---
title: form
date: 2021-07-22
sidebar: 'auto'
sidebarDepth: 2
categories:
    - Element-Plus 源码分析
tags:
    - Element-Plus 源码分析
---

# form

1. 逻辑图

   ![form](https://gitee.com/xiaolannuoyi/my_drawing_bed/raw/master/image/form.png)
   
   
   
2. 校验方法

   ```js
       const validate = (callback?: Callback) => {
         if (!props.model) {
           debugWarn('Form', 'model is required for validate to work!')
           return
         }
   
         let promise: Promise<boolean> | undefined
         // if no callback, return promise
         if (typeof callback !== 'function') {
           promise = new Promise((resolve, reject) => {
             callback = function (valid, invalidFields) {
               if (valid) {
                 resolve(true)
               } else {
                 reject(invalidFields)
               }
             }
           })
         }
   
         if (fields.length === 0) {
           callback(true)
         }
         let valid = true
         let count = 0
         let invalidFields = {}
         let firstInvalidFields
         for (const field of fields) {
           field.validate('', (message, field) => {
             if (message) {
               valid = false
               firstInvalidFields || (firstInvalidFields = field)
             }
             invalidFields = { ...invalidFields, ...field }
             if (++count === fields.length) {
               callback(valid, invalidFields)
             }
           })
         }
         if (!valid && props.scrollToError) {
           scrollToField(Object.keys(firstInvalidFields)[0])
         }
         return promise
       }
   ```

   >
   >1. 如果没有callback函数，返回promise，
   >使用方法如下
   >```js
   >const valid = await this.$refs.form.validate().catch((err)=>{return err})})
   >if (valid) {
   >  //验证通过的代码的
   >} else {
   >  //验证不通过的代码
   >}
   >```
   >2. 有`callback`,执行`callback`,返回undefined
   >
   >3. Promise 封装callback，学习
   >
   >   ```js
   >   promise = new Promise((resolve, reject) => {
   >     callback = function (valid, invalidFields) {
   >       if (valid) {
   >         resolve(true)
   >       } else {
   >         reject(invalidFields)
   >       }
   >     }
   >   })
   >   ```
   >
   >   若源码中没有这段代码，我们使用中封装代码，如下
   >
   >   ```js
   >   // 使用方法
   >   // const vaild = await pifyValidate(this.$refs.form.validate)
   >   export const pifyValidate = validateFn => {
   >       return new Promise(resolve => {
   >           validateFn(valid => {
   >               resolve(valid);
   >           });
   >       });
   >   };
   >   ```
   >
   >   
   >
   >

2. 

   
   
3. 
