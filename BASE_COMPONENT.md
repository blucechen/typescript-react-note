# react组件模式

## SFC---无状态组件
1. 基本实现
```ts
type Props = {
  onClick(e: React.MouseEvent<HTMLElement>): void,
  children?: React.ReactNode
}
const Button = ({onClick: handleClick, children}: Props) => (
  <button onClick={handleClick}>{children}</button>
)
```

2. React.SFC模式
在@types/react中已经预定义一个类型type SFC<P>，它也是类型interface StatelessComponent<P>的一个别名，此外，
它已经有预定义的children和其他（defaultProps、displayName等等…），所以我们不用每次都自己编写！ 
```ts
// React.SFC实现代码
// type SFC<P = {}> = StatelessComponent<P>;
// interface ReactElement<P> {
//   type: string | ComponentClass<P> | SFC<P>;
//   props: P;
//   key: Key | null;
// }
// type ValidationMap<T> = {[K in keyof T]?: Validator<T> };
// interface StatelessComponent<P = {}> {
//     (props: P & { children?: ReactNode }, context?: any): ReactElement<any> | null;//返回ReactElement
//     propTypes?: ValidationMap<P>;
//     contextTypes?: ValidationMap<any>;
//     defaultProps?: Partial<P>;
//     displayName?: string;
// }
type Props = {
  onClick(e: React.MouseEvent<HTMLElement>): void
}
const Button: React.SFC<Props> = ({onClick: handleClick, children}) => (
  <button onClick={handleClick}>{children}</button>
)
```

3. 针对defaultProps的处理1
TS虽然可以指定类型，但是无法检测到此处的默认值
```tsx
type Props = {
  onClick(e: MouseEvent<HTMLElement>): void;
  color?: string;
};

const Button: SFC<Props> = ({ onClick: handleClick, color, children }) => {
  // console.log(color.length)// 由于color在定义的时候是可选参数，也就是可能是undefined，所以会报错
  // 1. !运算符
  console.log(color!.length)
  // 2. ||
  color = color||''
  console.log(color)
  // 3. 高阶函数
  return ( 
    <button style={{ color }} onClick={handleClick}> {children} </button> 
  )
}
// 这里中， TS虽然可以指定类型，但是无法检测到此处的默认值
Button.defaultProps = {
  color: 'blue',
}
```

4. 针对defaultProps的处理2---高阶组件方法
```tsx
const defaultProps = { color: 'red' }
type DefaultProps = typeof defaultProps
type Props = {
  onClick: (e: React.MouseEvent<HTMLElement>) => void
} & DefaultProps

// 这里的类型就是Props，其中color为string啊
const Button: React.SFC<Props> = ({onClick: handleClick, color, children}) => {
  console.log( color.length )
  return (<button onClick={handleClick} style={{color}}>{children}</button>)
}


// 高阶组件
// 这里的对外提供的被包装的组件， 该组件的类型Props中会自动将DefaultProps中转成可选属性
const withDefaultProps =
<P extends object, DP extends Partial<P> = Partial<P>>(
  defaultProps: DP,
  Cmp: React.ComponentType<P>// P就是上面的 type Props// 不包括SFC中定义的内容
) => {

  // 找到必须的键组成的接口或者type,不是keyof这类的，要注意（就是P中有，而DP中没有的）
  type RequiredProps = Omit<P, keyof DP>;// 有可能是非必须的
  
  // RequiredProps转成必须的 DP的转成非必须的
  type Props = Partial<DP> & Required<RequiredProps>;

  Cmp.defaultProps = defaultProps;// 没有改变默认值

  return (Cmp as React.ComponentType<any>) as React.ComponentType<Props>;
}

const ButtonWithDefaultProps = withDefaultProps(defaultProps, Button);

// 匿名SFC方式使用
let XButton = withDefaultProps<Props>(defaultProps, ({ onClick: handleClick, color, children }) => (
  <button style={{ color }} onClick={handleClick}>
    {children}
  </button>
))

// 匿名class方式
let YButton = withDefaultProps(defaultProps, 
  class Button extends React.Component<Props, {}> {
    render() {
      let x = this.props
      return (<button style={{color: x.color}} onClick={x.onClick}>{x.children}</button>)
    }
  }
)

```

　

