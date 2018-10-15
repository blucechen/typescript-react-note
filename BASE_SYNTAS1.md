# 函数，接口，类，泛型
基本参考ts中文指南顺序
## 函数

#### 缺省值并设置默认值
```ts
function keepWholeObject(wholeObject: { a: string, b?: number }) {
    let { a, b = 1001 } = wholeObject;
}
```

#### 类型推断并解构默认赋值
```ts
function f({ a, b } = { a: "", b: 0 }): void { }
f(); // ok, default to { a: "", b: 0 }

function f({ a, b = 0 } = { a: "" }): void { }
f({ a: "yes" }); // ok, default b = 0

```

#### 额外的属性检查
option bags
```ts
//  对直接传入字面量- 会添加额外的属性检查导致报错
interface SquareConfig {
  color?: string;
  width?: number;
}
function createSquare(config: SquareConfig): { color: string; area: number } {
  return {color: '1', area: 1}
}
let mySquare = createSquare({ colour: "red", width: 100 })//由于额外的属性检查导致多余的属性出现时报错


// 解决方法1：
// 去掉额外的属性检查   ----    类型断言
let mySquare = createSquare({ colour: "red", width: 100 } as SquareConfig)

// 解决方法2：
// 添加一个变量
let config = { colour: "red", width: 100 }
let mySquare = createSquare(config)


// 解决方法3：
// 添加额外的字符串索引签名
interface SquareConfig {
  color?: string
  width?: number
  [key: string]: any// 本身就是可选的
}

```

#### 可索引的类型
我们也可以描述那些能够“通过索引得到”的类型
```ts
// 数字索引签名
interface StringArray {
    [index: number]: string
}

// 字符串索引签名---其他key的值类型必须与之相同
interface NumberDictionary {
    length: number; // 可以，length是number类型
    name: string // 错误，`name`的类型与索引类型返回值的类型不匹配
    [index: string]: number;
}

// 注意：
class Animal {
name: string;
}
class Dog extends Animal {
breed: string;
}
// 错误：使用数值型的字符串索引，有时会得到完全不同的Animal!
// 同时使用两种类型的索引，但是数字  索引的返回值必须是字符串索引返回值类型的子类型
interface NotOkay {
    [x: number]: Animal;
    [x: string]: Dog;
}

// 支持readonly属性
interface A {
  readonly [x: number]: string
}

``` 

## 接口
#### 入门
```ts
// 描述公共属性和方法
interface ClockInterface {
  currentTime: Date;
  setTime(d: Date): void;// 描述方法
  // new (x: string) error
}

// 当一个类实现了一个接口时，只对其实例成员进行类型检查。 
// constructor 存在于类的静态部分，所以不在检查的范围内。
class  X implements ClockInterface {
  currentTime: Date
  setTime(d: Date) {
    this.currentTime = d
  }
  constructor(x: string) {
  }
}

```

#### 检查constructor
```ts
// 构造函数签名
interface TimeConstru {
  new (args: string): TimeInstance
}

interface TimeInstance {
  tick: (args?: string) => Date
}

class Time implements TimeInstance {
  constructor(args: string) {
    
  }
  tick() {
    return new Date()
  }
}

function createTime(constructor: TimeConstru, args: string): TimeInstance {
  return new constructor(args)
}
let timer = createTime(Time, '')
```

#### 接口继承接口
```ts
  interface Shape {
    color: string;
  }
  interface Square extends Shape {
    sideLength: number;
  }
  let square = <Square>{};// 类型转换

  // 函数也是对象----类型断言
  interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
  }
  function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
  }
  let c = getCounter();
  c(10);
  c.reset();
  c.interval = 5.0;
```

#### 接口继承类
```ts
class Control {
  private state: any;
}
interface SelectableControl extends Control {// 继承了 Control，由于其包含了私有属性，造成实现类无法直接创建 
  select(): void;
}
class Button extends Control implements SelectableControl {
  select() { }
}
class TextBox extends Control {
  select() { }
}
// Error: Property 'state' is missing in type 'Image'. 因为 state 是私有成员，所以只能够是 Control 的子类们才能实现 SelectableControl 接口
class Imagex extends Control implements SelectableControl {
  state = 1
  select() { }
}
```

## 类
#### 基本概念
1. 默认为 public 
采用的是`结构性类型系统`，ts不在乎所声明的成员从何处来，只关心是否一样

2. private只能在类内部使用
当成员被标记成 private 时，它就不能在声明它的类的外部访问；
如果其中一个类型里包含一个 private 成员，那么只有当另外一个类型中也
存在这样一个 private 成员， 并且它们都是来自同一处声明时，我们才认为这两
个类型是兼容的
```ts
class Animal {
private name: string;
  constructor(theName: string) { this.name = theName; }
}
new Animal("Cat").name; // 错误: 'name' 是私有的.
```

