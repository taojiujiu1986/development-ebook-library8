# 全栈老赵讲 React Hooks：从 useState 到自定义 Hook

<!-- PAGEBREAK -->

![作者介绍图](05_full_book_draft_assets/asset-8534aff2fa.png)

<!-- PAGEBREAK -->

## 目录

- 第1章 为什么 React 要有 Hooks：从类组件到函数组件的转变
- 第2章 useState：让函数组件拥有状态
- 第3章 useEffect：副作用、依赖数组与重新渲染
- 第4章 闭包、依赖数组与常见 useEffect 陷阱
- 第5章 useRef：保存可变值、操作 DOM 与跨渲染保留数据
- 第6章 useMemo：缓存计算结果，理解性能优化的第一步
- 第7章 useCallback：稳定函数引用，配合子组件与性能优化
- 第8章 useContext：跨层传递数据，减少层层 props 传递
- 第9章 自定义 Hook：把重复逻辑抽出来，形成可复用能力
- 第10章 Hooks 组合思维：在真实项目里把多个 Hooks 串起来
- 第11章 常见 Hooks 错误与修复：从能跑到写对
- 第12章 小项目实战：用 Hooks 完成一个完整任务管理应用

# 第1章 为什么 React 要有 Hooks：从类组件到函数组件的转变

## 开篇引入：我们为什么要聊 Hooks？

如果你写过一阵子 React，大概率会有这种感受：组件能跑，页面也能出，但代码一复杂，就开始变得碎、乱、难维护。

一会儿要在 `constructor` 里初始化状态，一会儿要在 `componentDidMount` 里发请求，一会儿又要在 `componentDidUpdate` 里同步逻辑，最后还得一直盯着 `this` 有没有绑定对。每一块单看都没问题，合在一起却很容易把一个组件写成“拼图现场”。

Hooks 的出现，正是为了解决这些问题。它不是给函数组件补几个 API 那么简单，而是 React 对组件组织方式的一次调整：**让状态和副作用能力回到函数组件中，让逻辑按功能组织，而不是按生命周期切块。**

老赵先给你一句话：**Hooks 不是让你少写代码，而是让你更容易写对、写清楚、写可复用。**

---

## 一、类组件时代到底卡在哪里？

### 1.1 逻辑复用难：同类能力分散在不同组件里

在类组件时代，想复用一段逻辑，常见做法是高阶组件、render props 或抽象基类。问题是这些方案要么嵌套深，要么让组件结构变绕。

比如你想给多个页面加“窗口宽度监听”能力，往往得再包一层组件。原本清爽的页面结构，会被一层层包起来，最后你想看页面本身做了什么，得先穿过好几层壳。逻辑是复用了，但阅读和维护成本也上去了。

### 1.2 生命周期分散：同一件事被拆到好几个地方

类组件最典型的问题之一，是一个功能往往要分散写在多个生命周期里。

例如“页面挂载时请求数据、参数变化时重新请求、卸载时清理订阅”，你可能要分别处理：

- `componentDidMount`
- `componentDidUpdate`
- `componentWillUnmount`

这样的问题很现实：**同一功能被打散了。**  
你想看“搜索列表这个功能是怎么工作的”，却要在好几个生命周期之间来回跳。功能越多，遗漏清理、重复请求、逻辑分叉这些问题就越容易出现。

### 1.3 `this` 绑定复杂：写着写着就容易出错

类组件里，方法经常要处理 `this` 绑定。你可能写过这些场景：

- 构造函数里手动绑定
- 箭头函数避免丢失上下文
- 回调传递时担心 `this` 变掉

这类问题不一定致命，但很烦。尤其当你本来是在想业务逻辑，却还要分神处理 `this`，体验并不好。很多人学类组件时，都会有一种“代码不复杂，但细节很多”的疲惫感。

### 小例子：一个类组件计数器

```jsx
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  add = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    return <button onClick={this.add}>count: {this.state.count}</button>;
  }
}
```

这个例子本身不难，但一旦逻辑增多，比如加定时器、请求、清理、副作用同步，类组件的组织成本会很快上升。

### 老赵提醒你别踩坑

不要把“类组件难写”简单理解成“类组件过时了”。更准确地说，是**类组件的组织方式不太适合现代前端对复用和组合的要求**。你先理解这一点，后面才会明白 Hooks 解决的是结构问题，不只是语法问题。

---

## 二、Hooks 的设计目标是什么？

### 2.1 把状态能力带回函数组件

Hooks 最核心的价值，是让函数组件不再只是“展示型组件”。它可以拥有状态、副作用、上下文读取等能力。

也就是说，函数组件不再只是“接收 props、返回 UI”的轻量壳子，而可以成为真正承载业务逻辑的单元。

### 2.2 让副作用和状态更自然地绑定

在 Hooks 之前，状态和副作用常常按生命周期拆开。Hooks 则倾向于把相关逻辑放在一起，让你一眼就知道“这个功能”涉及哪些状态、哪些副作用、哪些清理动作。

这是一种更接近“按功能分组”的写法。比如“用户搜索”功能，查询条件、请求、loading、错误处理可以围绕同一个业务块组织起来，而不是散落在几个生命周期里互相找补。

### 2.3 降低逻辑复用门槛

Hooks 让复用逻辑更像“函数复用”，而不是“组件包装”。

如果你有一段“监听鼠标位置”的逻辑，过去可能要写成 HOC；现在可以把它抽成一个自定义 Hook，多个组件直接调用即可。代码更轻，也更贴近日常开发习惯。

### 小例子：函数组件加上状态

```jsx
function Counter() {
  const [count, setCount] = React.useState(0);

  return <button onClick={() => setCount(count + 1)}>count: {count}</button>;
}
```

同样是计数器，函数组件 + Hook 的写法更直接：状态就在函数里，UI 也在函数里，理解成本低很多。

### 老赵提醒你别踩坑

Hooks 不是“把类组件翻译成函数组件”。如果你只是照着生命周期去套 Hooks，很容易把代码写得还是很碎。正确思路是：**先想功能，再想怎么用 Hook 组织它。**

---

## 三、Hooks 改变了什么：从“按生命周期写”到“按功能写”

### 3.1 以前：按生命周期切块

类组件时代，我们常常这样组织代码：

- 初始化写一块
- 数据请求写一块
- 监听变化写一块
- 清理逻辑写一块

这看起来符合框架要求，但不符合人的思考方式。因为我们平时想的是“搜索列表”“表单提交”“窗口监听”，而不是“挂载时做什么、更新时做什么”。

### 3.2 现在：按功能聚合

Hooks 更适合把同一功能相关的逻辑写在一起。

例如“搜索列表”这个功能，可以把：

- 查询条件状态
- 请求数据逻辑
- loading 状态
- 请求完成后的处理

尽量放在一处。这样别人打开代码时，能顺着业务场景理解，而不是顺着生命周期猜。

### 3.3 这种变化带来的好处

- 更容易阅读
- 更容易抽象复用
- 更容易拆分复杂功能
- 更容易把业务逻辑和 UI 组合起来

这也是很多团队在新项目里优先使用函数组件和 Hooks 的原因。不是因为它更“潮”，而是因为它更适合现代工程里的协作与维护。

### 小例子：按功能组织搜索逻辑

```jsx
function SearchBox() {
  const [keyword, setKeyword] = React.useState("");

  const onSearch = () => {
    console.log("搜索：", keyword);
  };

  return (
    <>
      <input value={keyword} onChange={(e) => setKeyword(e.target.value)} />
      <button onClick={onSearch}>搜索</button>
    </>
  );
}
```

你会发现，状态、事件、行为是围绕“搜索”这个功能自然展开的，而不是先想生命周期。

### 老赵提醒你别踩坑

按功能组织，不代表一个组件可以无限塞内容。**功能聚合是为了清晰，不是为了把所有东西都堆进一个文件里。** 该拆子组件时还是要拆，别把 Hooks 当成万能大锅。

---

## 四、为什么函数组件更适合配合 Hooks？

### 4.1 函数组件更接近“输入输出”模型

函数组件本质上更像“输入 props，输出 UI”。结构轻，组合自然。Hooks 的设计让这个函数不再只能展示，而是在保持简洁的前提下拥有状态能力。

### 4.2 函数组件更方便抽象复用逻辑

Hooks 本身就是函数式思维的一部分，所以它和函数组件天然搭配。你可以把一段逻辑抽成 Hook，然后在多个函数组件里复用，这比类继承或包装组件更直接。

### 4.3 更适合现代前端的组合方式

现代前端越来越强调组合，而不是继承。函数组件 + Hooks 正好顺应这个方向：UI 可以组合，逻辑也可以组合。

### 小例子：复用页面进入日志逻辑

假设两个页面都需要“进入页面时记录日志”。过去你可能写一层包装；现在可以把这部分逻辑抽成 Hook，页面直接调用即可。

### 老赵提醒你别踩坑

函数组件不是“更高级的类组件”，它们的心智模型不一样。**不要带着类组件思维硬套 Hook。** 否则你会在依赖数组、闭包、重复渲染这些地方频繁踩坑。

---

## 五、本书接下来怎么学，才能真正上手 Hooks？

### 5.1 学习顺序要跟着问题走

这本书不会把你丢进一堆 API 里死记硬背，而是按实际开发顺序往前推进：

1. 先理解 Hooks 为什么出现  
2. 再学 `useState` 如何管理状态  
3. 再学 `useEffect` 如何处理副作用  
4. 再学 `useRef`、`useMemo`、`useCallback`、`useContext`  
5. 再进入自定义 Hook  
6. 最后学习如何在真实项目里组合 Hooks 解决问题

### 5.2 后面几章你会重点碰到什么

后面你会反复见到几个关键概念：

- **依赖数组**：决定副作用或记忆值什么时候更新
- **闭包**：会影响你读到的状态是不是“最新的”
- **重新渲染**：决定组件什么时候重新执行函数体
- **性能优化**：不是一上来就优化，而是先判断是否真的有问题

这些内容，才是 Hooks 学习的真正分水岭。

### 5.3 你应该带着什么问题继续学

建议你后面每学一个 Hook，都带着三个问题：

- 它解决什么场景？
- 它什么时候会出问题？
- 它和重新渲染、依赖、闭包之间是什么关系？

这样学下来，你不会只会写 API，而是会真正用 Hooks 做事。

### 小例子：观察函数组件的重新执行

把一个函数组件里的 `console.log` 打开，多点几次按钮，你会看到函数体会反复执行。这说明函数组件每次渲染都会重新运行。后面讲依赖数组和闭包时，这个现象非常关键。

### 老赵提醒你别踩坑

别把“学会调用 Hook”当成“掌握 Hook”。真正重要的是理解它在**重新渲染机制**里的位置。只会抄代码，遇到异步、缓存、联动状态时还是会乱。

---

## 结尾小结：先建立正确认知，再谈熟练使用

这一章先解决最重要的问题：**React 为什么要有 Hooks。**

你现在应该已经明白：

- 类组件时代有逻辑复用难、生命周期分散、`this` 绑定复杂等痛点
- Hooks 的设计目标，是把状态和副作用能力带回函数组件
- Hooks 让代码更适合按功能组织，而不是按生命周期切块
- 函数组件和 Hooks 天然适合组合与复用
- 后续学习要重点关注依赖数组、闭包、重新渲染和性能优化

如果你是第一次系统学 Hooks，老赵建议你先别急着追求“写得多”，先追求“想得对”。  
因为后面的每一个 Hook，几乎都建立在这一章的认知基础上。

下一章，我们就从最常用、也最容易上手的 `useState` 开始。

# 第2章 useState：让函数组件拥有状态

如果你之前写 React 组件时，心里一直有个感觉：**函数组件很轻巧，但好像“缺点什么”**。  
没错，缺的就是“状态”。没有状态，组件只能靠父组件传参和静态渲染；有了状态，组件才真正能记住用户刚刚做了什么、输入了什么、点了几次按钮、弹窗是开还是关。

老赵先把话说透：**useState 的价值，不只是让函数组件能存数据，而是让函数组件具备响应交互的能力。** 这也是 Hooks 学习的起点。你后面要学的 useEffect、useMemo、useCallback、自定义 Hook，本质上都建立在你对“状态”和“重新渲染”的理解之上。

---

## 一、useState 是什么？为什么函数组件需要它？

### 1.1 useState 的基本语法

useState 是 React 提供的一个 Hook，用来在函数组件里声明状态。最常见的写法是：

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      点击了 {count} 次
    </button>
  );
}
```

这里有三个关键点：

- `count`：当前状态值
- `setCount`：更新状态的方法
- `0`：初始值

你可以把它理解成：**组件第一次渲染时，React 帮你创建了一个“记忆格子”**，以后每次重新渲染，都能从这个格子里取到最新值。

和普通变量不一样，`useState` 管理的是会参与页面展示、并且需要在用户操作后持续保留的数据。比如按钮点击次数、输入框内容、弹窗开关，这些都需要被 React 记住。

### 1.2 初始值怎么设

useState 的初始值可以是普通值，也可以是函数：

```jsx
const [value, setValue] = useState(10);
const [data, setData] = useState(() => getDefaultData());
```

适合用函数初始化的场景：

- 初始化逻辑比较重
- 初始值需要计算
- 只想让它在首次渲染时执行一次

如果你直接写 `useState(getDefaultData())`，每次渲染都会执行函数，虽然只有第一次结果会作为初始值，但计算过程仍会发生。  
所以老赵建议你记住：**初始化函数只在第一次用，别把它当普通表达式。**

### 小例子

```jsx
function Welcome() {
  const [name, setName] = useState("老赵");

  return (
    <div>
      <p>欢迎你，{name}</p>
      <button onClick={() => setName("赵老师")}>切换称呼</button>
    </div>
  );
}
```

这个例子很简单，但它体现了 useState 的本质：点击按钮后，组件不是“修改了一个变量”，而是**通过状态更新触发界面重新渲染**。

### 老赵提醒你别踩坑

**useState 不是变量赋值，而是触发 React 重新渲染的声明式更新。**  
别写成 `count = count + 1`，那只是改变量，不会让页面更新。真正需要的是 `setCount(...)`。

---

## 二、哪些状态适合用 useState 管理？

### 2.1 适合用 useState 的典型场景

useState 最适合管理这些局部交互状态：

1. **表单输入**
   - 输入框内容
   - 下拉选择
   - 单选/多选

2. **开关类状态**
   - 弹窗显示/隐藏
   - 折叠面板展开/收起
   - loading 状态

3. **计数器类状态**
   - 点赞数
   - 数量加减
   - 页码

4. **局部 UI 状态**
   - 当前选中项
   - Tab 切换
   - 临时筛选条件

它们的共同点是：**属于当前组件内部，变化频繁，而且会直接影响页面展示。**

你写页面时可以先问一句：**这个值变了，页面要不要跟着变？** 如果要，那它大概率就适合进 state。

### 2.2 什么情况不该优先用 useState

不是所有数据都适合放进 useState。下面这些就要多想一步：

- 来自父组件、且不需要本地修改的值
- 多个组件都要共享的值
- 不需要触发 UI 变化的临时值
- 非渲染相关的中间变量

比如定时器 ID、上一次点击时间，这类数据常常更适合用 `useRef`，而不是 `useState`。  
原因很简单：它们需要“记住”，但不一定要让界面跟着刷新。状态更新会引发重新渲染，如果没有必要，那就是多余成本。

### 小例子

```jsx
function LoginForm() {
  const [username, setUsername] = useState("");
  const [submitting, setSubmitting] = useState(false);

  return (
    <form>
      <input
        value={username}
        onChange={(e) => setUsername(e.target.value)}
      />
      <button disabled={submitting}>
        {submitting ? "提交中..." : "登录"}
      </button>
    </form>
  );
}
```

这里 `username` 和 `submitting` 都会直接影响 UI，所以放在 state 里很自然。

### 老赵提醒你别踩坑

**别把所有东西都塞进 useState。**  
如果一个值变化了也不影响页面渲染，放进 state 反而会增加不必要的重新渲染。关键不是“有没有数据”，而是“是否真的需要参与 UI 更新”。

---

## 三、状态更新到底怎么写？直接赋值和函数式更新有啥区别？

### 3.1 直接赋值写法

最常见的更新方式是直接传入新值：

```jsx
setCount(count + 1);
```

它适合的场景很简单：当前拿到的 `count` 就是最新值，而且这次更新不依赖前一次更新结果。

### 3.2 函数式更新写法

另一种写法是传入函数：

```jsx
setCount(prevCount => prevCount + 1);
```

它的核心是：**让 React 把上一次的状态值传给你。**  
所以只要新值依赖旧值，就更推荐函数式更新。

比如连续加 3：

```jsx
setCount(prev => prev + 1);
setCount(prev => prev + 1);
setCount(prev => prev + 1);
```

这样每次都基于最新的前一个值计算，不容易出错。

### 3.3 什么时候必须用函数式更新

以下场景尤其要优先考虑函数式更新：

- 新值依赖旧值
- 同一个事件里连续更新多次
- 可能存在异步、闭包捕获旧值的问题

这已经碰到后面会反复讲的关键概念：**闭包与重新渲染。**  
先记住一句话：**函数组件里的变量，会随着每次渲染重新生成。**  
如果你在某个回调里拿的是旧渲染时的 `count`，直接 `setCount(count + 1)` 就可能不如函数式更新稳妥。

### 小例子

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  const addThree = () => {
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
  };

  return <button onClick={addThree}>{count}</button>;
}
```

函数式更新的价值就在这里：不用纠结当前读到的是不是旧值，React 会按顺序帮你把状态算出来。

### 老赵提醒你别踩坑

**别把当前渲染里的 count 当成永远最新。**  
在异步回调、定时器、连续触发更新里，闭包很容易让你拿到旧值。只要新状态依赖旧状态，优先用函数式更新，省心很多。

---

## 四、多次 setState 会发生什么？批量更新怎么理解？

### 4.1 多次 setState 不等于多次立即刷新

很多初学者会以为：

```jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

页面会立刻加 3。  
但实际并不是这样。

原因是：**React 不一定会在每次 setState 后立即刷新 DOM。**  
它通常会把同一轮里的更新做一定程度的合并，再统一重新渲染。你可以把它理解成：React 在帮你“攒一波操作”，减少无意义的重复渲染。

### 4.2 为什么会出现“看起来只加了一次”

如果你写的是：

```jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

这里的 `count` 都来自同一次渲染，值其实一样。  
所以三次调用很可能提交的是同一个结果，最终看起来只加了 1。

而函数式更新则不同：

```jsx
setCount(prev => prev + 1);
setCount(prev => prev + 1);
setCount(prev => prev + 1);
```

React 会按顺序拿前一个结果继续计算，所以更符合预期。

### 4.3 批量更新的直观理解

你可以这样理解：

- `setState` 不是“立刻改值”
- 它更像“提交一个更新请求”
- React 会在合适时机统一处理
- 这样可以提升性能，减少重复渲染

这也是为什么我们一直强调：**别把 state 当成同步变量。**

### 小例子

```jsx
function Demo() {
  const [num, setNum] = useState(0);

  const handleClick = () => {
    setNum(num + 1);
    setNum(num + 1);
    setNum(num + 1);
  };

  return <button onClick={handleClick}>{num}</button>;
}
```

如果你真想稳定加 3，改成函数式更新会更靠谱。

### 老赵提醒你别踩坑

**别在 setState 后立刻假设 state 已经更新。**  
你紧接着 `console.log(num)`，大概率看到的还是旧值。React 的更新是“提交请求 + 统一渲染”，不是同步改内存。

---

## 五、状态要拆开还是合并？怎么做取舍？

### 5.1 拆分状态：更清晰，也更容易维护

很多时候，状态可以拆成多个 useState：

```jsx
const [name, setName] = useState("");
const [age, setAge] = useState("");
const [loading, setLoading] = useState(false);
```

拆分的好处是：

- 每个状态职责单一
- 更新互不干扰
- 可读性更好
- 代码更容易维护

对于表单、开关、局部交互状态，这种方式通常更自然。

### 5.2 合并状态：适合强相关数据

有些状态彼此关系紧密，可以放一起：

```jsx
const [form, setForm] = useState({
  name: "",
  age: "",
});
```

它的好处是：

- 方便整体管理
- 便于一次性重置
- 适合结构化数据

但更新时要手动保留其他字段：

```jsx
setForm(prev => ({
  ...prev,
  name: "老赵",
}));
```

如果你直接写成：

```jsx
setForm({ name: "老赵" });
```

那 `age` 就没了。

### 5.3 什么时候拆，什么时候合？

老赵给你一个简单判断标准：

- **彼此独立、变化频率不同**：优先拆开
- **必须一起重置、一起提交、一起表达业务含义**：可以合并
- **对象层级复杂、更新频繁**：谨慎合并，避免每次都手动展开拷贝

实践里通常是：**先拆，再在需要时合并。**  
先把逻辑理顺，等业务真的需要整体处理时，再考虑收拢。

### 小例子

```jsx
function ProfileEditor() {
  const [profile, setProfile] = useState({ name: "", city: "" });

  return (
    <input
      value={profile.name}
      onChange={(e) =>
        setProfile(prev => ({ ...prev, name: e.target.value }))
      }
    />
  );
}
```

这个例子里，`profile` 是一个整体对象，但更新时只改 `name`，其余字段通过展开运算符保留下来。

### 老赵提醒你别踩坑

