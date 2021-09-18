---
title: radio
date: 2021-04-07
sidebar: 'auto'
sidebarDepth: 2
categories:
 - Element-Plus 源码分析
tags:
 - Element-Plus 源码分析
---

# radio

1. `label`放在最外层的作用是`扩大鼠标点击范围`，无论是点击在文字还是input上都能够触发响

   ```html
   <label class="el-radio">
     <!-- 单选框 -->
     <span class="el-radio__input"></span>
     <!-- 文字部分 -->
     <span class="el-radio__label"></span>
   </label>
   ```

   

   ```html
   <template>
     <label
       class="el-radio"
       :class="{
         [`el-radio--${radioSize || ''}`]: border && radioSize,
         'is-disabled': isDisabled,
         'is-focus': focus,
         'is-bordered': border,
         'is-checked': model === label
       }"
       role="radio"
       :aria-checked="model === label"
       :aria-disabled="isDisabled"
       :tabindex="tabIndex"
       @keydown.space.stop.prevent="model = isDisabled ? model : label"
     >
       <!-- :tabindex="tabIndex"
       @keydown.space.stop.prevent="model = isDisabled ? model : label"
       这个功能是为了用tab切换不同选项时，按空格可以快速选择目标项 -->
   
       <!-- 模拟圆形按钮 -->
       <span
         class="el-radio__input"
         :class="{
           'is-disabled': isDisabled,
           'is-checked': model === label
         }"
       >
         <!-- 圆形样式 -->
         <span class="el-radio__inner"></span>
         <!-- 真正的按钮 -->
         <!-- el-radio__original ⚠️注意这个样式 -->
         <input
           ref="radioRef"
           v-model="model"
           class="el-radio__original"
           :value="label"
           type="radio"
           aria-hidden="true"
           :name="name"
           :disabled="isDisabled"
           tabindex="-1"
           @focus="focus = true"
           @blur="focus = false"
           @change="handleChange"
         >
       </span>
       <!-- 文字部分 -->
       <!-- keydown.stop 阻止事件继续冒泡 -->
       <span class="el-radio__label" @keydown.stop>
         <slot>
           {{ label }}
         </slot>
       </span>
     </label>
   </template>
   
   ```

   