3. protected
可以被子类使用，可以对构造函数使用，这样其就不能被直接实例化，必须使用子类继承其再实例化
```ts
// 无法直接实例化
class Person {
  protected constructor(name: string) {
    console.log( name )
  }
}
class Student extends Person {
  constructor(name: string, age: number) {
    super(name)
  }
}
console.log( new Student('11', 1) )
```

4. readonly修饰符
你可以使用 readonly 关键字将属性设置为只读的。 只读属性必须在声明时或构
造函数里被初始化
```ts
class Octopus {
  readonly name: string;
  readonly numberOfLegs: number = 8;
  constructor (theName: string) {
  this.name = theName;
  }
}
let dad = new Octopus("Man with the 8 strong legs");
dad.name = "Man with the 3-piece suit"; // 错误! name 是只读的.

```

5. 参数属性
```ts
class Animal {
  constructor(private name: string) { }// 我们把声明和赋值合并至一处, readonly也是可以的
    move(distanceInMeters: number) {
    console.log(`${this.name} moved ${distanceInMeters}m.`);
  }
}
```

5. 存取器
存取器要求你将编译器设置为输出ECMAScript 5或更高
只带有 get 不带有 set 的存取器自动被推断为 readonly 
```ts
class Person {
  private pName: string// 私有化
  constructor( name: string) {
    this.pName = name
  }
  get name() {
    console.log( 'getName' )
    return this.pName
  }
  set name(newName) {
    console.log( 'setName' )
    this.pName = newName
  }
}
let p = new Person('ppp')
```


#### 抽象类
无法实例化
```ts
abstract class Person {
  name: string
  constructor(name: string) {
    this.name = name
  }
  getName() {
    return this.name
  }
  abstract setName():void//子类中必须实现该方法
}
class Stu extends Person {
  constructor(name: string) {
    super(name)
  }
  setName(): void {
  }
}
```

#### 高级技巧
1. 深入构造函数及编译后的代码
```ts
// ts
class Greeters {
  static g: string = ''
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }
  greet() {
    return "Hello, " + this.greeting;
  }
}

// 编译后：js
var Greeters = /** @class */ (function () {
    function Greeters(message) {
        this.greeting = message;
    }
    Greeters.prototype.greet = function () {
        return "Hello, " + this.greeting;
    };
    Greeters.g = '';
    return Greeters;
}());
```

2. typeof Greeter
我们创建了一个叫做 greeterMaker 的变量。 这个变量保存了这个类或者说保存了类构造函数。 然后我们使用 typeof Greeter ，意
思是取Greeter类的类型，而不是实例的类型。 或者更确切的说，"告诉我 Greeter 标识符的类型"，也就是构造函数的类型。 这个类型包含了类的所有
静态成员和构造函数
```ts
class Greeter {
  static standardGreeting = "Hello, there";
  greeting: string;
  greet() {
    if (this.greeting) {
      return "Hello, " + this.greeting;
    }
    else {
      return Greeter.standardGreeting;
    }
  }
}
let greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";
let greeter2: Greeter = new greeterMaker();
console.log(greeter2.greet());
```

3. 类当接口使用(接口继承类)
```ts
class Point {
  x: number;
  y: number;
}
interface Point3d extends Point {
  z: number;
}
let point3d: Point3d = {x: 1, y: 2, z: 3};
```

## 函数
1. 完整的函数定义
```ts
// 推断定义
let myAdd = function(x: number, y: number): number { return x + y; };
// 完整的定义
let myAdd: (baseValue: number, increment: number) => number =
function(x, y) { return x + y; };
```

2. 可选参数和默认参数
```ts
// 1. 参数个数(可选参数必须放在最后面)
function build(name: string, age?: number): string {
  return name
}
build('bName', 123)
build('bName', 123, 123)// error, 只接受1-2个参数

// 2. 默认值(也是可省略的)(可以不放在最后位置，如果不放在最后需要传入undefined获取默认值)
function build(name: string, age: number = 123): string {
  return name
}
build('bName')
build('bName', 234)
```

3. 箭头函数绑定上级作用域(this)
```ts
let deck = {
  suits: ["hearts", "spades", "clubs", "diamonds"],
  cards: Array(52),
  createCardPicker: function () {
    return () => {// this将会被绑定为deck
      let pickedCard = Math.floor(Math.random() * 52);
      let pickedSuit = Math.floor(pickedCard / 13);
      return { suit: this.suits[pickedSuit], card: pickedCard % 13 };
    }
  }
}
let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();
alert("card: " + pickedCard.card + " of " + pickedCard.suit);

```

