###devtool

---

webpack 的选项 devtool

```js
module.exports = {
    entry: __dirname + '/../src/main.js',
    output: {
        path: __dirname + '/../dist',
        filename: 'bundle.js'
    },
    devtool: 'eval-source-map'
}
```

| devtool选项                  | 配置结果                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| source-map                   | 产生一个单独的source-map文件，功能最完全，但会减慢打包速度   |
| eval-source-map              | 使用eval打包源文件模块，直接在源文件中写入干净完整的source-map，不影响构建速度，但影响执行速度和安全，建议开发环境中使用，生产阶段不要使用 |
| cheap-module-source-map      | 会产生一个不带映射到列的单独的map文件，开发者工具就只能看到行，但无法对应到具体的列（符号），对调试不便 |
| cheap-module-eval-source-map | 不会产生单独的map文件，（与eval-source-map类似）但开发者工具就只能看到行，但无法对应到具体的列（符号），对调试不便 |

