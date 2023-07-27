---
title: "表单验证的一点实践"
date: 2023-06-22T21:20:00+08:00
lastmod: 2023-07-27T22:31:00+08:00
draft: false
tags: ["表单生成器","yup","zod", "vee-validate"]
categories: ["编程"]
summary: " "
---


## 背景
表单验证的功能较为简单，但如果没有合适的轮子，一遍一遍去写繁琐的校验逻辑会让人崩溃。翻阅了几个表单库后，最终选择了 [VeeValidate](https://vee-validate.logaretm.com/v4/)。本文记录一下使用过程。

## 简介
VeeValidate 提供了两种使用方式，一种是组件式，另一种是 Composition API 式。组件式适合小点的场景，不需要做太多封装，使用简单；Composition API 式适合有着很多表单的中大型应用，有着较强的可定制性。我选择使用 Composition API，因为此次实践的目标接近于造一个表单生成器，而且这种方式更加无头。

## 已知问题
1. 本项目的输入控件包含了 NaiveUI 的输入控件（如 NInput, NSelect）。
2. 本项目的输入控件也包含自定义的一些输入控件。而 NaiveUI 的 NFormItem 组件，没有将 Trigger 开放出来。所以用 NaiveUI 的表单验证组件无法满足需求。

解决这俩个问题的办法不难，那就是使用 VeeValidate Composables 封装出一套跟 NaiveUI 类似的 CustomForm 及 CustomFormItem，并将 `useField` 的 `handleBlur` 等方法 通过 SlotProps 由 CustomFormItem 传给输入控件（无论是 NaiveUI 的控件还是自定义控件）。


## 过程
### Schema Validator
VeeValidate 提供了两种方案。一种是 [Yup](https://github.com/jquense/yup), 一种是 [Zod](https://github.com/colinhacks/zod)。Zod 的名声大一些使用者也多一些，但 Yup 也不差。因为 Zod 的国际化没有 Yup 这么简单，所以没有花费时间去深究而**直接选用了 Yup**。
除了这两个， 市面上还有一些其他 Schema Validator，比如近期出现的 [Valibot](https://github.com/fabian-hiller/valibot)，主打一个体积小。

### Internationalization
在 VeeValidate 中配置国际化要按照 Schema Validator 的方式来。对于 Yup 来说，主要就是声明多种错误类型的错误消息。
将下面代码放入某层 Provider 即可。
```JavaScript
import {setLocale} from 'yup';
const {t} = useI18n();
setLocale({
    mixed: {
        required: ({label}) => {
            return t('field_required', {label})
        },
        oneOf: ({label, resolved: options}) => {
            const optionList = options.map(optionValue => `"${t('option_' + optionValue)}"`).join(t('option_delimiter') + ' ');
            return t('not_one_of', {label, optionList})
        }
    },
    string: {
        min: ({label, min}) => {
            return t('string_too_short', {label, min});
        },
        max: ({label, max}) => {
            return t('string_too_long', {label, max});
        },
        email: ({label})=>{
            return t('string_not_valid_email', {label});
        }
    },
    array: {
        max: ({label, max})=>{
            return  t('array_too_long', {label, max});
        }
    }
})
```

### Partial Validation
Veelidate 的 `useForm` 提供了 `handleSubmit` 构造函数对表单所有字段进行验证，验证成功/失败后执行回调。
但它没有开放直观的 Partial Validataion 验证，所以我们得自己封装一个。  
所幸 `useForm` 开放了粒度更细的 `validateField` 和 `setFieldTouched`，基于此我们可以模拟 [handleSubmit 的行为](https://vee-validate.logaretm.com/v4/guide/composition-api/handling-forms/#submission-behavior)（比如验证的同时 Set Field Touched），封装出一个 Partial Validataion 方法：
```JavaScript
async function validatePartial(cb, errCb, fieldFilter = () => true) {
    // 筛选出需要验证的规则
    const fieldsNeedToValidate = Object.keys(props.rules.fields).filter(fieldFilter);
    // touch fields
    for (const field of fieldsNeedToValidate) {
        setFieldTouched(field, true);
    }
    // 验证 fields
    const validationResults = await Promise.all(fieldsNeedToValidate.map(field => validateField(field)));

    // 验证结果：
    if (validationResults.every(validationResult => validationResult.valid)) {
        // 所有项成功
        typeof cb === 'function' && cb();
    } else {
        // 任意一项失败
        typeof errCb === 'function' && errCb();
    }
}
```
Partial Validation 封装完成后，可以把全量验证和部分验证统一在一个函数中，并暴露给 CustomForm 组件外部，通过 Field Filter 区分即可。


### Cross-Field Validation
一个很常见的需求是联动验证两个输入控件。    

比如修改密码时，用户两次输入的新密码需要保持一致。yup 开放了 `ref` 来引用同一 Schema 中的其他 Field。那么对于这个场景，我们提供给 CustomForm 组件的 Rules 可以这样写： 
```js
const formRules = yup.object({
    oldPassword: yup.string().required(),
    newPassword: yup.string().required().min(8).max(128)
        .test('is-password-diverse', t('password_diversity_not_enough'), isPasswordDiverse),
    newPasswordRepeat: yup.string().required().oneOf([yup.ref('newPassword')], t('password_not_match'))
});
```
注意，如果是给每个 `useField` 分别传入 rule 而不是给 `useForm` 传入整个 rules，那么这种情况可能会有 BUG，ref 不会生效。


## 遇到的问题
VeeValidate 的 `useField` 暴露出的 `value` 是用来绑定到原生输入控件上的。  
也就是说，在 UI 组件库作者手中，`useField` 应该被直接写入自造的输入控件中，`value` 绑定到原生输入控件上，再通过 `syncVModel` 将 `value` 同步至输入控件外部。  
但我所面临的这个场景，输入控件既可能来自外部也可能来自内部，所以不宜使用 `value` 作为表单值的来源，这样一来验证时的值将无法与表单实际值保持一致。所以我用了扭曲的方法：Watch 输入控件的值，并通过 `useField` 所提供的 `setValue` 将值同步给 CustomForm 及 CustomFormItem。

## 结束
文章到此结束，陈述内容包含较多的细节 API，比较无聊，以后再也不写这么烂的文章了。    
倒是最近韩国科学家发了常温超导材料的论文，很抓眼球，非常刺激。
