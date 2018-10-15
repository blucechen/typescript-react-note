# 枚举，类型推论，类型兼容性，
## 枚举 ENUM 
关键字enum
 
1. 入门
```ts
// 1. 自增和自动赋初值
enum types {
  FACE, // 0
  SUSPECT = 10000.21,// 如果没有赋值，此处为1
  PASSER// 10001
}
// 2. 翻译后：
var type;
(function (type) {
    type[type["FACE"] = 0] = "FACE";
    type[type["SUSPECT"] = 10000.21] = "SUSPECT";
    type[type["PASSER"] = 10001.21] = "PASSER"; // 10001
})(type || (type = {}));

// 3. 使用
types.FACE // 0
types.0 // 'FACE'

// 计算后的值
function getV() {
  return 12.1
}

// 4. 赋值给某一项，必须保证其后面的枚举项都有初始化值
enum types {
  FACE,
  SUSPECT = getV(),// error PASSER不具有初始化表达式。
  PASSER,
}
enum types {
  FACE = 1,
  PASSER = '123f',// 同理，只可以最后面，或者其后面的有初始化值
  SUSPECT,
}
// 重复的key，types[1] = 'a', type.FACE = 1
enum types {
  FACE = 1,
  PASSER = '123f',// success 同理，只可以最后面，或者其后面的有初始化值
  a = 1,
  SUSPECT,
}
// 翻译：
var type;
(function (type) {
    type[type["FACE"] = 1] = "FACE";
    type["PASSER"] = "123f";
    type[type["a"] = 1] = "a";
    type[type["SUSPECT"] = 2] = "SUSPECT";
})(type || (type = {}));
```

2. 字符串枚举
翻译略有不同
```ts
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}
// 翻译：
var Direction;
(function (Direction) {
    Direction["Up"] = "UP";
    Direction["Down"] = "DOWN";
    Direction["Left"] = "LEFT";
    Direction["Right"] = "RIGHT";
})(Direction || (Direction = {}));
```

3. 常量和计算的成员

4. 枚举成员的类型
```ts
enum ShapeKind {
  Circle,
  Square,
}
interface Circle {
  kind: ShapeKind.Circle;
  radius: number;
}
interface Square {
  kind: ShapeKind.Square;
  sideLength: number;
}
let c: Circle = {
  kind: ShapeKind.Square,// ~~~~~~~~~~~~~~~~ Error!
  radius: 100,
}
```

5. 运行时的枚举（以对象的方式存在）
```ts
enum E {
  X,Y,Z
}
function getV(arg: {X: number}): number {
  return arg.X
}
getV(E)
```

6. const 枚举
编译时候删除
```ts
// 格式：
const enum E {
  A, B
}
// 特点：1.不进行实际编译（不像普通枚举一样生成对象，其不生产代码）
// 2. 常量枚举成员在使用的地方会被内联进来
// 3. 注意： 不可包含计算成员

// 编译结果
const enum Enum {
  A,
  B
}
console.log( Enum.A )
// ----------编译上面
console.log(0 /* A */);
```

7. 外部枚举 TODO
跟const 一样不编译



## 类型推论
一般指typescript 自动完成
1. 自动最佳类型推论
```ts
let x = [0, 1, null];// 推断出x: (number | null)[]
let zoo = [new Rhino(), new Elephant(), new Snake()];// 无法推断出其父类animal,会推断出(Rhino|Elephant|Snake)
// let zoo: (Rhino | Elephant | Snake)[]
```

2. 上下文类型



## 类型兼容性
#### 基本应用
1. 结构性类型系统
typescript基于结构类型系统(与之相对的是基于名义(nominal)类型系统)
> 结构类型是一种只使用其成员来描述类型的方式。 它正好与名义（nominal）类型形成对比。（译者注：在基于名
> 义类型的类型系统中，数据类型的兼容性或等价性是通过明确的声明和/或类型的名称来决定的。这与结构性类型系统不同，它是基于类型的组成结构，且不要求明确地声明

```ts
interface Named {
  k: number
}
class Person {
  constructor(){
    this.k = 123
  }
  k: number
}
let v: Named = new Person()// 类型是兼容的
```

2. 对象，函数，函数返回值，剩余参数的兼容性