4. this参数
解决3中`this.suits[pickedSuit]`出现any的现象(noImplicitThis模式下会提示报错)
提供一个显式的 `this` 参数，`this` 参数是个假的参数，它出现在参数列表的最前面
```ts
interface Card {
  suit: string;
  card: number;
}
interface Deck {
  suits: string[];
  cards: number[];
  createCardPicker(this: Deck, arg1: string): () => Card;
}
let deck: Deck = {
  suits: ["hearts", "spades", "clubs", "diamonds"],
  cards: Array(52),
  // NOTE: The function now explicitly specifies that its callee must be of type Deck
  createCardPicker: function (this: Deck, arg1: string) {// 不会作为实际的参数传进去
    return () => {
      let pickedCard = Math.floor(Math.random() * 52);
      let pickedSuit = Math.floor(pickedCard / 13);
      return { suit: this.suits[pickedSuit], card: pickedCard % 13 }//this不在是any
    }
  }
}
let cardPicker = deck.createCardPicker( 'string......');
let pickedCard = cardPicker();
```

5. 回调函数里面的this指向
```ts
interface UIElement {
  addClickListener(onclick: (this: void, e: Event) => void): void;
}
class AppEle implements UIElement {
  addClickListener(onclick: (this: void, e: Event) => void): void {
    console.log( onclick )
    // onclick()
  }
}
class Handler {
  info: string;
  // onClickBad(this: Handler, e: Event) {//指定this为Handler，表明必须在 Handler 的实例上调用
  onClickBad(this: void, e: Event) {//绑定this为void
    // oops, used this here. using this callback would crash at runtime
    // this.info = e.message;// 绑定了this为void，就不能使用this.info了，
    console.log('clicked!');
  }
  // 两者都需要，就需要使用箭头函数
  // onClickGood = (e: Event) => { this.info = e.message }
}
let h = new Handler();

let appE = new AppEle()
appE.addClickListener(h.onClickBad)
```

#### 重载
```ts
let suits = ["hearts", "spades", "clubs", "diamonds"];
function pickCard(x: { suit: string; card: number; }[]): number;
function pickCard(x: number): { suit: string; card: number; };
function pickCard(x: any): any {//这里并不是重载，只是重载的处理函数
  if (typeof x == "object") {
    let pickedCard = Math.floor(Math.random() * x.length);
    return pickedCard;
  }
  // Otherwise just let them pick the card
  else if (typeof x == "number") {
    let pickedSuit = Math.floor(x / 13);
    return { suit: suits[pickedSuit], card: x % 13 };
  }
}
let myDeck = [{ suit: "diamonds", card: 2 }, {suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
let pickedCard2 = pickCard(15);
```


## 泛型
#### 入门
看类型提示
```ts
function getValue<T>(params:T): T {
  return params
}
// 俩种使用方式
getValue(false)
getValue<boolean>(false)

// 数组
function getValue<T>(params: T[]): T {
  return params[0]
}
function getValue<T>(params: Array<T>): T {
  return params[0]
}
```

#### 泛型类型
1. 完整定义泛型函数的类型
类型参数放在在最前面
```ts
// 1
function getName<T>(params:T): T {
  return params
}
getName(1)

// 2.
let getNumber: <T>(args: T) => T = function (params) {
  return params
}
getNumber<number>(1)

// 3. 类接口方式
let getNumber: {<T>(arg: T): T} = function (params) {
  return params
}

// 4
const fn: <T>(arg: T) => {...}// 返回值会根据里面是否有return自动推断

```
2. 泛型放在接口上还是调用签名上
更好的泛型接口
```ts
// 1.泛型放置在调用签名上，无法指定T
interface Gen {
  <T>(args: T): T
}
function getV<U>(args: U): U {// 也必须使用泛型
  return args
}
const getValue: Gen = getV

// 2: 将泛型放置到接口上，必须指定T
interface Gen<T> {
  (params: T): T // 不是<T>(params: T): T
}
function getV<U>(params: U): U {// 也是必须是泛型
  return params
}
const getValue: Gen<number> = getV// 指定了类型
getValue(1)
```

3. 泛型类
```ts
class GenericNumber<T> {
zeroValue: T;
add: (x: T, y: T) => T;
}
let myGenericNumber = new GenericNumber<number>();
```

4. 泛型约束
具有某一属性
```ts
interface Len {
  length: number
}
interface Gen<T extends Len> {
  (params: T): T
}
function getV<U>(params: U): U {
  return params
}
const getValue: Gen<number> = getV
getValue(1)
```

5. 泛型约束2--从泛型中使用类型参数
从泛型中取某一key，并获取对应的value
```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}
let x = { a: 1, b: 2, c: 3, d: 4 };                                                                                              
getProperty(x, "a"); // okay
```

6. 泛型约束3--class类型
工厂函数中就需要在泛型中使用class类型
```ts
// 1.直接使用
function create<T>(c: { new(): T; }): T {
  return new c();
}
class Person {
  name = 'person'// 是绑定到实例上的
  getName(args: number): string {// 绑定到原型上
    return ''
  }
}
let res = create<Person>(Person)

// 2.继承
class BeeKeeper {
  hasMask: boolean;
}
class Animal {
  numLegs: number;
}
class Bee extends Animal {
  keeper: BeeKeeper;
}
function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}
createInstance(Bee).keeper.hasMask; // typechecks
```

