**对象状态更新一定要注意保留旧字段。**  
useState 不会帮你自动合并对象，和 class 组件里的 `setState` 不是一回事。你一旦忘了展开旧值，原来的字段就可能被覆盖掉。

---

## 六、用一个判断清单，快速决定怎么用 useState

你可以在项目里快速问自己四个问题：

1. 这个值会不会影响界面显示？
2. 这个值是不是只属于当前组件？
3. 这个值更新时，需不需要依赖上一次结果？
4. 这个值应该拆开管理，还是作为一个整体管理？

如果答案是：

- 会影响界面
- 属于当前组件
- 需要重新渲染

那它大概率就适合 useState。  
如果还依赖上一次结果，就优先考虑函数式更新。  
如果是多个相关字段，就再判断拆分还是合并。

这个判断清单看起来简单，但在真实项目里特别好用。你以后写表单、弹窗、分页、筛选器，都可以先过一遍这四个问题，很多状态设计上的犹豫会一下子变少。

### 小例子

```jsx
function TogglePanel() {
  const [open, setOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setOpen(prev => !prev)}>
        {open ? "收起" : "展开"}
      </button>
      {open && <p>这里是面板内容</p>}
    </div>
  );
}
```

这是一个典型的状态驱动 UI 的例子：点击按钮改变 `open`，页面根据状态决定显示什么。

### 老赵提醒你别踩坑

**不要为了“看起来规范”就强行把状态拆得过碎，或者强行合成一个大对象。**  
状态设计没有固定死板答案，关键看业务关系、更新方式和维护成本。

---

## 七、本章小结：先把 useState 用稳

这一章你最该带走的，不是某个语法细节，而是这几个核心认知：

- **useState 让函数组件拥有状态能力**
- **状态适合管理会影响渲染的局部交互数据**
- **更新状态时，优先理解重新渲染，而不是变量赋值**
- **新值依赖旧值时，优先用函数式更新**
- **对象状态要注意保留旧字段，拆分和合并要有判断标准**

后面我们讲 `useEffect`、闭包、依赖数组的时候，你会发现今天这些内容是地基。  
地基打得稳，后面学 Hooks 才不容易乱。

### 课后行动建议

你现在就可以做三件事：

1. 找一个自己写过的组件，把其中一个普通变量改成 `useState`
2. 分别用“直接赋值”和“函数式更新”写一个计数器
3. 找一个表单组件，思考哪些字段适合拆，哪些字段适合合并

老赵建议你别急着背 API，先把“状态如何驱动 UI”这件事想明白。  
这一步想透了，后面的 Hooks 学起来会顺很多。

# 第3章 useEffect：副作用、依赖数组与重新渲染

## 开篇引入：为什么我们需要 useEffect？

老赵先问你一个问题：**React 组件除了根据状态渲染界面，还要不要做别的事？**

当然要。真实项目里，组件不只负责“画 UI”，还得和外部世界打交道，比如：

- 页面加载后请求接口
- 监听窗口变化
- 订阅 WebSocket
- 操作 DOM 做聚焦、滚动、埋点
- 开启定时器、轮询数据

这些都不是纯渲染逻辑，而是**副作用**。

React 函数组件理想状态是“输入什么状态，就返回什么界面”。但组件挂到页面后，往往还要做一些渲染之外的事。于是 React 提供了 `useEffect`，专门处理这类副作用。

你可以把它理解成：

**渲染负责“画出来”，useEffect 负责“画完后去做事”。**

这也是 `useEffect` 的意义：把展示逻辑和外部交互逻辑分开，让组件更清晰。

### 小例子

公告栏组件，页面一打开就请求公告：

```jsx
import { useEffect, useState } from 'react';

function NoticeBar() {
  const [notice, setNotice] = useState('');

  useEffect(() => {
    fetch('/api/notice')
      .then(res => res.json())
      .then(data => setNotice(data.message));
  }, []);

  return <div className="notice">{notice || '加载中...'}</div>;
}
```

渲染只负责显示，真正的请求动作放在 `useEffect` 里。

### 老赵提醒你别踩坑

**别把所有逻辑都塞进 useEffect。**  
像格式化字符串、计算总价、拼接文案这类“根据当前状态直接推导”的内容，应该放在渲染阶段，不算副作用。`useEffect` 是处理外部交互的，不是万能胶。

---

## 一、useEffect 适合处理哪些副作用？

### 1. 请求数据：组件渲染后拉接口