## 有状态组件
1. 基本实现
```ts
const initailState = {clickCount:0}
// 1. 映射内容为只读
type State = Readonly<typeof initailState>

// 2. 将状态更新函数提取到类的外部作为纯函数
const add = (prevState: State) => ({
  clickCount: prevState.clickCount + 1
})
const odd = (prevState: State) => ({
  clickCount: prevState.clickCount - 1
})
class ButtonWidthState extends React.Component<object, State> {
  readonly state: State = initailState// 3. 显式的将state显示为只读， 同时将state的每个属性都指定为只读
  private handleAdd = () => {
    this.setState(add)
  }
  private handleOdd = () => {
    this.setState(odd)
  }
  render() {
    const {clickCount} = this.state
    return (
      <div>
        <Button onClick={this.handleAdd}>++</Button>
        <Button onClick={this.handleOdd}>--</Button>
        value: {clickCount} times!
      </div>
    )
  }
}

```

## 模式实现
> 比如说你需要构建一个可展开的菜单组件，它需要在用户点击它时显示子内容
> Toggleable组件

### **render回调/render属性模式**

1. Toggleable组件实现
```tsx
const initialState = {
  show: false,
};
type State = Readonly<typeof initialState>;// 将里面的每一项转成readonly


// children或render函数接受的参数
type ToggleableComponentProps = {
  show: State['show'];// 取出State中show的类型
  toggle: Toggleable['toggle'];// 从class中取出
};

type RenderCallback = (args: ToggleableComponentProps) => JSX.Element;

// 可以不传，但不能传不是这俩中函数的； 注意： 空格，换行也是内容
type Props = Partial<{
  children: RenderCallback;
  render: RenderCallback;
}>;

const isFunction = (fn: any): fn is Function => Object.prototype.toString.call(fn) === '[object Function]'

// 状态更新依然是纯函数的实现
const updateShowState = (prevState: State) => ({ show: !prevState.show })


// 控制state状态的改变， 对外提供的也就是这俩功能， 这就实现了Toggleable函数控制显示了
// 仅仅保留的状态管理， 而不关心渲染逻辑
export class Toggleable extends React.Component<Props, State> {
  readonly state: State = initialState;
  private toggle = (event: React.MouseEvent<HTMLElement>) => this.setState(updateShowState);

  render() {
    const { render, children } = this.props;// 由于render和children都将返回JSX.Element，所以是可以的
    const renderProps = {
      show: this.state.show,
      toggle: this.toggle,
    };

    if (render) {
      return render(renderProps);
    }
    return isFunction(children) ? children(renderProps) : null;// 类型保护
  }
}

```

2. how to use
```tsx
// 1.基本使用1
const Tg = () => (
  <Toggleable
    render={({ show, toggle }) => (
      <>
        <div onClick={toggle}>
          <h1>some title</h1>
        </div>
        {show ? <p>some content</p> : null}
      </>
    )}
  />
)



// 1.基本使用2
type Props = { title: string }
const ToggleableMenu: React.SFC<Props> = ({ title, children }) => (
  {/* 防止空格这种问题，这里如果使用render，建议使用单标签 */}
  <Toggleable
    render={({ show, toggle }) => (
      <>
        <div onClick={toggle}>
          <h1>{title}</h1>
        </div>
        {show ? children : null}
      </>
    )}
  />
)
// 1. 基本使用3 
const ToggleableMenu: React.SFC<Propsxx> = ({ title, children }) => (
  <Toggleable>
  {
    ({ show, toggle }) => (
      <>
        <div onClick={toggle}>
          <h1>{title}aax</h1>
        </div>
        {show ? children : null}
      </>
    )
  }
  </Toggleable>
)

// 2. 组合组件的使用
const Menu = () =>(
    <>
      <ToggleableMenu title="First Menu">Some content</ToggleableMenu>
      <ToggleableMenu title="Second Menu">Some content</ToggleableMenu>
      <ToggleableMenu title="Third Menu">Some content</ToggleableMenu>
    </>
  )

```

