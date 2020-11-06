---
title: 在Vue中使用svg-icon笔记
---

# svg相关目录情况

```
|-components
|--|-SvgIcon
|------|-index.vue #封装组件
|-icons
|--|-svg           #svg资源
|--|-index.js      #自动导入
|--|-svgo.yml      #去颜色,精简SVG
```

# SVG组件封装-index.vue

```vue
<template>
  <div v-if="isExternal" :style="styleExternalIcon" class="svg-external-icon svg-icon" v-on="$listeners" />
  <svg v-else :class="svgClass" aria-hidden="true" v-on="$listeners">
    <use :xlink:href="iconName" />
  </svg>
</template>

<script>
// doc: https://panjiachen.github.io/vue-element-admin-site/feature/component/svg-icon.html#usage
import { isExternal } from '@/utils/validate'

export default {
  name: 'SvgIcon',
  props: {
    iconClass: {
      type: String,
      required: true
    },
    className: {
      type: String,
      default: ''
    }
  },
  computed: {
    isExternal() {
      return isExternal(this.iconClass)
    },
    iconName() {
      return `#icon-${this.iconClass}`
    },
    svgClass() {
      if (this.className) {
        return 'svg-icon ' + this.className
      } else {
        return 'svg-icon'
      }
    },
    styleExternalIcon() {
      return {
        mask: `url(${this.iconClass}) no-repeat 50% 50%`,
        '-webkit-mask': `url(${this.iconClass}) no-repeat 50% 50%`
      }
    }
  }
}
</script>

<style scoped>
.svg-icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}

.svg-external-icon {
  background-color: currentColor;
  mask-size: cover!important;
  display: inline-block;
}
</style>

```

# 自动导入 index.js

```js
import Vue from 'vue'
import SvgIcon from '@/components/SvgIcon'// svg component

// register globally
Vue.component('svg-icon', SvgIcon)

const req = require.context('./svg', false, /\.svg$/)
const requireAll = requireContext => requireContext.keys().map(requireContext)
requireAll(req)

```

在main.js中 引入

```js
import '@/icons' // icon
```

# 使用svg-sprite-loader制作svg-sprite

```js
chainWebpack(config) {
    // set svg-sprite-loader
    config.module
      .rule('svg')
      .exclude.add(resolve('src/icons'))
      .end()
    config.module
      .rule('icons')
      .test(/\.svg$/)
      .include.add(resolve('src/icons'))
      .end()
      .use('svg-sprite-loader')
      .loader('svg-sprite-loader')
      .options({
        symbolId: 'icon-[name]'
      })
      .end()
}
```

# 精简SVG

这里我们需要使用到svgo插件

```yml
# svgo.yml
# replace default config

# multipass: true
# full: true

plugins:

  # - name
  #
  # or:
  # - name: false
  # - name: true
  #
  # or:
  # - name:
  #     param1: 1
  #     param2: 2

- removeAttrs:
    attrs:
      - 'fill'
      - 'fill-rule'

```

在package.json在增加script脚本指令

```json
"scripts": {
    "svgo": "svgo -f src/icons/svg --config=src/icons/svgo.yml"
}
```

执行shell命令,实现去除

```shell
npm run svgo
```

# 使用svg-icon

## 使用方式

```vue
<!-- icon-class 为 icon 的名字; class-name 为 icon 自定义 class-->
<svg-icon icon-class="password"  class-name='custom-class' />
```

## 改变颜色

svg-icon 默认会读取其父级的 color fill: currentColor;
你可以改变父级的color或者直接改变fill的颜色即可。

## 使用外链

支持使用外链的形式引入 svg。例如：

```vue
<svg-icon icon-class="https://xxxx.svg />
```

## 大小

如果你是从 iconfont下载的图标，记得使用如 Sketch 等工具规范一下图标的大小问题，不然可能会造成项目中的图标大小尺寸不统一的问题。
本项目中使用的图标都是 128*128 大小规格的。

[花裤衩大佬的-手摸手，带你优雅的使用 icon](https://juejin.im/post/6844903517564436493)
[Svg Icon 图标使用说明](https://panjiachen.gitee.io/vue-element-admin-site/zh/feature/component/svg-icon.html)