```jsx
import { useEffect, useState } from 'react';

function UserList() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetch('/api/users')
      .then(res => res.json())
      .then(data => setUsers(data));
  }, []);

  return (
    <ul>
      {users.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

请求依赖网络、后端和时间，属于典型副作用，应该放进 `useEffect`。

### 2. 订阅事件：监听外部变化

```jsx
useEffect(() => {
  const handleResize = () => {
    console.log('当前窗口宽度：', window.innerWidth);
  };

  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

组件负责订阅，卸载时负责退订。

### 3. 手动操作 DOM：聚焦、滚动、测量尺寸

```jsx
import { useEffect, useRef } from 'react';

function SearchBox() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} />;
}
```

这类操作通常发生在组件挂载后，适合放进 `useEffect`。

### 4. 定时器：轮询、倒计时、延迟执行

```jsx
useEffect(() => {
  const timer = setInterval(() => {
    console.log('每秒执行一次');
  }, 1000);

  return () => clearInterval(timer);
}, []);
```

定时器也是副作用，启动后要负责清理。

### 小例子

页面加载后自动拉取公告并显示：

- `useEffect` 负责请求
- `useState` 负责保存结果
- 渲染只负责展示

### 老赵提醒你别踩坑

**不是所有“会执行代码”的地方都该用 useEffect。**  
点击按钮提交表单，更像事件处理，不是挂载后同步副作用。先判断它是不是依赖生命周期或外部变化，再决定放不放进 `useEffect`。

---

## 二、依赖数组到底是什么？三种常见写法怎么选？

`useEffect` 最容易让人头疼的，不是“写不写”，而是**依赖数组怎么写**。

依赖数组告诉 React：**这个副作用依赖哪些值**。React 会根据这些值的变化，决定要不要重新执行 effect。

### 1. 不传依赖数组：每次渲染后都执行

```jsx
useEffect(() => {
  console.log('每次渲染后都执行');
});
```

只要组件重新渲染，effect 就会再跑一遍。适合少数需要每次渲染后同步的场景，但一般会太频繁。

### 2. 空数组 `[]`：只在首次挂载后执行一次

```jsx
useEffect(() => {
  console.log('只执行一次');
}, []);
```

表示这个副作用只跟组件生命周期有关，常见于：

- 初始化请求
- 注册一次监听
- 启动一次定时器

### 3. 依赖具体变量：变量变化时才重新执行

```jsx
useEffect(() => {
  console.log('searchText 变了，重新请求');
}, [searchText]);
```

这是最常见的写法。依赖变了，effect 才重新执行。

### 怎么选？

老赵给你一个简单判断：

- **只做一次初始化** → `[]`
- **跟某个状态/属性强相关** → `[xx]`
- **每次渲染后都要同步** → 不传数组

### 小例子

搜索框输入关键词后请求结果：

```jsx
useEffect(() => {
  if (!keyword) return;
  fetch(`/api/search?q=${keyword}`);
}, [keyword]);
```

关键词一变，就应该重新请求。

### 老赵提醒你别踩坑

**不要“想当然”乱写依赖数组。**  
依赖少了，effect 可能拿到旧数据；依赖多了，可能频繁触发，导致重复请求。依赖数组不是控制开关，而是对副作用输入条件的声明。

---

## 三、为什么依赖数组会影响效果执行时机？

这背后其实是 React 的**重新渲染机制**。

### 1. 状态变化会触发重新渲染

调用 `setState` 后，组件函数会重新执行，生成新的 UI。  
而 `useEffect` 的执行时机，是在这次渲染提交到页面之后。

所以可以理解为：

- **渲染阶段**：计算 UI
- **提交后**：执行 effect

也就是说，`useEffect` 不是“状态一改马上执行”，而是**渲染完成后再执行**。

### 2. React 会比较依赖项是否变化

React 会对比前后两次渲染中的依赖项。如果没变，effect 不执行；如果变了，就执行新的副作用。

```jsx
useEffect(() => {
  document.title = `当前计数：${count}`;
}, [count]);
```

`count` 变化时，浏览器标题也要同步更新，所以它必须出现在依赖里。

### 3. 为什么这很重要？

副作用常常需要和“最新状态”保持一致。  
比如切换筛选条件后重新请求数据：

```jsx
useEffect(() => {
  loadTabData(activeTab);
}, [activeTab]);
```

这里 `activeTab` 决定请求哪份数据，所以它必须被声明为依赖。

### 小例子

用户切换 tab 时重新加载当前数据：

```jsx
useEffect(() => {
  loadTabData(activeTab);
}, [activeTab]);
```

### 老赵提醒你别踩坑

**依赖数组不是随便写的。**  
它应该写“这段副作用真正依赖谁”。很多死循环问题，不是 `useEffect` 不能用，而是 effect 里又改了状态，状态变化再次触发 effect。写之前先想清楚：这段逻辑到底依赖什么？变化后是否真的要重跑？

---

## 四、清理函数有什么用？为什么它很关键？

`useEffect` 不只会执行，还能返回一个函数，这就是**清理函数**。

它的作用是：**在 effect 失效前，把留下的痕迹清理掉**。

### 1. 取消订阅

```jsx
useEffect(() => {
  const handler = () => console.log('resize');
  window.addEventListener('resize', handler);

  return () => {
    window.removeEventListener('resize', handler);
  };
}, []);
```

组件卸载时，或者依赖变化导致旧 effect 被替换时，清理函数会执行。

### 2. 清除定时器

```jsx
useEffect(() => {
  const timer = setInterval(() => {
    console.log('tick');
  }, 1000);

  return () => clearInterval(timer);
}, []);
```

不清理，组件卸载后定时器还在跑，容易引发内存泄漏和异常更新。

### 3. 避免过期副作用继续工作

请求数据时，如果用户很快切换页面，旧请求返回后可能还想更新已卸载的组件。  
这时就需要在清理阶段做取消、忽略或防抖处理，至少不要把结果错误地落到失效页面上。

### 小例子

监听键盘快捷键关闭弹窗：

```jsx
useEffect(() => {
  const onKeyDown = e => {
    if (e.key === 'Escape') console.log('关闭弹窗');
  };

  window.addEventListener('keydown', onKeyDown);
  return () => window.removeEventListener('keydown', onKeyDown);
}, []);
```

弹窗关闭后，监听也应该移除。

### 老赵提醒你别踩坑

**凡是注册了外部资源的 effect，基本都要考虑清理。**  
记住这个口诀：

> 有订阅就要退订，有定时器就要清除，有监听就要解绑。

---

## 五、重新渲染、状态变化与 effect 重新执行是什么关系？

这一节是 `useEffect` 的核心理解点，老赵建议你认真捋顺。

### 1. 状态变化会导致重新渲染

当你调用 `setCount(1)` 后：

1. 组件函数重新执行
2. React 计算新的 UI
3. DOM 更新
4. 根据依赖数组判断是否执行 effect

所以 effect 不是“状态一改马上执行”，而是**在新一轮渲染提交后执行**。

### 2. effect 中更新状态，要小心连锁反应

```jsx
useEffect(() => {
  setCount(count + 1);
}, [count]);
```

这会形成循环：

- `count` 变化
- effect 执行
- 又 `setCount`
- 再渲染
- 再执行……

所以在 effect 里更新状态时，一定要确认：这个更新会不会再次触发自己。

### 3. 闭包也会影响你看到的值

effect 中的函数会捕获当次渲染时的变量值。  
如果依赖没写对，就可能拿到旧值。

```jsx
useEffect(() => {
  const timer = setInterval(() => {
    console.log(count);
  }, 1000);
  return () => clearInterval(timer);
}, []);
```

这里如果 `count` 变化了，打印的可能还是初始值，因为闭包锁住了当次渲染时的 `count`。这就是很多人觉得“明明 state 变了，为什么 effect 里还是旧的”的原因。

### 4. 正确思路：让 effect 跟真实依赖同步

如果你要用最新的 `count`，通常应该把它放进依赖数组，或者通过其他方式保持最新引用。关键是：不要指望一个空依赖的 effect 自动“看见未来”。

### 小例子

用户修改筛选条件时重新请求列表：

```jsx
useEffect(() => {
  fetchList(filter);
}, [filter]);
```

这比“只执行一次再想办法读旧值”更符合 React 的数据流。

### 老赵提醒你别踩坑

**不要把“重新渲染”和“effect 重新执行”混为一谈。**  
真正决定 effect 是否重跑的是依赖数组。还有，effect 里读到的值，是那次渲染对应的值，不是永远最新的值。理解这一点，你会少掉很多坑。

---

## 六、把 useEffect 用好的实用检查清单

写 effect 前，老赵建议你先问自己：

1. 这是不是副作用？
2. 它依赖哪些外部变量？
3. 依赖变化时要不要重新执行？
4. 需不需要清理函数？
5. 会不会因为更新状态造成循环？
6. 有没有闭包导致的旧值问题？

### 小例子

做“搜索建议”功能时：

- 输入变化后发请求
- 请求前判断是否为空
- 请求完成后更新列表
- 组件卸载时避免旧请求继续影响页面

先把这些问题写清楚，再决定 `useEffect` 怎么写，比一上来就敲代码稳得多。

### 老赵提醒你别踩坑

**别为了“看起来像 React”而滥用 useEffect。**  
很多逻辑本来可以写成普通函数或事件处理函数，结果被塞进 effect，最后反而更难维护。老赵的建议是：先分清“渲染逻辑”“事件逻辑”“副作用逻辑”，再决定放哪儿。

---

## 结尾小结：把副作用放对位置，代码才稳

`useEffect` 的核心，不是语法，而是**什么时候做什么事**。

这一章你要记住三点：

- **副作用**：请求、订阅、DOM 操作、定时器，都适合放进 `useEffect`
- **依赖数组**：决定 effect 何时执行，是理解 `useEffect` 的钥匙
- **重新渲染与闭包**：状态变化会触发渲染，但 effect 是否执行、拿到什么值，要结合依赖和闭包一起看

如果你已经能分清“何时执行、为何执行、执行后是否清理”，那 `useEffect` 基本就入门了。

下一步，咱们继续看 `useRef`、`useMemo` 这些常用 Hook。你会发现，Hooks 真正厉害的地方，不是单个 API，而是它们之间的组合能力。

# 第4章 闭包、依赖数组与常见 useEffect 陷阱

## 开篇引入：为什么你总觉得 useEffect “不听话”？

老赵第一次带学员写 `useEffect`，最常见的抱怨就是：  
“我明明改了状态，为什么 effect 里拿到的还是旧值？”  
或者：  
“为什么依赖数组一改，页面就开始疯狂重渲染？”

这章我们就把这件事讲透。你要先记住：`useEffect` 不是自动执行的魔法，它只是**在组件渲染之后，根据依赖变化执行副作用**。而闭包、依赖数组、重新渲染这三件事一混在一起，就很容易让人误判。

本章目标很明确：  
1. 搞懂闭包在 Hooks 里的表现；  
2. 明白为什么 effect 和事件处理函数会拿到旧状态；  
3. 学会判断依赖数组怎么写，避免数据不同步、重复执行、循环触发这些坑。

很多 `useEffect` 的问题，不是 API 难，而是你对“函数什么时候创建、什么时候执行、什么时候拿到的值”还不够敏感。老赵今天就带你把这层窗户纸捅破。

---

## 一、闭包是什么，以及它在 Hooks 里的典型表现？

### 1.1 闭包不是玄学，它就是“记住当时的环境”

按本书语境，闭包可以理解为：**函数在创建时，会捕获它所处作用域里的变量，并在之后继续使用这些变量。**

要注意，React 里还有一个更关键的事实：**每次组件重新渲染，函数都会重新创建一次。**  
所以你写的函数，并不一定一直在接收“最新状态”，它可能只是在执行“某次渲染时创建的那一版逻辑”。

### 1.2 闭包在 useEffect 里的样子

```jsx
function Demo() {
  const [count, setCount] = React.useState(0);

  React.useEffect(() => {
    const timer = setInterval(() => {
      console.log(count);
    }, 1000);

    return () => clearInterval(timer);
  }, []);

  return <button onClick={() => setCount(count + 1)}>+1</button>;
}
```

按钮点了很多次，但定时器里打印的可能永远是 `0`。  
原因很简单：`useEffect` 只在首次渲染后执行一次，`setInterval` 里的回调闭包，捕获的是**第一次渲染时的 `count`**。状态在变，回调却还活在旧渲染里。

### 1.3 闭包在真实开发中的常见表现

闭包问题不只会出现在定时器里，还常见于：

- `setTimeout` 延迟执行；
- `setInterval` 周期执行；
- 事件订阅回调；
- Promise 异步链；
- 第三方库长期持有的回调。

只要函数被延后执行，或者被外部系统长期保存，就要小心它拿到的是不是旧值。

### 1.4 小例子：异步回调里拿到旧名字

```jsx
function Demo() {
  const [name, setName] = React.useState("老赵");

  const greet = () => {
    setTimeout(() => {
      alert("你好，" + name);
    }, 1000);
  };

  return <button onClick={greet}>打招呼</button>;
}
```

如果点击按钮后立刻修改 `name`，一秒后弹出的可能还是旧名字。  
因为 `setTimeout` 里执行的是**当次点击时创建的闭包**。

### 1.5 老赵提醒你别踩坑

**闭包不是 bug，错的是你把“旧渲染里的值”当成了“最新值”。**  
如果某个逻辑需要持续读取最新状态，不要默认靠闭包拿值，要么补全依赖让 effect 重新创建，要么用 `useRef` 保存最新值。  
老赵给你一句口诀：**凡是“晚点再执行”的函数，都要先怀疑闭包。**

---

## 二、为什么 effect、事件处理函数会拿到旧状态或旧 props？

### 2.1 因为每次渲染都会生成新的函数

React 组件本质上就是函数。函数一执行，就会创建一套新的局部变量、新的函数引用、新的闭包环境。  
所以：

```jsx
function handleClick() {
  console.log(count);
}
```

并不是“同一个 handleClick 一直在用最新 count”，而是**每次渲染都会生成一个新的 handleClick**。

如果这个函数直接绑定在按钮上，通常没问题，因为点击时拿到的是当前渲染对应的函数。  
但如果它被异步逻辑、定时器、外部订阅持有，就很可能保留旧的那一版数据。

### 2.2 effect 里的旧值，从哪儿来的？

`useEffect` 里的代码不是实时运行的，它是在渲染提交后执行。  
如果依赖数组没写对，React 就不会重新执行这个 effect，里面的逻辑就一直停留在旧闭包里。

于是会出现两类问题：

- **数据不同步**：状态变了，effect 仍然用旧值请求、计算、订阅；
- **逻辑失效**：比如监听某个 `roomId`，切换房间后 effect 没重新执行。

### 2.3 小例子：看似简单的按钮，实际上也可能读旧值

```jsx
function Demo() {
  const [name, setName] = React.useState("老赵");

  const greet = () => {
    setTimeout(() => {
      alert("你好，" + name);
    }, 1000);
  };

  return (
    <>
      <button onClick={greet}>打招呼</button>
      <input value={name} onChange={e => setName(e.target.value)} />
    </>
  );
}
```

点击“打招呼”后的一秒内，如果你改了输入框，弹窗里显示的仍可能是点击那一刻的名字。  
这不是 React 没跟上，而是 `setTimeout` 里的回调，早就把当时的 `name` 记住了。

### 2.4 老赵提醒你别踩坑

**“函数里打印的是旧值”不代表 React 有问题，通常是你拿闭包当实时通道用了。**  
判断一个函数是否会“过期”，看它是不是会被延迟执行、被缓存、被外部系统持有。只要有这种情况，就要警惕旧闭包。

---

## 三、依赖数组遗漏会导致什么问题？

### 3.1 本质问题：React 不知道你还依赖它

`useEffect(fn, deps)` 的意思是：  
“只要 deps 没变，就复用上一次的结果，不重新执行 `fn`。”

所以如果你在 `fn` 里用了某个值，却没把它放进依赖数组，React 就不会在它变化时重跑 effect。结果就是：

- **数据不同步**；
- **逻辑失效**；
- **bug 难排查**。

### 3.2 小例子：漏依赖导致请求参数不更新

```jsx
function UserPanel({ userId }) {
  const [user, setUser] = React.useState(null);

  React.useEffect(() => {
    fetch(`/api/user/${userId}`)
      .then(res => res.json())
      .then(setUser);
  }, []); // 漏了 userId

  return <div>{user?.name}</div>;
}
```

当 `userId` 切换时，页面不会重新请求，仍然显示旧用户信息。  
这个问题最坑人，因为接口、状态、UI 看起来都“差不多”，但实际上已经错位了。

### 3.3 依赖遗漏的判断标准

老赵给你一个实用判断：  
**如果某个值参与了 effect 里的计算、判断、请求、订阅、清理，就大概率应该进入依赖。**

你可以问自己三个问题：

1. 这个值变了，effect 的逻辑是否应该变？
2. 这个值是否直接参与当前副作用的结果？
3. 如果不加它，会不会用到旧值？

只要答案偏向“会”，就应该加。

### 3.4 老赵提醒你别踩坑

**不要为了“只执行一次”就硬删依赖。**  
很多人把依赖数组写空，是为了逃避重复执行，但最后换来的是静默 bug。真正只执行一次的逻辑，要么本来就不依赖外部变化，要么就要想办法把依赖问题处理干净，而不是直接藏起来。

---

## 四、依赖数组写多了会带来什么问题？

### 4.1 依赖不是越多越好

另一种常见误区是：  
“既然漏依赖不行，那我就把所有用到的都放进去。”

听起来稳，实际上也可能出问题。因为有些依赖每次渲染都会变，比如对象、数组、函数引用。  
一旦它们变了，effect 就会重复执行。

这里就牵扯到一个关键点：**引用是否稳定。**  
值看起来一样，不代表引用没变；引用一变，依赖判断就会认为“变了”。

### 4.2 典型后果：重复执行、性能浪费、无限循环

#### 重复执行

effect 里如果做了请求、订阅、重置状态，只要依赖变化频繁，就会不断重跑，造成性能浪费。

#### 无限循环

最典型的是：effect 里更新了某个状态，而这个状态又在依赖数组里。

```jsx
function Demo() {
  const [count, setCount] = React.useState(0);

  React.useEffect(() => {
    setCount(count + 1);
  }, [count]);

  return <div>{count}</div>;
}
```

这会形成循环：`count` 变 -> effect 跑 -> `setCount` -> `count` 再变 -> effect 再跑。

### 4.3 小例子：对象依赖导致反复执行

```jsx
function Demo({ id }) {
  const query = { id };

  React.useEffect(() => {
    console.log("重新查询");
  }, [query]);

  return null;
}
```

因为 `query` 每次渲染都是新对象，所以即使 `id` 没变，`query` 也变了，effect 还是会重新执行。  
这也是搜索、筛选、表单联动场景里最容易踩的坑：明明只是点了别的按钮，查询却一直重新发。

### 4.4 老赵提醒你别踩坑

**依赖数组里放“每次都会新建”的东西，要特别小心。**  
对象、数组、函数引用都可能让你误以为“值没变”，实际上引用已经变了。遇到这种情况，先想办法：

- 拆分逻辑；
- 用 `useMemo` / `useCallback` 稳定引用；
- 或者不要把整个对象直接塞进依赖，改成依赖更稳定的原子值。

---

## 五、如何判断依赖项是否应该加入？

### 5.1 一个实用原则：凡是“用到了，就要负责到底”

判断依赖项，不要只看代码里有没有出现名字，而要看它在副作用里扮演什么角色。

你可以按这个思路判断：

#### 第一步：找出 effect 里真正参与逻辑的变量
包括：

- 请求参数；
- 条件判断的值；
- 订阅名称、房间号、token；
- 需要同步到外部系统的状态；
- cleanup 里要用到的值。

#### 第二步：判断它变化后，副作用是否应该重新执行
如果应该，就进依赖。

#### 第三步：判断这个依赖是否稳定
如果不稳定，就考虑：

- 能不能拆分 effect；
- 能不能把对象拆成原子依赖；
- 能不能用 ref 保存“最新但不触发重跑”的值。

### 5.2 一个小示例：怎么做更合理

```jsx
function SearchBox({ keyword }) {
  React.useEffect(() => {
    const timer = setTimeout(() => {
      console.log("搜索：", keyword);
    }, 300);

    return () => clearTimeout(timer);
  }, [keyword]);

  return null;
}
```

这里 `keyword` 明确参与了延迟搜索逻辑，所以它应该进依赖。  
加进去之后，旧定时器会被清理，新输入会重新创建 timer，这正是我们想要的效果。

### 5.3 检查清单：写 effect 前先问自己

- 这个副作用是同步外部系统，还是只是顺手写点逻辑？
- 里面用了哪些外部变量？
- 哪些变量变化后，副作用结果应该变化？
- 有没有对象、数组、函数引用导致依赖不稳定？
- 有没有因为“只想跑一次”而故意漏依赖？

### 5.4 老赵提醒你别踩坑

**依赖判断不是拍脑袋，而是“副作用和数据变化是否一致”的问题。**  
如果拿不准，宁可先让 effect 正确，再去优化性能；不要一上来就为了“少执行几次”把正确性牺牲掉。

---

## 结尾小结：先认清闭包，再谈依赖数组

这一章你要真正记住的，不是某个语法细节，而是三个核心认识：

1. **闭包会保留创建时的环境**，所以函数可能拿到旧状态或旧 props；  
2. **依赖数组的作用是告诉 React 什么时候重新执行副作用**，漏了会不同步，写乱了会重复执行或死循环；  
3. **判断依赖项的关键，不是看代码里出现了什么，而是看这个值变了，副作用是否应该重新运行。**

老赵给你的行动建议很简单：

- 写 `useEffect` 前，先想清楚它到底在同步什么；
- 把 effect 里用到的关键外部值列出来；
- 不要为了省事乱删依赖；
- 遇到旧值问题，先怀疑闭包和依赖，而不是先怀疑 React。

把这一章吃透，你后面学 `useRef`、`useMemo`、`useCallback` 的时候，会明显轻松很多。因为它们本质上，都是在帮你更好地处理“值、引用、渲染、闭包”之间的关系。

# 第5章 useRef：保存可变值、操作 DOM 与跨渲染保留数据

## 开篇引入：为什么你会需要 useRef？

老赵先问你一个很常见的问题：**一个值，我既想在组件里一直保存着，又不想它一变就触发重新渲染，该怎么办？**  
再进一步，如果你还想拿到某个输入框、按钮、视频播放器的真实 DOM 节点，或者想保存一个定时器 ID、上一次的输入值，那 `useRef` 就该登场了。

很多人刚学 Hooks 时，会下意识把所有“会变的东西”都塞进 `useState`，结果是：该刷新的地方刷了，不该刷新的地方也刷了；该保存的值保存了，但组件频繁重渲染；需要读取“最新值”的时候，反而读到旧闭包里的旧数据。

所以，`useRef` 的价值就在于：**它提供一个跨渲染保持不变的容器，里面装的值可以改，但修改它不会触发重新渲染。**

你可以把它理解成一个“稳定的小盒子”。组件每次渲染，函数都会重新执行，但这个盒子还是原来的盒子，里面放什么由你自己决定。正因为如此，`useRef` 在保存临时值、控制 DOM、处理中间态时特别好用。

### 老赵提醒你别踩坑

**`useRef` 不是“更高级的 state”，而是“另一种用途的容器”。**  
如果你的目标是让页面随着值变化而更新，那就别勉强用 ref，优先用 `useState`。

---

## 1. useRef 的基本结构：ref 对象与 current 属性

### 1.1 useRef 到底返回了什么？

`useRef` 返回的是一个对象，通常长这样：

```jsx
const countRef = useRef(0);
```

它不是直接返回 `0`，而是返回一个类似下面的对象：

```jsx
{ current: 0 }
```

以后你访问和修改的都是：

```jsx
countRef.current
```

这个 `current` 就是你真正存值的地方。你可以把它理解成“引用的内容”。只要这个 ref 对象还在，`current` 这个位置就一直存在。

### 1.2 它为什么适合“跨渲染保存数据”？

因为这个 ref 对象本身在组件的多次渲染之间是**同一个对象**。也就是说：

- 组件第一次渲染时创建它；
- 后续重新渲染时，还是拿到同一个 ref 对象；
- 你改的是 `current`，不是这个对象的引用；
- React 不会因为 `current` 变化而自动重新渲染。

这就是 `useRef` 和普通变量最大的区别。普通变量每次函数执行都会重置，而 ref 不会。  
所以你在函数组件里声明普通变量，哪怕写在最外层，也只是“本次渲染临时有效”；而 ref 则能把值稳稳地留住。

### 1.3 小例子：保存一个点击次数但不刷新界面

```jsx
import { useRef } from "react";

function ClickTracker() {
  const clickCountRef = useRef(0);

  const handleClick = () => {
    clickCountRef.current += 1;
    console.log("点击次数：", clickCountRef.current);
  };

  return <button onClick={handleClick}>点我</button>;
}
```

这个例子里，点击次数变了，但 UI 不变。因为我们只是修改了 `current`，没有让组件重新渲染。  
如果这个次数只是用来记录日志、上报数据、做内部判断，那 ref 很合适；如果你想把次数展示在页面上，就应该配合 `useState`。

### 老赵提醒你别踩坑

**不要把 `useRef` 当成“不会变的 state”。**  
它保存的是“可变数据”，但这个变化不会自动反映到界面上。  
如果你希望页面显示更新，就应该用 `useState`；如果只是想存一个“内部值”，再考虑 `useRef`。

---

## 2. useRef 适合保存什么：DOM 引用、定时器 ID、上一次值、临时可变数据

### 2.1 保存 DOM 引用：直接操作真实节点

这是 `useRef` 最经典的使用场景之一。  
在 React 里，大多数时候我们都提倡声明式开发，不直接碰 DOM。但总有一些场景必须拿到真实节点，比如自动聚焦、滚动定位、调用播放器或图表实例方法。

```jsx
import { useRef } from "react";

function SearchBox() {
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current?.focus();
  };

  return (
    <>
      <input ref={inputRef} />
      <button onClick={focusInput}>聚焦输入框</button>
    </>
  );
}
```

这里 `ref={inputRef}` 会让 React 在挂载后把对应 DOM 节点放进 `inputRef.current`。  
你调用 `focus()` 的时候，实际上是在对真实 DOM 做命令式操作。

### 2.2 保存定时器 ID：方便清理副作用

定时器、轮询、节流防抖的计时器，都是典型的“内部可变值”。它们需要被保存下来，以便后续清理，但并不需要展示在页面上。

```jsx
import { useEffect, useRef } from "react";

function Timer() {
  const timerRef = useRef(null);

  useEffect(() => {
    timerRef.current = setInterval(() => {
      console.log("每秒执行一次");
    }, 1000);

    return () => {
      clearInterval(timerRef.current);
    };
  }, []);

  return <div>计时中</div>;
}
```

如果不用 ref，你也能把定时器 ID 存到普通变量里，但一旦重新渲染，变量就没了。ref 则能把它稳稳保留住。

### 2.3 保存上一次的值：实现“前后对比”

`useRef` 很适合记录上一次的 prop 或 state。这个能力特别实用，比如判断值是否变化、比较前后差异、做过渡动画前的状态记录。

```jsx
import { useEffect, useRef } from "react";

function PrevValueDemo({ value }) {
  const prevRef = useRef();

  useEffect(() => {
    prevRef.current = value;
  }, [value]);

  return <p>当前值：{value}，上一次值：{prevRef.current}</p>;
}
```

第一次渲染时，`prevRef.current` 还是 `undefined`，但从第二次开始，它就能保存上一轮的值了。  
核心思路很简单：**先把当前值存进去，下一轮再读取上一轮留下的内容。**

### 2.4 保存临时可变数据：不需要驱动界面的“内部状态”

比如请求进行中的标记、防抖/节流的计时信息、异步任务的取消标志、组件内部缓存对象，这些东西如果只是内部使用，不需要展示到页面上，`useRef` 往往比 `useState` 更合适。因为它们变化太频繁，或者只是中间过程，不值得每次都让页面重新渲染一次。

### 小例子：防止重复提交

```jsx
import { useRef } from "react";

function SubmitButton() {
  const submittingRef = useRef(false);

  const handleSubmit = async () => {
    if (submittingRef.current) return;
    submittingRef.current = true;

    try {
      await fakeRequest();
    } finally {
      submittingRef.current = false;
    }
  };

  return <button onClick={handleSubmit}>提交</button>;
}
```

这个例子里，`submittingRef` 只是用来做“锁”。它不需要展示在界面上，所以没必要用 state。  
如果你还想在按钮上显示“提交中”，那可以把视觉状态交给 `useState`，把内部锁交给 `useRef`，两者分工明确。

### 老赵提醒你别踩坑

**定时器 ID、请求标记、上一次值这些东西，通常不该放进 state。**  
它们不是“界面状态”，而是“组件内部状态”。放进 state 只会增加无意义的渲染。

---

## 3. useRef 与 useState 的区别：是否触发重新渲染

这是本章最重要的判断标准之一。

### 3.1 useState：状态变化会推动视图更新

```jsx
const [count, setCount] = useState(0);
```

当你调用 `setCount` 后，React 会重新渲染组件，页面上的 `count` 也会更新。  
这正是 state 的职责：**一旦变化，就让 UI 跟着变。**

### 3.2 useRef：值变化不会触发重新渲染

```jsx
const countRef = useRef(0);
countRef.current += 1;
```

这里虽然值变了，但 React 不会自动更新 UI。  
因为 ref 的设计初衷不是驱动界面，而是保存一个“稳定可写”的引用容器。

### 3.3 怎么选？

你可以用一个简单标准来判断：

- **需要影响页面展示**：用 `useState`
- **只是保存数据，不需要影响渲染**：用 `useRef`

如果你一时拿不准，就问自己一句：**这个值变了以后，页面要不要变？**  
要变，就 state；不要变，就 ref。

### 3.4 小例子：两者对比

```jsx
import { useRef, useState } from "react";

function CompareDemo() {
  const [renderCount, setRenderCount] = useState(0);
  const refCount = useRef(0);

  const handleClick = () => {
    setRenderCount((c) => c + 1);
    refCount.current += 1;
  };

  return (
    <div>
      <p>state 值：{renderCount}</p>
      <p>ref 值：{refCount.current}</p>
      <button onClick={handleClick}>同时修改</button>
    </div>
  );
}
```

你会发现：`state` 的变化会刷新页面，而 `ref.current` 的变化本身不会推动渲染；只是因为 `state` 更新导致重新渲染后，页面才顺便显示了最新的 `ref.current`。  
这也是很多初学者最容易误解的地方：**不是 ref “不更新”，而是它的更新不负责触发界面刷新。**

### 老赵提醒你别踩坑

很多人误以为“修改了 ref 页面为什么不变”。答案很简单：**因为它本来就不负责驱动渲染。**  
如果你需要界面同步变化，记住：**别硬上 ref，先想 state。**

---

## 4. 如何用 useRef 解决读取最新值的问题？

### 4.1 问题从哪里来：闭包与重新渲染

这里老赵要重点提醒你：React 函数组件每次渲染，函数都会重新执行。  
这意味着：

- 每次渲染都会创建新的变量；
- 事件处理函数、异步回调可能“抓住”当时那一刻的变量；
- 如果回调晚一点执行，它看到的可能是旧值。

这就是闭包在 Hooks 里最常见的“坑”。

比如：

```jsx
import { useEffect, useState } from "react";

function Demo() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setTimeout(() => {
      console.log(count);
    }, 3000);

    return () => clearTimeout(id);
  }, []);
}
```

这里 `count` 可能永远打印首次渲染时的值，因为 `useEffect` 的依赖数组是空的，里面拿到的是首次渲染时的闭包。  
这不是 React “没更新”，而是你的回调拿着旧时刻的变量快照。

### 4.2 用 ref 保存最新值

解决思路是：**让 ref 始终保存最新值，异步回调需要时再去读它。**

```jsx
import { useEffect, useRef, useState } from "react";

function LatestValueDemo() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);

  useEffect(() => {
    countRef.current = count;
  }, [count]);

  const logLater = () => {
    setTimeout(() => {
      console.log("最新 count：", countRef.current);
    }, 3000);
  };

  return (
    <>
      <button onClick={() => setCount((c) => c + 1)}>加一</button>
      <button onClick={logLater}>3 秒后打印最新值</button>
    </>
  );
}
```

关键点是：每当 `count` 变化，我们都同步把最新值写入 `countRef.current`。  
这样一来，不管回调什么时候执行，读取到的都是当前最新的值。

### 4.3 什么时候特别有用？

这种写法在以下场景特别常见：

- 防抖函数内部读取最新表单值；
- WebSocket 回调里读取最新配置；
- 定时器里读取最新 state；
- 异步请求完成时判断当前页面状态是否还有效。

这些场景有一个共同点：**回调执行时间不确定，但你又希望它读到最新数据。**  
这时候 ref 就很有价值。

### 4.4 小例子：避免打印旧值

```jsx
function ChatInput() {
  const [text, setText] = useState("");
  const textRef = useRef(text);

  useEffect(() => {
    textRef.current = text;
  }, [text]);

  const sendLater = () => {
    setTimeout(() => {
      alert("发送内容：" + textRef.current);
    }, 2000);
  };

  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={sendLater}>两秒后发送</button>
    </>
  );
}
```

如果不借助 ref，定时器回调里很容易拿到旧的 `text`。  
通过 ref，我们把“最新输入值”保存起来，保证延迟执行时拿到的不是旧闭包。

### 老赵提醒你别踩坑

**不要以为“把值放到 ref 里就万事大吉”。**  
如果 `ref.current` 的更新时机不对，你读到的还是旧值。  
通常要记得：**在 state 变化后，把最新值同步进 ref。**

---

## 5. 在表单、动画、焦点控制中的实际应用

### 5.1 表单：管理非受控输入或临时缓存

在复杂表单里，有些字段你不一定想每输入一个字符都触发渲染。  
这时可以用 ref 暂存输入内容，或者读取 DOM 的真实值。特别是在“只在提交时读取”的场景，ref 会比受控组件更轻一些。

```jsx
function SimpleForm() {
  const inputRef = useRef(null);

  const handleSubmit = () => {
    console.log(inputRef.current?.value);
  };

  return (
    <>
      <input ref={inputRef} defaultValue="赵老师" />
      <button onClick={handleSubmit}>提交</button>
    </>
  );
}
```

这里输入框是非受控的，值由 DOM 自己维护，提交时再统一读取。  
这类方式适合简单收集信息、低频读取的场景。

### 5.2 焦点控制：提升交互体验

页面加载后自动聚焦搜索框、校验失败后聚焦错误字段，这些都很常见。  
它们不属于“数据变化”，而是“操作界面行为”，因此 ref 很合适。

```jsx
function AutoFocusInput() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} />;
}
```

你可以把它想成：组件挂载完成后，老赵帮你把光标自动放到该去的地方。

### 5.3 动画与外部实例：保存第三方对象

有些动画库、图表库会返回一个实例对象，你不需要它参与渲染，但你要在卸载时销毁它。  
这时可以用 ref 保存实例，避免每次渲染都重建。

```jsx
function VideoPlayer() {
  const playerRef = useRef(null);

  useEffect(() => {
    playerRef.current = createPlayer();
    return () => {
      playerRef.current?.destroy();
    };
  }, []);

  return <div id="player" />;
}
```

这类对象通常生命周期跟组件一致。用 ref 来保存，既稳定又方便清理。

### 5.4 小例子：提交后自动聚焦下一输入框

```jsx
function FormStep() {
  const nextInputRef = useRef(null);

  const goNext = () => {
    nextInputRef.current?.focus();
  };

  return (
    <>
      <input placeholder="第一项" />
      <input ref={nextInputRef} placeholder="第二项" />
      <button onClick={goNext}>跳到下一项</button>
    </>
  );
}
```

这个例子虽然简单，但很能说明 ref 的价值：**它让你在需要的时候，精确控制某个 DOM 行为。**

### 老赵提醒你别踩坑

**如果某个 DOM 操作能用受控组件和 state 解决，就别急着直接操作 DOM。**  
`useRef` 是工具，不是默认方案。能声明式解决的，尽量声明式；需要命令式控制时，再用 ref。

---

## 6. 本章小结：useRef 的判断清单

你可以记住这几个问题来快速判断：

1. **这个值要不要驱动 UI 更新？**  
   - 要：`useState`
   - 不要：`useRef`

2. **这个值会不会跨渲染继续保留？**  
   - 会：`useRef` 很合适

3. **是不是 DOM 节点、定时器、实例对象、上一次值？**  
   - 这些通常都适合放进 `useRef`

4. **是不是异步回调里要读取最新值？**  
   - 可以考虑用 ref 保存最新状态

5. **是不是为了减少无意义渲染？**  
   - 可以把“非展示型数据”从 state 挪到 ref

如果你把这些判断标准记熟，后面学 `useEffect`、`useMemo`、`useCallback` 的时候，你会越来越清楚：哪些东西该参与渲染，哪些东西只是“后台保存”。

---

## 结尾建议：把 useRef 当成“稳定容器”来用

老赵最后给你一句最实用的话：**`useRef` 不是用来“代替 state”的，而是用来保存“不需要触发渲染，但又要在多次渲染之间持续存在”的数据。**

你学会它之后，很多 React 里的老问题会变简单：

- 组件内部怎么存临时值；
- 怎么拿到 DOM；
- 怎么记住上一次的状态；
- 怎么在异步场景下读到最新数据；
- 怎么减少一些没必要的重新渲染。

下一步你可以做两件事：

1. 把自己项目里“只是为了存值”的 `useState` 找出来，看看能不能换成 `useRef`；
2. 刻意练习一次“输入框聚焦 + 定时器 + 上一次值记录”的小练习，把 `useRef` 用熟。

**老赵提醒你别踩坑**：  
别把 ref 变成“随手塞东西的黑洞”。每次使用前先问自己一句：**这个值到底需不需要驱动页面更新？**  
想清楚这一点，`useRef` 才会真正成为你的好帮手。

# 第6章 useMemo：缓存计算结果，理解性能优化的第一步

## 开篇引入：为什么我们会需要 `useMemo`

老赵先说句实在话：**不是所有“慢”都值得优化，也不是所有重复渲染都该立刻上 `useMemo`**。很多同学一觉得卡，就急着加缓存，结果代码更绕，性能未必真变好。

但如果你在项目里遇到这些情况，`useMemo` 就很有用：

- 列表筛选、统计、格式化频繁执行；
- 组件重渲染时要做一遍比较重的派生计算；
- 某个对象或数组传给子组件后，因为引用变化导致子组件总是重渲染；
- 明明关键数据没变，某段计算却反复执行。

这时 `useMemo` 的作用就很明确：**把“根据依赖算出来的值”先记住，只有依赖变了才重新算**。它不是为了“少写代码”，而是为了减少不必要的重复计算，顺带保持引用稳定。

老赵总结一句：**先算一次，后面能复用就复用。**

### 小例子

```jsx
function PricePanel({ goods }) {
  const total = useMemo(() => {
    console.log('重新计算总价');
    return goods.reduce((sum, g) => sum + g.price * g.count, 0);
  }, [goods]);

  return <div>总价：{total}</div>;
}
```

如果父组件只是切换了主题色，`goods` 没变，这个计算就不会重跑。

### 老赵提醒你别踩坑

`useMemo` **不是“让组件不渲染”**。组件该渲染还是会渲染，它只是帮你缓存某个计算结果。别把它和“防止渲染”混为一谈。

---

## 一、`useMemo` 解决的是什么问题：重复计算与不必要的派生值重算

### 1. 什么是“派生值”

在 React 里，很多值不是直接来自接口，而是从已有数据“算出来”的，比如：

- 根据购物车商品算总价；
- 根据搜索词筛选列表；
- 根据表单状态生成提示文案；
- 根据用户权限拼出可见菜单。

这些都叫**派生值**。如果计算过程比较重，而组件又因为别的状态频繁重渲染，就会出现每次都重复计算的问题。

### 2. `useMemo` 在这里做了什么

```jsx
const total = useMemo(() => {
  return items.reduce((sum, item) => sum + item.price, 0);
}, [items]);
```

含义很简单：

- `items` 没变，复用上次结果；
- `items` 变了，重新计算。

这就是 `useMemo` 最典型的价值：**避免派生值在无关渲染中被重复计算**。

### 小例子

```jsx
function PricePanel({ goods }) {
  const total = useMemo(() => {
    console.log('重新计算总价');
    return goods.reduce((sum, g) => sum + g.price * g.count, 0);
  }, [goods]);

  return <div>总价：{total}</div>;
}
```

页面里如果还有“展开/收起”按钮，点击它会让组件重渲染，但只要 `goods` 没变，总价就不必重算。

### 老赵提醒你别踩坑

别把“从 props/state 算出来的值”一股脑都包上 `useMemo`。**`useMemo` 的重点不是“算出来的值”，而是“值是否值得缓存”**。如果只是一个简单表达式，缓存的收益可能还没有维护它的成本大。

---

## 二、什么场景适合用 `useMemo`：昂贵计算、依赖派生、稳定引用

### 1. 适合昂贵计算

如果计算成本明显偏高，比如：

- 大数组排序、过滤、归并；
- 复杂统计；
- 深层对象处理；
- 高频渲染下的重复计算；

那就可以考虑 `useMemo`。

```jsx
const filteredUsers = useMemo(() => {
  return users.filter(user => user.name.includes(keyword));
}, [users, keyword]);
```

如果 `users` 很大，而组件里还有别的 state 经常变化，这个缓存就能省掉不少重复工作。

### 2. 适合依赖派生

有些值本质上就是“从状态或 props 派生出来的结果”，只要依赖不变，结果通常也不变，这就很适合 memo。

```jsx
const fullName = useMemo(() => {
  return `${firstName} ${lastName}`;
}, [firstName, lastName]);
```

这个例子本身不重，但它说明了 `useMemo` 的思路：**结果由依赖决定**。

### 3. 适合稳定引用

对象和数组每次创建都是新引用，即使内容一样，引用也变了。

```jsx
const config = { pageSize: 10, showBorder: true };
```

如果每次渲染都直接写这句，`config` 每次都是新对象。传给依赖浅比较的子组件或 Hook 时，容易触发额外更新。这时可以用：

```jsx
const config = useMemo(() => ({
  pageSize: 10,
  showBorder: true
}), []);
```

### 小例子

```jsx
function UserList({ users, keyword }) {
  const visibleUsers = useMemo(() => {
    return users.filter(u => u.name.includes(keyword));
  }, [users, keyword]);

  return visibleUsers.map(u => <div key={u.id}>{u.name}</div>);
}
```

这里既有筛选逻辑，也有列表渲染，`useMemo` 就很合理。

### 老赵提醒你别踩坑

`useMemo` 的“稳定引用”确实有用，但别为了稳定而稳定。简单字符串拼接、简单数学运算，通常没必要上它。**优化也有成本，别把小题做成大题。**

---

## 三、`useMemo` 的依赖机制与缓存失效时机

### 1. 依赖数组决定缓存是否刷新

`useMemo` 的第二个参数是依赖数组：

```jsx
const value = useMemo(() => compute(a, b), [a, b]);
```

规则和 `useEffect` 类似：

- 依赖没变：复用缓存值；
- 依赖变了：重新执行计算函数，生成新值。

也就是说，`useMemo` 不是永远缓存，而是**按依赖条件缓存**。

### 2. 缓存什么时候失效

通常有三种情况：

1. **依赖变化**：最常见；
2. **组件重新挂载**：卸载后再挂载，缓存就没了；
3. **开发环境特殊行为**：某些开发模式下，React 可能会有额外检查，看起来像是执行更频繁，理解依赖机制本身就行。

### 3. 依赖数组怎么写才靠谱

依赖数组不能乱写，也不能偷懒。应当把**计算结果真正依赖的所有外部值**都列进去。

```jsx
const result = useMemo(() => {
  return list.filter(item => item.type === type);
}, [list, type]);
```

如果漏掉 `type`，结果就可能过期。  
这类问题很隐蔽：页面不报错，但数据会在某些时机悄悄不对。

### 小例子

```jsx
function SearchBox({ list, keyword }) {
  const matched = useMemo(() => {
    console.log('筛选列表');
    return list.filter(item => item.includes(keyword));
  }, [list, keyword]);

  return <div>匹配结果：{matched.length}</div>;
}
```

只要 `keyword` 变化，筛选就会重新执行；如果只是别的状态变了，缓存就可以继续用。

### 老赵提醒你别踩坑

别把 `useMemo` 当成逃避依赖数组的工具。**依赖写错，缓存就会把错误一起缓存下来**。很多“数据怎么没更新”的问题，最后一查就是少写了依赖。

---

## 四、一个完整业务例子：列表筛选、复杂统计与优化前后对比

老赵带你看一个更贴近业务的例子。假设你做的是商品管理页，有搜索、分类切换和统计卡片。用户每输入一个字，组件都会重新渲染；如果不做处理，筛选和统计也会跟着反复执行。

### 1. 问题现象

```jsx
function ProductDashboard({ products, category, keyword }) {
  const visibleProducts = products
    .filter(p => p.category === category)
    .filter(p => p.name.includes(keyword))
    .sort((a, b) => b.sales - a.sales);

  const totalSales = visibleProducts.reduce((sum, p) => sum + p.sales, 0);

  return (
    <div>
      <p>总销量：{totalSales}</p>
      {visibleProducts.map(p => (
        <div key={p.id}>{p.name}</div>
      ))}
    </div>
  );
}
```

这段代码没错，但只要父组件任何状态变化，哪怕只是切换弹窗，筛选、排序和统计都会重新执行。

### 2. 为什么慢

慢的点不在“代码行数”，而在：

- `filter`、`sort` 会遍历数据；
- `reduce` 又做了一次统计；
- 数据量一大，重复执行成本就上来了；
- 页面里无关状态一多，这些计算就会被反复带跑。

### 3. 怎么用 `useMemo`

```jsx
function ProductDashboard({ products, category, keyword }) {
  const visibleProducts = useMemo(() => {
    console.log('重新计算筛选和排序');
    return products
      .filter(p => p.category === category)
      .filter(p => p.name.includes(keyword))
      .sort((a, b) => b.sales - a.sales);
  }, [products, category, keyword]);

  const totalSales = useMemo(() => {
    console.log('重新计算总销量');
    return visibleProducts.reduce((sum, p) => sum + p.sales, 0);
  }, [visibleProducts]);

  return (
    <div>
      <p>总销量：{totalSales}</p>
      {visibleProducts.map(p => (
        <div key={p.id}>{p.name}</div>
      ))}
    </div>
  );
}
```

这样就实现了：

- `products/category/keyword` 不变时，列表筛选和排序复用结果；
- `visibleProducts` 不变时，总销量也复用结果；
- 只有真正相关的数据变化时，才重新计算。

### 4. 优化前后对比

优化前：

- 每次重渲染都要重新筛选、排序、统计；
- 计算逻辑直接堆在渲染里；
- 数据越大，卡顿越明显。

优化后：

- 无关状态变化不会重复做重活；
- 计算逻辑更明确；
- 结果更稳定，页面更顺滑。

### 小例子

上面的 `ProductDashboard` 就是典型例子：**先定位重复计算，再决定是否缓存**。

### 老赵提醒你别踩坑

别以为把所有计算都拆成 `useMemo` 就一定更快。`useMemo` 只在“重复计算明显”的情况下更划算；如果数据量小、计算轻，反而可能让代码更复杂。**真正的优化不是“多”，而是“准”**。

---

## 五、`useMemo` 不是万能优化：避免过早优化和滥用

### 1. 先判断，再优化

老赵建议你按这个顺序来：

1. 先把功能写对；
2. 再确认是否真的有性能问题；
3. 最后再决定要不要加 `useMemo`。

因为 `useMemo` 也不是零成本：

- 要维护依赖数组；
- React 要做缓存判断；
- 代码可读性会下降；
- 过度使用会让组件变难懂。

### 2. 哪些情况通常不值得用

- 计算很轻，比如字符串拼接、简单判断；
- 组件渲染频率低；
- 数据量小；
- 子组件根本不关心引用变化；
- 只是“感觉可能慢”，没有实际迹象。

比如：

```jsx
const title = useMemo(() => `欢迎，${name}`, [name]);
```

这不是不能写，而是多数时候没必要。直接写：

```jsx
const title = `欢迎，${name}`;
```

更清晰，也更符合日常阅读习惯。

### 3. 什么时候值得用

你可以问自己三个问题：

- 这个值是不是由其他数据派生出来的？
- 它的计算过程是不是明显有点重？
- 它是不是会因为无关渲染而被重复计算？

如果至少两个答案是“是”，就值得考虑 `useMemo`。

### 小例子

```jsx
function ProductTable({ products, keyword }) {
  const visibleProducts = useMemo(() => {
    return products
      .filter(p => p.name.includes(keyword))
      .sort((a, b) => b.sales - a.sales);
  }, [products, keyword]);

  return visibleProducts.map(p => <div key={p.id}>{p.name}</div>);
}
```

这里的过滤和排序会重复执行，而且结果直接影响列表渲染，使用 `useMemo` 比较合理。

### 老赵提醒你别踩坑

**别为了“看起来在优化”而优化。** 真正的性能优化，第一步不是加 Hook，而是先找到瓶颈。否则你得到的可能不是更快的代码，而是更复杂、更难维护的代码。

---

## 六、如何判断一个场景是否真的值得 memo

### 1. 老赵的判断步骤

你可以按下面这套流程判断：

#### 第一步：看这是不是派生值
如果值是根据 props/state 算出来的，它才有可能适合 `useMemo`。

#### 第二步：看计算是否“重”
标准不是“写了多少行”，而是：

- 数据量是否大；
- 是否要遍历、过滤、排序；
- 是否要复杂转换；
- 是否在高频渲染中反复出现。

#### 第三步：看它是否会导致无关更新
如果这个值被传给子组件、依赖某些 Hook，或者作为配置对象使用，那么引用稳定性就很关键。

#### 第四步：看代码复杂度是否值得
如果加了 `useMemo` 后代码明显更绕，而收益很小，那就不值得。

### 2. 一个简单的判断模型

老赵给你一个实用模型：**“重不重、稳不稳、值不值”**

- **重不重**：计算是否昂贵；
- **稳不稳**：引用是否需要稳定；
- **值不值**：优化收益是否大于维护成本。

如果这三个维度里只有一个弱弱地“可能”，那就先别上 `useMemo`。

### 小例子

```jsx
function CategoryPanel({ items, activeType }) {
  const summary = useMemo(() => {
    return items
      .filter(item => item.type === activeType)
      .map(item => item.name)
      .join('、');
  }, [items, activeType]);

  return <p>{summary}</p>;
}
```

如果 `items` 很多、筛选频繁变化，这个缓存是值得的；如果只是几个固定选项，那直接计算也完全够用。

### 老赵提醒你别踩坑

判断是否 memo，不要只看“当前快不快”，还要看“后面会不会变复杂”。有些场景现在不重，但数据规模一上来就会卡，这时可以提前留好结构，但别过早堆优化。

---

## 结尾小结：先理解缓存，再谈优化

这一章你真正要记住的，不是 `useMemo` 这个 API 本身，而是它背后的思路：

- 它缓存的是**计算结果**，不是组件渲染；
- 它适合**昂贵计算、派生值、稳定引用**；
- 它是否生效，完全取决于**依赖数组**；
- 它不是万能药，**过早优化**反而会增加复杂度；
- 判断是否值得用，核心看 **重不重、稳不稳、值不值**。

老赵建议你在项目里遇到性能问题时，先别急着写 `useMemo`。先找出真正的重复计算点，再决定要不要缓存。  
这才是一个成熟的 Hooks 使用习惯。

下一章我们继续讲 `useCallback`。你会看到它和 `useMemo` 很像，但关注点不一样。搞懂这俩，很多“为什么子组件老重渲染”的问题，就能顺手解决了。

# 第7章 useCallback：稳定函数引用，配合子组件与性能优化

前面讲完 `useMemo`，老赵顺手把一个特别容易被误解的 Hook 拿出来单讲：`useCallback`。很多人第一次见它都会问：“这不就是把函数包了一层吗？有什么用？”

答案很直接：**`useCallback` 缓存的是函数引用，不是函数执行结果。**

它的价值不在“高级”，而在“稳定”。当你把函数传给子组件、把函数放进依赖数组，或者要配合 `React.memo` 做渲染优化时，`useCallback` 往往就是关键一环。

更准确一点说，本章真正要解决的核心问题是：**为什么要稳定函数引用、什么时候才需要稳定、稳定之后到底能解决什么真实痛点。**  
如果只是为了“看起来更专业”去包一层 `useCallback`，那基本就是给自己加负担；但如果你能判断哪些函数会因为引用变化引发子组件重渲染、依赖抖动或闭包过期，那它就会变成很实用的工具。

---

## 一、useCallback 的作用：它到底缓存了什么？

一句话记住：**`useCallback(fn, deps)` 的意思是，依赖不变时，返回同一个函数引用。**

它不是帮你记住函数内容，而是帮你记住“这个函数还是不是原来那个”。在 React 里，函数也是值；组件每次重新渲染，函数定义通常都会重新创建一次。内容一样，不代表引用一样。

### 小例子

```jsx
import { useCallback, useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  const handleAdd = useCallback(() => {
    setCount(c => c + 1);
  }, []);

  return <button onClick={handleAdd}>加 1：{count}</button>;
}
```

这里 `handleAdd` 依赖为空，尽量保持同一个引用。之所以能安全写成 `[]`，是因为 `setCount` 是稳定的，而且使用了函数式更新，没有直接依赖 `count`。如果你写成 `setCount(count + 1)`，那就要把 `count` 放进依赖里，否则会拿到旧值。

### 适用场景

`useCallback` 最常见的场景有三类：

- **传给子组件**，尤其是子组件用了 `React.memo`
- **作为其他 Hook 的依赖项**，比如 `useEffect`
- **需要稳定函数引用**，避免不必要的联动更新

### 老赵提醒你别踩坑

别把 `useCallback` 当成性能万金油。  
**只有当“函数引用变化”会带来实际代价时，它才有意义。**

---

## 二、为什么函数引用变化会影响子组件渲染？

理解 `useCallback` 的关键，就是明白：**函数引用变化，会让 React 认为 props 变了。**

`React.memo` 会对 props 做浅比较：基本类型通常按值看，引用类型按地址看。函数本质上也是引用类型，所以如果你这样写：

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    console.log("点击了");
  };

  return <Child onClick={handleClick} />;
}
```

每次 `Parent` 重新渲染，`handleClick` 都是新引用。即使逻辑没变，`Child` 收到的 `onClick` 也变了；如果 `Child` 用了 `React.memo`，它就更容易重新渲染。

### 小例子

```jsx
const Child = React.memo(function Child({ onClick }) {
  console.log("Child render");
  return <button onClick={onClick}>子组件按钮</button>;
});
```

父组件状态一变，函数引用也变，`Child` 就会跟着“被动刷新”。

### 实战场景

#### 未使用 useCallback

```jsx
import React, { useState } from "react";

