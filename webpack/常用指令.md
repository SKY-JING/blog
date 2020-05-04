<div class="my_title" style="text-align: center; font-weight: 700; font-size: 2em;">webpack常用指令</div>

```
webpack：启动 执行一次开发时的编译

webpack -w：如果你想当改变一个文件而让webpack实时编译

webpack -p：执行一次生成环境的编译（压缩）

webpack -d：对文件进行解压缩，提供source map，方便调式代码方便调试文件

webpack --config customconfig.js：如果你想把默认的配置文件webpack.config.js改成自定义文件

webpack --watch：在开发时持续监控增量编译（很快）

webpack --display-error-details：显示更多报错信息

webpack --display-chunks：展示编译后的分块

webpack --colors：显示静态资源的颜色

webpack --progress：显示编译进度

webpack --display-reasons：显示更多引用模块原因

webpack --profile：输出性能数据，可以看到每一步的耗时

webpack --display-modules：默认情况下node_modules下的模块会被隐藏，加上这个参数可以显示这些被隐藏的模块

webpack --sort-chunks-by,--sort-assets-by,--sort-modules-by：将modules/chunks/assets进行列表排序

webpack -help：查看webpack帮忙文档
```