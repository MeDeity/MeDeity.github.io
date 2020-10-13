---
title: [Vue]Vue使用过程踩坑记录
---
一. Vue中动态绑定img的src属性,图片无法显示的问题

使用Vue的v-bind属性绑定img元素的src属性时,发现src数据正常渲染,但是网页上却无法显示图片,经搜索,动态生成的路径需加上require方可成功加载图片

```vue
<template>
  <div class="grid-item">
    <img class="grid-item-img" :src="icon" alt="图标">
    <div>{{ title }}</div>
  </div>
</template>

<script>
export default {
  props: {
    title: String,
    iconName: String
  },
  computed: {
    icon: function() {
      const result = require('@/assets/common/' + this.iconName + '.png')
      return result
    }
  }
}
</script>

<style lang="scss" scoped>
.grid-item {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;

    .grid-item-img {
       width: 52px;
       height: 52px;
       margin-bottom: 5px;
    }
}
</style>

二. elementUI table隐藏边框
在通用css类(修正css)中,添加以下css,并在<el-table>元素的父类中新增class="customer-table"
```css
<style lang="scss">
.customer-table {
  .el-table,.el-table--border,.el-table--group{
    td,th.is-leaf{
      border: none;
      cursor: pointer;
    }
    &::before{
      height: 0;
    }
  }
}
</style>
```

三. elementUI table el-table-column宽度设置百分比
由于需要保证table不会出现横向滚动,因此尝试在el-table-column使用百分比宽度,
但发现width='20%'不会生效.改用min-width = '2'可以生效

```html
<el-table-column
    prop="name"
    label="资源名称"
    min-width="5"/>

<el-table-column
    prop="date"
    label="生产时间"
    min-width="5"/>
```

>特别注意,不能限制 scoped="scoped"