### **组件注入**
跟上面的一样的效果（唯一的弊端在于defaultProps无法检测类型）
```tsx
import * as React  from 'react';

const Component = React.Component// 这是个类
type ReactNode = React.ReactNode
type ComponentType<T> = React.ComponentType<T>
type MouseEvent<T> = React.MouseEvent<T>


const initialState = { show: false };
const defaultProps = { props: {} as { [name: string]: any } };

export type ToggleableComponentProps<P extends object = object> = {
  show: State['show'];
  toggle: Toggleable['toggle'];
} & P;

type State = Readonly<typeof initialState>;
type DefaultProps = typeof defaultProps;
type RenderCallback = (args: ToggleableComponentProps) => JSX.Element;

type Props = Partial<
  {
    children: RenderCallback | ReactNode;
    render: RenderCallback;
    component: ComponentType<ToggleableComponentProps<any>>;// 定义组件中接受的参数
  } & DefaultProps
  >
  
const isFunction = <T extends Function>(fn: any): fn is T => Object.prototype.toString.call(fn) === '[object Function]'

const updateShowState = (prevState: State) => ({ show: !prevState.show });


export class Toggleable extends Component<Props, State> {
  static readonly defaultProps: Props = defaultProps;
  readonly state: State = initialState;

  render() {
    const {
      component: InjectedComponent,
      props,
      render,
      children,
    } = this.props;
    const renderProps = {
      show: this.state.show,
      toggle: this.toggle,
    };

    if (InjectedComponent) {
      return (
        <InjectedComponent {...props} {...renderProps}>
          {children}
        </InjectedComponent>
      );
    }
    if (render) {
      return render(renderProps);
    }
    return isFunction(children) ? children(renderProps) : null;
  }

  private toggle = (event: MouseEvent<HTMLElement>) =>
    this.setState(updateShowState);
}


type MenuItemProps = { title: string };
// 1
const MenuItem: React.SFC<MenuItemProps & ToggleableComponentProps> = ({
  title,
  toggle,
  show,
  children,
}) => (
  <>
    <div onClick={toggle}>
      <h1>{title}</h1>
    </div>
    {show ? children : null}
  </>
);


type ToggleableMenuProps = MenuItemProps;
const ToggleableMenuViaComponentInjection: React.SFC<ToggleableMenuProps> = ({
  title,
  children,
}) => (
  // 由于Toggleable中对props的校验是{[key: string ]: any},所以这里无法校验
  // <Toggleable component={MenuItem} props={{ title: 123 , a: 'x'}}>
  <Toggleable component={MenuItem} props={{ title }}>
    {children}
  </Toggleable>
)

export {ToggleableMenuViaComponentInjection}

export default class Menu extends React.Component {
  render() {
    return (
      <>
        <ToggleableMenuViaComponentInjection title="First Menu">
          Some content
        </ToggleableMenuViaComponentInjection>
        <ToggleableMenuViaComponentInjection title="Second Menu">
          Another content
        </ToggleableMenuViaComponentInjection>
        <ToggleableMenuViaComponentInjection title="Third Menu">
          More content
        </ToggleableMenuViaComponentInjection>
      </>
    );
  }
}
```










## React中定义的类型
1. React.ReactNode
> children: React.ReactNode
2. JSX.Element
```ts
type ReactNode = ReactChild | ReactFragment | ReactPortal | string | number | boolean | null | undefined;

type Key = string | number;
type ReactText = string | number;
type ReactChild = ReactElement<any> | ReactText;
type ReactFragment = {} | ReactNodeArray;
interface ReactPortal {
  key: Key | null;
  children: ReactNode;
}

interface ReactElement<P> {
  type: string | ComponentClass<P> | SFC<P>;
  props: P;
  key: Key | null;
}

type SFC<P = {}> = StatelessComponent<P>;
interface StatelessComponent<P = {}> {
  (props: P & { children?: ReactNode }, context?: any): ReactElement<any> | null;
  propTypes?: ValidationMap<P>;
  contextTypes?: ValidationMap<any>;
  defaultProps?: Partial<P>;
  displayName?: string;
}

interface JSX.Element extends React.ReactElement<any> { }
```


## <></>的使用
react顶层只能有一个标签，可以使用<></>




　　　








