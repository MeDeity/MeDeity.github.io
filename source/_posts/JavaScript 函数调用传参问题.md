---
title: JavaScript 函数调用传参问题
---
一. 函数调用传参问题

```vue
<template>
  <!--无参函数调用-->
  <div @click="methodWithOutParams" />
  <!--函数调用带括号-->
  <div @click="methodReturnFunc()" />
  <!--有参函数调用-->
  <div @click="()=>{methodWithParams(yourParams)}" />
</template>

<script>
export default {
  name: 'Dialog',
  data() {
    return {}
  },
  methods: {
    methodWithOutParams() {
      console.info('无参数的方法被调用')
    },
    //函数执行后仍返回函数对象
    methodReturnFunc(){
        //do something
        return methodWithParams("methodReturnFunc")
    }
    methodWithParams(val) {
      console.info('含参数的方法被调用' + val)
    }
  }
}
</script>
```

1. vue的@click事件在进行函数调用时,传递的是函数的地址(指针),所以不带括号.
2. 如果带括号表示直接执行该函数并将执行后的返回值绑定到@click事件上(这种情况下返回值应该也是函数才行) - methodReturnFunc
3. 如果函数携带参数

二、箭头函数的语法

1. 没有参数的函数应该写成一对圆括号

```
() => { statements }
```

2. 当只有一个参数时，圆括号是可选的

```
(singleParam) => { statements }
singleParam => { statements }
```

3. 如果只有一条语句,可以省略大括号;同样如果只有一条语句 会自动添加return

```
(param1, param2, …, paramN) => { statements }
(param1, param2, …, paramN) => expression
//相当于：(param1, param2, …, paramN) =>{ return expression; }
```