const SaveButton = React.memo(function SaveButton({ onSave }) {
  console.log("SaveButton render");
  return <button onClick={onSave}>保存</button>;
});

function EditorPage() {
  const [title, setTitle] = useState("");
  const [content, setContent] = useState("");

  const handleSave = () => {
    console.log("保存草稿：", { title, content });
  };

  return (
    <div>
      <input value={title} onChange={e => setTitle(e.target.value)} />
      <textarea value={content} onChange={e => setContent(e.target.value)} />
      <SaveButton onSave={handleSave} />
    </div>
  );
}
```

这里只要标题或正文变化，`handleSave` 就会变成新函数引用。即使 `SaveButton` 包了 `React.memo`，它也会因为 `onSave` 变化而重新渲染。

#### 使用 useCallback 之后

```jsx
import React, { useCallback, useState } from "react";

const SaveButton = React.memo(function SaveButton({ onSave }) {
  console.log("SaveButton render");
  return <button onClick={onSave}>保存</button>;
});

function EditorPage() {
  const [title, setTitle] = useState("");
  const [content, setContent] = useState("");

  const handleSave = useCallback(() => {
    console.log("保存草稿：", { title, content });
  }, [title, content]);

  return (
    <div>
      <input value={title} onChange={e => setTitle(e.target.value)} />
      <textarea value={content} onChange={e => setContent(e.target.value)} />
      <SaveButton onSave={handleSave} />
    </div>
  );
}
```

这时，只有 `title` 或 `content` 真正变化时，`handleSave` 才会更新；如果父组件因为别的状态重渲染，`SaveButton` 就更容易保持稳定。

这里要注意：**`useCallback` 不是让子组件永远不渲染，而是减少“无意义的因为函数引用变化而渲染”。**

### 老赵提醒你别踩坑

别一看到子组件重渲染，就只怀疑数据变了。  
**很多时候，是函数引用变了，不是业务数据变了。**

---

## 三、useCallback 适合哪些场景？

`useCallback` 更像“引用稳定器”，不是“所有事件都要包一层”。

### 1. 传给子组件

当子组件用了 `React.memo`，又想避免父组件状态变化导致它无意义重渲染时，可以用 `useCallback`。

### 小例子

```jsx
const Button = React.memo(function Button({ onSave }) {
  console.log("Button render");
  return <button onClick={onSave}>保存</button>;
});

function Page() {
  const [text, setText] = useState("");

  const handleSave = useCallback(() => {
    console.log("保存内容:", text);
  }, [text]);

  return <Button onSave={handleSave} />;
}
```

### 2. 作为依赖项

当某个 `useEffect` 依赖函数时，如果这个函数每次都变，effect 也会频繁触发。此时 `useCallback` 可以把依赖控制住。

### 小例子

```jsx
const fetchData = useCallback(() => {
  // 请求逻辑
}, [userId]);

