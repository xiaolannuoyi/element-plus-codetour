---
title: layout
date: 2021-04-07
sidebar: 'auto'
sidebarDepth: 2
categories:
 - Element-Plus 源码分析
tags:
 - Element-Plus 源码分析
---

# Layout

> 涉及到的组件`row` `col`

源码内容

省略

## gutter

`gutter`的作用是让row里面的col产生出间隔来，**但是注意容器的最左和最右侧是没有间隔的**

![img](https://gitee.com/xiaolannuoyi/my_drawing_bed/raw/master/image/1653de08acbf5815~tplv-t2oaga2asx-watermark.awebp)

上图就是最终示意图，黑框就是`<el-row>`的宽度范围，里面是`<el-col>`组件，

 这个组件的宽度其实按`<el-row>`百分比来计算，而且`box-sizing`是`border-box`,

注意`gutter`属性是定义在父级的`<el-row>`上，子级的col通过 `inject` 可以拿到该属性，

```js
//row.ts
const gutter = computed(() => props.gutter)
provide('ElRow', {
  gutter,
})
```

然后给`<el-col>`分配`padding-left`和`padding-right`,因此每个col都有左右padding，

```js
//col.ts
const { gutter } = inject('ElRow', { gutter: { value: 0 } })

const style = computed(() => {
      if (gutter.value) {
        return {
          paddingLeft: gutter.value / 2 + 'px',
          paddingRight: gutter.value / 2 + 'px',
        }
      }
      return {}
    })
```



上图中每个col占宽25%，gutter的宽度就是col的padding的2倍，但是注意最左侧和最右侧是没有padding的，那么问题来了，怎么消去最左和最右的padding? 这里就是`<el-row>`负的margin起的作用，如果不设置上面的计算属性的style，那么左右2侧就会有col的padding，因此这里负的margin抵消了col的padding，且该值为 `-gutter/2+'px'`

```js
//row.ts
//row的左右margin，用于抵消col的padding，这里通过gutter计算出实际margin
const style = computed(() => {
  const ret = {
    marginLeft: '',
    marginRight: '',
  }
  if (props.gutter) {
    ret.marginLeft = `-${props.gutter / 2}px`
    ret.marginRight = ret.marginLeft
  }
  return ret
})
```

注意如果初看上面的图，一般的想法是col之间用margin来间隔，其实是不行的，而用padding来间隔就很简单，width按百分比来分配就行(box-sizing要设置为`border-box`)

## Provide/inject

```js
//row.ts
const gutter = computed(() => props.gutter)
provide('ElRow', {
  gutter,
})

//col.ts
const { gutter } = inject('ElRow', { gutter: { value: 0 } })
```

> why
>
> 这里`provide`的`gutter`是`computed`,为什么不直接用`props.gutter`?
>
> ```js
> //row.ts
> provide('ElRow',props.gutter)
> //col.ts
> const gutter = inject('ElRow')
> ```
>
> 
>
> **`provide/inject` 绑定*并不是*响应式的。我们可以通过传递一个 `ref` property 或 `reactive` 对象给 `provide` 来改变这种行为。**
>
> [#1441](https://github.com/element-plus/element-plus/issues/1441)

## h函数

这里使用`h函数`是因为`tag`不确定



## fix gutter bug 

[#1366](https://github.com/element-plus/element-plus/issues/1366)

```js
if (gutter.value) {
  ret.push('is-guttered')
}
```

❓此代码感觉没啥用。。。

