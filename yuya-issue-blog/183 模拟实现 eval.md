
## 实现原理: [`iife`](https://developer.mozilla.org/zh-CN/docs/Glossary/%E7%AB%8B%E5%8D%B3%E6%89%A7%E8%A1%8C%E5%87%BD%E6%95%B0%E8%A1%A8%E8%BE%BE%E5%BC%8F) + [`new Function`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function)

> `eval()` 是一个危险的函数， 它使用与调用者相同的权限执行代码。如果你用 `eval()` 运行的字符串代码被恶意方（不怀好意的人）修改，您最终可能会在您的网页/扩展程序的权限下，在用户计算机上运行恶意代码。更重要的是，第三方代码可以看到某一个 `eval()` 被调用时的作用域，这也有可能导致一些不同方式的攻击。相似的 `Function` 就不容易被攻击。

### `safeEval/index.ts`
```ts
import { obj2json } from '../obj2json';
import { TAnyObject } from '../types';

/**
 * 比原生 eval 安全,比 ast 轻量的 实现
 * @param returnCode - 要 eval 后返回的结果
 * @param vars - 代码字符串需要的变量
 * @param expression - 返回语句(return)前的 表达式
 */
export const safeEval = (returnCode: string, vars: TAnyObject = {}, expression?: string) => {
  const varsStr = Object.entries(vars).reduce((prev, next) => {
    const [key, value] = next;
    prev += `const ${key} = ${obj2json(value)};`;
    return prev;
  }, '');
  return new Function(
    `"use strict";${varsStr};${expression ? expression : ''};return (${returnCode});`
  )();
};

// test
const params = {
  name: 'jiangzhiguo',
  email: 'jiangzhiguo2010@qq.com',
  address: {
    city: 'beijing',
    street: 'xxxx',
  }
}
const keypath = ['params','address','number']
const value = 1000
const newParams = safeEval('params', { params }, `${keypath.join('.')}=${value}`);
console.log(newParams.address.number === value) // true
safeEval('console.log(111)') // 111
```
![image](https://user-images.githubusercontent.com/22312092/92742853-994e5f80-f3b2-11ea-9223-9dfd443d8e71.png)

### `types/index.ts`
```ts
// 对象的索引类型，只有三种，string、number、symbol
export type TPropertyName = string | number | symbol;

export type TAnyObject = Record<TPropertyName, any>;
```

### `obj2json/index.ts`
```ts
/**
 * 对象转字符串,保留值为 undefined,NaN,Infinity,-Infinity,function,symbol
 * @param obj
 * @notice 该方法不可逆,即不能 JSON.parse
 */
export const obj2json = (obj: TAnyObject): string => {
  try {
    return JSON.stringify(obj, (key, value) => {
      if ([undefined, NaN, Infinity, -Infinity].includes(value) || typeof value === 'function') {
        return `{{{${value}}}}`;
      }
      if (typeof value === 'symbol') {
        value = value.toString();
        return `{{{${value}}}}`;
      }
      return value;
    })
      .replace(/"{{{/g, '')
      .replace(/}}}"/g, '');
  } catch (err) {
    throw new Error(err);
  }
};
```
源码: <https://github.com/jsany/any/blob/main/packages/shared-utils/src/safeEval/index.ts>

__如有不足,欢迎指正👏__