useEffect(() => {
  fetchData();
}, [fetchData]);
```

### 3. 配合 `React.memo`

`memo` 没起效时，除了检查对象和数组 props，也要看看是不是传了一个“每次都新的函数”。

### 老赵提醒你别踩坑

`useCallback` 不是“加了就更快”。  
如果子组件本来就会因为别的 props 变化而重渲染，或者函数创建成本很低，那它未必带来收益，反而增加理解成本。

---

## 四、useCallback 和 useMemo 有什么区别与联系？

一句话先记住：

- **`useMemo` 缓存的是“值”**
- **`useCallback` 缓存的是“函数”**

更准确地说，`useCallback(fn, deps)` 可以理解成 `useMemo(() => fn, deps)` 的语义化写法。两者底层思路接近，都是依赖不变时返回上次结果。

如果你总是分不清，也可以用一个最简单的判断方法：  
**先问自己，我要缓存的是“值”，还是“函数”？**  
- 缓存计算结果、对象、数组、派生数据，通常用 `useMemo`
- 缓存回调函数引用，通常用 `useCallback`

### 对比理解

#### useMemo

```jsx
const total = useMemo(() => {
  return list.reduce((sum, item) => sum + item.price, 0);
}, [list]);
```

#### useCallback

```jsx
const handleClick = useCallback(() => {
  console.log("clicked");
}, []);
```

### 小例子

```jsx
function Demo({ list }) {
  const total = useMemo(() => list.length, [list]);

  const logTotal = useCallback(() => {
    console.log(total);
  }, [total]);

  return <button onClick={logTotal}>打印总数</button>;
}
```

### 老赵提醒你别踩坑

别把 `useCallback` 和 `useMemo` 当成两个长得像的 API，然后机械套用。  
**先看你要稳定的是“值”还是“函数引用”**，再决定用哪个。

---

## 五、useCallback 的依赖数组怎么写才靠谱？

`useCallback` 的依赖数组，决定了函数什么时候更新。你可以把它理解为：**函数内部用到了哪些外部变量，这些变量就应该进依赖。**

老赵建议你写依赖数组时，先做一个简单检查：

1. 回调里直接读取了哪些 state？
2. 回调里直接读取了哪些 props？
3. 回调里调用了哪些外部函数？
4. 这些外部函数本身是否稳定？
5. 有没有通过函数式更新避开某些状态依赖？

### 基本原则

凡是在回调里直接使用的外部状态、props、函数，都要考虑放进依赖数组。

#### 小例子

```jsx
function Profile({ userId }) {
  const [name, setName] = useState("");

  const handleSubmit = useCallback(() => {
    console.log(userId, name);
  }, [userId, name]);

  return <button onClick={handleSubmit}>提交</button>;
}
```

### 常见误区

#### 误区 1：为了稳定故意少写依赖

```jsx
const handleSubmit = useCallback(() => {
  console.log(name);
}, []);
```

这会导致闭包拿到旧的 `name`。函数看似稳定，逻辑却已经过期。

#### 误区 2：把所有东西都塞进去，不思考

依赖不是越多越好，但该进的必须进。依赖特别多时，往往说明你该拆分逻辑了。

### 闭包与重新渲染

这是 `useCallback` 最容易翻车的地方。函数会捕获创建它那次渲染中的变量，这就是闭包。少写依赖时，React 不会帮你自动刷新闭包里的值，结果就是：

- UI 已更新
- 回调里还是旧值
- 你以为“函数没生效”

#### 小例子

```jsx
function ChatBox() {
  const [message, setMessage] = useState("hello");

  const sendMessage = useCallback(() => {
    alert(message);
  }, [message]);

  return <button onClick={sendMessage}>发送</button>;
}
```

如果这里写成 `[]`，按钮点下去可能永远弹出第一次渲染时的 `message`。这就是典型的闭包问题。

### 依赖写错会带来什么问题？

老赵把它拆成两类：

- **依赖少了**：函数引用稳定了，但拿到的是旧数据，容易出现闭包 bug
- **依赖多了**：函数引用频繁变化，`memo`、`useEffect` 的优化效果会被削弱

所以依赖数组不是“越少越好”，而是“越准确越好”。

### 老赵提醒你别踩坑

**别为了稳定引用牺牲数据正确性。**  
Hooks 里，错误的依赖比多一点渲染更麻烦，因为它会带来很难排查的逻辑 bug。

---

## 六、真实项目里怎么用 useCallback 才算会用？

在项目里，`useCallback` 往往不是单独出现的，而是和 `memo`、`useEffect`、`useMemo` 组合起来，用来控制渲染范围。

### 一个常见判断流程

1. 子组件是否真的频繁重渲染
2. 这些渲染是否影响体验或性能
3. 是否因为函数引用变化导致 props 变化
4. 是否可以用 `useCallback` 稳住引用
5. 稳定后是否还要配合 `React.memo`

老赵建议你以后遇到性能问题，先按这个顺序排查。

### 小例子

```jsx
const UserList = React.memo(function UserList({ onSelect }) {
  console.log("UserList render");
  return <div onClick={onSelect}>用户列表</div>;
});

function App() {
  const [theme, setTheme] = useState("light");

  const handleSelect = useCallback(() => {
    console.log("选择用户");
  }, []);

  return (
    <>
      <button onClick={() => setTheme(t => (t === "light" ? "dark" : "light"))}>
        切换主题
      </button>
      <UserList onSelect={handleSelect} />
    </>
  );
}
```

切换主题时，`App` 会重渲染，但 `handleSelect` 引用稳定，`UserList` 也更容易保持不变。若它内部还有列表、图表、复杂布局，这种稳定性就更有价值。

### 性能优化的基本思路

老赵给你一个简单原则：

- **先保证正确**
- **再看性能**
- **最后做针对性优化**

`useCallback` 不是第一步，而是确认有必要后再做的优化手段。真正有效的优化通常来自：

- 减少无意义的 props 变化
- 拆分大组件
- 缩小组件职责
- 配合 `memo`、`useMemo` 使用

也就是说，`useCallback` 的目标不是“让函数不创建”，而是**减少无意义的子组件更新**。如果一个函数不会传给子组件，也不会进入依赖数组，通常就没有必要为了它去包一层。

### 老赵提醒你别踩坑

不要把所有事件处理函数都套一层 `useCallback`。那样只会增加依赖复杂度，降低可读性。  
**性能优化不是 Hook 越多越专业，而是优化点越精准越好。**

---

## 本章小结：你应该怎么掌握 useCallback？

你可以把 `useCallback` 记成一句话：  
**当我需要让函数引用保持稳定，以便配合子组件渲染优化、依赖控制或缓存逻辑时，就考虑 useCallback。**

最后老赵给你一份实战检查清单：

- 这个函数会不会传给子组件？
- 子组件有没有 `React.memo`？
- 这个函数会不会作为依赖项？
- 函数内部有没有用到外部状态或 props？
- 依赖数组是否完整？
- 去掉 `useCallback` 后，功能是否仍然正确？
- 保留它后，是否真的减少了无意义渲染？
- 这个优化是否值得付出额外的理解成本？

如果你能答上来，说明你已经不是“会写 `useCallback`”，而是开始“会判断何时该用它”了。Hooks 的功夫，不在 API 本身，而在你能不能把它放到正确的位置上。

# 第8章 useContext：跨层传递数据，减少层层 props 传递

## 开篇引入：为什么到了这里，我们需要 useContext

老赵先带你回忆一下：组件层级浅的时候，数据靠 `props` 一路往下传没问题；但一旦组件树变深，比如“页面 → 布局 → 侧边栏 → 菜单 → 按钮”，中间很多组件其实根本不用这份数据，却还得一层层帮忙转交，这就是常说的 **props drilling**。

`useContext` 解决的，就是这种“跨很多层传数据”的场景。它很适合传一些**全局感强、但又没必要上重型状态管理**的内容，比如：

- 主题色、暗黑模式
- 语言切换
- 登录态、用户信息
- 全局配置、权限信息
- 运行环境参数

赵老师提醒你一句：`useContext` 不是“什么都往里塞”的万能桶，它更像一条**共享通道**。它能解决跨层传递，但不适合硬扛高频、复杂、频繁局部变化的数据。

### 小例子

```jsx
const ThemeContext = React.createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Page />
    </ThemeContext.Provider>
  );
}

function Page() {
  return <Toolbar />;
}

function Toolbar() {
  return <ThemeButton />;
}

function ThemeButton() {
  const theme = React.useContext(ThemeContext);
  return <button>当前主题：{theme}</button>;
}
```

这里 `ThemeButton` 不用再通过 `Page`、`Toolbar` 一层层接收 `theme`，直接读取上下文即可。

### 老赵提醒你别踩坑

别为了“省 props”就把所有状态都塞进 Context。**能局部传就局部传，只有确实跨层、且多个组件都要用时，再考虑 Context。**

---

## Context Provider 与 Consumer 的基本工作方式

useContext 的核心，离不开两个角色：**Provider** 和 **Consumer**。

- **Provider**：负责提供数据
- **Consumer**：负责读取数据

现代 React 里我们更常用 `useContext` 直接读取，本质上和 `Context.Consumer` 是一回事。

### 它是怎么工作的？

1. 先创建 Context。
2. 用 `Provider` 包住需要共享的区域，并通过 `value` 传值。
3. 下层组件用 `useContext(Context)` 读取。
4. 只要 `Provider` 的 `value` 变化，消费这个 Context 的组件就可能重新渲染。

### 小例子

```jsx
const LangContext = React.createContext('zh-CN');

function App() {
  return (
    <LangContext.Provider value="en-US">
      <Header />
    </LangContext.Provider>
  );
}

function Header() {
  const lang = React.useContext(LangContext);
  return <div>当前语言：{lang}</div>;
}
```

这就是最基础的“上面给，下面拿”。如果顶部导航、侧边栏、底部工具条都要显示语言信息，这种方式会比逐层传 `lang` 省心很多。

### 老赵提醒你别踩坑

`Provider` 的 `value` 如果每次渲染都是一个新对象，消费组件就会更容易被连带刷新。比如：

```jsx
<ConfigContext.Provider value={{ theme, lang }}>
```

就算内容没变，只要引用变了，也可能触发多余更新。

---

## useContext 与 props drilling 的对比

第一次接触 `useContext`，很多人会想：是不是以后都不用 `props` 了？老赵告诉你，不是。

### props drilling 的特点

- 优点：数据流清晰，传什么一眼能看见
- 缺点：层级深时很烦，中间层组件被迫“转手”

### useContext 的特点

- 优点：避免中间层层传递，适合共享数据
- 缺点：共享范围太大时，依赖关系会变模糊

### 怎么选？

- 父子之间简单传参：**优先 props**
- 跨很多层，且多个组件都要读：**考虑 Context**
- 数据高频变化、逻辑复杂：**先想清楚是否更适合别的状态方案**

### 小例子

用户信息需要在 `Header`、`Avatar`、`Sidebar`、`ProfileMenu` 中使用，且它们分布在不同层级，这时用 `UserContext` 会比逐层传 `user` 更自然。

```jsx
const UserContext = React.createContext(null);

function App() {
  const user = { name: '老赵', role: 'admin' };

  return (
    <UserContext.Provider value={user}>
      <Layout />
    </UserContext.Provider>
  );
}

function Avatar() {
  const user = React.useContext(UserContext);
  return <span>{user.name}</span>;
}
```

这里 `Layout`、`Sidebar`、`Header` 不需要关心 `user`，但 `Avatar` 可以直接拿到数据。

### 老赵提醒你别踩坑

`useContext` 不是“更高级的 props”。**别为了架构感，把简单传参搞复杂。**

---

## Context 适合什么场景，不适合什么场景

讲实战，不能只讲“能用”，还得讲“该不该用”。老赵建议你用一个简单判断法：**共享、跨层、低频变化**，这三个条件越满足，Context 越适合。

### 适合的场景

- 主题切换：深层组件都要知道当前主题
- 语言切换：整个页面都需要同一套文案语言
- 登录态：头部、菜单、个人中心都可能读取用户信息
- 全局配置：接口前缀、权限开关、站点信息

### 不太适合的场景

- 页面内部很局部的表单状态
- 输入框每敲一个字就变化的数据
- 变化非常频繁、且只影响少量组件的状态
- 需要复杂派生、回滚、时间旅行的业务状态

这些场景更适合 `useState`、`useReducer`，或者更完整的状态管理方案，而不是一股脑塞进 Context。

### 小例子

如果只有一个弹窗内部的开关状态，直接放在弹窗组件里就够了；没必要为了“共享”把它抬到全局 Context。

### 老赵提醒你别踩坑

**Context 不是全局状态管理的替代品，更不是所有数据的收纳盒。** 该局部就局部，该共享才共享。

---

## Context 值变化会带来什么渲染影响

这一节很关键，也是很多人第一次用 Context 时最容易忽略的地方。

### 问题核心：值一变，谁会重新渲染？

当 `Provider` 的 `value` 变化时，所有读取这个 Context 的组件，都可能重新渲染。注意，这不是“只更新真正用到变化字段的组件”，而是**依赖这个 Context 的组件整体刷新**。

比如你把“用户信息”和“主题色”放在一个 Context 里：

```jsx
const AppContext = React.createContext(null);
```

如果某次只改了主题色，但头像组件也读了这个 Context，那头像组件也会跟着重渲染。

### 这意味着什么？

- Context 很适合“共享”
- 但不适合把大量、变化频繁、彼此无关的状态塞一起

### 依赖数组、闭包和重新渲染的关联

这里顺手把前面讲过的概念串起来：

- **重新渲染**：Provider 的值变化，会触发消费组件更新
- **闭包**：回调里如果拿的是旧值，容易出现“看起来变了，实际读到还是老数据”
- **依赖数组**：在 `useMemo`、`useCallback` 里稳定 Context 的 value 时，依赖写错，会导致值更新不及时或无意义地频繁变动

比如：

```jsx
function App() {
  const [theme, setTheme] = React.useState('light');
  const [lang, setLang] = React.useState('zh-CN');

  const value = React.useMemo(() => ({ theme, lang, setTheme, setLang }), [
    theme,
    lang,
  ]);

  return (
    <AppContext.Provider value={value}>
      <Page />
    </AppContext.Provider>
  );
}
```

这样可以避免每次渲染都创建新对象，减少无意义刷新。这里 `useMemo` 的作用不是“神奇地优化一切”，而是**让 Context 的 value 引用更稳定**。

### 小例子

```jsx
function ThemeSwitch() {
  const { theme, setTheme } = React.useContext(AppContext);

  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      切换主题
    </button>
  );
}
```

如果 `AppContext` 被拆得合理，切换主题时只会影响主题相关组件，而不会把整个页面都拖着重渲染。

### 老赵提醒你别踩坑

Context 更新不是“按字段精细更新”，更像是“订阅这个上下文的组件一起动”。**拆分 Context，往往比硬优化更有效。**

---

## 一个贴近业务的完整流程：主题切换怎么做

老赵给你补一个更像真实项目的例子。假设我们要做一个后台系统，支持主题切换。

### 第一步：创建 Context

```jsx
const ThemeContext = React.createContext(null);
```

### 第二步：在顶层提供状态

```jsx
function App() {
  const [theme, setTheme] = React.useState('light');

  const value = React.useMemo(() => ({ theme, setTheme }), [theme]);

  return (
    <ThemeContext.Provider value={value}>
      <Layout />
    </ThemeContext.Provider>
  );
}
```

### 第三步：在深层组件中消费

```jsx
function ThemeButton() {
  const { theme, setTheme } = React.useContext(ThemeContext);

  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      当前主题：{theme}
    </button>
  );
}
```

### 第四步：让多个层级都能用

```jsx
function Sidebar() {
  return (
    <div>
      <ThemeButton />
    </div>
  );
}
```

这时候 `Sidebar` 不需要知道主题逻辑，`ThemeButton` 直接拿到上下文即可。现实项目里，语言切换、用户权限、登录态，都是类似的套路。

### 老赵提醒你别踩坑

别把 `theme`、`lang`、`user`、`permission` 全塞进一个大对象里图省事。**越是看起来集中，越容易把更新范围放大。**

---

## 如何合理拆分 Context，避免一个值拖垮整个树

如果你把“登录态、主题、语言、配置、权限”全放进一个 `AppContext`，表面上集中，实际上会带来两个问题：

1. 任意一个值变化，整个消费树都可能刷新
2. 上下文职责混乱，后期维护很难

### 拆分原则

老赵建议按“变化频率”和“业务职责”拆：

- 主题一个 Context
- 语言一个 Context
- 用户信息一个 Context
- 配置一个 Context

这样做的好处是：

- 谁变谁更新，影响范围更小
- 语义更清晰
- 更容易排查问题

### 小例子

```jsx
const ThemeContext = React.createContext('light');
const UserContext = React.createContext(null);

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <UserContext.Provider value={{ name: '老赵' }}>
        <Layout />
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}
```

如果只有主题变化，用户相关组件未必会跟着刷新。拆开以后，更新边界也更清楚。

### 老赵提醒你别踩坑

不要把 Context 当成“项目级大杂烩仓库”。**拆分不是多写几个 Provider 而已，它是在控制渲染边界。**

---

## 结尾小结：把 useContext 用在该用的地方

这一章你只要记住一句话：**useContext 适合解决跨层共享数据，但不适合无边界地承载所有状态。**

今天我们讲了它的几个关键点：

- 它适合主题、语言、登录态、全局配置
- 它通过 Provider 提供数据，消费组件读取数据
- 它能替代一部分 props drilling，但不是全部
- 它的值变化会带来重新渲染，要注意性能影响
- 最好按职责拆分 Context，减少一个值拖垮整棵树

### 最后给你一个行动建议

下次项目里碰到“某个数据要一层层往下传”的场景，先别急着上 Context。先判断：

- 是否真的跨层？
- 是否很多组件都要用？
- 是否适合独立成一类共享数据？

如果答案成立，再用 `useContext`。这样你会发现它既简洁，又不容易把架构弄乱。

### 老赵提醒你别踩坑

记住，Context 的价值不是“替代 props”，而是“把共享数据放到合适的位置”。**位置对了，代码就顺；位置错了，后面优化会很痛。**

# 第9章 自定义 Hook：把重复逻辑抽出来，形成可复用能力

前面我们已经把 `useState`、`useEffect`、`useRef` 这些基础 Hook 讲明白了。到了真实项目里，你会很快发现一个问题：**页面越多，重复逻辑越容易散得到处都是**。请求数据、处理表单、监听窗口大小、做防抖、读写缓存，这些逻辑如果每个组件都写一遍，代码会越来越乱，维护成本也会越来越高。

这时候，自定义 Hook 就派上用场了。赵老师常说：**Hook 不只是让你少写 class，更重要的是把逻辑从组件里拎出来，变成可复用、可组合、可维护的能力。**  
这一章我们就讲清楚：什么逻辑适合抽成自定义 Hook，怎么命名、怎么设计输入输出，怎么把状态、effect、ref、callback 组合起来，最后再看几个最常见的案例。咱们不只看“成品”，还要把“抽象过程”走一遍，这样你以后在项目里才能真的会封装。

---

## 一、什么样的逻辑适合抽成自定义 Hook？

不是所有代码都适合抽成 Hook。判断标准很简单：**如果一段逻辑在多个组件里反复出现，而且主要由“状态 + 副作用 + 行为”构成，和 UI 强相关但又不属于某个具体页面，就很适合抽成自定义 Hook。**

换句话说，Hook 抽的是“能力”，不是“页面”。页面会变，能力往往更稳定。只要你的逻辑满足“复用性强、边界清晰、可以独立描述”，就值得认真考虑抽出来。

### 适合抽取的典型场景

1. **数据请求逻辑**：列表页、详情页、搜索页都要处理“加载中 / 成功 / 失败 / 重新请求”。
2. **表单逻辑**：输入值管理、校验、提交状态、错误提示。
3. **窗口尺寸、滚动位置、可见性监听**：这些逻辑和浏览器环境有关，和业务组件本身无关。
4. **缓存、节流、防抖、轮询**：这类逻辑经常需要在多个页面复用。
5. **订阅类逻辑**：比如 WebSocket、事件总线、媒体查询监听等。

### 不适合强行抽 Hook 的情况

- 只是某个组件里的一次性逻辑，没有复用价值。
- 抽出来后参数过多、返回值过乱，反而更难懂。
- 只是想“把代码搬出去”，但逻辑边界并不清晰。

### 判断口诀

你可以问自己三个问题：

- 这段逻辑是否和 UI 渲染解耦？
- 是否在多个地方重复出现？
- 是否可以通过输入参数改变行为，通过返回值提供能力？

如果答案大多是“是”，那就值得抽成自定义 Hook。

### 小例子

两个页面都需要“请求用户列表 + 管理 loading/error/data”，与其各写一遍，不如抽成 `useUsers`：

```jsx
function useUsers() {
  const [data, setData] = React.useState([]);
  const [loading, setLoading] = React.useState(false);

  React.useEffect(() => {
    setLoading(true);
    fetch('/api/users')
      .then(res => res.json())
      .then(setData)
      .finally(() => setLoading(false));
  }, []);

  return { data, loading };
}
```

页面里只管展示，不必重复写请求流程。

### 老赵提醒你别踩坑

别看到重复代码就急着抽 Hook。**重复的是“代码长相”，还是“逻辑本质”**，这两件事不一样。真正值得抽的是逻辑本质相同、只是参数不同的部分。

---

## 二、自定义 Hook 的命名规则与基本结构是什么？

### 命名规则：必须以 `use` 开头

这是 React Hooks 的约定，也是规则。只要你写的是 Hook，就应该以 `use` 开头，比如：

- `useFetch`
- `useForm`
- `useWindowSize`
- `useLocalStorage`

这样做有两个好处：

1. 一眼就能看出它是 Hook；
2. ESLint 能识别它，检查 Hook 规则是否正确。

### 基本结构：本质上是“可复用的函数”

自定义 Hook 本质上就是一个普通函数，只不过它内部可以调用别的 Hook。

```jsx
function useSomething() {
  const [state, setState] = React.useState();
  React.useEffect(() => {}, []);
  return {};
}
```

它和组件最大的区别在于：

- **组件负责返回 UI**
- **自定义 Hook 负责返回状态和行为**

也就是说，Hook 不直接渲染页面，而是“提供能力”。

### 最常见的结构

通常会包含这几部分：

1. 初始化状态：`useState`
2. 副作用处理：`useEffect`
3. 需要保留的引用：`useRef`
4. 对外暴露的方法：`useCallback`
5. 最终返回结果：对象或数组

### 小例子

```jsx
function useToggle(initial = false) {
  const [value, setValue] = React.useState(initial);

  const toggle = React.useCallback(() => {
    setValue(v => !v);
  }, []);

  return [value, toggle];
}
```

调用时很直接：

```jsx
const [open, toggleOpen] = useToggle(false);
```

这个 Hook 很简单，但已经具备了自定义 Hook 的基本形态：内部管理状态，向外暴露能力。

### 老赵提醒你别踩坑

自定义 Hook 里也要遵守 Hook 规则：**只能在顶层调用，不能写在条件判断、循环、普通函数内部**。别因为自己写的是函数，就把 Hook 规则放飞了。

---

## 三、从页面里的重复逻辑，怎么一步步抽成自定义 Hook？

很多人一上来就想写“完美 Hook”，结果把自己绕晕。赵老师建议你别急，**先从页面中的重复逻辑出发，按步骤拆**，这样最稳。

### 第一步：先找重复点

假设有两个页面：

- 商品列表页：打开页面要请求商品列表，管理 `loading`、`error`、`data`
- 订单列表页：打开页面也要请求列表，管理同样三套状态

它们真正重复的不是页面 UI，而是“请求流程”：

- 先设置 loading
- 请求接口
- 成功后设置 data
- 失败后设置 error
- 最后关闭 loading

这就是抽象入口。

### 第二步：拆输入和输出

抽 Hook 之前，先回答两个问题：

- 调用方需要传什么？
- 调用方需要拿到什么？

以请求逻辑为例，输入可能是：

- `url`
- `immediate`
- `transformData`
- `onSuccess`
- `onError`

输出可能是：

- `data`
- `loading`
- `error`
- `run`
- `reset`

这样一看，接口边界就清楚了。

### 第三步：把副作用封进去

请求、监听、订阅、定时器这些都属于副作用，应该集中在 Hook 内部处理。组件只负责触发和展示，不要让组件到处散落副作用代码。

### 第四步：把行为方法暴露出去

如果这个 Hook 需要手动刷新、重置、重新订阅，就把这些方法通过 `useCallback` 返回出去。这样调用方拿到的不是一堆内部细节，而是一组稳定能力。

### 一个完整的抽象过程示例

先看页面里散落的逻辑：

```jsx
function UserListPage() {
  const [data, setData] = React.useState([]);
  const [loading, setLoading] = React.useState(false);
  const [error, setError] = React.useState(null);

  React.useEffect(() => {
    setLoading(true);
    fetch('/api/users')
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <p>加载中...</p>;
  if (error) return <p>出错了</p>;

  return <ul>{data.map(user => <li key={user.id}>{user.name}</li>)}</ul>;
}
```

如果订单页也这么写一遍，重复就出来了。现在我们开始抽：

#### 1）识别重复点
重复的是请求状态管理，不是列表展示本身。

#### 2）拆输入输出
输入：接口地址、是否自动请求  
输出：数据、状态、重新请求方法

#### 3）封装副作用
把请求过程和状态更新都放进 Hook。

#### 4）形成复用模块

```jsx
function useRequest(url, options = {}) {
  const { immediate = true } = options;
  const [data, setData] = React.useState([]);
  const [loading, setLoading] = React.useState(false);
  const [error, setError] = React.useState(null);

  const run = React.useCallback(() => {
    setLoading(true);
    setError(null);

    return fetch(url)
      .then(res => res.json())
      .then(result => {
        setData(result);
        return result;
      })
      .catch(err => {
        setError(err);
        throw err;
      })
      .finally(() => setLoading(false));
  }, [url]);

  React.useEffect(() => {
    if (immediate) run();
  }, [immediate, run]);

  return { data, loading, error, run };
}
```

页面里就变成：

```jsx
function UserListPage() {
  const { data, loading, error } = useRequest('/api/users');

  if (loading) return <p>加载中...</p>;
  if (error) return <p>出错了</p>;

  return <ul>{data.map(user => <li key={user.id}>{user.name}</li>)}</ul>;
}
```

这样一来，页面只保留展示逻辑，重复请求逻辑就被抽走了。

### 老赵提醒你别踩坑

抽 Hook 时，**不要把组件渲染本身也塞进去**。Hook 的职责是提供数据和行为，不是替你返回一整套 JSX。还有，别一个 Hook 里什么都干，越大越难维护，最后就失去复用意义了。

---

## 四、如何把状态、effect、ref、callback 组合成一个 Hook？

一个成熟的 Hook 往往不是只包一层 `useState`，而是把多个能力组合起来。你可以把它理解成：**一个 Hook 不是一个点，而是一条完整的工作链。**

### 1）状态：记录业务过程

状态用来描述“当前处于什么状态”。比如请求 Hook 常见状态：

- `data`
- `loading`
- `error`

状态是给 UI 看的，它决定页面上显示“转圈中”“空状态”“错误提示”还是“正常列表”。

### 2）effect：处理副作用

`useEffect` 负责在合适时机执行副作用，比如：

- 发请求
- 订阅事件
- 监听窗口变化
- 清理资源

它把“渲染”和“外部世界同步”分开，这是 Hook 里非常重要的一层边界。

### 3）ref：保存不会触发重渲染的值

`useRef` 很适合保存这些东西：

- 定时器 ID
- 最新请求标记
- 上一次的值
- DOM 引用

它的特点是：**值变了，但不需要重新渲染**。有些数据只是为了逻辑判断，不需要展示在界面上，这时候用 `ref` 就很合适。

### 4）callback：稳定行为引用

`useCallback` 适合把对外暴露的方法包起来，避免每次渲染都创建新函数。尤其当这个函数会传给子组件或依赖别的 Hook 时，稳定性就很重要。

如果不处理，函数引用每次都变，子组件可能会重复渲染，effect 也可能被反复触发，逻辑会很吵。

### 一个完整思路：请求 Hook

```jsx
function useRequest(url) {
  const [data, setData] = React.useState(null);
  const [loading, setLoading] = React.useState(false);
  const [error, setError] = React.useState(null);
  const latestUrlRef = React.useRef(url);

  const run = React.useCallback(() => {
    setLoading(true);
    setError(null);
    latestUrlRef.current = url;

    fetch(url)
      .then(res => res.json())
      .then(result => {
        if (latestUrlRef.current === url) setData(result);
      })
      .catch(err => setError(err))
      .finally(() => setLoading(false));
  }, [url]);

  React.useEffect(() => {
    run();
  }, [run]);

  return { data, loading, error, run };
}
```

这里可以看到一个典型组合：

- `useState` 管状态
- `useRef` 记住当前请求对应的 URL
- `useCallback` 固定 `run`
- `useEffect` 负责自动触发请求

这就是自定义 Hook 的核心价值：**把容易散开的逻辑收拢成一套稳定能力**。

### 小例子

商品详情页和用户详情页都可以复用 `useRequest`，只要传入不同 URL 就行。页面层只负责展示数据，不必重复写请求、loading 和错误处理。

### 老赵提醒你别踩坑

别把所有逻辑都塞进一个 Hook 里。**一个自定义 Hook 最好只解决一个明确问题**。如果一个 Hook 既要请求、又要分页、又要搜索、又要缓存，最后通常会变成巨无霸 Hook，难维护也难测试。

---

## 五、抽象时如何设计输入参数与返回值？

很多人写自定义 Hook，问题不在实现，而在**不知道怎么设计 API**。这部分决定了这个 Hook 好不好用。

### 输入参数怎么设计？

原则只有一个：**让调用方通过参数控制行为，不要把业务写死在 Hook 内部。**

常见参数类型有：

- 基础参数：`url`、`initialValue`
- 配置对象：`{ immediate, cacheKey, onSuccess }`
- 回调函数：`onError`、`transformData`

一般来说，**参数少的时候用位置参数，参数多的时候用对象参数**。对象参数更清晰，也方便扩展。

比如请求 Hook，不只是“给个 URL 就完了”，还可能要支持：

- 是否立即请求
- 成功后做额外处理
- 是否缓存结果
- 是否手动触发

这时用对象参数更自然。

### 返回值怎么设计？

返回值要围绕“调用方下一步需要什么”来设计。通常有两种形式：

#### 1）返回数组
适合和 `useState` 风格接近的简单场景。

```jsx
const [value, setValue] = useSomething();
```

#### 2）返回对象
适合字段较多、语义较清晰的场景。

```jsx
const { data, loading, error, reload } = useRequest();
```

一般来说，**复杂 Hook 优先返回对象**，因为可读性更好，不容易记错顺序。

### 设计检查清单

抽象前先问：

- 这个 Hook 的“输入”是什么？
- 输出给调用方的“能力”是什么？
- 调用方最常操作的动作是什么？
- 需不需要提供“重新执行”“重置”“取消”等方法？

这些问题想清楚了，Hook 的 API 基本就不会太差。

### 小例子

表单 Hook 可以这样设计：

```jsx
function useForm(initialValues) {
  const [values, setValues] = React.useState(initialValues);

  const handleChange = React.useCallback((name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));
  }, []);

  const reset = React.useCallback(() => {
    setValues(initialValues);
  }, [initialValues]);

  return { values, handleChange, reset };
}
```

输入是初始值，输出是表单值、修改方法和重置方法。调用方拿到后就能直接接到页面里用。

### 老赵提醒你别踩坑

返回值不要“看起来很全”，而要“刚刚好”。很多人喜欢把内部状态全暴露出去，结果调用方开始直接改内部细节，Hook 的封装价值就没了。

---

## 六、常见自定义 Hook 案例：请求、表单、窗口尺寸、缓存数据

下面看几个最实用的案例。它们几乎覆盖了日常开发里最常见的复用需求。

### 1）请求 Hook：统一处理加载、错误、刷新

适合场景：列表加载、详情加载、搜索请求。

核心能力：

- 自动请求
- 手动刷新
- 管理 `loading/error/data`
- 避免重复代码

如果一个页面里到处都是 `setLoading(true)`、`catch`、`finally`，那基本就说明请求逻辑该抽了。

### 2）表单 Hook：统一处理输入、校验、重置

适合场景：登录表单、编辑表单、筛选表单。

核心能力：

- 管理输入状态
- 统一 `onChange`
- 支持重置和提交
- 可继续扩展校验逻辑

它的好处是，表单页面不必再为每个输入框单独写一堆状态和事件处理函数。

### 3）窗口尺寸 Hook：监听浏览器变化

适合场景：响应式布局、判断移动端、控制弹窗尺寸。

核心能力：

- 初始化获取窗口大小
- 监听 `resize`
- 组件卸载时清理监听

```jsx
function useWindowSize() {
  const [size, setSize] = React.useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  React.useEffect(() => {
    const onResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };

    window.addEventListener('resize', onResize);
    return () => window.removeEventListener('resize', onResize);
  }, []);

  return size;
}
```

页面只关心“宽高是多少”，不需要关心监听和清理细节。

### 4）缓存 Hook：优先读缓存，再决定是否请求

适合场景：本地存储用户偏好、保存草稿、记住筛选条件。

核心能力：

- 启动时从缓存读取
- 值变化时同步缓存
- 提升体验，减少重复请求

```jsx
function useLocalStorage(key, defaultValue) {
  const [value, setValue] = React.useState(() => {
    const saved = localStorage.getItem(key);
    return saved ? JSON.parse(saved) : defaultValue;
  });

  React.useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}