```ts
// 1. 函数： 更严格的可以接受不那么严格的
let fnA = function (arg1: string): number {
  return 1
}
let fnB = function (arg1: string, arg2: boolean): number {
  return 1
}
// fnA = fnB // error
fnB = fnA


// 2. 对象： 不那么精确的可以接受更精确的
let objA: {a: string} = {a: 'aaa'};
let objB = {a: 'aaaa', b: 123}
objA = objB
// objA = {a: 'aaaa', b: 123}   // 这是error


// 3. 函数返回值： 同对象
let fnA = function (arg1: string) {
  return {a: 1, b: ''}
}
let fnB = function (arg1: string, arg2: boolean) {
  return {a: 1, b: '', c: false}
}
// fnA = fnB
fnB = fnA// 更多信息存在于fnB的返回值


// 4. 剩余参数
function invokeLater(args: any[], callback: (...args: any[]) =>void) {
}
invokeLater([1, 2], (x, y) => console.log(x + ', ' + y));
invokeLater([1, 2], (x?, y?) => console.log(x + ', ' + y));

interface FnA {
  (...args: any[]): void
}
let fnA: FnA = function (a, b?, c?, ...arg) {
  console.log( a, b, c , arg)
}
```

3. 函数 重载的兼容性
需要符合上面的 **允许'不可靠'属性**
```ts
function getV(arg: string): number
function getV(arg: number, arg2: string): number[]
function getV(arg: any/* , arg2: string */):any {
  if( typeof arg === 'string' ) {
      return 1
  }else {
    return [1]
  }
}
getV(1, '')
```


4. ENUM 的兼容性
枚举类型与数字类型兼容，并且数字类型与枚举类型兼容。不同枚举类型之间是不兼容的


5. class的兼容性
> 类与对象字面量和接口差不多，但有一点不同：类有静态部分和实例部分的类型。比较两个类类型的对象时，只有实例的成员会被比较。 静态成员和构造函数不在比较的范围内;
> 如果目标类型包含一个私有成员，那么源类型必须包含来自同一个类的这个私有成员,即继承
```ts
class Animal {
  feet: number;
  static m1: number = 1
  constructor(name: string, numFeet: number) { }
}
class Size {
  feet: number;
  static m1: string = ''
  constructor(numFeet: number) { }
}
let a: Animal = new Animal('', 1);
let s = new Size(1);
a = s; //OK
s = a; //OK
```

6. 泛型的兼容性
```ts
// 1. 没有使用泛型
interface Empty<T> { }
class A implements Empty<number> {
  constructor(params: string) {}
  a: 'aaa'
}
let x: Empty<number> = new A('');
let y: Empty<string> = {b: 'bb'};
x = y; // okay, y matches structure of x

// 2. 使用了具体的泛型
interface NotEmpty<T> {
data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;
x = y; // error, x and y are not compatible

// 3. 没有指定具体的泛型
let identity = function <T>(x: T): T {
  return x
}
let reverse = function <U>(y: U): U {
  return y
}
identity = reverse; // Okay because (x: any)=>any matches (y: any)=>any
```

### 高级类型
1. 交叉类型
交叉类型是将多个类型合并为一个类型, 取其并集；eg: `T & U`表明同时具有了T和U的类型的成员
```ts
// let x = {} as T & U表明希望将x转为T和U的联合类型
function extend<T, U>(first: T, second: U): T & U {
  let result = {} as T & U;// react写法， result就同时具有了T和U的属性，类型断言
  // let result = <T & U>{};// ts一种写法
  for (let id in first) {
    (result as any)[id] = (first as any)[id];
  }
  for (let id in second) {
    if (!result.hasOwnProperty(id)) {
      (result as any)[id] = (second as any)[id];
    }
  }
  return result;
}
class Person {
  constructor(public name: string) { }
}
interface Loggable {
  log(): void;
}
class ConsoleLogger implements Loggable {
  log() {}
}
var jim = extend(new Person("Jim"), new ConsoleLogger());
var n = jim.name;
jim.log();
```

2. 联合类型1--联合
```ts
function padLeft(value: string, padding: string | number) {
  // 如果使用padding,则只能使用string和number的共有成员
}
let indentedString = padLeft("Hello world", true)
```

3. 联合类型2--类型断言&类型保护
`obj as type`类型断言，
`parameterName is Type`类型保护
`typeof v === 'typename'`,`typeof v !== 'typename'`类型保护（typename 仅支持 'number','string','boolean','symbol'）
`instanceof`类型保护