2. **el-radio__original**

   > ```css
   > .el-radio__original {
   >  opacity: 0;
   >  outline: none;
   >  position: absolute;
   >  z-index: -1;
   >  top: 0;
   >  left: 0;
   >  right: 0;
   >  bottom: 0;
   >  margin: 0;
   > }
   > ```
   >
   > 真正的input透明度为0，且是绝对定位脱离文档流，因此我们看不到，但是是有大小的，注意不是`display:none`或者`visibility:hidden`,如果是none或者hidden的话则无法触发鼠标点击了，**只有`opacity:0`才能达到目的,这是个需要注意的地方**
   >
   > ![image-20210414152424491](https://gitee.com/xiaolannuoyi/my_drawing_bed/raw/master/image/image-20210414152424491.png)

3. disabled

   >样式上的`disabled`绑定在`label` 上，功能上的`disabled `绑定在`input` 上
   >
   >```js
   >const isDisabled = computed(() => {
   >    return isGroup.value
   >      ? radioGroup.disabled || props.disabled || elForm.disabled
   >      : props.disabled || elForm.disabled
   >  })
   >```
   >
   >`isGroup`、`elForm`的取值都是通过`inject`注入
   >
   >```js
   >const radioGroup = inject(radioGroupKey, {} as RadioGroupContext)
   >const isGroup = computed(() => radioGroup?.name === 'ElRadioGroup')
   >```
   >
   >

4. ```
   role="radio"
   :aria-checked="model === label"
   :aria-disabled="isDisabled"
   ```

   `role`role的作用是描述一个非标准的tag的实际作用。比如用div做button，那么设置div 的 role="button"，辅助工具就可以认出这实际上是个button。

   `aria-*`的作用就是描述这个tag在可视化的情境中的具体信息

   > 此tag 是一个`radio`，`checked`和`disabled` 的属性分别是什么

5. ```js
   :tabindex="tabIndex"
   @keydown.space.stop.prevent="model = isDisabled ? model : label"
   ```

   tabindex规定了按下tab键该元素获取焦点的顺序,同样是个计算属性

   ```js
   const tabIndex = computed(() => {
       return (isDisabled.value || (isGroup.value && model.value !== props.label)) ? -1 : 0
     })
   ```

   >- tabindex=负值 (通常是tabindex=“-1”)，表示元素是可聚焦的，但是不能通过键盘导航来访问到该元素，用JS做页面小组件内部键盘导航的时候非常有用。
   >
   >- `tabindex="0"` ，表示元素是可聚焦的，并且可以通过键盘导航来聚焦到该元素，它的相对顺序是当前处于的DOM结构来决定的。
   >
   >- tabindex=正值，表示元素是可聚焦的，并且可以通过键盘导航来访问到该元素；它的相对顺序**按照**tabindex 的数值`递增而滞后获焦 `。如果多个元素拥有相同的 **tabindex**，它们的相对顺序按照他们在当前DOM中的先后顺序决定。
   >
   >  
   >
   >`@keydown.space.stop.prevent="model = isDisabled ? model : label"`
   >
   >​    *这个功能是为了用tab切换不同选项时，按空格可以快速选择目标项* (先通过tab 切换聚焦，在通过space选择当前聚焦的项目)

6. focus   

   ```js
   //radio.vue
   const focus = ref(false)//控制focus样式 click时，不显示focus样式
   
   <label  :class="{
         'is-focus': focus,
         ...
       }">
   
   <input @focus="focus = true"
          @blur="focus = false">
   ```

   

   ```css
   .el-radio:focus:not(.is-focus):not(:active):not(.is-disabled) .el-radio__inner {
       box-shadow: 0 0 2px 2px #409eff;
   }
   ```

   >按` tab切换`，会出现`foucs`样式，但不会触发`@focus事件`因为label上添加的`:tabindex="tabIndex"`,而input的`:tabindex="-1"`，所以虽然有样式，但不会触发input的`@focus事件`
   >
   >当`click`时，同时也会出现`focus`样式，触发`@focus事件`,此时不想有`focus`的样式，所以添加一个`is-focus`

7. model

   ```js
   const model = computed<string | number | boolean>({
         get() {
           return isGroup.value ? radioGroup.modelValue : props.modelValue
         },
         set(val) {
           if (isGroup.value) {
             radioGroup.changeEvent(val)//injdect radioGroup 修改radioGroup的v-model值
           } else {
             ctx.emit(UPDATE_MODEL_EVENT, val)//update:modelValue 修改radio的v-model值
           }
           radioRef.value.checked = props.modelValue === props.label//dom 对象上的 checked 属性
         },
       })
   ```

   `Model `通过`computed`计算而来，

   get : 获取值

   Set:  修改值

8. nextTick

   ```js
   function handleChange() {
     nextTick(() => {
       ctx.emit('change', model.value)
     })
   }
   ```

   此处为什么使用nextTick？感觉不使用也没问题❓

   > 我认为 radio-group 中已确保了执行循序，所以radio的change 无论是单独使用还是在group内总是最后执行的，所以我认为使不使用`nextTick`都可以。希望有人帮忙解惑，谢谢
   >
   > 以下是测试说明
   
   nextTick 使用测试
   
   ```html
   <template>
       <el-button @click="test1">test1</el-button>
       {{ name }}
       <el-button @click="test2">test2</el-button>
   </template>
   
   <script setup lang="ts">
   import { nextTick, onBeforeUpdate, onUpdated, ref, watch } from 'vue-demi'
   
   onBeforeUpdate(() => {
     console.warn('onBeforeUpdate')
   })
   onUpdated(() => {
     console.warn('onUpdated')
   })
   
   const test1 = () => {
     console.warn('test1')
     nextTick(() => {
       console.warn('nextTick')
     })
   }
   const name = ref('')
   const test2 = () => {
     console.warn('test2')
     name.value = Math.random()
     nextTick(() => {
       console.warn('nextTick')
     })
   }
   </script>
   ```
   
   > test1按钮输出：test1->nextTick
   >
   > test2按钮输出：test2->onBeforeUpdate->onUpdated->nextTick
   >
   > ⚠️name要放到页面，才会触发更新。
   
   所以当只调用radio时，有无nextTick，是一样的。因为不涉及到页面的更新。
   
   > Radio
   
   再次讨论使用`radio-group`时
   
   ```js
   const changeEvent = value => {
     console.warn('group changeEvent')
     ctx.emit(UPDATE_MODEL_EVENT, value)//update:modelValue
     nextTick(() => {
       console.warn('group changeEvent nextTick')
       ctx.emit('change', value)
     })
   }
   ```
   
   > 顺序
   >
   > ![image-20210918235840685](https://gitee.com/xiaolannuoyi/my_drawing_bed/raw/master/image/image-20210918235840685.png)
   
    

# radio-group

1. nextTick

   ```js
   const changeEvent = value => {
         ctx.emit(UPDATE_MODEL_EVENT, value)//update:modelValue
         nextTick(() => {
           ctx.emit('change', value)
         })
       }
   ```

   使用nextTick的顺序图，如上图。

   

   >```js
   >const changeEvent = value => {
   >      ctx.emit(UPDATE_MODEL_EVENT, value)//update:modelValue
   >      ctx.emit('change', value)
   >}
   >```
   >
   >![image-20210919000724489](https://gitee.com/xiaolannuoyi/my_drawing_bed/raw/master/image/image-20210919000724489.png)

   此处nextTick作用，保证执行顺序。 先更新值 `ctx.emit(UPDATE_MODEL_EVENT, value)`，后触发页面change事件`ctx.emit('change', value)`，保证`v-model`变化会优先于`change`执行。
   如果没有 nextTick，执行顺序相反，
   因为ctx.emit(UPDATE_MODEL_EVENT, value)，更新值的变化，会导致页面更新，所以会在`下次页面更新时执行`。而chang事件会在`当前执行`。

2. 选中切换

   ```js
   // '@element-plus/utils/aria'
   export const EVENT_CODE = {
     tab: 'Tab',
     enter: 'Enter',
     space: 'Space',
     left: 'ArrowLeft', // 37
     up: 'ArrowUp', // 38
     right: 'ArrowRight', // 39
     down: 'ArrowDown', // 40
     esc: 'Escape',
     delete: 'Delete',
     backspace: 'Backspace',
   }
   //radio-group.vue
     const handleKeydown = e => { // 左右上下按键 可以在radio组内切换不同选项
         const target = e.target
         const className = target.nodeName === 'INPUT' ? '[type=radio]' : '[role=radio]'//radio||label
         const radios = radioGroup.value.querySelectorAll(className)
         const length = radios.length
         const index = Array.from(radios).indexOf(target)
         const roleRadios = radioGroup.value.querySelectorAll('[role=radio]')
         let nextIndex = null
         switch (e.code) {
           case EVENT_CODE.left:
           case EVENT_CODE.up:
             //⬅️⬆️ 选中前一个
             e.stopPropagation()//阻止冒泡
             e.preventDefault()//阻止默认行为
             //如果当前选中的是第一个，则下一个选中的是最后一个，否则是前一个
             nextIndex = index === 0 ? length - 1 : index - 1
             break
           case EVENT_CODE.right:
           case EVENT_CODE.down:
             //➡️⬇️ 选中后一个
             e.stopPropagation()
             e.preventDefault()
             //如果当前选中的是最后一个，则下一个选中的是第一个，否则是后一个
             nextIndex = (index === (length - 1)) ? 0 : index + 1
             break
           default:
             break
         }
         if (nextIndex === null) return
         roleRadios[nextIndex].click()
         roleRadios[nextIndex].focus()
       }
   ```

   

# radio-button

1. `label`放在最外层的作用是`扩大鼠标点击范围`，无论是点击在文字还是input上都能够触发响

2. 同上

   ```css
   .el-radio-button__orig-radio {
       opacity: 0;
       outline: none;
       position: absolute;
       z-index: -1;
   }
   ```

   