```

这类 Hook 在做“记住我”“保存搜索条件”“编辑草稿自动保存”时特别常用。

### 小例子

搜索页的筛选条件可以用 `useLocalStorage` 保存。用户刷新页面后，筛选条件还能保留，体验会好很多。

### 老赵提醒你别踩坑

像 `window`、`localStorage` 这类浏览器 API，不要直接在顶层无脑访问。**如果你的代码可能在服务端渲染环境运行，要先判断环境是否可用**，否则会报错。

---

## 七、结尾小结：先抽“稳定模式”，再做“能力复用”

自定义 Hook 的核心，不是“把代码搬进函数里”，而是把**重复出现的稳定逻辑**提炼成可复用能力。你可以记住这条路线：

1. 找到重复逻辑；
2. 判断它是否与 UI 解耦；
3. 用 `useState` 管状态；
4. 用 `useEffect` 管副作用；
5. 用 `useRef` 保存不触发渲染的数据；
6. 用 `useCallback` 暴露稳定方法；
7. 通过输入参数控制行为，通过返回值提供能力。

如果你把这套思路吃透，后面再看 Hooks 组合思维和真实项目实战，就会轻松很多。真正厉害的 React 项目，不是每个组件都写很多逻辑，而是把逻辑拆成一个个小 Hook，再组合成更大的能力。

### 行动建议

今天你可以先做两件事：

- 找一个项目里重复最多的逻辑，尝试抽成一个自定义 Hook；
- 给这个 Hook 写清楚“输入参数是什么、输出结果是什么、最核心的方法是什么”。

写完后再检查：这个 Hook 是否真的减少了重复？调用方是否更容易理解？如果答案是肯定的，那这次抽象就是有价值的。

### 老赵提醒你别踩坑

自定义 Hook 不是“为了复用而复用”。**复用的前提是边界清晰、职责单一、接口克制**。只要记住这一点，写出来的 Hook 才会越用越顺，而不是越用越乱。

# 第10章 Hooks 组合思维：在真实项目里把多个 Hooks 串起来

## 开篇引入：为什么“会用 Hook”不等于“会做项目”

老赵见过不少同学，`useState`、`useEffect`、`useRef`、`useMemo` 都会单独写，但一到真实项目就开始乱：状态不知道放哪，副作用到处飞，缓存和计算混在一起，Context 也容易滥用。最后页面能跑，可维护性很差。

这就是“Hooks 组合思维”要解决的问题。

真实项目里，React 不是让你把所有 Hook 堆在一个组件里，而是根据业务，把**状态、副作用、缓存、共享数据**拆开，再组合起来。你要学的不是某个 Hook 的语法，而是：**在什么地方用什么 Hook，为什么这么配，怎么配得清楚、稳妥、可维护。**

Hook 不是孤立存在的，它们更像积木。单个积木不难，难的是搭出结构稳定、层次清晰、还能扩展的房子。

### 小例子

一个商品列表页，看起来只是“输入关键词，展示列表”，背后通常已经包含多个 Hook 的协作：

- `useState` 维护关键词、分页、加载状态
- `useEffect` 在条件变化时请求数据
- `useMemo` 计算过滤后的展示结果
- `useRef` 保存上一次请求信息，避免重复触发
- `useCallback` 提供稳定的查询函数
- 必要时用 `useContext` 取登录用户或权限信息

### 老赵提醒你别踩坑

别把“会写 Hook”误以为“会做项目”。  
**单个 Hook 是知识点，多个 Hook 组合起来才是项目能力。**

---

## 一、从业务需求出发：先拆状态、副作用、缓存和共享数据

老赵建议你先别急着写代码，先把业务问题翻译成四类：

1. **状态**：页面上哪些值会变？
   - 输入框内容
   - 加载状态
   - 弹窗开关
   - 当前页码、筛选条件

2. **副作用**：哪些事不是纯渲染，必须和外部系统交互？
   - 请求接口
   - 监听滚动
   - 订阅消息
   - 操作 `localStorage`

3. **缓存/计算**：哪些值可以由已有数据推导，且计算成本高？
   - 过滤后的列表
   - 排序结果
   - 汇总统计

4. **共享数据**：哪些数据需要跨组件传递？
   - 登录用户信息
   - 主题色
   - 语言环境
   - 全局权限

对应关系很直接：

- 状态用 `useState`
- 副作用用 `useEffect`
- 需要保留但不触发渲染的值用 `useRef`
- 计算结果用 `useMemo`
- 函数引用稳定性用 `useCallback`
- 跨层共享用 `useContext`
- 复用逻辑时提炼成自定义 Hook

关键不是“把所有东西都找个 Hook 装进去”，而是先看它在业务里扮演什么角色。角色分清了，结构就不会乱。

### 完整需求拆解案例：商品搜索列表页

假设要做一个“商品搜索列表页”：

- 输入关键词后查询商品
- 切换分页时重新拉取数据
- 列表只展示符合筛选条件的商品
- 页面右上角显示当前登录用户昵称
- 搜索频繁时避免重复请求
- 查询逻辑以后能复用到别的页面

按“业务需求 → 状态 → 副作用 → 缓存 → 共享数据 → 自定义 Hook”拆解就很清楚：

- **业务需求**：搜索商品、分页展示、显示用户信息、复用查询逻辑
- **状态**：`keyword`、`page`、`pageSize`、`loading`、`list`
- **副作用**：请求商品列表
- **缓存**：筛选后的展示列表、分页统计
- **共享数据**：当前登录用户信息
- **自定义 Hook**：把请求列表和查询条件管理封装成 `useProductSearch`

这样写代码时，逻辑不会乱，每一块都有归属。

### 小例子

搜索列表页常见组合是：

- 输入关键词：`useState`
- 关键词变化后请求接口：`useEffect`
- 防止重复请求：`useRef`
- 过滤后的展示数据：`useMemo`
- 把查询条件传给列表头和筛选栏：`useContext` 或父组件统一管理

### 老赵提醒你别踩坑

别一上来就问“这个地方该用哪个 Hook”，先问“这个需求属于哪类问题”。  
**先分类，再选 Hook**，这是组合思维的起点。

---

## 二、多 Hook 协同的思考顺序：先分职责，再定组合

很多初学者喜欢直接堆 Hook，结果组件内部一团乱：

- `useState` 管输入
- `useEffect` 管请求
- `useMemo` 管列表
- `useCallback` 管事件
- `useRef` 管节流
- 再加一堆 `useContext`

问题不是 Hook 多，而是**职责没分清**。

如果一个组件里什么都管，它就会变成“超级组件”：能拿数据、能请求、能过滤、能缓存、还能控制弹窗。短期省事，长期一定难维护，因为你改一个点，可能牵动一串逻辑。

### 正确顺序是什么？

老赵给你一个实战顺序：

#### 第一步：先分职责

把页面拆成几个职责块：

- 展示区
- 筛选区
- 列表区
- 弹窗区
- 数据请求区

职责一分开，你就知道哪些逻辑该放一起，哪些该拆出去。

#### 第二步：再看数据流

判断哪些状态属于当前组件，哪些该上提到父组件或 Context。

比如列表页里：

- 搜索框输入值，通常属于搜索区
- 列表数据，通常属于页面主区域
- 用户信息，来自 Context
- 弹窗开关，可能只属于局部组件

不要让所有状态都堆在最外层，也不要把状态拆得碎到看不懂。

#### 第三步：再定 Hook 组合

根据职责决定怎么组合：

- 页面级：`useState + useEffect + useMemo`
- 表单级：`useState + useRef + useCallback`
- 共享数据：`useContext + 自定义 Hook`
- 列表页：`useState + useEffect + useMemo`

先有结构，再有 Hook；不是先有 Hook，再硬拼结构。

### 小例子

一个“商品列表页”可以这样拆：

- `page`、`pageSize`、`keyword` 用 `useState`
- 关键词或页码变化触发 `useEffect` 拉数据
- `filteredGoods` 用 `useMemo`
- 防抖搜索的定时器用 `useRef`
- 点击“查询”按钮的处理函数用 `useCallback`

这不是“堆功能”，而是在“安排分工”。

### 老赵提醒你别踩坑

不要为了“看起来高级”强行组合 Hook。  
**Hook 组合不是炫技，是为了让职责各归其位。**

---

## 三、常见组合模式：状态 + 副作用、状态 + 计算、Context + 自定义 Hook

### 1）状态 + 副作用：最常见的数据请求模式

**场景**：用户修改筛选条件，页面自动重新请求列表。

思路很直接：

- `useState` 存筛选条件
- `useEffect` 监听条件变化
- 请求完成后更新列表和加载状态

副作用不是渲染的一部分，它是渲染之后对外部世界的动作。比如请求接口、更新标题、订阅事件，都属于副作用。

#### 小例子

```jsx
const [keyword, setKeyword] = useState('');
const [list, setList] = useState([]);
const [loading, setLoading] = useState(false);

useEffect(() => {
  let cancelled = false;
  setLoading(true);

  fetchGoods(keyword).then(res => {
    if (!cancelled) {
      setList(res);
      setLoading(false);
    }
  });

  return () => {
    cancelled = true;
  };
}, [keyword]);
```

这里的关键不是“会写请求”，而是理解：**状态变化触发副作用，副作用完成后再回写状态。**  
如果没有清理逻辑，旧请求可能后返回并覆盖新结果，这就是典型竞态问题。

---

### 2）状态 + 计算：把“派生数据”交给 useMemo

**场景**：原始列表很多，但页面只展示过滤后的结果。

这类场景里，原始数据和派生数据一定要分开想。

- 原始数据是你真正存的
- 派生数据是你“算出来”的

如果把派生数据也放进 `useState`，就容易重复存储、逻辑复杂，源数据变了还可能忘记同步。

更合理的方式是：

- 原始列表放 `useState`
- 过滤条件放 `useState`
- 过滤结果用 `useMemo`

#### 小例子

```jsx
const visibleList = useMemo(() => {
  return list.filter(item => item.name.includes(keyword));
}, [list, keyword]);
```

这段代码的意思很明确：只要 `list` 或 `keyword` 没变，就没必要重新过滤。

不过老赵提醒你，`useMemo` 不是随便包一下就叫优化。数据量不大、计算很轻时，过度使用反而更绕。它适合的是：**计算确实有成本，或者你需要稳定引用时。**

---

### 3）Context + 自定义 Hook：把共享数据和访问方式一起封装

**场景**：登录用户、主题、权限等数据需要多层组件共享。

常见做法是：

- `Context` 提供数据
- 自定义 Hook 封装读取和校验逻辑

#### 小例子

```jsx
const UserContext = createContext(null);

