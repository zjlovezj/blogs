# 前言

修复官方的plugin的bug，让子类可以继承父类的static属性。

```js
// A.js
export default class {
    static a() {
        console.log('A.a()');
    }
}
```

```js
// B.js
import A from './A';

export default class extends A{
    static b() {
        console.log('B.b()');
    }
}
```

```js
// test.js
import A from './A';
import B from './B';

console.log(A);
console.log(B);

A.a();
// B.a(); // 在IE10以下出错
B.b();
```

## bug描述

在Babel对代码进行转换后，生成一下代码。  
Object.setPrototypeOf 不能在IE10以下运行。  

```js
if (superClass)
    Object.setPrototypeOf
        ? Object.setPrototypeOf(subClass, superClass)
        : _defaults(subClass, superClass);
```

需要 babel-plugin-transform-object-set-prototype-of-to-assign 将 Object.setPrototypeOf 改写为自定义方法。  

但是官方的插件 babel-plugin-transform-object-set-prototype-of-to-assign 转换的代码如下：  

```js
if (superClass)
    Object.setPrototypeOf
        ? _defaults(subClass, superClass)
        : (subClass.__proto__ = superClass);
```

这个插件没有注意到前面的判断语句，直接进行了函数替换。  
我们重写的插件就是要让这个插件改写这里的代码为：  

```js
if (superClass)
    Object.setPrototypeOf
        ? Object.setPrototypeOf(subClass, superClass)
        : _defaults(subClass, superClass);
```

## 解决

修改 babel-plugin-transform-object-set-prototype-of-to-assign 插件。  
使得它在转换以上代码时，如果前面有 Object.setPrototypeOf 的判断，就采用新的代码。

```js
// babel-plugin-transform-object-set-prototype-of-to-assign: lib/index.js

"use strict";

exports.__esModule = true;

exports.default = function() {
  return {
    visitor: {
      CallExpression: function CallExpression(path, file) {
        if (path.get("callee").matchesPattern("Object.setPrototypeOf")) {
          if (
            path.parent.test &&
            path.parent.test.object.name === "Object" &&
            path.parent.test.property.name === "setPrototypeOf" &&
            path.parent.alternate.left &&
            path.parent.alternate.left.object.name === "subClass" &&
            path.parent.alternate.left.property.name === "__proto__"
          ) {
            file.addHelper("defaults");
            path.parentPath.replaceWithSourceString(`Object.setPrototypeOf
                ? Object.setPrototypeOf(subClass, superClass)
                : _defaults(subClass, superClass)`);
          } else if (path.parent.test === undefined) {
            path.node.callee = file.addHelper("defaults");
          }
        }
      }
    }
  };
};

module.exports = exports["default"];

```

## 感想

我们虽然可以使用ES6的方式来编写代码，让Babel转换的成可以在旧浏览器可以运行的代码，但Babel的插件并不是万能。

## 参考

### [Babel IE10 提示](https://babeljs.io/docs/usage/caveats/)
### [Babel plugins 和 presets 执行顺序](https://babeljs.io/docs/plugins/#pluginpreset-ordering)
### [Babel plugin 开发用到的api](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md)
