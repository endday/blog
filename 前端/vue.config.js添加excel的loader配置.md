---
title: vue.config.js添加excel的loader配置
url: https://www.yuque.com/endday/blog/atge7i
---

默认webpack没有处理excel文件的
可以直接使用file-loader来读取excel，就不需要放在public文件夹了，还能带上hash

```javascript
module.exports = {
  chainWebpack: config => {
    // 针对a标签上直接相对路径引用
    // 不建议加这个配置, 会影响到所有a标签
    // 而且该配置无法应用到el-link这种针对a标签的组件
     config.module
      .rule('vue')
      .use('vue-loader')
      .tap(options => {
        options.transformAssetUrls = {
          // 默认的配置
          video: ['src', 'poster'],
          source: 'src',
          img: 'src',
          image: ['xlink:href', 'href'],
          use: ['xlink:href', 'href'],
          a: 'href' // 额外添加的
        }
        return options
      })
    // 这个配置需要import excel文件
    config.module
      .rule('excel')
      .test(/\.(xls|xlsx|csv)$/)
      .use('file-loader')
      .loader('file-loader')
      .options({
        name: 'libs/[name].[hash:8].[ext]'
      })
  }
}
```