function useUser() {
  const user = useContext(UserContext);
  if (!user) throw new Error('useUser 必须在 Provider 内使用');
  return user;
}
```

这样做的好处有两个：

1. 组件不用知道 Context 的细节
2. 访问方式统一，错误更容易定位

以后如果要加权限检查、默认值处理、空值保护，也可以直接放进 `useUser()`，组件只管拿结果，不必关心底层实现。

### 老赵提醒你别踩坑

`useMemo` 不是万能性能优化器，`Context` 也不是全局状态垃圾桶。  
**组合模式的目标是职责清晰，不是把所有东西都塞进一个大组件。**

---

## 四、在页面、表单、列表、搜索、弹窗等场景里怎么组合？

### 1）页面：把页面当作“状态总控”

页面通常是组合中心，负责：

- 拉数据
- 管筛选条件
- 控制局部弹窗
- 下发共享信息

常见组合：

- `useState` 管页面状态
- `useEffect` 拉取初始化数据
- `useMemo` 处理展示数据
- `useCallback` 传递稳定事件

页面级组件更像协调者，不是“什么都自己干”的总包工头。

### 2）表单：状态多，但要避免无意义重渲染

表单常见问题是输入频繁变化，导致组件抖动。

常见组合：

- `useState` 管字段值
- `useRef` 记录上一次提交信息或表单实例
- `useCallback` 包装提交、重置、校验函数
- 复杂字段拆到子组件

比如搜索表单里，用户每输入一个字符，页面都会重新渲染。你要关注的不是“能不能渲染”，而是“有没有必要让表单外的部分一起重渲染”。没必要就拆组件、稳函数引用、减少无关更新。

### 3）列表：重点是请求、分页、筛选和缓存

常见组合：

- `useState` 管分页、筛选、排序
- `useEffect` 触发请求
- `useMemo` 缓存筛选结果
- `useRef` 保存上次请求标识，避免竞态

列表页最容易把请求逻辑、页码变化、筛选变化混在一起。可以把“查询条件”抽成一个对象，由 `useEffect` 统一监听，但要注意对象引用是否稳定，否则容易无意中重复请求。

### 4）搜索：关键词变化频繁，注意节流/防抖

常见组合：

- `useState` 管关键词
- `useRef` 保存定时器
- `useEffect` 监听关键词变化
- `useCallback` 处理搜索提交

搜索场景最怕输入过快导致接口被频繁打。防抖通常很必要：先延迟一小段时间，用户停下来后再请求。这里 `useRef` 很适合保存定时器 id，因为它不需要触发渲染，只要在多次输入之间保留住这个中间变量。

### 5）弹窗：状态简单，但要注意生命周期

常见组合：

- `useState` 控制显示隐藏
- `useEffect` 在打开/关闭时做清理
- `useRef` 记录上一次焦点元素或弹窗实例

弹窗虽然简单，实际常涉及焦点恢复、滚动锁定、监听 ESC 关闭等细节。别小看这些副作用，一旦没处理好，体验会很差。

### 小例子

一个“搜索弹窗”：

- 输入关键词：`useState`
- 输入防抖：`useRef + useEffect`
- 搜索结果列表：`useMemo`
- 打开关闭弹窗：`useState`
- 点击外部关闭：`useEffect`

可以很清楚地看到，Hook 不是孤立写的，而是围绕一个业务场景协同工作。

### 老赵提醒你别踩坑

场景越常见，越容易写得随意。  
**页面、表单、列表、搜索、弹窗，每一种都要先想“谁负责状态，谁负责副作用，谁负责计算”。**

---

## 五、如何让 Hook 组合既清晰又可维护？

### 1）保持单一职责

一个 Hook 最好只做一类事：

- 只管数据请求
- 只管表单状态
- 只管弹窗开关
- 只管共享用户信息

不要把“请求 + 过滤 + 弹窗 + 埋点”都塞进一个自定义 Hook。那样看起来像封装，后面改起来像拆炸弹。

### 2）把派生状态和源状态分开

源状态是你真正存的数据；派生状态是由它算出来的。

比如：

- 源状态：`list`、`keyword`
- 派生状态：`visibleList`

派生状态尽量用 `useMemo`，不要再用 `useState` 复制一份，否则很容易不一致。记住一句话：**能算出来的，就尽量别再存一份。**

### 3）谨慎处理依赖数组

依赖数组不是随便填几个值，而是告诉 React：**这个副作用或缓存依赖哪些外部变量。**

常见原则：

- 用到什么，就尽量写什么
- 不要故意省略依赖
- 如果依赖函数频繁变化，考虑 `useCallback`
- 如果对象/数组每次都新建，考虑拆分或稳定引用

依赖数组写不好，问题通常会出在两个方向：

- 少写了：逻辑不更新，拿到旧值
- 多写了：频繁触发，造成重复请求或重复计算

所以它不是语法细节，而是组合 Hooks 时最容易翻车的地方之一。

### 4）把可复用逻辑抽成自定义 Hook

当你发现一组 Hook 组合在多个地方重复出现，就可以抽出来：

- 请求列表
- 防抖搜索
- 弹窗焦点管理
- 权限校验

这时你复用的不只是代码，而是一套业务能力。

### 5）可执行的组合步骤：老赵给你一套现成流程

以后你遇到一个新页面，可以按下面五步走：

1. **先拆职责**：这个页面有哪些状态、哪些副作用、哪些派生值、哪些共享数据
2. **再选 Hook**：状态用 `useState`，副作用用 `useEffect`，缓存用 `useMemo`，稳定函数用 `useCallback`
3. **再组合数据流**：谁先变，谁后算，谁触发请求，谁负责展示
4. **再检查依赖**：依赖数组是否完整，函数和对象引用是否稳定
5. **最后看性能和可维护性**：有没有重复计算，组件是不是过重，逻辑是不是容易复用

### 小例子

`useSearchList` 可能内部包含：

- `useState` 管关键词、列表、加载状态
- `useEffect` 发请求
- `useMemo` 计算展示列表
- `useCallback` 提供搜索、重置方法

组件拿到这个 Hook 后，只负责渲染，不再关心细节。页面代码会短很多，职责也更集中。

### 老赵提醒你别踩坑

自定义 Hook 不是“把代码搬家”。  
**真正好的抽象，是把重复的行为模式提炼出来，而不是把一堆变量简单封装。**

---

## 六、一个完整的实战思路：把组合过程写成项目里的动作

老赵再把前面的思路串成一条线，你以后写真实项目就可以照着做。

### 需求：做一个“商品搜索管理页”

这个页面要实现：

- 顶部显示当前用户信息
- 中间有搜索框和筛选项
- 下方展示商品列表
- 支持分页、搜索、防抖
- 支持弹窗编辑商品
- 查询逻辑可复用到别的列表页

### 组合过程

#### 1. 先拆职责
- 用户信息：共享数据
- 搜索条件：页面状态
- 列表请求：副作用
- 展示筛选：派生计算
- 弹窗编辑：局部状态

#### 2. 再选 Hook
- `useContext` 获取用户信息
- `useState` 管搜索条件、分页、弹窗状态
- `useEffect` 负责拉取商品列表
- `useRef` 保存防抖定时器和请求标识
- `useMemo` 计算最终展示列表
- `useCallback` 处理搜索、重置、打开弹窗
- 自定义 Hook `useProductSearch` 封装列表请求

#### 3. 再串数据流
- 用户输入关键词
- `useState` 更新关键词
- `useEffect` 监听关键词和分页
- `useRef` 控制防抖与竞态
- 请求返回后更新列表状态
- `useMemo` 计算最终展示结果
- 列表项点击后打开弹窗编辑

#### 4. 再检查性能和依赖
- 防止搜索每敲一个字都打接口
- 防止旧请求覆盖新请求
- 防止无关组件跟着重渲染
- 防止依赖数组漏写导致数据不同步

### 小例子

如果把“商品搜索管理页”拆成组件，大致会是：

- `PageHeader`：显示用户信息，依赖 `useContext`
- `SearchBar`：管理输入和提交，依赖 `useState`、`useCallback`
- `GoodsTable`：展示列表，依赖 `useMemo`
- `EditModal`：控制弹窗开关，依赖 `useState`、`useEffect`
- `useProductSearch`：封装查询逻辑，内部使用 `useEffect`、`useRef`

这样一来，每个组件的职责都很清楚。后面你想改搜索逻辑，只改 Hook；想改展示样式，只改表格；想换用户信息来源，只改 Context 层。维护成本会低很多。

### 老赵提醒你别踩坑

别让一个组件承担过多职责，也别为了“复用”把所有东西都抽得看不懂。  
**真实项目里最值钱的不是代码最少，而是结构最稳、后面最好改。**

---

## 结尾小结：组合思维的核心，是让 Hook 为业务服务

这一章你要记住的，不是某个具体 Hook 怎么写，而是组合思维的核心方法：

1. **先按业务拆问题**：状态、副作用、缓存、共享数据  
2. **再按职责选 Hook**：谁管什么，就让谁负责  
3. **用常见模式组织代码**：状态 + 副作用、状态 + 计算、Context + 自定义 Hook  
4. **在真实场景中落地**：页面、表单、列表、搜索、弹窗都能套这个思路  
5. **追求清晰和可维护**：少耦合、少重复、少滥用

老赵建议你下一次写项目时，不要先敲代码，先写一张小纸条：

- 这个页面有哪些状态？
- 哪些是副作用？
- 哪些是派生值？
- 哪些数据要共享？
- 哪些逻辑可以抽成 Hook？

你会发现，Hook 一旦按职责组合起来，React 项目会顺手很多。

**真正的高手，不是会堆 Hook，而是会把 Hook 串得明明白白。**

# 第11章 常见 Hooks 错误与修复：从能跑到写对

很多同学学 Hooks 时，最常见的状态就是：**代码能跑，但不稳；页面能出结果，但一改就乱；本地没报错，上线后问题一堆**。老赵跟你说，Hooks 真正拉开差距的地方，不是会不会写 `useState`、`useEffect`，而是出问题时能不能快速判断：**这是依赖数组的问题，还是闭包的问题，还是状态拆分和副作用设计的问题。**

这一章不讲花活，专门讲“常见坑”和“修法”。学会以后，你写 Hooks 会更像在做工程，而不是在撞运气。老赵的思路也很简单：**先看现象，再看依赖，再看闭包，再看状态流，最后再谈优化。**

---

## 11.1 依赖数组错了，会发生什么？

依赖数组是 Hooks 里最容易写错的地方。写得不对，轻则数据不同步，重则重复请求、循环渲染，页面直接卡死。

### 1）依赖数组为什么这么关键？

`useEffect`、`useMemo`、`useCallback` 都可能依赖外部变量。依赖数组的作用，就是告诉 React：**什么时候需要重新执行这段逻辑**。  
漏写依赖，函数可能拿到旧值；乱写依赖，组件可能频繁重新执行。说白了，依赖数组就是这段逻辑的“触发条件清单”，少了会不同步，多了会过度触发。

### 2）常见错误：循环渲染、重复请求、数据不同步

- **循环渲染**：`useEffect` 里修改了依赖本身，导致不断触发
- **重复请求**：依赖每次渲染都变，接口不停重发
- **数据不同步**：接口返回了新数据，但 effect 没重新执行

这类问题最麻烦的地方在于，它们很多时候不是“报错”，而是“逻辑不对”。

### 3）小例子：最小复现与修复

#### 错误写法：把会变化的 state 再写回去

```jsx
function Demo() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setCount(count + 1);
  }, [count]);

  return <p>{count}</p>;
}
```

这段代码会形成循环：`count` 变了，effect 执行；effect 里又改 `count`。如果目标只是首次进入页面加一次，应该把依赖改空，或者把初始化逻辑挪到更合适的位置。

#### 修复写法：只在初始化时执行

```jsx
function Demo() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setCount(1);
  }, []);

  return <p>{count}</p>;
}
```

再比如这个：

```jsx
useEffect(() => {
  fetchList({ page, keyword });
}, [{ page, keyword }]);
```

对象每次渲染都会重新创建，依赖永远“变了”，effect 也会一直跑。更合理的写法是：

```jsx
useEffect(() => {
  fetchList({ page, keyword });
}, [page, keyword]);
```

### 4）判断标准

你可以问自己三个问题：

1. 这段副作用到底依赖谁？
2. 依赖变了以后，是否真的应该重新执行？
3. effect 里有没有反过来修改依赖项？

有一个说不清，依赖数组大概率还没想明白。

### 5）修复思路

- 把真正参与计算和副作用触发的变量列出来
- 避免把临时对象、临时函数直接塞进依赖数组
- 依赖函数时，先想它是否需要稳定引用
- 如果 effect 里又修改了依赖，先检查是不是“自我触发”
- 如果只是想拿到最新值，不一定非要把它放进依赖数组，先看是否该改成 `useRef` 或函数式更新

### 老赵提醒你别踩坑

**别把“我希望它重新执行”理解成“我把所有变量都塞进依赖数组”。**  
依赖数组不是越全越好，而是要和副作用的真实输入一致。写依赖不是拍脑袋，是在描述业务关系。还有一点很重要：**先确认逻辑和数据流，再决定是不是 Hooks 的问题**，别一出错就怪 React。

---

## 11.2 闭包旧值问题：为什么异步回调总拿到老数据？

闭包是 Hooks 的经典坑。函数组件里每一次函数执行，都会“记住”当次渲染的变量值。  
所以一旦进入定时器、Promise、事件监听这类异步场景，回调里拿到的可能不是最新状态，而是**旧闭包里的旧值**。

### 1）问题是怎么来的？

函数组件每次渲染都会重新执行，形成新的作用域。某次渲染里创建的回调，会保存当时的状态快照。后面状态更新了，回调不一定自动更新。  
所以同样一段逻辑，写在普通函数里和写在组件里，表现可能不一样。

### 2）典型场景

- `setTimeout` 里读到旧 state
- `setInterval` 一直累加错误
- 异步请求返回时，状态已经切换，但回调还在用旧参数
- 事件监听函数一直拿旧值

共同点是：**回调执行的时机晚于创建时机**，就容易拿旧数据。

### 3）小例子：最小复现与修复

#### 错误写法：定时器里读到了旧值

```jsx
function Demo() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setTimeout(() => {
      console.log(count);
    }, 1000);
  };

  return <button onClick={handleClick}>打印</button>;
}
```

如果点击后立刻把 `count` 改了，1 秒后打印出来的还是旧值。原因就是闭包拿的是当次渲染时的 `count`。

#### 修复写法 1：函数式更新

```jsx
function Demo() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setCount(prev => prev + 1);
    }, 1000);

    return () => clearInterval(timer);
  }, []);

  return <p>{count}</p>;
}
```

这里用函数式更新拿“上一次状态”，就不会被旧闭包卡住。

#### 修复写法 2：把最新值放进 ref

```jsx
function Demo() {
  const [keyword, setKeyword] = useState("");
  const latestKeyword = useRef(keyword);

  useEffect(() => {
    latestKeyword.current = keyword;
  }, [keyword]);

  const handleSearch = () => {
    setTimeout(() => {
      console.log(latestKeyword.current);
    }, 1000);
  };

  return <button onClick={handleSearch}>搜索</button>;
}
```

如果异步回调里需要读取最新值，但这个值又不该驱动重渲染，可以考虑 `useRef`。

### 4）修复思路

遇到异步回调时，先判断：

- 这段回调是不是会延迟执行？
- 执行时是不是可能已经不是当前渲染了？
- 读取的是状态还是只是一个临时值？
- 是否应该改成函数式更新？

### 老赵提醒你别踩坑

**闭包不是 bug，本身是 JavaScript 的正常行为；错的是你没意识到它会“冻结当次渲染的值”。**  
尤其是异步回调里，别想当然地以为读到的一定是最新 state。你看到的是“当时的快照”，不是“现在的现场”。

---

## 11.3 state 拆分不合理，为什么会让组件越来越乱？

很多人一开始写组件，喜欢把所有状态塞进一个对象里，或者反过来把一件事拆成很多零散 state。两种极端都会出问题。

### 1）不合理的 state 拆分

状态设计要服务于业务变化，而不是为了“看起来整齐”。

#### 常见误区
- 把完全无关的数据强行放在一个对象里，导致更新一处就要拷贝整个对象
- 把本该一起变化的状态拆得太散，结果逻辑分散、阅读困难
- 一个表单、一个弹窗、一个加载态、一个分页态，全塞在同一个组件里

状态越多，不一定越乱；真正乱的是你没有把“谁和谁一起变”想清楚。

### 2）过度使用 useEffect

`useEffect` 不是“只要有变化就写进去”的万能工具。  
很多同学会把**本来可以直接在渲染阶段计算出来的值**，硬塞进 `useEffect` 再 setState 一次，结果多了一层同步，代码更绕。

比如：

```jsx
const [fullName, setFullName] = useState("");

useEffect(() => {
  setFullName(firstName + " " + lastName);
}, [firstName, lastName]);
```

这里 `fullName` 只是派生值，不该单独存。更好的方式是直接计算：

```jsx
const fullName = `${firstName} ${lastName}`;
```

这样不仅更短，还能减少一次渲染和同步逻辑。

### 3）小例子：按变化边界拆状态

做表单时，可以按“变化边界”拆状态：

```jsx
const [form, setForm] = useState({
  name: "",
  email: ""
});
const [submitting, setSubmitting] = useState(false);
```

这里 `form` 和 `submitting` 职责不同，放一起不一定更好；但 `name` 和 `email` 作为同一个表单对象的一部分，放一起反而更容易管理。

### 4）判断标准

如果一个值能从现有 state 直接算出来，通常不要再存一份。  
如果某个 effect 只是为了“同步状态”，先想想能不能删掉它。  
如果某个状态只会在一个事件里临时使用，那就别把它升级成全局状态思维。

### 老赵提醒你别踩坑

**能计算出来的值，就别存成 state；能在事件里完成的逻辑，就别搬进 useEffect。**  
`useEffect` 是处理副作用，不是处理“懒得写计算”的。状态设计越清楚，Hooks 才越好用。

---

## 11.4 useMemo、useCallback、useRef：什么时候有用，什么时候是滥用？

这三个 Hook 很容易被过度神化。很多人一遇到性能焦虑，就先套一层 `useMemo`、`useCallback`，好像加了就高级。其实不然。

### 1）useMemo 的误用

`useMemo` 的目标是缓存**计算结果**，避免不必要的重复计算。  
但如果计算本身很轻，或者依赖每次都变，那缓存意义很小，反而增加理解成本。

### 小例子

```jsx
const filteredList = useMemo(() => {
  return list.filter(item => item.name.includes(keyword));
}, [list, keyword]);
```

如果列表很大、过滤很频繁，这很合理。  
但如果只是一个简单拼接，那就没必要强上。先把代码写直白，通常比提前优化更重要。

### 2）useCallback 的误用

`useCallback` 缓存的是**函数引用**，常用于把稳定回调传给子组件。  
但如果子组件根本不关心引用是否变化，你缓存它也没收益。

### 小例子

```jsx
const handleSubmit = useCallback(() => {
  onSubmit(formData);
}, [onSubmit, formData]);
```

当这个函数被传给 `React.memo` 子组件时，才更有价值。  
如果只是本组件内部自己用，通常没必要为了“看起来优化了”而包一层。

### 3）useRef 的误用

`useRef` 适合存：

- 不参与渲染变化的数据
- DOM 引用
- 需要跨渲染保存但又不想触发重渲染的值

但它不是“替代 state”的万能盒子。  
如果值变化应该驱动 UI 更新，你塞进 ref，页面不会自动刷新。

### 小例子

```jsx
const latestKeyword = useRef("");

