# SPECIAL
1. typeof
```ts
const initialState = { 
  clicksCount: 0 
}

// 读取到clickCount为number，形成类似于type initialState = {clickCount: number} 的类型
type State = Readonly<typeof initialState>
let o = {} as typeof initialState
```

2. 关于 `<K extends keyof T>` 与 `<T extends A>` 的认识
```ts
interface A {
  a: string,
  aa: boolean,
  aaa: string
}

// 1. extends keyof A
type EA = 'a'|'aa'
type keyA = keyof A// keyof 索引类型查询符，其被extends的和接口的是不同的
let ka: keyA = 'a'

function getA<T extends keyof A>(arg1: T, arg2: Exclude<keyof A, EA>) {
}
type Aextends = 'a'
getA<Aextends>('a', 'aaa')
getA('aa', 'aaa')


// 2. extends A
function getB<T extends A>(params: T) {}
getB({a: '', aa: false, aaa: '', ccc: ''})
```

3. Exclude 与Omit 于Rquired
```ts
/**
 * 1. From T pick a set of properties K
 */
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

/**
 * 2. Exclude from T those types that are assignable to U
 * 从T中排除那些也符合U的(如果是keyof的话，就是从T中找到那些U中没有的)
 * 注： 一般是结合keyof使用，不是对两个接口使用 (当然，对接口使用也是可以的)
 * c
 */
type Exclude<T, U> = T extends U ? never : T;
// eg: 
interface A {
  a: string,
  aa: boolean,
  aaa: string
}
type EA = 'a'|'aa'
type AA = 'a'|'aa'|'aaa'
type EX_A = Exclude<keyof A, EA>// 'aaa'，返回
type EX_A = Exclude<AA, EA>// 'aaa'，返回


type temp = Exclude<keyof T, K>// 从T中找到那些K中没有的键
/**
 * 3. 从T中找到那些K中没有的键，然后组成新的type,
 * 新的type是｛key1: type1, key2: type2｝这类的，不是keyof这类的
 */
type Omit<T, K> = Pick<T, Exclude<keyof T, K>>// K一般是keyof这类的，一般不会是一个接口


// 另一种实现
type DiffPropertyNames<T extends string | number | symbol, U> = 
  {[P in T]: P extends U ? never: P}[T]

type Omit<T, K> = Pick<T, DiffPropertyNames<keyof T, K>>
```

4. 关于泛型中的默认值
`<P extends object, DP extends Partial<P> = Partial<P>>` = 后面是默认值
```ts
// 说明
// DP如果没有传入，就是用Partial<P>
type T_P<P extends object, DP extends Partial<P> = Partial<P>> = {
  readonly [key in keyof DP]: DP[key]
}
let objobj: T_P<PS> = {// yes
  city: '',
  age: 12
}
let objobj: T_P<PS, T_TP> = {// error
  city: '',
  age: 12,
}
```


5. 基本数据类型之 Function
isFunction
```ts
// 1. 更为合理
const isFunction = <T extends Function>(val: any): val is T => typeof val === 'function'

// 2.
const isFunction = (fn: any): fn is Function => Object.prototype.toString.call(fn) === '[object Function]'

// 3.
const isFunction = (fn: any): fn is ()=>void => Object.prototype.toString.call(fn) === '[object Function]'
```


6. 引用第三方的类型
```ts
// 1.
class TES {
  a = 'fd'
}
let x: TES['a'] = 123

// 2.
type TES = {
  a: string
}
let x: TES['a'] = '123'

// 3
interface TES {
  a: string
}
let x: TES['a'] = '123'


```











