```ts
// 1. 类型断言
interface Bird {
  fly: () => void;
  layEggs(): void;// 挂载到原型上
}
interface Fish {
  swim(): any;
  layEggs(): any;
}
class BBird implements Bird {
  fly() {}
  layEggs() {}
}
function getSmallPet(): Fish | Bird {
  return new BBird()
}
console.log( new BBird )
let pet = getSmallPet();
pet.layEggs(); // okay
(pet as Bird).fly(); // 多次断言

```
```ts
// 类型保护1 ---  parameterName is Type
function isBird(pet: Fish | Bird): pet is Bird {//
  return (pet as Bird).fly !== undefined;
}
if (isBird(pet)) {
  pet.fly();
}
else {
  pet.swim();
}
console.log( pet )// 还是只能在if-else里面使用

// 类型保护2 --- typeof v === typename , typeof v !== typename 针对原始类型无需抽象成函数
function isString(v: any): v is string {
  return typeof v === 'string'
}
console.log( isString({}) )
function getV(arg: string | number) {
  // if (isString(arg)) {
  if(typeof arg === 'string')
    return arg + 1
  }else {
    return arg
  }
}


// 类型保护3 --- instanceof  类似于typeof
interface Padder {
  getPaddingString(): string
}
class SpaceRepeatingPadder implements Padder {
  constructor(private numSpaces: number) { }
  getPaddingString() {
    return Array(this.numSpaces + 1).join(" ");
  }
}
class StringPadder implements Padder {
  constructor(private value: string) { }
  getPaddingString() {
    return this.value;
  }
}
function getRandomPadder() {
  return Math.random() < 0.5 ? new SpaceRepeatingPadder(4) : new StringPadder(" ");
}
let padder: Padder = getRandomPadder();
if (padder instanceof SpaceRepeatingPadder) {
  padder; // 类型细化为'SpaceRepeatingPadder'
}
if (padder instanceof StringPadder) {
  padder; // 类型细化为'StringPadder'
}

```

4. null 和undefined的处理
>TypeScript会把 null 和 undefined 区别对待
>默认情况下，类型检查器认为 null 与 undefined 可以赋值给任何类型。 null 与 undefined 是所有其它类型的一个有效值
>--strictNullChecks 标记可以解决此错误：当你声明一个变量时，它不会自动地包含 null 或 undefined
```ts
// 1. 不包含null或undefined
let s = "foo";
s = null; // 错误, 'null'不能赋值给'string'
let sn: string | null = "bar";
sn = null; // 可以

// 2. 可选参数和可选属性-自动加上undefined但不加上null
function f(x: number, y?: number) {
  return x + (y || 0);
}
f(1, 2);
f(1);
f(1, undefined);
f(1, null);// 类型“null”的参数不能赋给类型“number | undefined”的参数。

// 3. 可选属性
class C {
a: number;
b?: number;
}
let c = new C()
c.b = 13;
c.b = undefined; // ok
c.b = null; // error, 'null' is not assignable to 'number | undefined'

```

5. null的类型保护和类型断言
采用 identifier!从类型里去掉null和undefined
```ts
function f(sn: string | null): string {
  return sn || "default";
}

// identifier!去掉null和undefined
function broken(name: string | null): string {
  function postfix(epithet: string) {
    return name.charAt(0) + '. the ' + epithet; // error, 'name' is possibly null
  }
  name = name || "Bob";
  return postfix("great");
}
function fixed(name: string | null): string {
  function postfix(epithet: string) {
    return name!.charAt(0) + '. the ' + epithet; // ok
  }
  name = name || "Bob";
  return postfix("great");
}

```


#### 类型别名
1. 基本使用
```ts
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
  if (typeof n === 'string') {
    return n;
  }
  else {
    return n();
  }
}
// 编译代码：
function getName(n) {
  if (typeof n === 'string') {
    return n;
  }
  else {
    return n();
  }
}
```

2. 奇怪的类型别名
可以在类型别名里引用自己
类型别名不能出现在声明右侧的任何地方
```ts
// 1.
type LinkedList<T> = T & { next: LinkedList<T> };
interface Person {
  name: string;
}
let y = {}
let people: LinkedList<Person> = y as LinkedList<Person>;
people.next = y as LinkedList<Person>
people.name = '123'

// 2.
type Tree<T> = {
  value: T;
  left: Tree<T>;
  right: Tree<T>;
}
let x = {}
let y: Tree<string> = <Tree<string>>x
y.value = ''
y.left = <Tree<string>>x
y.right = <Tree<string>>x

// 3. 类型别名不能出现在声明右侧的任何地方
type Yikes = Array<Yikes>; // error
```

3. 类型别名 vs 接口
> a.鼠标悬停显示信息不同;
> b.别名不能被extends和complements

4. 字符串字面量类型
```ts
type Easing = "ease-in" | "ease-out" | "ease-in-out";
function getV(params: Easing): Easing {
  return params
}
function getVV(params: 'abc') {// 此处只能传进abc这一type
  return params
}
getV('ease-in')
getVV('abc')
getVV('abcc')// error
```

6. 数字字面量类型(同上)