useEffect(() => {
  latestKeyword.current = keyword;
}, [keyword]);
```

这里可以把最新 keyword 保存起来，供异步回调读取，但它不会触发界面刷新。

### 4）性能优化基本思路

先记住一句话：**先保证正确，再考虑性能；先定位瓶颈，再做缓存。**  
优化顺序一般是：

1. 先排除逻辑错误
2. 再观察是否有明显重复计算、重复渲染
3. 最后决定是否使用 `useMemo`、`useCallback`
4. `useRef` 用来保存“需要持久化但不需要渲染”的值

很多时候，真正拖慢页面的，不是少了一个 `useMemo`，而是状态层级设计不合理、列表太大却没做拆分、请求和渲染没有分开处理。

### 老赵提醒你别踩坑

**不要为了“避免重渲染”而滥用 Hook。**  
很多时候，真正的问题不是“渲染太多”，而是“状态和副作用设计错了”。优化不是堆 Hook，而是让数据流更合理。

---

## 11.5 怎么用调试思路定位 Hooks 问题？

Hooks 出问题时，最怕的是瞎猜。老赵建议你按“现象—边界—依赖—闭包—状态流”这条线去查。

### 1）先看现象

先明确问题属于哪类：

- 页面一直刷新？
- 请求发了很多次？
- 数据总是旧的？
- 子组件频繁重渲染？
- 点击后状态不生效？

先分类，后排查。别一上来就改代码，那样很容易越改越乱。

### 2）再查边界

看问题是在：

- 首次渲染发生
- 某个交互后发生
- 异步返回时发生
- 切换路由后发生

边界一清楚，很多问题就会缩小范围。比如只在首次渲染发生的，常见是初始化逻辑或依赖数组；只在异步返回时发生的，常见是闭包或卸载后更新状态。

### 3）重点检查依赖

把相关 Hook 的依赖数组一个个核对：

- 是否漏依赖
- 是否多依赖
- 依赖对象/函数是否每次都变

如果依赖里出现了对象或函数，先思考它们是否稳定。很多“莫名其妙反复执行”，其实都和不稳定引用有关。

### 4）检查闭包和状态流

问自己：

- 这个回调拿的是不是旧值？
- 这个状态是不是应该放在 ref？
- 这个值是不是派生值，不该单独存？
- 这个 effect 是不是在替代计算逻辑？

只要你能把状态的来源、流向、使用时机说清楚，问题往往就不难找。

### 5）小例子：接口重复请求怎么排

如果你发现接口重复请求，先别急着加防抖。  
先看：

```jsx
useEffect(() => {
  fetchData(page, keyword);
}, [page, keyword]);
```

如果 `keyword` 是每次输入都变化的值，那请求频繁是正常的；如果 `page` 在 effect 中又被修改，就可能形成循环。先判断是不是逻辑设计问题，再决定是否做节流、防抖、缓存或拆分 effect。换句话说，**先看是不是数据流写错了，再看是不是需要优化体验**。

### 老赵提醒你别踩坑

**调试 Hooks，别只盯着代码表面，要顺着“渲染—依赖—回调—状态更新”这条链路看。**  
很多 bug 不是某一行写错，而是数据流设计错了。你要查的是“为什么它会这样跑”，不是只看“这一行能不能执行”。

---

## 11.6 常见错误与修法对照表

为了方便你复习，老赵给你整理一个速查表。遇到问题时，先对号入座，再动手改。

| 问题现象 | 常见原因 | 典型修法 |
| --- | --- | --- |
| `useEffect` 反复执行、页面卡住 | 依赖数组写错，effect 修改了依赖本身 | 检查依赖是否真实需要，避免自我触发 |
| 请求重复发送 | 依赖对象/函数不稳定，或者输入变化太频繁 | 拆分依赖，必要时节流、防抖，或用 `useMemo` 稳定引用 |
| 异步回调拿到旧 state | 闭包保存了旧渲染值 | 用函数式更新，或用 `useRef` 保存最新值 |
| 页面状态不同步 | 派生值被重复存进 state，或 effect 设计过重 | 直接计算派生值，减少不必要的 `useEffect` |
| 子组件频繁重渲染 | 父组件传入的函数/对象每次都变 | 视情况使用 `useCallback`、`useMemo` |
| 想保存值但不想触发渲染 | 状态应该是“记录型数据”而不是“展示型数据” | 用 `useRef` 保存 |

---

## 11.7 本章小结：从“能跑”走向“写对”

这一章其实只想帮你建立一个意识：**Hooks 代码不是写完就算，关键是它在渲染、依赖、闭包和副作用之间是否自洽。**

你可以把排错动作记成一个简单清单：

1. 先判断问题类型：重复执行、旧值、不同步、性能差
2. 再查依赖数组：是否该依赖的没依赖，不该依赖的乱依赖
3. 再查闭包：异步回调是不是拿了旧状态
4. 再查状态设计：是不是把派生值也存成了 state
5. 最后再决定是否使用 `useMemo`、`useCallback`、`useRef`

如果你能把这一套方法养成习惯，后面无论是写表单、写列表、写搜索，还是写复杂页面，都会稳很多。  
老赵最后送你一句话：**Hooks 不是难在 API，而是难在“你是否理解每次渲染都在发生什么”。**

下一章我们就把这些排错经验继续往前接，看看怎么把 Hooks 真正组合起来，做出一个更接近真实项目的完整页面。

# 第12章 小项目实战：用 Hooks 完成一个完整任务管理应用

## 开篇引入：为什么要用一个完整项目把 Hooks 串起来

前面我们把 `useState`、`useEffect`、`useRef`、`useMemo`、`useCallback`、`useContext` 和自定义 Hook 都拆开讲过了。真正的考验，不是“我知道每个 Hook 是什么”，而是“我能不能把它们组合起来，解决一个完整业务”。

这一章，老赵带你做一个**任务管理应用**。它不花哨，但很贴近真实项目：有任务列表、筛选、搜索、新增、编辑、状态切换，还要能把数据持久化到本地。做完它，你会更直观地理解：Hooks 不是零散 API，而是一套组织页面和逻辑的方式。

这章重点不是把页面堆出来，而是看清楚：**状态怎么放、逻辑怎么拆、数据怎么流、性能怎么控**。

### 项目目标
- 展示任务列表
- 支持按关键字搜索
- 支持按状态筛选
- 支持新增、编辑、完成任务
- 支持本地缓存，刷新页面数据不丢
- 尽量减少不必要渲染
- 把重复逻辑抽成自定义 Hook

### 小例子
比如你输入“报表”，列表只显示标题里包含“报表”的任务；你把某条任务标记完成，刷新后状态仍保留。这个动作背后，其实已经包含了状态管理、派生数据、持久化和性能优化。

### 老赵提醒你别踩坑
别一上来就写组件。先拆需求，再决定哪些状态放组件里，哪些逻辑抽成 Hook。顺序反了，后面很容易越写越乱。

---

## 1. 项目需求拆解：任务列表、筛选、搜索、编辑、新增、状态持久化

这个项目的核心，其实是“任务数据的生命周期管理”。

### 1.1 任务列表
最基本的是展示任务数组，每条任务至少包含：
- `id`
- `title`
- `done`
- `createdAt`

先把数据结构想清楚很重要，因为新增、编辑、切换状态，都是围绕这份数据展开的。

### 1.2 筛选与搜索
筛选一般分三种：
- 全部
- 已完成
- 未完成

搜索则是按标题关键字过滤。

这里要注意：**筛选和搜索是派生数据，不是原始数据**。原始数据还是任务数组，展示结果只是根据条件计算出来的视图。

### 1.3 新增与编辑
新增任务就是往数组里追加一项；编辑任务是找到对应 `id` 后更新标题。实际项目里还要考虑：
- 新增后是否清空输入框
- 编辑时输入框是否回填
- 是否允许空标题提交
- 编辑状态和新增状态如何切换

这些细节决定了页面是否真能用。

### 1.4 状态持久化
为了让刷新后数据还在，需要把任务写入本地存储，并在初始化时读出来。

老赵建议你把**初始化数据**和**保存数据**分开思考：初始化负责“从哪来”，保存负责“到哪去”。这个思路会让 `useEffect` 的职责更清楚。

### 小例子
如果本地有 3 条任务，页面加载时先从缓存读取；如果缓存为空，就用默认初始列表。这样用户第一次打开也不会看到空白页。

### 老赵提醒你别踩坑
筛选结果不要直接当成“真实数据”去修改。你改的是过滤后的数组，不是源数据，最后常会出现“列表变了但原数据没变”的问题。

---

## 2. 页面结构与数据流：先搭骨架，再填逻辑

做这种项目，老赵一般建议先定页面结构，再定数据流。因为如果边写 UI 边想逻辑，很容易把状态散得到处都是。

### 2.1 页面可以拆成几个区域
- 顶部标题区
- 搜索与筛选区
- 新增/编辑表单区
- 任务列表区
- 任务统计区

这样拆的好处是，每个区域只关心自己的状态和动作，职责会清楚很多。

### 2.2 数据流怎么走
一个比较顺的流程是：
1. 页面初始化，从本地缓存读取任务
2. 用户输入搜索词或筛选条件
3. 页面根据原始任务计算出可见列表
4. 用户新增、编辑、完成任务
5. 任务变化后同步写入缓存

这里最关键的是“原始状态”和“展示状态”分离。原始状态只保存任务本体，展示状态由 `useMemo` 算出来，这样结构才不会乱。

### 小例子
比如用户先选“未完成”，再输入“前端”，最终列表会只显示“未完成且标题包含前端”的任务。这里并没有改任务本身，只是在算可见结果。

### 老赵提醒你别踩坑
页面结构没拆清楚之前，千万别急着把所有状态都往一个组件里塞。一个大组件把新增、编辑、筛选、缓存全包了，后面维护起来会很痛苦。

---

## 3. 使用 useState、useEffect、useRef、useMemo、useCallback、useContext 构建页面

这一部分是整个应用的骨架。可以理解为：**状态负责内容，副作用负责外部同步，引用负责临时保存，计算负责派生视图，共享负责跨层通信**。

### 3.1 useState：管理页面状态
`useState` 负责最直接的状态：
- 任务列表 `tasks`
- 搜索词 `keyword`
- 当前筛选条件 `filter`
- 表单输入 `title`
- 编辑态 `editingId`

只要状态变化会引起界面变化，优先考虑 `useState`。比如搜索词一变，列表展示就会跟着变；编辑中的任务 id 一变，按钮文案也会变。

### 3.2 useEffect：处理副作用和持久化
`useEffect` 常用于：
- 页面初始化读取本地缓存
- 任务数据变化时写入本地缓存
- 监听外部副作用

最典型的是本地缓存同步：页面初次加载时读一次缓存，后续任务变化时再写回去。这样就形成了一个完整的数据闭环。

记住，`useEffect` 不是“专门做请求”的，它的本质是**同步 React 和外部世界**。本地存储、浏览器事件、定时器、第三方库，都属于它的工作范围。

### 3.3 useRef：保存不参与渲染的值
`useRef` 适合：
- 聚焦输入框
- 保存上一次值
- 存储不需要触发渲染的临时信息

比如新增任务后自动把焦点放到输入框里，就很适合 `useRef`。因为“聚焦”不是界面状态，但它是实际交互需求。

### 3.4 useMemo：缓存派生结果
搜索和筛选结果可以用 `useMemo` 包起来，避免每次渲染都重新计算。

它适合：
- 计算成本稍高的派生数据
- 依赖明确、结果可缓存的场景

比如先筛选状态，再按关键字过滤，最终得到可见任务列表。这个结果不是状态，而是根据原始数据算出来的。用 `useMemo`，能让计算只在依赖变化时重新执行。

### 3.5 useCallback：稳定函数引用
当我们把方法传给子组件时，比如“切换任务状态”“删除任务”，可以用 `useCallback` 保持函数引用稳定，减少不必要渲染。

要记住：`useCallback` 不是为了“让函数更快”，而是为了**让函数引用更稳定**。如果子组件用了 `React.memo`，这个稳定性就很有意义。

### 3.6 useContext：共享全局能力
如果任务列表里的某些操作要在多个层级使用，比如主题、语言、全局通知、任务操作方法，可以用 `Context` 共享。

在这个项目里，可以把任务管理相关方法放到上下文里，让深层组件不用层层传参。比如任务项组件只关心“显示什么、点按钮时怎么通知上层”，不用知道整个页面状态。

### 小例子
```jsx
const visibleTasks = useMemo(() => {
  return tasks
    .filter(t => filter === 'all' ? true : filter === 'done' ? t.done : !t.done)
    .filter(t => t.title.includes(keyword));
}, [tasks, filter, keyword]);
```

这段代码很实用：**原始数据不动，展示结果现算**。这样状态更清晰，也更不容易写乱。

### 老赵提醒你别踩坑
`useMemo` 和 `useCallback` 不是性能魔法。计算很轻、组件很小的时候，硬上这些 Hook 反而会让代码更难读。先保证正确性，再考虑优化。

---

## 4. 关键实现：把新增、编辑、切换状态串成一条业务链

真正写业务时，Hooks 不是一个个单独使用的，而是一起配合。

### 4.1 新增任务
流程通常是：
- 输入框输入标题
- 点击新增
- 校验标题是否为空
- 创建新任务对象
- 追加到任务数组
- 清空输入框
- 重新聚焦输入框

这里会用到 `useState` 管输入值，`useRef` 做聚焦，`useEffect` 或回调函数处理提交后的收尾动作。

### 4.2 编辑任务
编辑比新增多一步“回填”：
- 点击编辑
- 把当前任务标题放进输入框
- 记录正在编辑的任务 id
- 提交时走更新逻辑
- 提交后退出编辑态

这个场景很容易让人把“新增”和“编辑”写成两套重复代码。老赵建议你把提交动作抽成统一处理：有 `editingId` 就更新，没有就新增。

### 4.3 切换状态
切换完成状态其实就是更新某条任务的 `done` 字段。这里最常见的问题，是函数里拿到的任务列表不是最新值。

所以建议你使用函数式更新：
```jsx
setTasks(prev =>
  prev.map(task =>
    task.id === id ? { ...task, done: !task.done } : task
  )
);
```
这样就算当前回调来自旧闭包，也能基于最新状态做修改。

### 小例子
如果你在任务列表里点“完成”，页面只需要改一项数据，别把整个流程写得很重。一个清晰的 `map` 更新，往往比到处复制状态更可靠。

### 老赵提醒你别踩坑
编辑态和新增态不要混成一团。很多人写到后面发现：点“新增”也在回填旧值，点“编辑”又清不掉状态，这就是状态边界没划清。

---

## 5. 一处性能优化：让筛选与列表渲染更稳

真实项目里，性能优化不一定要很复杂，但一定要有思路。

### 5.1 为什么这里值得优化
当任务变多后，搜索、筛选和列表渲染都会变重。尤其是输入框每敲一个字，列表都跟着重新计算，如果逻辑写得散，就会出现明显的重复渲染。

### 5.2 优化思路
- 用 `useMemo` 缓存筛选结果
- 用 `useCallback` 稳定任务操作函数
- 把列表项拆成子组件，并配合 `React.memo`
- 把不参与渲染的临时值放进 `useRef`

比如可见任务列表可以只在 `tasks`、`filter`、`keyword` 变化时重新计算，而不是每次父组件渲染都重算一遍。

### 小例子
如果搜索框输入时，页面上只有搜索条件变化，任务项组件却还在频繁重绘，那就要检查是不是给子组件传了新的对象或新函数引用。

### 老赵提醒你别踩坑
别一看到“性能”就乱加缓存。缓存本身也有成本。先看是否存在明显重复计算、频繁重渲染和引用不稳定，再决定要不要优化。

---

## 6. 一处常见错误修复：闭包、依赖数组和重复渲染

这一节是实战里最容易翻车的地方，老赵专门给你掰开说。

### 6.1 闭包导致的旧状态问题
函数组件每次渲染都会生成新的变量和函数快照。如果你在定时器、异步回调或事件处理里用了旧函数，很可能读到的是旧任务列表。

解决方法通常有两个：
- 依赖数组写全，保证函数更新
- 使用函数式更新，避免直接依赖旧状态

### 6.2 依赖数组写错
比如本地缓存同步：
```jsx
useEffect(() => {
  localStorage.setItem('tasks', JSON.stringify(tasks));
}, []);
```
这就错了。因为它只在第一次执行，后面任务变了也不会写入缓存。正确做法是把 `tasks` 放进依赖数组。

同样地，`useMemo` 和 `useCallback` 里如果漏写依赖，也会拿到旧值，出现“界面看着变了，逻辑却没跟上”的问题。

### 6.3 重复渲染的常见源头
- 父组件状态太多，导致整个树一起更新
- 每次渲染都创建新对象、新数组、新函数
- 子组件没拆分，导致局部变化拖累全局

修复思路不是一味加缓存，而是先看状态边界，再看引用稳定性。

### 小例子
如果任务标题输入框一变，整个任务列表都重新渲染，先别急着怪 `useMemo`。先检查是不是列表组件跟输入状态绑在一个父组件里了，或者传参时每次都生成新对象。

### 老赵提醒你别踩坑
依赖数组不是“可选项”。漏一个依赖，轻则数据不更新，重则缓存错乱、闭包拿旧值。写 Hooks 时，依赖要老老实实核对。

---

## 7. 真实项目中的 Hooks 组合思维：不是单个 Hook，而是一套配合

Hooks 在真实项目里，最有价值的地方就是“组合”。

### 7.1 常见组合方式
- `useState + useMemo`：管理输入并计算筛选结果
- `useEffect + useRef`：初始化数据、持久化缓存、自动聚焦
- `useCallback + React.memo`：稳定函数引用，减少子组件重渲染
- `useContext + 自定义 Hook`：集中管理任务操作能力

### 7.2 组合不是堆砌
一个成熟的页面，不是把所有 Hook 都摆上来，而是让它们各司其职：
- 状态放在该放的地方
- 副作用集中处理
- 计算结果按需派生
- 可复用逻辑抽成 Hook
- 跨层数据用 Context 传递

### 7.3 一个真实落地的思路
比如“任务快捷操作”这个需求：
- 搜索框需要 `useRef` 做聚焦
- 搜索结果需要 `useMemo`
- 任务列表项操作需要 `useCallback`
- 全局通知可以放 `Context`
- 快捷键逻辑抽成 `useHotkeys`

你会发现，单个需求背后往往是多个 Hook 协作完成的。

### 小例子
按 `Ctrl+K` 聚焦搜索框，输入关键词后自动过滤列表，这个看似简单的功能，其实就串起了 `useRef`、`useState`、`useMemo` 和自定义 Hook。

### 老赵提醒你别踩坑
Hooks 组合的关键，不是“用得多”，而是“边界清”。每个 Hook 只做自己擅长的事，页面才会稳，后面才好扩展。

---

## 8. 项目复盘：把每个 Hook 放回它该在的位置

做完这个任务管理应用，你会发现 Hooks 不只是“替代 class 生命周期”，而是在重塑架构思路。

### 8.1 从“生命周期思维”转向“状态与副作用分离”
- `useState` 管状态
- `useEffect` 管副作用
- `useRef` 管不触发渲染的数据
- `useMemo` 管派生结果
- `useCallback` 管稳定引用
- `useContext` 管跨层共享

这样拆开后，页面不再靠一个巨型组件硬撑。每个 Hook 都有清晰职责，代码自然更容易维护。

### 8.2 从“写页面”转向“组合能力”
真实项目里，我们几乎不会只用一个 Hook。常见模式是：
- `useState + useMemo`：管理输入并计算筛选结果
- `useEffect + useRef`：初始化、缓存、聚焦输入
- `useCallback + React.memo`：稳定传参，减少子组件渲染
- `useContext + 自定义 Hook`：集中管理业务能力

组合的本质，不是把 Hook 堆在一起，而是让它们各自做自己最擅长的事。你一旦有了这个意识，写复杂页面就不容易慌。

### 8.3 你应该建立的检查清单
每次做 Hooks 页面时，可以问自己：
1. 这个状态是否真的需要渲染？
2. 这段逻辑是不是副作用？
3. 依赖数组是否完整？
4. 是否存在闭包拿旧值？
5. 是否有重复逻辑可以抽成 Hook？
6. 是否有不必要的子组件重渲染？

这个清单很好用。老赵建议你每次做练习项目，都拿它过一遍。

### 小例子
任务页可以拆成：
- 顶部搜索栏
- 新增表单
- 筛选工具条
- 任务列表
- 任务项

每个组件只拿自己需要的数据和方法，整体会更清爽。搜索栏不关心任务项的展示细节，任务项也不需要知道整个筛选状态，这就是职责分离。

### 老赵提醒你别踩坑
Hooks 的高级感，不在于你用了多少个 Hook，而在于你有没有把状态、副作用、计算、共享、复用这几件事分清楚。

---

## 9. 项目知识映射表：把 Hooks 和功能模块对上号

这一页你可以当作复习清单。老赵建议你看完项目后，把每个 Hook 在项目中的位置再过一遍。

| Hook / 机制 | 项目中的位置 | 作用 |
|---|---|---|
| `useState` | 任务列表、搜索词、筛选条件、编辑态、输入框 | 管理页面核心状态 |
| `useEffect` | 初始化读取缓存、任务变化写缓存 | 处理副作用与持久化 |
| `useRef` | 输入框聚焦、临时值保存 | 保存不触发渲染的数据 |
| `useMemo` | 过滤后的可见任务列表 | 缓存派生结果，减少重复计算 |
| `useCallback` | 新增、编辑、切换状态、删除等操作函数 | 稳定函数引用，减少子组件更新 |
| `useContext` | 全局任务操作、通知、主题等 | 跨层共享能力，避免层层传参 |
| 自定义 Hook | `useTasksManager`、`useInput`、`useLocalStorageState`、`useHotkeys` | 抽象复用逻辑，让页面更清晰 |

### 小例子
你以后看到一个任务页，就可以先问：搜索词是谁管的？缓存是谁写的？列表结果是谁算的？这几个问题一拆开，Hooks 的架构感就出来了。

### 老赵提醒你别踩坑
复习时不要只记 API 名字。一定要把 Hook 和“它在项目里解决了什么问题”对应起来，这才是真正掌握。

---

## 结尾小结：把 Hooks 真正用进项目里

这一章我们用一个完整任务管理应用，把前面所有知识串成了一条线：

- 先拆需求，明确状态和派生数据
- 用 `useState` 管基础状态
- 用 `useEffect` 处理初始化和持久化
- 用 `useRef` 处理焦点与临时值
- 用 `useMemo` 和 `useCallback` 做基础性能优化
- 用 `useContext` 做跨层共享
- 再把重复逻辑抽成自定义 Hook
- 最后用组合思维重构整个页面结构

如果你能独立写出这个项目，说明你已经不只是“会用 Hooks”，而是开始具备“用 Hooks 组织应用”的能力了。

### 行动建议
接下来你可以做两件事：
1. 把这个任务管理应用自己复写一遍，尽量不看答案
2. 再加两个功能：任务排序、任务批量删除，继续练习 Hooks 组合思维

老赵最后送你一句话：**Hooks 不是背 API，Hooks 是学会把页面拆成可管理的能力单元。**

---

更多内容请访问：[https://tutor.lao-zhao.com/](https://tutor.lao-zhao.com/)
