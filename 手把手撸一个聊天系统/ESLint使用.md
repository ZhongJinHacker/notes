### ESLint

#### 主要用于js代码的编码规范约束



```json
{
    // 解析选项
    "parserOptions": {
        //emcaVersion用来指定你想要使用的 ECMAScript 版本
        "ecmaVersion": 2018, // 2018 这里也可以填9
        // 设置为 "script"(默认) 或 "module"（如果你的代码是 ECMAScript 模块)
        "sourceType": "module"
    },
    // 确定运行环境
    "env": {
      	// 使用es6语法
        "es6": true,
        // 运行在node环境
        "node": true
    },
    // 指定babel解析器
    "parser": "babel-eslint",
    // 对应package.json -> devDependencies的eslint-plugin-react
    "plugins": ["react"],
    // 规则
    "rules": {
        // 要求结尾有分号； 2 -> error
        "semi": [2, "always"],
        // 是否需要new操作符，0 -> never
        "new-cap": [0],
        // 2 -> error 直接报错；4 -> 4个空格；switchcase缩进用4个空格
        "indent": [2, 4, { "SwitchCase": 1 }],
        // 尾随逗号 2 -> error 直接报错； only-multiline 允许但不要求
        "comma-dangle": [2, "only-multiline"],
        //
        "space-before-function-paren": [2, "never"],
        //
        "operator-linebreak": [2, "before"],
        //
        "no-floating-decimal": [0],
        //
        "react/jsx-indent": [2, 4],
        //
        "react/jsx-indent-props": [2, 4],
        //
        "react/jsx-boolean-value": [2, "always"],
        //
        "react/prop-types": [0],
        //
        "jsx-quotes": [2, "prefer-double"]
    },
    //
    "extends": ["standard", "standard-react"]
}

```



参考： https://eslint.org/

