# Symbol, 模块, 命名空间, JSX
## Symbol
1. es6和es5的instanceof的不同方式
ts中需要配置`target: es5`或者`target: es6`
当配置为`es5`的时候无法实现改写类中的静态 [Symbol.hasInstance]方法，只能修改原型链
```ts
// es5： a instaceof Fn ---> a是否在Fn.prototype的原型链上
function T1() {
}
function T2() {
}
T2.prototype = new T1
let t2 = new T2()
console.log( t2 instanceof T1 )
console.log( t2 instanceof T2 )

// es6：改写class 的静态方法 static [Symbol.hasInstance](obj: any): boolean
class T1 {
  static [Symbol.hasInstance](obj) {// 会被调用
    console.log( obj )
    return true
  }
}
let obj1 = {a: 'aaa'}
console.log( obj1 instanceof T1 )
```

2. for-of的实现Symbol.iterator
当配置为`target: es5`无法实现非数组类型的迭代器
当配置为`target: es6`可以实现内置迭代器
```ts
class T1 {
  static [Symbol.hasInstance](obj) {
    return true
  }
  constructor() {
    this.a = 'a'
    this.b = 'b'
  }
  *[Symbol.iterator]() {
    for (const key in this) {
      if (this.hasOwnProperty(key)) {
        yield this[key]
      }
    }
  }
}

let t1 = new T1()
for (const key1 of t1) {
  console.log( key1, 'jinyue.....' )
}
console.log( {} instanceof T1 )
```


## 模块
术语变迁说明：`内部模块`现在称做`命名空间`，`外部模块`现在则简称为`模块`
eg: `module X {` 相当于现在推荐的写法 `namespace X {`

> TypeScript与ECMAScript 2015一样，任何包含顶级 import 或者 export 的文件都被当成一个模块。
> 相反地，如果一个文件不带有顶级的 import 或者 export 声明，那么它的内容被视为全局可见的（因此对模块也是可见的）

1. 导出
```ts
class ZipCodeValidator implements StringValidator {
  isAcceptable(s: string) {
    return s.length === 5 && numberRegexp.test(s);
  }
}

export const a = 12
export { ZipCodeValidator };
export { ZipCodeValidator as mainValidator };// 导出重命名
```

2. 重新导出
```ts
// testExport1
class T1 {
  constructor() {
  }
  t1: string = '1'
}
export {T1}
export {T1 as T1_1}
export {T3 as T1_3} from './testExport3'


// A.ts - T1_1，K再A.ts中都是无法使用的
export {T1_1 as K} from '../ts/testExport1'
```

3. 全部导出
```ts
// testExport1
export {T1}
export {T1 as T1_1}
// A.ts
export * from '../ts/testExport1'

// B.ts
import {T1, T1_1} from 'A.ts'
```

4. 默认导出
```ts
// or0
export default function (s: string) {
  return s.length === 5 && numberRegexp.test(s);
}
// or1
export default class A() {}
// or2
export default "123";
```

5. Commonjs AMD模块化
> TypeScript模块支持 export = 语法以支持传统的CommonJS和AMD的工作流模型;
> 但是 面向 ECMAScript 模块时，不能使用导出分配。请考虑改用 "export default" 或另一种模块格式（react中用不了）
```ts
// 配置了CommonJS或AMD的环境中可以使用（注：不是所有的环境都可以使用的）
// ZipCodeValidator.ts
let numberRegexp = /^[0-9]+$/;
class ZipCodeValidator {
  isAcceptable(s: string) {
    return s.length === 5 && numberRegexp.test(s);
  }
}
export = ZipCodeValidator;
// B.ts
import zip = require("./ZipCodeValidator");
```


## 模块（外部模块）
TODO



## 命名空间（内部模块）
TODO
```ts
// 模块内部命名空间，引用外部的TODO
namespace Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean;
  }
  const lettersRegexp = /^[A-Za-z]+$/;
  const numberRegexp = /^[0-9]+$/;
  export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
      return lettersRegexp.test(s);
    }
  }
  export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
      return s.length === 5 && numberRegexp.test(s);
    }
  }
}
let strings = ["Hello", "98052", "101"];
let validators: { [s: string]: Validation.StringValidator; } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();
for (let s of strings) {
  for (let name in validators) {
    console.log(`"${s}" - ${validators[name].isAcceptable(s) ? "matches" : "does not match"} ${name}`);
  }
}
```


## JSX









