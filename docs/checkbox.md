---
title: echeckbox
date: 2021-04-07
sidebar: 'auto'
sidebarDepth: 2
categories:
    - Element-Plus 源码分析
tags:
    - Element-Plus 源码分析
---

# echeckbox

1. Label

    ```html
    <label class="el-checkbox">
        <!-- 复选框 -->
        <span class="el-checkbox__input"></span>
        <!-- 文字部分 -->
        <span class="el-checkbox__label"></span>
    </label>
    ```

2. 选中样式

    ```css
    .el-checkbox__inner {
        //🔲 复选框样式
        display: inline-block;
        position: relative;
        border: 1px solid #dcdfe6;
        border-radius: 2px;
        box-sizing: border-box;
        width: 14px;
        height: 14px;
        background-color: #fff;
        z-index: 1;
        transition: border-color 0.25s cubic-bezier(0.71, -0.46, 0.29, 1.46), background-color 0.25s cubic-bezier(0.71, -0.46, 0.29, 1.46);
    }
    .el-checkbox__inner:after {
        //✅样式
        box-sizing: content-box;
        content: '';
        border: 1px solid #fff;
        border-left: 0;
        border-top: 0;
        height: 7px;
        left: 4px;
        position: absolute;
        top: 1px;
        transform: rotate(45deg) scaleY(0);
        width: 3px;
        transition: transform 0.15s ease-in 0.05s;
        transform-origin: center;
    }
    .el-checkbox__input.is-checked .el-checkbox__inner:after {
        //✅显示
        transform: rotate(45deg) scaleY(1);
    }
    ```

    > 这是`el-checkbox__inner`类的 after 伪元素，里面是一个只有右下 border 的长方形经过旋转 45 度后的图形，也就是一个勾的形状，所以这个勾只是纯粹的 css 实现而已，好处是简化了 html 结构，并且还用了`transition`来添加点击后勾变大的动画效果,这里是通过过渡`transform`的`scaleY`的值来实现，未选中时`scaleY`为 0，选中时为 1，就实现了勾放大的效果
    > **因此要善用伪元素，会简化很多不必要的代码**

3. slot

    ```html
    <!-- 文字部分 -->
    <span v-if="$slots.default || label" class="el-checkbox__label">
        <slot></slot>
        <template v-if="!$slots.default">{{ label }}</template>
    </span>
    ```

    > 我感觉 radio 的这里比这个要简洁，完全可以替换成[后背插槽](https://www.vue3js.cn/docs/zh/guide/component-slots.html#%E5%90%8E%E5%A4%87%E5%86%85%E5%AE%B9)
    >
    > ```html
    > //radio
    > <span class="el-checkbox__label">
    >     <slot>
    >         {{ label }}
    >     </slot>
    > </span>
    > ```

4. el-checkbox\_\_original 隐藏 input 样式

5. focus

    ```css
    .el-checkbox__input.is-foucs .el-checkbox__inner {
        border-color: #409eff;
    }
    ```

    > 这里跟 radio 不一样，这里是有`is-focus`添加 focus 样式。

6. 初始化

    ```js
    setStoreValue(props, { model });

    const setStoreValue = (props: ICheckboxProps, { model }: PartialReturnType<typeof useModel>) => {
        function addToStore() {
            //如果是array,且当前model中未选中当前值时，将其选中
            if (Array.isArray(model.value) && !model.value.includes(props.label)) {
                model.value.push(props.label);
            } else {
                model.value = props.trueLabel || true;
            }
        }
        //props.checked 为true时，修改model的值 使props.checked与model值 一致
        props.checked && addToStore();
    };
    ```

7. isChecked

    ```js
    <span :class="{ 'is-checked': isChecked,}"></span>
    <input :checked="isChecked"/>
    //是否被选中
      const isChecked = computed(() => {
        const value = model.value
        //toTypeString 源于vue
        //const objectToString = Object.prototype.toString;
        //const toTypeString = (value) => objectToString.call(value);
        //1.boolean 2.array 3.trueLabel 三种情况的判断
        if (toTypeString(value) === '[object Boolean]') {
          return value
        } else if (Array.isArray(value)) {
          return value.includes(props.label)
        } else if (value !== null && value !== undefined) {
          return value === props.trueLabel
        }
      })
    ```

8) model

    ```js
    const model = computed({
        get() {
          return isGroup.value
            ? store.value
            : props.modelValue ?? selfModel
            // 「双问号」?? 如果给定变量值为 null 或者 undefined，则使用双问号后的默认值，否则使用该变量值。
        },

        set(val: unknown) {
          if (isGroup.value && Array.isArray(val)) { // checkbox-group
            //是否突破限制 判断
            isLimitExceeded.value = false
            // min定义了 且 当前选中长度 < min
            if (checkboxGroup.min !== undefined && val.length < checkboxGroup.min.value) {
              isLimitExceeded.value = true
            }
            // max定义了 且 当前选中长度 > max
            if (checkboxGroup.max !== undefined && val.length > checkboxGroup.max.value) {
              isLimitExceeded.value = true
            }
            //未突破限制 修改checkboxGroup的value值
            // ?.链式调用。
            isLimitExceeded.value === false && checkboxGroup?.changeEvent?.(val)
          } else {
            emit(UPDATE_MODEL_EVENT, val)//update:modelValue 修改checkbox的value值
            selfModel = val as boolean
          }
        },
      })


    function handleChange(e: InputEvent) {
        if (isLimitExceeded.value) return  //如果突破限制 就不能修改值
        const target = e.target as HTMLInputElement
        const value = target.checked
          ? props.trueLabel ?? true
          : props.falseLabel ?? false

        emit('change', value, e)
      }
    ```

    > 不过这块有问题
    >
    > 就是 当有 min>1 时，初始化数组为[]，不能选中值。[#1742](https://github.com/element-plus/element-plus/issues/1742)
    >
    > 所以在学习的时候，也是能发现源码中存在的一些问题的 😂

9) watch

    ```js
    // 监听value变化，与表单验证有关
    watch(
        () => props.modelValue,
        (val) => {
            elFormItem.formItemMitt?.emit('el.form.change', [val]);
        }
    );
    ```

10.
