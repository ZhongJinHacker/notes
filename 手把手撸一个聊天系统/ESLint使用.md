### ESLint

#### 主要用于js代码的编码规范约束



##### 以下是react的约束模板



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
        //函数与括号之间是否有空格：2 -> error; never -> 不允许
        "space-before-function-paren": [2, "never"],
        //2->error; before -> 操作符在前
        "operator-linebreak": [2, "before"],
        //0->允许
        "no-floating-decimal": [0],
        //jsx缩进问题 要求4个空格；2->error
        "react/jsx-indent": [2, 4],
        //jsx 中props属性缩进
        "react/jsx-indent-props": [2, 4],
        //允许使用布尔值
        "react/jsx-boolean-value": [2, "always"],
        // 0 关闭 ，prop-types检查
        "react/prop-types": [0],
        // 对所有不包含双引号的jsx属性值 强制使用双引号
        "jsx-quotes": [2, "prefer-double"]
    },
    // 扩展配置 
    // 对应devDependencies的 
    // eslint-config-standard和eslint-config-standard-react
    "extends": ["standard", "standard-react"]
}

```



#### 参考： 

https://eslint.org/

https://github.com/yannickcr/eslint-plugin-react

https://cloud.tencent.com/developer/section/1135629



