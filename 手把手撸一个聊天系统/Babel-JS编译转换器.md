### Babel

Babel 是一个广泛使用的变异转码器，用于将ES6、ES7的代码转换为ES5的代码（原因是部分浏览器暂时仅支持到ES5）

---

```json
{
  // 先 stage-0处理 -> react 处理 -> es2015处理
  // babel-preset-es2015; babel-preset-react;babel-preset-stage-0
  "presets": ["es2015", "react", "stage-0"],
  // 使用polyfill插件和 decorators插件，用来转换一些特别的语法
  "plugins": ["babel-polyfill", "transform-decorators-legacy"],
  "env": {
    "production": {
      "presets": ["react-optimize"]
    },
    "development": {
      "presets": ["react-hmre"]
    }
  }
}
```

