# JSX
开不下去了，先这样吧


## JSX
固有元素与基于值的元素(固有标签和自定义标签)

### 固有元素
1. 查找
固有元素使用特殊的接口 JSX.IntrinsicElements 来查找
```ts
declare namespace JSX {
  interface IntrinsicElements {
    foo: any
  }
}
<foo />; // 正确
<bar />; // 错误
```

### 基于值的元素
1. 查找
基于值的元素会简单的在它所在的作用域里按标识符查找
```ts
import MyComponent from "./myComponent";
<MyComponent />; // 正确
<SomeOtherComponent />; // 错误
```

#### 分类
TypeScript首先会尝试将表达式做为无状态函数组件进行解析。如果解析成功，那么TypeScript就完成了表
达式到其声明的解析操作。如果按照无状态函数组件解析失败，那么TypeScript会继续尝试以类组件的形式进行解析
1. 无状态函数组件（SFC）
```ts
interface FooProp {
  name: string;
  X: number;
  Y: number;
}
declare function AnotherComponent(prop: {name: string});
function ComponentFoo(prop: FooProp) {
  return <AnotherComponent name=prop.name />;
}
const Button = (prop: {value: string}, context: { color: string }) => <button>

// 重载
interface ClickableProps {
  children: JSX.Element[] | JSX.Element
}
interface HomeProps extends ClickableProps {
  home: JSX.Element;
}
interface SideProps extends ClickableProps {
  side: JSX.Element | string;
}
function MainButton(prop: HomeProps): JSX.Element;
function MainButton(prop: SideProps): JSX.Element {
  // ...
}
```


2. 类组件
元素类的类型和元素实例的类型
```ts



```



