7. 可辨识的联合
你可以合并`单例类型`，`联合类型`，`类型保护`和`类型别名`来创建一个叫做`可辨识联合`的高级模式，它也称做`标签联合`或`代数数据类型`
```ts
interface Square {// 单例类型
  kind: "square"; //kind 属性称做可辨识的特征或标签
  size: number;
}
interface Rectangle {
  kind: "rectangle";//kind 属性称做可辨识的特征或标签
  width: number;
  height: number;
}
interface Circle {
  kind: "circle";//kind 属性称做可辨识的特征或标签
  radius: number;
}

type Shape = Square | Rectangle | Circle;// 可辨识的联合,类型别名，类型联合
function area(s: Shape) {
  switch (s.kind) {
    case "square": return s.size * s.size;//类型保护
    case "rectangle": return s.height * s.width;
    case "circle": return Math.PI * s.radius ** 2;
  }
}


//**************** 针对上的一个完整性检查**************************
// 背景： 添加了一个临时的shape 例如： triangle,时候，area无法覆盖全部的信息,但没有开发提示
// 解决： 1. area(s:Shape): number， 明确返回值
// 解决： 2.使用never检查
function assertNever(x: never): never {
  throw new Error("Unexpected object: " + x);
}
type Shape = Square | Rectangle | Circle | Triangle;// 可辨识的联合,类型别名，类型联合

function area(s: Shape) {
  switch (s.kind) {
    case "square": return s.size * s.size;//类型保护
    case "rectangle": return s.height * s.width;
    case "circle": return Math.PI * s.radius ** 2;
    default: return assertNever(s);// 如果没有全部覆盖，会导致asserNever报错
  }
}
```

#### (续)高级类型
1. 多态的this类型,可以返回this
```ts
class BasicCalculator {
  public constructor(private value: number = 0) { }
  private y: string
  public currentValue(): number {
    return this.value;
  }
  public add(operand: number): this {// 可以返回this
    this.value += operand;
    return this;
  }
  public multiply(operand: number): this {
    this.value *= operand;
    return this;
  }
}

class MC extends BasicCalculator {
  constructor(private x: number){
    super(x)
  }
  odd(o: number): this {
    // this.value -= o
    return this
  }
}

```

2. 索引类型
eg:`<T, K extends keyof T>`
```ts
function getV<T, K extends keyof T>(obj: T, keys: K[]): T[K][] {
  return keys.map(v => obj[v])
}
let obj = {
  name: 'x',
  age: 12,
  city: 'shenzhen'
}
let city = getV(obj, ['city', 'age'])// 自动识别key
```

3. 索引类型和字符串索引类型
`[key: string]: T`, `[key: number]: T`
```ts
interface Map<T> {
  [key: string]: T;
}
let keys: keyof Map<number>; // string
let value: Map<number>['foo']; // number
```

4. 映射类型
从旧类型创建新类型`K in keyof T`, `K extends keyof T`
```ts
// 1. 
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
}
type Partial<T> = {
  [P in keyof T]?: T[P];
}

// how to use
type PersonPartial = Partial<Person>;
type PersonPartial = Readonly<Person>;

// 2. 不改变原来的修饰符
type RO<T> = {
  [k in keyof T]?: T[k]|null
}
interface O {
  readonly k1: string
}
type RO_O = RO<O>// 修饰出来的k1依然具有readonly

```

5. 更为复杂的映射类型TODO
```ts
type Proxy<T> = {
  get(): T;
  set(value: T): void;
}
type Proxify<T> = {
  [P in keyof T]: Proxy<T[P]>;
}
function proxify<T>(o: T): Proxify<T> {
  let res = {} as Proxify<T>

  for (const key in o) {
    if (o.hasOwnProperty(key)) {
      res[key] = {
        get():T[keyof T] {//ype '() => T[keyof T]' is not assignable to type '() => T[Extract<keyof T, string>]'
          return o[key]
        },
        set(val: T[keyof T]) {
          o[key] = val
        }
      }
    }
  }
  return res
}
let proxyProps = proxify({a: 'a', b: 13});
proxyProps.b.set(12)
```


6. ts标准库类型映射
```ts
interface A {
  name?: string
  readonly age?: number
  city?: string
}

type RA = Readonly<A>
type PA = Partial<A>
type RequiredAA = Required<A>
type PiA = Pick< A, keyof A>
type PiAA = Pick< A, 'name'|'age'>
type ReA = Record<'length'|'slice', A>// 不同态，会创建新的

// 标准库
type Readonly<T> = {
  readonly [key in keyof T]: T[key]
}
type Partial<T> = {
  [key in keyof T]?: T[key]
}
type Pick<T, keys extends keyof T> = {
  [key in keys]: T[key]
}
type Record<keys extends string, T> = {
  [key in keys]: T
}
type Required<T> = {
    [P in keyof T]-?: T[P];
};

```

7. 由映射类型进行推断
```ts
type Proxy<T> = {
  get(): T;
  set(value: T): void;
}
type Proxify<T> = {
  [P in keyof T]: Proxy<T[P]>;
}
function unproxify<T>(t: Proxify<T>): T {
  let result = {} as T;
  for (const k in t) {
    result[k] = t[k].get();
  }
  return result;
}
let originalProps = unproxify(proxyProps);
```



















