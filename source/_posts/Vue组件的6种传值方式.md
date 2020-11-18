---
title: 'Vue组件的6种传值方式'
date: 2020-03-31
categories: 
  - 前端-Vue
tags:
  - 知识点
  - 面试
---

### 父子组件通信Demo

父组件

```
<template>
  <div>
    <child :users="users"></child>
    <div @updateEvent="update">{{dataFromChild}}</div>
  </div>
</template>
<script>
  import child from '@/component/child'
  export default {
    data(){
      return {
        users: ["张三","李四","王五"],
        dataFromChild:''
      }
    },
    methods:{
      update(event){
        this.dataFromChild = event;
      }
    }
  }
</script>
```

子组件child

```
<template>
  <ul>
    <li v-for="user in users" @click="updateName">{{user}}</li>
  </ul>
</template>
<script>
  import child from '@/component/child'
  export default {
    props:{
      users:{
        type:Array,
        require:true
      }
    },
    methods:{
      updateName(){
        this.$emit("updateEvent","新名称");
      }
    }

  }
</script>
```

### 父子组件通信

Vue中的父子组件是通过Prop/$emit关键字完成通信功能.解析以上示例
