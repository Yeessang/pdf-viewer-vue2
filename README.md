# pdf-viewer-vue2

一款Vue2框架开发的pdf阅读器组件，如果您使用的是Vue3，可以查看[Vue3PDF阅读器组件地址](https://www.npmjs.com/package/pdf-viewer-vue3)


## demo

[demo地址](https://codesandbox.io/p/devbox/nice-keller-75ns2d?file=%2Fsrc%2FApp.vue%3A1%2C1-39%2C1)

## Feature

- 文本选中、复制
- 缩略图
- 目录
- 翻页、跳转页
- 单页/双页视图
- 缩放/容器宽/容器高/原尺寸/自定义尺寸
- 打印
- 搜索文本
- 旋转
- 横向/竖向滚动
- 移动端

## usage

```vue
<template>
  <div id="app">
    <div>
      <button @click="openFile">选择文件</button>
      <input v-show="false" ref="fileInput" type="file" @change="handleFileChange">
      <PDF
        ref="pdfComp"
        style="width: 100%; height: 600px; margin: auto;"
      ></PDF>
      <PDF
        ref="pdfMobileComp"
        @pagesLoaded="(v) => log('pagesLoaded', v)"
        @pageRendered="(v) => log('pageRendered', v)"
        @pageChanging="(v) => log('pageChanging', v)"
        @findChange="(v) => log('findChange', v)"
        @scaleChanging="(v) => console.log('scaleChanging 缩放比例改变', v)"
        style="width: 300px; height: 500px; margin-left: 20px;"
      ></PDF>
    </div>
  </div>
</template>

<script>
import PDF from 'pdf-viewer-vue2';
import 'pdf-viewer-vue2/package/pdf-viewer.css';

export default {
  name: 'App',
  components: {
    PDF
  },
  methods: {
    async handleFileChange(event) {
      const files = event.target.files
      const file = files[0]
      if (file) {
        const buffer = await file.arrayBuffer()
        this.$refs.pdfMobileComp?.loadFile(buffer);
        const reReadBuffer = await file.arrayBuffer()
        this.$refs.pdfComp?.loadFile(reReadBuffer)
      }
    },
    openFile() {
      this.$refs.fileInput?.click();
    },
    log(name, v) {
      console.log(name, v)
    }
  }
}
</script>

<style>
</style>


```

