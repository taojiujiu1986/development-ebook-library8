# 全栈老赵讲 用 React 快速做一个 AI 聊天网页

<!-- PAGEBREAK -->

![作者介绍图](05_full_book_draft_assets/asset-8534aff2fa.png)

<!-- PAGEBREAK -->

## 目录

- 第1章 先把项目跑起来——环境准备与 React 项目初始化
- 第2章 搭出聊天页面骨架——页面布局与基础对话 UI
- 第3章 把消息管起来——对话消息状态管理
- 第4章 做出发送动作——输入框交互与消息发送流程
- 第5章 接上大模型——接口请求封装与 API 调用
- 第6章 让回复像聊天一样出现——流式响应与打字效果
- 第7章 让内容更像真实聊天——Markdown 渲染与代码高亮
- 第8章 把聊天记录存住——消息历史与本地存储
- 第9章 别让用户等得太慌——错误处理与限流提示
- 第10章 让聊天更好用——滚动、自动聚焦与体验优化
- 第11章 手机上也能顺手用——移动端适配与响应式优化
- 第12章 上线给别人看——部署到 Vercel 与常见问题排查

# 第1章 先把项目跑起来——环境准备与 React 项目初始化

做 AI 聊天网页，第一步不是急着接大模型接口，而是先搭好一个**能跑、能改、能扩展的 React 项目**。后面无论是消息列表、输入框、流式输出，还是 Markdown 渲染、代码高亮、历史记录和部署上线，都要落在这个基础上。

这一章的目标只有一个：**尽快把项目启动成功，并在浏览器里看到首屏。**  
只要这一步通了，后面所有功能才有地方落地。

---

## 一、先把开发环境准备好

### 1. 只检查三样东西

新手最容易卡住的地方，不是代码，而是环境。这里先确认三项就够了：

- **Node.js**：运行前端工具链
- **包管理器**：npm / pnpm / yarn 任选一个，新手先用 npm
- **编辑器和浏览器**：推荐 VS Code + Chrome/Edge

先别研究太多原理，重点是保证后面能正常安装、启动和预览。

### 2. 快速验证环境

打开终端，执行：

```bash
node -v
npm -v
```

能正常输出版本号，就说明 Node 和 npm 可用。  
本书后续建议使用 **Node 18+**，版本太旧最容易在安装依赖时出问题。

### 3. 截图说明思路

建议统一按这个模板准备截图：

- **截图目的**：证明环境已就绪
- **截图内容**：终端执行 `node -v`、`npm -v`
- **标注重点**：版本号能正常显示
- **读者关注点**：不要出现“命令找不到”或空输出

### 4. 常见坑

- **Node 版本太旧**：建议升级到 18 或更高
- **终端识别不到 Node**：通常是环境变量问题，重开终端或重新安装
- **包管理器混用**：同一个项目里别一会儿 npm、一会儿 pnpm，容易把锁文件弄乱

---

## 二、用 Vite 初始化 React 项目

### 1. 为什么直接选 Vite

本书追求的是“快”。Vite 创建 React 项目速度快、启动快、热更新也顺手，非常适合从 0 到 1 做 AI 聊天网页。先把项目做出来，最重要。

### 2. 初始化项目并进入目录

在准备好的文件夹里执行：

```bash
npm create vite@latest ai-chat-web -- --template react
cd ai-chat-web
npm install
npm run dev
```

这四步是一条连续操作链：创建项目、进入目录、安装依赖、启动开发服务器。

启动后，终端会给出本地地址，比如：

```bash
http://localhost:5173/
```

打开浏览器访问它，如果看到 Vite 默认页面，说明项目已经跑起来了。

### 3. 先认识最核心的目录

初始化后，先记住这几个文件就行：

```bash
ai-chat-web/
├─ src/
│  ├─ App.jsx
│  ├─ main.jsx
│  └─ assets/
├─ public/
├─ index.html
├─ package.json
└─ vite.config.js
```

重点看三个地方：

- **App.jsx**：后面主要写聊天页面
- **main.jsx**：项目入口，把 React 挂到页面上
- **package.json**：记录依赖和脚本命令

### 4. 截图说明思路

建议准备两张图：

- **截图目的**：证明项目创建成功
- **截图内容**：终端执行 `npm create vite@latest` 的结果，以及目录结构
- **标注重点**：能看到项目名 `ai-chat-web`
- **读者关注点**：初始化命令没有报错，目录已经生成

### 5. 常见坑

- **项目名输错**：建议直接复制命令，少手打
- **目录没切对**：`npm run dev` 必须在项目根目录执行
- **初始化后没装依赖**：`npm install` 不做，项目起不来

---

## 三、安装后续要用的核心依赖

### 1. 先把常用包装上

为了后面少返工，这一章先把核心依赖装好：

```bash
npm install axios marked highlight.js
```

### 2. 这几个包分别负责什么

- **axios**：封装接口请求更方便
- **marked**：把 AI 返回的 Markdown 文本渲染成网页内容
- **highlight.js**：给代码块做高亮展示

AI 聊天的返回内容通常不只是纯文本，还会包含标题、列表、代码块，所以这几个依赖后面几乎一定会用到。

### 3. 截图说明思路

建议截图一张终端安装成功的画面：

- **截图目的**：证明依赖安装成功
- **截图内容**：`npm install` 完成后的终端信息，最好能看到 `package.json` 里新增依赖
- **标注重点**：安装无报错
- **读者关注点**：不要出现 `failed`、`ERR`、`network error`

### 4. 常见坑

- **安装慢或失败**：先检查网络，必要时切换镜像源
- **包版本冲突**：如果之前装过别的方案，建议删掉 `node_modules` 和锁文件后重新装
- **一次装太多包**：新手阶段先围绕需求安装，别把项目搞复杂

---

## 四、启动开发服务器，确认首屏正常显示

### 1. 启动命令

在项目目录下执行：

```bash
npm run dev
```

如果终端出现类似提示，就说明开发服务器启动成功：

```bash
VITE vX.X.X  ready in XXX ms
Local: http://localhost:5173/
```

### 2. 先做一个最小页面验证

把 `src/App.jsx` 改成最简单的内容，确认热更新正常：

```jsx
export default function App() {
  return (
    <div style={{ padding: 24 }}>
      <h1>AI 聊天网页项目已启动</h1>
      <p>下一步我们会把它做成真正的聊天界面。</p>
    </div>
  );
}
```

保存后，浏览器应该自动刷新并显示新内容。  
如果页面能立刻变化，说明 React 项目、热更新、浏览器预览这条链路已经打通。

### 3. 这一页要看到什么

你现在只需要确认三件事：

1. 页面能打开，不是空白页
2. 没有红色报错覆盖页面
3. 修改 `App.jsx` 后，浏览器能自动刷新

这意味着项目已经具备继续开发聊天 UI、接口请求封装、消息状态管理的基础。

### 4. 截图说明思路

建议统一准备三张图：

- **截图目的**：证明项目真正跑起来了
- **截图内容**：终端显示 `npm run dev` 成功、浏览器打开 `localhost:5173`、修改 `App.jsx` 后页面变化
- **标注重点**：开发服务器地址、页面首屏、热更新效果
- **读者关注点**：不是“建了项目”，而是“项目能运行、能刷新”

### 5. 常见坑

- **浏览器打不开地址**：检查端口号是否正确，是否被占用
- **页面没自动刷新**：确认文件已保存，且终端没有报错
- **白屏但终端正常**：打开浏览器控制台看报错，常见是 JSX 写法错误

---

## 五、本章你要完成的最小检查清单

完成这一章后，至少要确认这些事都成立：

- 能执行 `node -v` 和 `npm -v`
- 能用 Vite 创建 React 项目
- 能进入项目目录并安装依赖
- 能启动本地开发服务器
- 能在浏览器看到首屏
- 能修改 `App.jsx` 并看到页面变化

如果这些都完成了，说明你已经拿到了一个**最小可运行起点**。  
后面要做的，就是在这个稳定骨架上继续加功能：**对话 UI、接口请求封装、对话消息状态管理、流式响应、Markdown 渲染、代码高亮、消息历史、本地存储、错误提示和部署上线。**

---

## 本章小结

这一章追求的不是功能多，而是一个字：**稳**。  
AI 聊天网页最容易失败的地方，不是“不会调接口”，而是项目一开始就没跑通，后面每一步都在修环境。现在你已经把最小可运行项目搭好了，下一章就可以直接进入聊天页面布局，把输入框、消息列表和按钮先做出来。

**下一步建议：**  
先确认你已经成功打开 `http://localhost:5173/`，并把项目目录结构看一遍。只要基础站稳了，后面的 AI 接入会顺很多。

# 第2章 搭出聊天页面骨架——页面布局与基础对话 UI

这一章先不接 AI 接口，先把聊天页的“壳”搭出来。对新手来说，最容易出问题的不是接口，而是页面结构乱、消息区不滚动、输入框乱跑。**先把能展示、能交互、像聊天页的基础界面做出来**，后面再把 AI 回复接进去就顺了。

---

## 一、先规划整体布局：顶部标题、消息区、输入区

一个最基础的 AI 聊天页，通常就三块：

1. **顶部标题区**：显示页面名称或简单状态。
2. **消息展示区**：显示用户和机器人的对话。
3. **底部输入区**：输入内容、发送消息、回车发送。

最稳的结构就是：**顶部固定，中间自适应，底部固定**。后面加 API、流式输出、历史记录，都是围绕这三块扩展。

### 页面结构示意

```txt
┌──────────────────────┐
│  全栈老赵 AI Chat    │
├──────────────────────┤
│                      │
│  消息 1              │
│  消息 2              │
│  消息 3              │
│                      │
├──────────────────────┤
│ [输入框...........] [发送] │
└──────────────────────┘
```

### 截图说明思路
- 截图 1：完整页面，确认三段式布局清楚。
- 截图 2：消息较多时，消息区能单独滚动。
- 截图 3：输入框始终固定在底部。

### 常见坑
- 消息列表直接撑高页面，输入框被挤走。
- 外层容器没设高度，`height: 100%` 不生效。
- 消息区不滚动，页面越聊越长。

---

## 二、先把项目里的基础组件拆开：ChatHeader、MessageList、ChatInput

别把所有内容堆在一个文件里，最简单的拆法就是三个组件：

- **ChatHeader**：顶部标题。
- **MessageList**：消息列表。
- **ChatInput**：输入和发送。

这样做的好处很直接：结构清楚、样式好管、后面接 AI 时只改局部。现在不需要复杂封装，先让页面跑起来最重要。

### 组件结构示例

```jsx
// App.jsx
import { useState } from "react";
import ChatHeader from "./components/ChatHeader";
import MessageList from "./components/MessageList";
import ChatInput from "./components/ChatInput";

export default function App() {
  const [messages, setMessages] = useState([
    { role: "bot", content: "你好，我是你的 AI 助手。" },
    { role: "user", content: "帮我写一个登录页。" },
  ]);

  const handleSend = (text) => {
    setMessages((prev) => [...prev, { role: "user", content: text }]);
  };

  return (
    <div className="chat-page">
      <ChatHeader />
      <MessageList messages={messages} />
      <ChatInput onSend={handleSend} />
    </div>
  );
}
```

```jsx
// components/ChatHeader.jsx
export default function ChatHeader() {
  return (
    <div className="chat-header">
      <h1>全栈老赵 AI Chat</h1>
      <p>先做界面，再接大模型</p>
    </div>
  );
}
```

```jsx
// components/MessageList.jsx
export default function MessageList({ messages }) {
  return (
    <div className="message-list">
      {messages.map((msg, index) => (
        <div key={index} className={`message ${msg.role}`}>
          {msg.content}
        </div>
      ))}
    </div>
  );
}
```

```jsx
// components/ChatInput.jsx
import { useState } from "react";

export default function ChatInput({ onSend }) {
  const [value, setValue] = useState("");

  const handleSend = () => {
    const text = value.trim();
    if (!text) return;
    onSend?.(text);
    setValue("");
  };

  return (
    <div className="chat-input">
      <input
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="输入你的问题..."
        onKeyDown={(e) => {
          if (e.key === "Enter") handleSend();
        }}
      />
      <button onClick={handleSend}>发送</button>
    </div>
  );
}
```

### 常见坑
- 组件拆了，但样式还是全局乱写。
- 消息区和输入区没做布局约束。
- 输入框不是受控组件，后面不好接发送逻辑。

---

## 三、用 CSS 先做稳基础界面：先保证布局不塌

这一章先求稳定，不求花哨。先用纯 CSS 做出最小可用样式，后面再换成 Tailwind 也行。目标不是“好看”，而是“像一个真正能聊天的网页”。

### 基础 CSS 示例

```css
* {
  box-sizing: border-box;
}

html,
body,
#root {
  height: 100%;
  margin: 0;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  background: #f5f7fb;
}

.chat-page {
  height: 100%;
  display: flex;
  flex-direction: column;
  max-width: 900px;
  margin: 0 auto;
  background: #fff;
}

.chat-header {
  padding: 16px 20px;
  border-bottom: 1px solid #eee;
}

.message-list {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
  background: #f9fafc;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.message {
  max-width: 75%;
  padding: 12px 14px;
  border-radius: 14px;
  line-height: 1.6;
  word-break: break-word;
}

.message.user {
  align-self: flex-end;
  background: #1677ff;
  color: #fff;
  border-bottom-right-radius: 4px;
}

.message.bot {
  align-self: flex-start;
  background: #fff;
  color: #222;
  border: 1px solid #e8e8e8;
  border-bottom-left-radius: 4px;
}

.chat-input {
  display: flex;
  gap: 10px;
  padding: 16px;
  border-top: 1px solid #eee;
  background: #fff;
}

.chat-input input {
  flex: 1;
  height: 44px;
  border: 1px solid #ddd;
  border-radius: 10px;
  padding: 0 14px;
  outline: none;
}

.chat-input button {
  width: 88px;
  border: none;
  border-radius: 10px;
  background: #1677ff;
  color: #fff;
  cursor: pointer;
}
```

关键就三句：

- `.chat-page` 用 `flex-direction: column`
- `.message-list` 用 `flex: 1`
- `.message-list` 用 `overflow-y: auto`

只要这三处对了，聊天页的基本形态就不会跑偏。

### 截图说明思路
- 截图 1：完整页面，确认中间消息区撑满。
- 截图 2：用户和机器人消息左右区分明显。
- 截图 3：输入区固定在底部，不挤压消息区。

### 常见坑
- 忘了给 `html、body、#root` 设置 `height: 100%`。
- `message-list` 没有 `flex: 1`。
- 消息太长时没加 `word-break: break-word`，气泡被撑爆。
- 外层容器用了 `overflow: hidden`，结果消息根本滚不动。

---

## 四、把用户消息和机器人消息区分开，别做成留言板

聊天页最重要的一眼识别，就是谁在说话。最简单的方式：

- **用户消息靠右、蓝底白字**
- **机器人消息靠左、白底灰边**

这里只要把角色和样式对应上就行，不用先做头像、时间戳这些扩展功能。

### 消息数据结构建议

```jsx
const messages = [
  { role: "bot", content: "你好，我是 AI 助手。" },
  { role: "user", content: "帮我写一个 React 聊天页。" },
];
```

渲染时根据 `role` 区分样式：

```jsx
export default function MessageList({ messages }) {
  return (
    <div className="message-list">
      {messages.map((msg, index) => (
        <div key={index} className={`message ${msg.role}`}>
          {msg.content}
        </div>
      ))}
    </div>
  );
}
```

如果后面要加头像、时间戳，也建议继续沿用这个结构，别中途换格式。先把消息的“身份识别”做清楚，后面加 Markdown 渲染、代码高亮时也不会乱。

### 判断标准
不看内容，只看样式，就能分出用户消息和机器人消息，这一步就算过关。

### 常见坑
- 所有消息都用同一个 class，看起来像留言板。
- 颜色对比太弱，手机上看不清。
- 只改颜色没改 `align-self`，左右位置还是乱的。

---

## 五、把输入框做成最小闭环：能输入、能发送、能回显

输入区不能只是摆设，至少要做到：

1. 输入框能输入文字  
2. 点击发送能把内容加到消息列表里  
3. 发完后清空输入框  
4. 支持回车发送

这样你就有了最小可用闭环，后面接 AI 时只要把“机器人回复”换成接口返回结果。**这一章做到这里，已经具备“能展示”的基础形态了。**

如果想进一步提升体验，后面再加 `Shift + Enter` 换行、`Enter` 发送；本章先把最基本的跑通。

### 常见坑
- 输入框不受控，清空时界面和状态不同步。
- 没处理回车发送，体验不完整。
- 空字符串也能发送，消息区会出现空气泡。
- 点击发送后没有自动聚焦回输入框，连续聊天不顺手。

---

## 六、移动端也要能看：别只在大屏上调样式

AI 聊天网页最后大概率会被手机打开，所以这一步不要跳过。最简单的目标是：小屏幕下消息能正常显示，输入区不遮挡，气泡不要太宽。

### 基础适配

```css
@media (max-width: 640px) {
  .chat-page {
    max-width: 100%;
  }

  .chat-header {
    padding: 12px 16px;
  }

  .message-list {
    padding: 16px;
  }

  .message {
    max-width: 88%;
  }

  .chat-input {
    padding: 12px;
  }

  .chat-input button {
    width: 72px;
  }
}
```

先确保手机浏览器打开后不乱套。后面再继续优化输入框高度、字体大小、按钮触控区域。

### 截图说明思路
- 截图 1：桌面端完整聊天页。
- 截图 2：手机宽度下的聊天页，确认气泡不挤压。
- 截图 3：长消息和多条消息时，滚动体验正常。

### 常见坑
- 只在电脑浏览器调样式，手机上一打开就挤成一团。
- 气泡最大宽度太大，导致移动端一条消息占满屏幕。
- 输入区按钮太小，手机上很难点中。

---

## 七、本章你要检查的不是“好不好看”，而是“像不像聊天页”

做完后，按这份清单验收：

- 页面是否分成顶部、消息区、输入区三段？
- 消息区是否能独立滚动？
- 用户消息和机器人消息是否明显区分？
- 输入框是否支持输入和发送？
- 小屏幕下是否还能正常显示？

### 你现在应该达到的效果

到这里，你已经做出一个**可以继续接 AI 的聊天壳子**。下一章只要把接口请求逻辑接进来，用户发一句，机器人回一句，应用就真正活起来了。

### 常见坑
- 只在电脑上测试，没看手机宽度下是否会挤压。
- 页面结构写对了，但交互没跑通。
- 没多发几条长消息测试，后面接真实内容才发现布局崩了。

---

## 本章小结

这章的核心只有一件事：把聊天网页的骨架搭稳。

- 先规划 **标题区、消息区、输入区**
- 再拆成 **ChatHeader、MessageList、ChatInput**
- 用 CSS 做对 **布局、气泡、滚动**
- 最后保证输入框能形成最小交互闭环
- 顺手把 **移动端适配** 也打个底

**下一步建议：**  
先把页面静态样式跑起来，再把消息数据改成数组渲染，为下一章接入 AI API 做准备。只要这一步稳了，后面的接口、流式输出、Markdown 渲染、历史记录都会顺很多。

# 第3章 把消息管起来——对话消息状态管理

前面页面框架已经搭起来了，但聊天应用真正跑起来，靠的不是按钮长什么样，而是**消息能不能被正确记录、更新、清空和渲染**。  
这一章不讲 AI 原理，只解决最实际的问题：**怎么把用户消息、AI 回复、输入状态统一管起来**。这一步做好了，后面的接口请求、流式输出、历史保存才有基础。

> 做完这一章，你的聊天页就不再是静态页面，而是一个能承接后续 API 和流式输出的消息底座。后面第 5 章到第 8 章，都会直接用这里这套消息结构。

---

## 3.1 先定规则：消息数据结构怎么设计？

做聊天页，先想清楚：**一条消息要有哪些信息**。  
这里建议统一成这套字段：

- `id`：唯一标识，更新某条消息时必须用
- `role`：消息角色，固定为 `user`、`assistant`
- `content`：消息正文
- `time`：消息时间
- `status`：消息状态，固定为 `sending`、`streaming`、`done`、`error`

### 推荐的数据结构

```jsx
const createMessage = ({ role, content, status = "done" }) => ({
  id: crypto.randomUUID(),
  role,
  content,
  status,
  time: new Date().toLocaleTimeString(),
});
```

如果你担心兼容性，也可以不用 `crypto.randomUUID()`，改成时间戳加随机数。重点不是生成方式，而是**必须有唯一 id**，不然后面流式更新时很容易改错对象。

### 这套字段为什么够用？

- `role` 让你知道这条消息是谁发的
- `content` 决定页面显示什么
- `time` 方便做聊天记录
- `status` 区分“正在发”“正在生成”“已完成”“出错了”
- `id` 让你可以只改某一条，不会把整个数组搞乱

### 截图说明思路

建议截图一张“消息数据结构示意图”：
- 左边是聊天气泡
- 右边是开发者工具里的数组对象
- 标出 `role/content/status/time/id` 五个字段  
这张图的作用是让读者马上明白：**页面上的每一条气泡，其实都对应数组里的一个对象**。

### 常见坑

- 只存 `content`，后面分不清用户消息和 AI 消息
- 没有 `id`，流式更新时只能整数组替换，容易乱
- `status` 名称一会儿用 `loading`，一会儿用 `streaming`，后面会越来越乱

---

## 3.2 全书统一一种方案：用 `useState` 先把消息管稳

这一章开始，建议全书统一用 **`useState` 管消息数组**。  
原因很简单：新手上手快，代码少，和后面接口请求、流式更新也好接。

### 基础写法

```jsx
import { useState } from "react";

export default function ChatPage() {
  const [messages, setMessages] = useState([]);
  const [inputValue, setInputValue] = useState("");

  return <div>{/* 页面内容 */}</div>;
}
```

这里先记住一个原则：  
**消息列表和输入框是两份状态，不能混在一起。**  
消息列表负责保存聊天记录，输入框状态只负责当前正在输入的文字。

### 为什么不用先讲 `useReducer`？

因为你现在最需要的是把页面做出来，而不是先研究状态管理的高级写法。  
`useState` 够你完成下面这些事：

- 新增一条消息
- 更新某一条消息
- 清空全部消息
- 切换消息状态

这四件事，就是聊天页的核心骨架。

### 截图说明思路

建议截图两张对比：
1. `messages` 数组为空时的开发者工具
2. 发送一条消息后，数组里多出一条对象  
这样读者可以直观看到：**状态不是写在页面里，而是写在 React 的内存状态里**。

### 常见坑

- 把数组直接赋值给新变量后修改，结果 React 不更新
- `setMessages(messages)` 容易拿到旧值
- 后面如果要连续追加消息，必须优先使用函数式更新

---

## 3.3 新增、更新、清空：先把三个最小操作写出来

消息状态管理的核心动作其实就三个：**新增一条、更新一条、清空全部**。  
只要这三件事打通，聊天页就能进入可用状态。

### 1）新增消息

```jsx
const addMessage = (msg) => {
  setMessages((prev) => [...prev, msg]);
};
```

这里一定用函数式写法 `prev => ...`，这样能避免连续操作时拿到旧状态。

### 2）更新某一条消息

```jsx
const updateMessage = (id, patch) => {
  setMessages((prev) =>
    prev.map((item) =>
      item.id === id ? { ...item, ...patch } : item
    )
  );
};
```

这个方法在后面的流式输出里特别重要。AI 回复一边生成，你就一边改同一条消息的 `content` 和 `status`。

### 3）清空消息

```jsx
const clearMessages = () => {
  setMessages([]);
};
```

清空功能一般放在“新对话”或“重置”按钮里，方便用户快速重新开始。

### 一个最小可用例子

```jsx
const onSend = () => {
  const userMsg = createMessage({ role: "user", content: inputValue, status: "done" });
  addMessage(userMsg);

  const assistantMsg = createMessage({ role: "assistant", content: "", status: "sending" });
  addMessage(assistantMsg);
};
```

这里先插入用户消息，再预留一条 AI 消息占位。后面接口返回内容时，就更新这条 AI 消息。  
这也是后面所有功能的基础：**先占坑，再填内容**。

### 截图说明思路

建议截图“发送前”和“发送后”的聊天界面：
- 发送前：消息列表为空
- 发送后：用户消息出现，AI 消息先显示“正在输入”或空白占位  
这样可以帮助读者理解：**消息不是一次性全部出现，而是先建记录，再逐步补内容**。

### 常见坑

- 新增时忘了展开旧数组，结果旧消息被覆盖
- 更新时直接改对象属性，React 不一定刷新
- 清空消息后，输入框和 loading 状态没同步重置

---

## 3.4 把输入中、发送中、完成、错误这几个状态切清楚

聊天应用好不好用，很多时候不取决于功能多，而取决于状态是不是清楚。  
这里建议把消息状态固定成四种：

- `sending`：用户消息已提交，AI 还没开始返回
- `streaming`：AI 正在一边生成一边输出
- `done`：消息已完成
- `error`：请求失败或被限流

### 状态切换思路

1. 用户输入完成，点击发送  
2. 立即把用户消息设为 `done`
3. 插入一条 AI 占位消息，状态设为 `sending`
4. 接口开始返回后，AI 消息切换为 `streaming`
5. 结束后改成 `done`
6. 出错时改成 `error`

### 示例代码

```jsx
const handleSend = async () => {
  if (!inputValue.trim()) return;

  const userMsg = createMessage({
    role: "user",
    content: inputValue,
    status: "done",
  });

  const assistantMsg = createMessage({
    role: "assistant",
    content: "",
    status: "sending",
  });

  addMessage(userMsg);
  addMessage(assistantMsg);
  setInputValue("");

  try {
    updateMessage(assistantMsg.id, { status: "streaming" });
    // 这里后面接接口请求与流式更新
    updateMessage(assistantMsg.id, { content: "AI 回复完成", status: "done" });
  } catch (err) {
    updateMessage(assistantMsg.id, {
      content: "请求失败，请稍后重试",
      status: "error",
    });
  }
};
```

这里先别纠结接口细节，你只要看懂状态怎么变就行。  
后面接大模型 API 时，你会发现：**只需要把返回内容不断填进同一条 assistant 消息里**。

### 截图说明思路

可以做一张三状态对比图：
- `sending`：显示加载中
- `streaming`：气泡里文字逐步增加
- `done`：正常完整展示  
这张图很适合新手理解状态切换，不需要讲太多术语，看图就懂。

### 常见坑

- `inputValue` 清空太早，用户回头看不到自己刚发了什么
- 发送中的 AI 占位消息没先插入，后面没法更新
- 状态名乱写，比如有时叫 `loading`，有时叫 `streaming`，容易维护混乱

---

## 3.5 一个能直接用的消息管理组件

下面给你一个可直接参考的最小版本，重点是结构清楚，后面好接 API。

```jsx
import { useState } from "react";

const createMessage = ({ role, content, status = "done" }) => ({
  id: crypto.randomUUID(),
  role,
  content,
  status,
  time: new Date().toLocaleTimeString(),
});

export default function ChatPage() {
  const [messages, setMessages] = useState([]);
  const [inputValue, setInputValue] = useState("");

  const addMessage = (msg) => setMessages((prev) => [...prev, msg]);

  const updateMessage = (id, patch) => {
    setMessages((prev) =>
      prev.map((item) => (item.id === id ? { ...item, ...patch } : item))
    );
  };

  const clearMessages = () => setMessages([]);

  const handleSend = async () => {
    if (!inputValue.trim()) return;

    const userMsg = createMessage({ role: "user", content: inputValue });
    const assistantMsg = createMessage({
      role: "assistant",
      content: "正在思考...",
      status: "sending",
    });

    addMessage(userMsg);
    addMessage(assistantMsg);
    setInputValue("");

    try {
      updateMessage(assistantMsg.id, {
        content: "这里后面接大模型接口返回内容",
        status: "done",
      });
    } catch (e) {
      updateMessage(assistantMsg.id, {
        content: "发送失败，请重试",
        status: "error",
      });
    }
  };

  return (
    <div>
      <div>
        {messages.map((msg) => (
          <div key={msg.id}>
            <strong>{msg.role}</strong>：{msg.content} ({msg.status})
          </div>
        ))}
      </div>

      <textarea
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="输入你的问题"
      />

      <button onClick={handleSend}>发送</button>
      <button onClick={clearMessages}>清空</button>
    </div>
  );
}
```

这个版本虽然简单，但已经把后面最关键的事情准备好了：

- 消息可新增
- 消息可更新
- 消息可清空
- 每条消息有自己的状态

### 截图说明思路

建议截图浏览器页面和开发者工具同时出现：
- 页面上能看到用户消息、AI 占位消息、状态文字
- 开发者工具里能看到 `messages` 数组内容  
这张图能证明你的数据结构和页面展示是对应的。

### 常见坑

- `key` 不要用数组下标，后面消息更新会乱
- `handleSend` 里不要忘记判空，否则空消息也会进数组
- 发送失败时要把消息状态改成 `error`，不要只打印日志

---

## 3.6 本章小结：先把状态管住，后面才接得稳

这一章只做了一件事：**把聊天消息从“页面里的文本”变成“可管理的数据”**。  
你只要记住下面这条线就行：

- 先设计消息结构
- 再统一用 `useState` 管理消息数组
- 然后实现新增、更新、清空
- 最后加上发送中、流式中、完成、错误这些状态

### 你现在应该完成的动作

1. 在项目里定义统一的消息对象结构
2. 用 `useState` 把消息数组管起来
3. 完成消息新增、更新、清空方法
4. 在界面上能看到消息状态变化
5. 打开开发者工具检查数组是否正确变化

### 最后提醒几个高频坑

- 不要直接改原数组
- 不要忽略 `id`
- 不要把消息状态和输入框状态混在一起
- 不要等到接接口时才想“消息怎么更新”

把这一章做好，下一章接大模型接口时，你就不会手忙脚乱。因为你已经有了一个可靠的消息底座，后面无论是普通请求还是流式输出，都只是往这个底座上“填内容”而已。

# 第4章 做出发送动作——输入框交互与消息发送流程

做 AI 聊天网页，真正的分水岭不是“页面像聊天框”，而是**用户能不能顺畅把一句话发出去**。这一章只做一件事：把输入框、发送按钮、回车发送、换行输入、简单校验和消息入列打通。先把“发出去”这一步做稳，后面的 API 接入、流式输出、历史保存才有意义。

> 本章你要做出的效果很简单：能输入、能发送、能换行、能防重复、能把用户消息立刻显示到列表里。

---

## 一、实现受控输入框与发送按钮状态联动

输入框要做成受控组件：**输入框里写什么，state 就是什么；state 变了，界面也跟着变**。这样后面才能清空输入、控制按钮状态、发送前校验。

### 参考实现
```jsx
import { useState } from "react";

export default function ChatInput({ onSend }) {
  const [input, setInput] = useState("");

  const canSend = input.trim().length > 0;

  return (
    <div className="chat-input">
      <textarea
        value={input}
        placeholder="输入你的问题，Enter 发送，Shift+Enter 换行"
        onChange={(e) => setInput(e.target.value)}
        rows={3}
      />
      <button
        onClick={() => {
          if (!canSend) return;
          onSend(input.trim());
          setInput("");
        }}
        disabled={!canSend}
      >
        发送
      </button>
    </div>
  );
}
```

### 要达到的效果
- 输入时，页面实时显示内容
- `input` 状态和 textarea 内容一致
- 空内容时按钮不可点，有内容时按钮可点

### 截图说明思路
建议截两张：
- **空输入状态**：textarea 为空，发送按钮灰掉
- **输入内容后状态**：输入一段文字，按钮可点击

### 常见坑
- 忘了写 `value={input}`，后续不好清空
- 用 `defaultValue` 代替 `value`，状态容易脱节
- 只改样式不改逻辑，空内容也能发送

---

## 二、支持 Enter 发送、Shift+Enter 换行

聊天输入框里，**Enter 通常表示发送，Shift+Enter 才是换行**。不处理好，用户体验会很差：一按回车就提交，甚至直接刷新页面。

### 代码写法
```jsx
import { useState } from "react";

export default function ChatInput({ onSend }) {
  const [input, setInput] = useState("");

  const handleSend = () => {
    const text = input.trim();
    if (!text) return;
    onSend(text);
    setInput("");
  };

  const handleKeyDown = (e) => {
    if (e.key === "Enter" && !e.shiftKey) {
      e.preventDefault();
      handleSend();
    }
  };

  return (
    <div className="chat-input">
      <textarea
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyDown={handleKeyDown}
        rows={3}
        placeholder="Enter 发送，Shift+Enter 换行"
      />
      <button onClick={handleSend} disabled={!input.trim()}>
        发送
      </button>
    </div>
  );
}
```

### 操作步骤
1. 给 textarea 加 `onKeyDown`
2. 判断是否按下 `Enter`
3. 没按 `Shift` 时执行 `preventDefault()`
4. 调用发送逻辑
5. `Shift+Enter` 保留默认行为，继续换行

### 截图说明思路
建议演示两种效果：
- 按 **Enter** 后消息被发送
- 按 **Shift+Enter** 后输入框出现换行

### 常见坑
- 忘了 `preventDefault()`，导致表单刷新
- 用 `onKeyPress`，兼容性不如 `onKeyDown`
- 没区分 `Shift+Enter`，用户无法正常换行

---

## 三、发送前做简单校验：空内容、重复提交、防抖处理

真实用户不会总是规规矩矩：会输入空格误点发送，也可能在网络慢时连点按钮。最少要挡住三类问题：**空内容、重复提交、短时间重复触发**。

### 参考实现
```jsx
import { useRef, useState } from "react";

export default function ChatInput({ onSend }) {
  const [input, setInput] = useState("");
  const [sending, setSending] = useState(false);
  const lastSendTimeRef = useRef(0);

  const handleSend = async () => {
    const text = input.trim();
    if (!text) return;
    if (sending) return;

    const now = Date.now();
    if (now - lastSendTimeRef.current < 800) return;
    lastSendTimeRef.current = now;

    try {
      setSending(true);
      await onSend(text);
      setInput("");
    } finally {
      setSending(false);
    }
  };

  return (
    <div className="chat-input">
      <textarea
        value={input}
        onChange={(e) => setInput(e.target.value)}
        rows={3}
      />
      <button onClick={handleSend} disabled={!input.trim() || sending}>
        {sending ? "发送中..." : "发送"}
      </button>
    </div>
  );
}
```

### 这段代码的作用
- `trim()` 去掉前后空格，避免空白消息
- `sending` 防止请求未结束又点一次
- `lastSendTimeRef` 避免极短时间内重复触发
- 按钮状态跟随发送状态变化

### 截图说明思路
截一个“发送中”的状态：
- 按钮显示“发送中...”
- 按钮被禁用
- 输入框内容保留，等待请求结束

### 常见坑
- 只禁用按钮，不禁用回车，还是会重复发送
- `sending` 还没变 `true` 就被连点两次
- 防抖时间设太长，正常操作会显得卡

---

## 四、发送后清空输入框，并把用户消息加入列表

发送消息不是把文本扔出去，而是要**立刻出现在聊天列表里**。这样用户才会感觉“我已经发出去了”。

### 推荐流程
1. 读取输入内容
2. 先把用户消息加入消息列表
3. 清空输入框
4. 再进入后续 AI 请求流程

### 示例代码
```jsx
import { useState } from "react";

export default function ChatPage() {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState("");

  const handleSend = async (text) => {
    const userMsg = {
      id: crypto.randomUUID(),
      role: "user",
      content: text,
      time: Date.now(),
    };

    setMessages((prev) => [...prev, userMsg]);
    setInput("");

    // 这里后面再接 AI 请求
  };

  return (
    <div>
      <div className="message-list">
        {messages.map((msg) => (
          <div key={msg.id} className={msg.role}>
            {msg.content}
          </div>
        ))}
      </div>

      <textarea value={input} onChange={(e) => setInput(e.target.value)} />
      <button onClick={() => handleSend(input.trim())}>发送</button>
    </div>
  );
}
```

### 这里要注意
- 用户消息最好先显示，不要等 AI 返回
- `setMessages` 用函数式更新，避免连续发送时拿到旧状态
- `id` 要唯一，后面做流式更新、删除都方便

### 截图说明思路
建议截图展示：
- 发送前：输入框里有文字，消息列表还没变化
- 发送后：输入框清空，列表中立刻新增一条用户消息

### 常见坑
- 先清空再读取，结果发出去的是空字符串
- 直接 `setMessages([...messages, userMsg])`，连续发送时容易丢状态
- 消息入列太晚，界面像“没反应”

---

## 五、把输入交互收成一个可复用的小流程

到这里，你已经有了一个能直接用的发送骨架。后面接 API 时，不要把逻辑散在页面各处，最好把“发送”收成一个明确流程：**校验输入 → 标记发送中 → 入列用户消息 → 清空输入框 → 交给下一步请求**。

### 一个可直接套用的整合版
```jsx
import { useRef, useState } from "react";

export default function ChatInput({ onSend }) {
  const [input, setInput] = useState("");
  const [sending, setSending] = useState(false);
  const lastSendTimeRef = useRef(0);

  const handleSend = async () => {
    const text = input.trim();
    if (!text || sending) return;

    const now = Date.now();
    if (now - lastSendTimeRef.current < 800) return;
    lastSendTimeRef.current = now;

    try {
      setSending(true);
      await onSend(text);
      setInput("");
    } finally {
      setSending(false);
    }
  };

  const handleKeyDown = (e) => {
    if (e.key === "Enter" && !e.shiftKey) {
      e.preventDefault();
      handleSend();
    }
  };

  return (
    <div className="chat-input">
      <textarea
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyDown={handleKeyDown}
        rows={3}
        placeholder="输入内容，Enter 发送，Shift+Enter 换行"
      />
      <button onClick={handleSend} disabled={!input.trim() || sending}>
        {sending ? "发送中..." : "发送"}
      </button>
    </div>
  );
}
```

### 本章你已经打通的闭环
- 能输入
- 能判断空内容
- Enter 能发，Shift+Enter 能换行
- 发送时不能重复提交
- 发送后输入框自动清空，消息进入列表

---

## 本章小结：先把“发出去”这件事做稳

这一章最重要的不是某个语法点，而是你已经搭出了**聊天输入流程的基本骨架**：

- 输入框用受控组件管理
- 按钮状态跟随内容变化
- Enter 发送，Shift+Enter 换行
- 发送前做空内容和重复提交保护
- 发送后清空输入框，并把用户消息加入列表

### 你接下来应该做什么
把这一章的代码接到上一章的页面结构里，重点检查：
- 能不能正常打字
- 能不能回车发送
- 能不能换行
- 空消息会不会被拦住
- 发送后输入框会不会清空
- 消息会不会马上出现在列表里

### 本章最常见的坑
- 回车触发表单刷新
- 输入框失控，状态和界面不同步
- 连续点击导致多次发送
- 空消息也被提交
- 发送后没有清空，用户以为没成功

先把“发送动作”做对，后面接 AI 接口时，你会轻松很多。

# 第5章 接上大模型——接口请求封装与 API 调用

前面我们已经把聊天页面、输入框和消息列表搭好了。  
现在要做最关键的一步：**让前端真正把消息发给大模型，并把回复显示出来**。

你不需要先懂太多 AI 原理，只要先把这条链路打通：

**用户输入 → 前端封装请求 → 调用大模型接口 → 拿到回复 → 写回消息列表**

这一步跑通后，你的 AI 聊天网页就从“静态页面”变成“可交互应用”了。

---

## 1. 先选对接口方式：前端直连还是后端代理

新手最容易卡在这里。做第一个 AI 聊天网页，先记住一句话：

**能跑起来优先，能上线更要安全。**

所以本章统一用一种方式：**前端请求自己的后端代理接口**。  
这样有三个好处：

- API Key 不会暴露在浏览器里
- 更容易解决跨域问题
- 以后换模型，主要改后端

你可以把它理解成这条链路：

`页面输入框 -> React 组件 -> sendChatRequest() -> /api/chat -> 后端转发给大模型 -> 返回回复 -> 写回消息列表`

### 推荐的请求形态

聊天接口通常用 `POST`，请求体里带上：

- `messages`：消息数组
- `model`：模型名
- `temperature`：回答随机性

前端不需要关心模型内部怎么推理，只要把消息发过去，再把结果拿回来。

### 本章最小可完成任务

你只要完成下面这件事，就算本章过关：

> 在输入框里输入一句话，点击发送后，页面能收到 AI 回复，并把回复显示到消息列表里。

先别追求花哨效果，先追求**通路跑通**。

### 截图说明思路

建议配一张“接口调用链路图”截图：

- 左边：输入框和发送按钮
- 中间：`sendChatRequest()`
- 右边：`/api/chat`
- 最后：消息列表里的 AI 回复

图上标清“用户输入”“请求发送”“响应返回”“写回页面”四步，读者一眼就能明白流程。

### 常见坑

- 前端直连第三方接口，被 **跨域** 拦住
- 把 **API Key 直接写在前端代码**，上线后等于公开
- 接口要求 `POST`，你却按 `GET` 发请求
- 只看页面不看网络请求，出错了不知道错在哪

---

## 2. 统一封装请求函数：地址、请求头、请求体、错误处理

不要把 `fetch` 散落在组件里。正确做法是单独封装一个接口文件，比如 `src/api/chat.js`。  
这样后面你要加流式输出、改返回格式、换后端地址，都不用到处改。

### `src/api/chat.js`

```javascript
export async function sendChatRequest(messages) {
  const res = await fetch(import.meta.env.VITE_CHAT_API_URL, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      messages,
      model: "gpt-4o-mini",
      temperature: 0.7,
    }),
  });

  if (!res.ok) {
    const errorText = await res.text();
    throw new Error(`请求失败：${res.status} ${errorText}`);
  }

  return await res.json();
}
```

这段代码是本章最重要的基础。  
先把它跑通，后面很多功能都会基于它扩展。

### 这几个部分分别干什么

#### 1）请求地址
`import.meta.env.VITE_CHAT_API_URL`  
接口地址不要写死。放到环境变量里，开发和上线切换会方便很多。

#### 2）请求头
`Content-Type: application/json`  
告诉后端：我发的是 JSON 数据。

#### 3）请求体
统一传 `messages`，再附上 `model` 和 `temperature`。  
消息数组可能长这样：

```javascript
[
  { role: "user", content: "你好" },
  { role: "assistant", content: "你好，我可以帮你什么？" }
]
```

核心不是“传一句话”，而是**把完整对话上下文传给接口**，这样模型才能接着聊。

#### 4）错误处理
不要默认“请求一定成功”。一旦接口返回 400、401、500，页面就可能没反应。  
所以一定要判断：

- `res.ok` 是否为真
- 失败时抛出错误
- 在组件里统一提示用户

### 截图说明思路

建议配两张图：

1. `chat.js` 中 `fetch` 的地址、请求头和请求体  
2. 浏览器 DevTools 的 Network 面板，显示请求方法、状态码和响应内容

这样读者能从“代码”和“网络请求”两个角度理解接口调用。

### 常见坑

- 忘记写 `JSON.stringify`
- `Content-Type` 写错，后端收不到 JSON
- `VITE_` 前缀没加，环境变量读不到
- 接口返回不是 JSON，却直接 `res.json()`，导致解析失败

---

## 3. 用环境变量管理 API 地址和代理配置

接口信息不要写死在代码里。  
正确做法是放到 `.env.local`，这样本地调试、上线部署都能切换。

### 新建 `.env.local`

```env
VITE_CHAT_API_URL=/api/chat
```

如果你当前还在测试阶段，也可以先写成真实后端地址：

```env
VITE_CHAT_API_URL=https://your-domain.com/api/chat
```

这里不把 Key 放到前端。**真正的密钥应该留在后端环境变量里，由后端去调用大模型服务。**  
前端只负责访问你自己的 `/api/chat`，更安全，也更适合部署。

### 在代码里读取

```javascript
const apiUrl = import.meta.env.VITE_CHAT_API_URL;
```

你可以在开发时临时打印一下，确认地址有没有读到：

```javascript
console.log("当前接口地址：", import.meta.env.VITE_CHAT_API_URL);
```

### 推荐的项目结构

```bash
src/
  api/
    chat.js
  components/
    ChatInput.jsx
    ChatList.jsx
  App.jsx
.env.local
```

### 截图说明思路

建议截图展示这三处：

- `.env.local` 的接口地址
- `chat.js` 里读取环境变量的位置
- 控制台里打印出的实际地址

重点是让读者知道：**接口配置和页面逻辑是分离的**，以后改地址不用翻组件。

### 常见坑

- 改了 `.env.local` 却没重启项目，变量没生效
- 环境变量名没加 `VITE_`
- 把 `.env.local` 提交到 Git 仓库，导致配置泄露
- 本地代理能用，部署后地址却没同步更新

---

## 4. 发送消息并把 AI 回复写回消息列表

接口封装好了，接下来就是把消息真正发出去，并把结果显示在页面上。  
这一节要完成两件事：

1. 用户点击发送后，先把自己的消息写入列表  
2. 接口返回后，再把 AI 回复追加进去

### `App.jsx`

```jsx
import { useState } from "react";
import { sendChatRequest } from "./api/chat";

function App() {
  const [messages, setMessages] = useState([
    { role: "assistant", content: "你好，我是你的 AI 助手。" },
  ]);
  const [input, setInput] = useState("");
  const [loading, setLoading] = useState(false);
  const [errorTip, setErrorTip] = useState("");

  const handleSend = async () => {
    if (!input.trim() || loading) return;

    const userMessage = { role: "user", content: input };
    const nextMessages = [...messages, userMessage];

    setMessages(nextMessages);
    setInput("");
    setLoading(true);
    setErrorTip("");

    try {
      const data = await sendChatRequest(nextMessages);

      const assistantMessage = {
        role: "assistant",
        content:
          data.reply ||
          data.choices?.[0]?.message?.content ||
          data.message ||
          "没有收到回复",
      };

      setMessages([...nextMessages, assistantMessage]);
    } catch (error) {
      console.error(error);
      setErrorTip("请求失败，请稍后再试。");
      setMessages([
        ...nextMessages,
        { role: "assistant", content: "请求失败，请稍后再试。" },
      ]);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div style={{ padding: 20, maxWidth: 720, margin: "0 auto" }}>
      <div style={{ marginBottom: 12, color: "#d33" }}>{errorTip}</div>

      <div>
        {messages.map((msg, index) => (
          <div key={index} style={{ marginBottom: 12 }}>
            <b>{msg.role}：</b>
            <span>{msg.content}</span>
          </div>
        ))}
      </div>

      <textarea
        value={input}
        onChange={(e) => setInput(e.target.value)}
        rows={4}
        style={{ width: "100%", marginTop: 20 }}
        placeholder="输入你想问的问题"
      />
      <button onClick={handleSend} disabled={loading} style={{ marginTop: 12 }}>
        {loading ? "发送中..." : "发送"}
      </button>
    </div>
  );
}

export default App;
```

### 这段流程要记住

#### 1）先乐观更新
用户点发送后，先把自己的消息显示出来。  
这样页面会立刻有反馈，不会像“卡住了”。

#### 2）再发请求
把当前消息数组传给接口，让模型看到完整上下文。

#### 3）再追加 AI 回复
接口成功后，解析返回值，把 AI 内容写回消息列表。

#### 4）失败也要兜底
如果请求失败，至少显示一句“请求失败，请稍后再试”。  
不要让页面空白，也不要让用户怀疑是不是按钮坏了。

### 这里还有一个小优化

上面代码里我写了：

```javascript
const reply =
  data.reply ||
  data.choices?.[0]?.message?.content ||
  data.message ||
  "没有收到回复";
```

这是为了兼容不同接口返回格式。  
有的后端返回 `reply`，有的返回 `choices[0].message.content`。  
先做兼容，能减少调试成本。

### 截图说明思路

建议配两张截图：

- 发送后，用户消息先出现在列表里
- 接口返回后，AI 回复紧跟在后面

这两张图连起来看，能非常直观地说明“消息写回”的过程。

### 常见坑

- 直接改旧的 `messages`，导致新消息丢失
- `setMessages` 还没更新完就去读旧值，顺序错乱
- 请求失败没有兜底提示，页面看起来没反应
- 发送按钮没加 `loading`，用户连续点几次导致重复请求

---

## 5. 请求返回后怎么排查：看请求参数、看响应内容、看消息更新

这一节不是单独加功能，而是教你**怎么确认接口到底有没有通**。  
第一次接入大模型时，最常见的问题不是“不会写代码”，而是“明明写了代码，为什么没结果”。

建议按下面三步查：

### 第一步：看请求参数有没有发对

在 `sendChatRequest(messages)` 里，确认这几个内容：

- 请求地址是不是对的
- `method` 是不是 `POST`
- `body` 有没有 `JSON.stringify`
- 传出去的 `messages` 有没有内容

你也可以临时加一行：

```javascript
console.log("发送的消息：", messages);
```

### 第二步：看响应内容长什么样

不同接口返回结构不一样。  
有的会直接返回：

```json
{ "reply": "你好" }
```

有的会返回：

```json
{
  "choices": [
    {
      "message": {
        "content": "你好"
      }
    }
  ]
}
```

所以先在浏览器 Network 面板里看返回值，再决定前端怎么解析。

### 第三步：看消息列表有没有更新

如果接口已经返回了，但页面没变化，通常是状态更新写错了。  
检查点包括：

- `setMessages([...nextMessages, assistantMessage])` 是否执行了
- 是否在错误分支里覆盖了正确消息
- `loading` 是否影响了按钮状态或渲染逻辑

### 截图说明思路

建议统一用这三张图做本章截图说明：

1. **请求配置截图**：`chat.js` 里的地址、请求头、请求体  
2. **接口响应截图**：Network 面板里的返回内容  
3. **消息更新截图**：页面中用户消息和 AI 回复连续出现

这样读者能顺着“参数配置 → 接口返回 → 页面变化”把整条链路看明白。

### 常见坑

- 只看页面，不看 Network，定位不了问题
- 响应已经返回，但你取错字段了
- 请求失败后错误被吞掉，没有打印日志
- 前端状态更新了，但组件渲染条件写错了，页面看不到

---

## 6. 给新手的验收清单：做到这里就算通了

这一章不要求你一次做得很完美，但一定要先把“能聊”做出来。  
你可以按下面这份清单验收：

- [ ] `.env.local` 里已经写好接口地址
- [ ] `src/api/chat.js` 已经封装了请求函数
- [ ] 请求方法是 `POST`
- [ ] 请求体使用了 `JSON.stringify`
- [ ] `App.jsx` 能把用户输入写进消息列表
- [ ] 接口返回后，AI 回复能追加到页面
- [ ] 请求失败时页面会显示提示
- [ ] 浏览器 Network 面板里能看到请求和响应

如果这 8 项都完成了，说明你已经把前端和大模型真正接上了。

---

## 7. 本章常见坑汇总：Key 暴露、跨域、格式错误、解析失败

这一章最容易出问题的地方，基本集中在下面几类。

### 坑 1：API Key 暴露
**错误写法：**

```javascript
const apiKey = "sk-xxxxxx";
```

这会直接把密钥暴露给浏览器。  
正确做法是：

- 前端只请求自己的后端接口
- Key 放到后端环境变量
- 由后端转发请求给模型服务

### 坑 2：跨域问题
如果你直接请求第三方接口，浏览器可能会拦截。  
常见表现是：

- 控制台报 CORS
- 请求看起来发出去了，但前端拿不到结果

**解决思路：**

- 用后端代理
- 开发时配置代理
- 上线后尽量让前端和代理接口同域

### 坑 3：请求体格式错误
很多接口对字段名要求很严格。  
比如有的要求：

```json
{ "messages": [...] }
```

有的要求：

```json
{ "model": "...", "messages": [...] }
```

字段少一个、名字错一个，都可能直接 400。  
所以不要猜，直接对照接口文档和后端约定。

### 坑 4：返回值解析失败
接口返回格式不统一时，前端最容易写死路径。  
建议保留兼容写法：

```javascript
const reply =
  data.reply ||
  data.choices?.[0]?.message?.content ||
  data.message ||
  "";
```

如果后面换模型、换后端，这种写法会省很多时间。

### 坑 5：接口报错后 UI 没反馈
这是新手非常常见的问题。  
请求失败后，页面还停留在“发送中...”或者什么都不显示，用户就会以为程序卡住了。  
所以一定要在 `catch` 里做两件事：

- 打印错误日志
- 给页面加一条明确提示

---

## 本章小结：先把“能聊”做出来

这一章你完成的是整个 AI 聊天网页最核心的闭环：  
**从输入到调用接口，再到显示回复。**

你现在应该已经掌握：

- 用环境变量管理接口地址
- 把请求封装成独立函数
- 用 React 状态管理消息列表
- 发送消息后把 AI 回复写回页面
- 用 Network 面板和控制台排查问题
- 提前规避 Key 暴露、跨域、格式错误、解析失败这些常见坑

### 下一步建议

如果你已经跑通了本章代码，下一章就可以继续做两个最影响体验的功能：

1. **流式输出与打字效果**  
   让 AI 回复像边生成边显示，而不是一下子蹦出来

2. **Markdown 渲染与代码高亮**  
   让 AI 回复里的代码块、标题、列表显示得更像“真正的聊天产品”

先别急着做花哨效果，先把接口链路跑稳。  
**能稳定地发出去、收回来、展示出来，就是一个合格的 AI 聊天网页。**

# 第6章 让回复像聊天一样出现——流式响应与打字效果

前面我们已经把页面、输入框、消息列表都搭好了。接下来最影响“像不像一个真正的 AI 聊天网页”的，就是**回复出现的方式**。

如果 AI 一次性把整段答案“啪”地吐出来，功能虽然能用，但体验更像接口刷新。真正像聊天产品的做法，是**边生成边显示**：内容一小段一小段地出现，用户能明显感觉到系统正在思考。

这一章我们只做三件事：

1. 让前端接住后端的**流式响应**
2. 把增量内容**实时追加到同一条 AI 消息**
3. 加一个简单的**打字效果和“正在输入”提示**

---

## 一、先把思路理顺：流式响应不是一次性返回

普通接口通常这样写：

```js
const data = await response.json();
```

意思是：等接口把完整结果都返回，前端再统一显示。

但流式接口不一样。它会把内容拆成很多小片段，前端边接边渲染。比如一段回答可能被拆成：

- `你`
- `好`
- `，`
- `我可以帮你`

所以这里不能再用 `response.json()`，而要改成**读取流**。你可以把它理解成：

- `fetch` 负责发请求
- `response.body` 里保存可读流
- `getReader()` 一段一段读取
- 每读到一小块，就更新页面上的 AI 消息

### 截图说明思路

建议放两张对比图：

- **图 6-1**：普通一次性返回，整段内容在接口完成后才出现
- **图 6-2**：流式输出，回复一边增长一边显示，底部带“AI 正在输入...”

这两张图的重点是让读者直观看出：**流式输出更像真实聊天产品**。

### 常见坑

- 误以为 `response.json()` 可以处理流式接口
- 没检查 `response.body` 是否存在
- 后端返回的不是纯文本分片，前端没做解析就直接拼接

---

## 二、用 `fetch + ReadableStream` 读取增量数据

下面这段代码是本章最核心的请求封装。它不负责渲染，只负责把流式内容一段段吐给调用方。

```jsx
export async function sendChatMessage(messages, onChunk, onDone, onError) {
  try {
    const response = await fetch("/api/chat", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ messages }),
    });

    if (!response.ok) {
      throw new Error(`请求失败：${response.status}`);
    }

    if (!response.body) {
      throw new Error("当前环境不支持流式响应");
    }

    const reader = response.body.getReader();
    const decoder = new TextDecoder("utf-8");

    while (true) {
      const { value, done } = await reader.read();
      if (done) break;

      const chunk = decoder.decode(value, { stream: true });
      onChunk(chunk);
    }

    onDone?.();
  } catch (err) {
    onError?.(err);
  }
}
```

这段代码的关键就四个点：

- `getReader()`：拿到流读取器
- `reader.read()`：每次读取一小段
- `TextDecoder("utf-8")`：把二进制转成字符串，避免中文乱码
- `onChunk(chunk)`：把增量内容交给页面更新

### 这里为什么要固定用 UTF-8

因为流式数据是分片传输的，如果编码不统一，就容易出现中文乱码或字符断裂。新手先记住一句话就够了：

> 前后端都尽量统一 UTF-8，前端解码时一定要用 `TextDecoder("utf-8")`。

### 截图说明思路

建议截图这段请求封装，并标注两个地方：

- `getReader()` 是读取流的入口
- `onChunk(chunk)` 是实时刷新界面的关键

这样读者能把“代码”和“页面变化”直接对应起来。

### 常见坑

- 忘了判断 `response.body`
- 每次循环都重新创建 `TextDecoder`
- `reader.read()` 的 `done` 判断漏写
- 后端不是流式返回，前端却按流去读

---

## 三、把增量内容追加到同一条 AI 消息

这一步最容易写错。很多人拿到一个 chunk 就新增一条消息，结果消息列表会越来越长，变成一堆碎片。正确做法是：**先插入一条空的 AI 消息，再把后续 chunk 追加到这同一条消息里。**

### 正确的消息结构

```js
[
  { role: "user", content: "你好" },
  { role: "assistant", content: "你好，我可以帮你..." }
]
```

用户消息先固定下来，AI 消息则随着流式数据不断增长。

### React 里的写法

```jsx
import { useState } from "react";
import { sendChatMessage } from "./api";

export default function Chat() {
  const [messages, setMessages] = useState([]);
  const [inputValue, setInputValue] = useState("");
  const [loading, setLoading] = useState(false);

  const handleSend = async () => {
    if (!inputValue.trim() || loading) return;

    const userMsg = { role: "user", content: inputValue };
    const assistantMsg = { role: "assistant", content: "" };

    setMessages((prev) => [...prev, userMsg, assistantMsg]);
    setInputValue("");
    setLoading(true);

    const nextMessages = [...messages, userMsg];

    await sendChatMessage(
      nextMessages,
      (chunk) => {
        setMessages((prev) => {
          const copy = [...prev];
          const lastIndex = copy.length - 1;

          copy[lastIndex] = {
            ...copy[lastIndex],
            content: copy[lastIndex].content + chunk,
          };

          return copy;
        });
      },
      () => setLoading(false),
      () => {
        setLoading(false);
        setMessages((prev) => {
          const copy = [...prev];
          const lastIndex = copy.length - 1;

          if (copy[lastIndex]?.role === "assistant" && !copy[lastIndex].content) {
            copy[lastIndex] = {
              ...copy[lastIndex],
              content: "抱歉，回复中断了，请重试。",
            };
          }

          return copy;
        });
      }
    );
  };

  return null;
}
```

### 这段代码的关键点

1. **先插入空的 assistant 消息**  
   后续只更新最后一条，不用反复判断有没有 AI 消息。

2. **每次 chunk 到来都只更新最后一条**  
   不要新增消息，不要把每个片段当成独立对话。

3. **用函数式更新 `setMessages`**  
   更稳，不容易拿到过期状态。

4. **流式中断时要收尾**  
   如果请求失败，至少关闭加载状态，必要时补一句“回复中断了，请重试”。

### 截图说明思路

建议做一组对比截图：

- **图 6-3**：正确做法，AI 消息内容逐步变长
- **图 6-4**：错误做法，每个 chunk 都变成单独一条消息
- **图 6-5**：流式中断后，页面仍能正常收尾，不会卡在加载状态

### 常见坑

- 把每个 chunk 都当成新消息，消息列表会爆炸
- `setMessages([...messages, ...])` 直接依赖旧闭包，容易拿到过期状态
- 用户连续点击发送，导致多个请求互相覆盖
- 流中断后没有收尾，`loading` 一直不关闭

---

## 四、加一个“正在输入”的提示和基础打字感

流式输出已经有聊天感了，再补一层体验：让用户知道系统还没结束。

### 4.1 “正在输入”提示

最简单的方式是，当 `loading` 为 `true` 时显示提示：

```jsx
{loading && (
  <div className="typing">
    AI 正在输入...
  </div>
)}
```

配一个简单样式就够了：

```css
.typing {
  font-size: 14px;
  color: #888;
  padding: 8px 12px;
}
```

这个提示的作用很直接：告诉用户**不是卡住了，是还在生成**。

### 4.2 简单打字效果

如果后端本来就是分片返回，前端天然就有一点“打字感”。你还可以在 AI 消息末尾加一个闪烁光标，让它更像聊天产品：

```css
.cursor {
  display: inline-block;
  width: 8px;
  animation: blink 1s step-end infinite;
}

@keyframes blink {
  50% {
    opacity: 0;
  }
}
```

然后在渲染 AI 消息时带上光标：

```jsx
<div className="assistant-msg">
  {msg.content}
  {loading && <span className="cursor">|</span>}
</div>
```

### 4.3 这一层体验做到什么程度就够了

新手先把下面三点做好就行：

- 回复不是整段瞬间出现
- 用户能看到“正在生成”
- 流结束后提示自动消失

先做到这一步，聊天网页的“活感”就出来了。

### 截图说明思路

建议截图三种状态：

- 回复进行中，消息末尾有闪烁光标
- 回复未完成，底部显示“AI 正在输入...”
- 回复完成后，提示自动消失

### 常见坑

- `loading` 忘了在结束时置为 `false`
- 只在发送时显示提示，没有覆盖整个流式阶段
- 光标动画太抢眼，影响正文阅读

---

## 五、流式失败时怎么降级，避免页面卡死

实际项目里，流式不一定每次都顺利。浏览器兼容、网络抖动、后端中断、接口格式变化，都可能让流读到一半停掉。所以你最好提前准备一个**收尾策略**：

### 方案一：优先流式，失败时给出提示

上面的 `onError` 已经做了最基础的处理：关闭 `loading`，并给最后一条空助手消息补一句错误提示。这样页面不会空着，也不会一直转圈。

### 方案二：退化为完整回复

如果后端同时支持普通完整返回，可以在检测到流式不可用时，改用一次性返回。思路很简单：

- 先尝试流式
- 如果 `response.body` 不存在，或者读流报错
- 就走普通 `json()` 或普通文本返回逻辑

你可以把它理解为：**能流就流，不能流就先保证能用**。对新手来说，这比一上来追求完美更重要。

### 截图说明思路

可以补一张“异常状态”截图：

- 流式中断后，AI 消息显示“回复中断了，请重试”
- 输入框仍然可用
- 发送按钮恢复正常

这能告诉读者：失败也要优雅收场。

### 常见坑

- 只考虑成功，不考虑中断
- 出错后没有关闭 `loading`
- 流式和普通模式切换时，消息结构不一致

---

## 六、移动端适配别忘了：流式内容会把页面顶乱

流式输出本身没问题，但如果消息区域高度、滚动位置、输入框固定方式没做好，手机上很容易出现“内容一边长，一边把输入框顶跑”的情况。最少要注意这几点：

```css
.chat-list {
  overflow-y: auto;
  height: calc(100vh - 140px);
  padding: 12px;
}

@media (max-width: 768px) {
  .chat-list {
    height: calc(100vh - 120px);
    padding: 8px;
  }

  .typing {
    font-size: 13px;
  }
}
```

如果你已经做了输入框底部固定，那流式内容增长时要保持滚动条自动贴底，不然用户会感觉“消息在跳”。

### 截图说明思路

建议补一张手机视图截图：

- 长回复正在流式显示
- 页面底部输入框固定可见
- 消息区自动滚动，不遮挡内容

### 常见坑

- 流式内容增长后，输入框被顶出屏幕
- 手机上没有自动滚动到底部
- 字体太大，导致一行消息被拆得太碎

---

## 七、本章小结：先做出“会动”的聊天，再追求更精致

到这里，你已经把 AI 聊天网页里最关键的“实时感”做出来了。回顾一下，本章真正要掌握的是：

- 用 `fetch + ReadableStream` 接流
- 用 `TextDecoder` 把字节转成字符串
- 将增量内容追加到同一条 AI 消息
- 用 `loading` 和光标做出“正在输入”的感觉
- 流式失败时能收尾或降级
- 提前避开乱码、重复追加、解析错误这些常见坑

### 本章验收标准

如果你已经做到下面 4 条，说明这一章可以算过关：

1. AI 回复不是一次性出现，而是一段段增长
2. 页面底部能显示“AI 正在输入...”
3. 流式中断时页面不会卡死
4. 手机端看起来也能正常聊天，不会乱布局

### 你现在可以立刻做的事

1. 把现有普通请求改成流式读取
2. 在消息列表里预留一条空的 assistant 消息
3. 每收到一个 chunk，就更新最后一条消息
4. 加一个简单的“AI 正在输入...”提示
5. 用两张截图对比前后效果

如果你把这一章跑通，整个项目就已经从“能发消息”升级成“像真的 AI 聊天产品”。下一步，我们就继续做更实用的能力：**消息历史、本地存储、错误提示和部署上线**。

# 第7章 让内容更像真实聊天——Markdown 渲染与代码高亮

AI 一旦开始输出长文本，最先暴露的问题通常不是“答得对不对”，而是“好不好读”。一大段纯文本、没有层次的列表、没高亮的代码块，会让页面立刻显得很粗糙。

这一章只做一件事：**把 AI 回复渲染得像一个真正可用的聊天产品**。你会学到如何把 Markdown 正确显示出来，如何给代码块加高亮，以及如何把这些效果稳定地放进聊天气泡里。

---

## 7.1 识别 AI 回复中的 Markdown 内容并进行渲染

### 为什么要做这一步

大模型很爱输出 Markdown：标题、列表、引用、代码块、链接、加粗等。如果你直接把它当普通字符串显示，用户看到的只是原始文本，层次感会差很多。

比如 AI 回复里明明有这样的内容：

```md
## 安装步骤

1. 先安装依赖
2. 再启动项目
3. 最后打开页面

```ts
console.log("hello");
```
```

如果不处理，页面上就会变成一整段普通文字。

### 实现思路

不用自己写 Markdown 解析器，直接用现成组件更稳。只要消息来自 AI，并且希望支持富文本，就统一走 Markdown 渲染。

先安装依赖：

```bash
npm install react-markdown remark-gfm rehype-sanitize
```

然后封装一个消息组件：

```tsx
// components/MarkdownMessage.tsx
import ReactMarkdown from "react-markdown";
import remarkGfm from "remark-gfm";
import rehypeSanitize from "rehype-sanitize";

type Props = {
  content: string;
};

export default function MarkdownMessage({ content }: Props) {
  return (
    <div className="markdown-body">
      <ReactMarkdown
        remarkPlugins={[remarkGfm]}
        rehypePlugins={[rehypeSanitize]}
      >
        {content}
      </ReactMarkdown>
    </div>
  );
}
```

这里重点有两个：

1. `remarkGfm`：支持表格、任务列表、删除线等常见语法  
2. `rehypeSanitize`：避免危险 HTML 被直接执行，降低 XSS 风险  

### 截图说明思路

建议截三张图：

- 原始文本直接显示，内容很乱
- 接入 Markdown 后，标题和列表有了层次
- 代码块、链接、引用也能正确渲染

这样读者能很直观看到差异。

### 常见坑

- 把 Markdown 当普通文本输出，看起来像日志
- 直接渲染 HTML，容易引入安全问题
- 模型输出不规范，比如代码块没闭合，导致样式错乱

---

## 7.2 接入 Markdown 渲染库并控制安全性

### 为什么不能只“能显示”就完事

聊天网页会接收外部内容，只要内容来自模型或用户输入，就要考虑安全性。支持 Markdown 后，最怕模型输出 `<script>`、`<img onerror=...>` 这类危险内容。

所以前端至少要做基础防护：**允许常见格式，禁止危险 HTML 执行**。

### 推荐做法

```tsx
import ReactMarkdown from "react-markdown";
import remarkGfm from "remark-gfm";
import rehypeSanitize from "rehype-sanitize";

export function SafeMarkdown({ text }: { text: string }) {
  return (
    <ReactMarkdown
      remarkPlugins={[remarkGfm]}
      rehypePlugins={[rehypeSanitize]}
      components={{
        a: ({ href, children }) => (
          <a href={href} target="_blank" rel="noreferrer">
            {children}
          </a>
        ),
      }}
    >
      {text}
    </ReactMarkdown>
  );
}
```

这里顺手把链接处理成新窗口打开，并加上 `rel="noreferrer"`，更稳妥。你也可以给外链统一加样式，避免用户看不出哪里能点。

### 你可以怎么测

准备几段测试文本：

```md
# 标题

- 列表 1
- 列表 2

> 这是一段引用

[打开示例](https://example.com)
```

再准备一段带危险标签的文本，看它是否会被安全过滤：

```md
<script>alert('xss')</script>
```

你不需要让它“变聪明”，你只需要确认它“不会乱执行”。

### 截图说明思路

可以做一个对比图：

- 左边：原始字符串，显示的是标签文本
- 右边：Markdown 渲染结果，但危险 HTML 不会执行

### 常见坑

- 以为“AI 不会恶意输出”，其实模型输出不可完全信任
- 直接开启危险 HTML 支持，图省事但风险大
- 忘记处理链接跳转，用户点了链接后离开聊天页

---

## 7.3 配置代码高亮方案，让代码块更易读

### 为什么代码块要单独处理

AI 聊天里最常见的高价值内容之一就是代码。用户复制代码、查看思路、对比修改时，如果代码块没有高亮、没有背景、没有换行处理，体验会很差。

所以代码块的目标不是“能显示”，而是“能看懂、能复制、不会撑坏布局”。

### 推荐方案

在 `react-markdown` 的基础上，为 `code` 标签单独接入高亮组件。

```bash
npm install react-syntax-highlighter
```

```tsx
// components/MarkdownMessage.tsx
import ReactMarkdown from "react-markdown";
import remarkGfm from "remark-gfm";
import rehypeSanitize from "rehype-sanitize";
import { Prism as SyntaxHighlighter } from "react-syntax-highlighter";
import { oneDark } from "react-syntax-highlighter/dist/esm/styles/prism";

export default function MarkdownMessage({ content }: { content: string }) {
  return (
    <div className="markdown-body">
      <ReactMarkdown
        remarkPlugins={[remarkGfm]}
        rehypePlugins={[rehypeSanitize]}
        components={{
          code({ inline, className, children, ...props }) {
            const match = /language-(\w+)/.exec(className || "");
            if (!inline && match) {
              return (
                <SyntaxHighlighter
                  style={oneDark}
                  language={match[1]}
                  PreTag="div"
                  customStyle={{ borderRadius: 8, overflowX: "auto", margin: "12px 0" }}
                  wrapLongLines={false}
                  {...props}
                >
                  {String(children).replace(/\n$/, "")}
                </SyntaxHighlighter>
              );
            }
            return (
              <code className={className} {...props}>
                {children}
              </code>
            );
          },
        }}
      >
        {content}
      </ReactMarkdown>
    </div>
  );
}
```

### 这段代码的核心逻辑

- 普通内联代码：用原生 `code`
- 带语言标记的代码块：交给 `SyntaxHighlighter`
- `overflowX: "auto"`：避免长代码把页面撑坏
- `wrapLongLines={false}`：默认保留代码结构

你可以理解为：**小段代码轻量显示，大段代码重点渲染**。

### 截图说明思路

建议截图展示：

- 没有高亮时：代码块黑白一片
- 接入高亮后：JS、TS、TSX 关键字已经着色
- 长代码行：能横向滚动，不会破坏布局

### 常见坑

- 语言名识别不到，模型没写 ```js 而是写了别名
- 代码块换行问题，长行撑出容器，需要 `overflow-x`
- 高亮库太重，首屏变慢，要注意按需选择主题和库

---

## 7.4 优化表格、引用、链接等常见格式的展示

### 这些内容为什么重要

真实聊天里，AI 不只会吐代码，也会给表格对比、引用说明、参考链接。只把文本渲染出来还不够，要让用户一眼看懂。

比如 AI 可能会输出对比表：

```md
| 方案 | 优点 | 缺点 |
|---|---|---|
| Markdown | 易实现 | 需要样式 |
| 富文本 | 更灵活 | 成本更高 |
```

如果表格样式没处理好，内容就会挤成一团。

### 表格

`remark-gfm` 已经支持表格语法，接下来补样式即可。

```css
.markdown-body table {
  width: 100%;
  border-collapse: collapse;
  margin: 12px 0;
}

.markdown-body th,
.markdown-body td {
  border: 1px solid #e5e7eb;
  padding: 8px 10px;
  vertical-align: top;
  word-break: break-word;
}

.markdown-body th {
  background: #f9fafb;
}
```

### 引用

```css
.markdown-body blockquote {
  border-left: 4px solid #6366f1;
  padding: 8px 12px;
  color: #4b5563;
  background: #f8fafc;
  margin: 12px 0;
}
```

### 链接

```css
.markdown-body a {
  color: #2563eb;
  text-decoration: underline;
  word-break: break-word;
}
```

### 列表与段落

```css
.markdown-body p {
  margin: 8px 0;
}

.markdown-body ul,
.markdown-body ol {
  padding-left: 20px;
  margin: 8px 0;
}

.markdown-body li {
  margin: 4px 0;
}
```

### 进一步的小优化

如果你希望表格在手机端不至于炸开，可以给外层容器加横向滚动：

```css
.markdown-body {
  overflow-x: auto;
}
```

这样当表格列很多时，用户还能左右滑动查看，不会把整个气泡撑爆。

### 截图说明思路

建议做一个组合对比截图：

- 表格：列对齐、边框清晰
- 引用：左侧有蓝色竖线
- 链接：蓝色下划线，点击明确
- 列表：缩进自然，层级分明

### 常见坑

- 表格宽度超出容器，手机端直接炸掉
- 引用和普通段落样式差不多，看不出层级
- 长链接不换行，撑破气泡
- 列表缩进太浅，层次不明显

---

## 7.5 让 Markdown 在聊天气泡里更像“产品”

### 不只是能显示，更要像聊天

Markdown 渲染完成后，还要和聊天 UI 融合。重点是：**内容要在气泡里自然排版，不能像网页文章直接贴进去**。

很多人做到这一步，会发现“功能是有了，但看起来还是怪”。通常不是渲染错了，而是气泡的内边距、行距、字体、段落间距没调好。

### 建议的容器样式

```css
.markdown-body {
  line-height: 1.75;
  font-size: 14px;
  color: #111827;
  word-break: break-word;
}

.markdown-body pre {
  margin: 12px 0;
  border-radius: 8px;
}

.markdown-body code {
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
}
```

如果它放在聊天气泡里，外层容器也要注意：

- 气泡不要太窄，否则表格和代码会很挤
- 气泡不要太满，否则行距再好也显拥挤
- 左右边距要一致，视觉上才稳

### 你要检查的点

- 段落间距是否自然
- 列表缩进是否合理
- 代码块是否不会溢出
- 手机端是否能正常阅读
- 链接、表格、引用是否在深浅色背景下都清楚

建议在开发时准备一条测试消息，专门包含标题、列表、引用、代码块、链接五种内容，然后反复看效果。

### 本章最小可完成任务

做到下面这四步，就算本章过关了：

1. `react-markdown` 正确渲染 AI 回复  
2. `remark-gfm` 让表格、列表、删除线可用  
3. `react-syntax-highlighter` 给代码块加上高亮  
4. 聊天气泡里不再出现溢出、换行错乱和样式崩坏  

### 下一步继续做什么

下一章你就可以把“已经能读”的回复，进一步升级成“正在生成”的体验，也就是流式输出和打字效果。那时候，Markdown 渲染和高亮会继续发挥作用，AI 一边说，你一边看，整个聊天感就真正出来了。

### 截图说明思路

建议最后做一个“聊天气泡完整效果图”：

- 上方是用户提问
- 下方是 AI 回复，里面包含标题、列表、代码块、链接
- 再补一张移动端截图，验证可读性
- 如果条件允许，再加一张“普通文本 vs Markdown 渲染后”的对比图

### 常见坑

- 全局字体和 Markdown 字体冲突
- 代码块字体过大，气泡变形
- Markdown 区域内边距太小，内容挤成一团
- 移动端没做横向滚动，表格直接溢出屏幕
- 过度追求花哨主题，反而让内容不够清晰

---

## 本章小结：先把“读得舒服”做出来

这一章的目标很明确：**让 AI 回复从原始文本变成可阅读的聊天内容**。

你已经完成了这些事：

1. 识别 AI 回复中的 Markdown 内容  
2. 接入 Markdown 渲染库并控制安全性  
3. 给代码块配置高亮与横向滚动  
4. 优化表格、引用、链接、列表等格式  
5. 让 Markdown 和聊天气泡样式统一  

### 你现在就该做的事

- 先把 `react-markdown` 接进项目
- 再补 `remark-gfm` 和 `rehype-sanitize`
- 然后单独处理代码块高亮
- 最后微调表格、引用、链接样式
- 在手机端再检查一遍展示效果

如果这一章做对了，下一步接入流式输出时，AI 说出来的内容就不只是“有了”，而是真的像一个产品了。

# 第8章 把聊天记录存住——消息历史与本地存储

前面已经能正常和 AI 聊天了，但一刷新页面，聊天记录就没了。演示时尤其尴尬，聊到一半一刷新，整个对话直接“失忆”。

这一章专门解决这个问题：**把消息历史和会话状态存到本地**。这样用户刷新后还能继续聊，支持新建会话、清空会话、切换会话，并且控制存储大小，避免越存越乱。

先不用后端数据库，直接用 `localStorage` 把核心体验做出来。对新手来说，这一步最值：它能最快把项目从“能聊”变成“像个产品”。

---

## 8.1 先定好存什么：消息、会话、当前会话

先把数据结构想清楚：**到底存什么？**

最少要存三类数据：

1. **消息列表**：用户发了什么，AI 回了什么。
2. **会话 ID**：当前正在聊的是哪一个会话。
3. **会话列表**：如果支持多会话，就要保存多个入口。

建议统一成一套结构，后面恢复、切换、裁剪都会更顺手。

### 示例：定义本地存储格式

```ts
// storage.ts
export type ChatRole = "user" | "assistant";

export interface ChatMessage {
  id: string;
  role: ChatRole;
  content: string;
  createdAt: number;
}

export interface ChatSession {
  id: string;
  title: string;
  messages: ChatMessage[];
  updatedAt: number;
}

const STORAGE_KEY = "ai_chat_sessions_v1";
const CURRENT_SESSION_KEY = "ai_chat_current_session_v1";

export function loadSessions(): ChatSession[] {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (!raw) return [];
    const data = JSON.parse(raw);
    return Array.isArray(data) ? data : [];
  } catch {
    return [];
  }
}

export function saveSessions(sessions: ChatSession[]) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(sessions));
}

export function loadCurrentSessionId(): string | null {
  return localStorage.getItem(CURRENT_SESSION_KEY);
}

export function saveCurrentSessionId(id: string) {
  localStorage.setItem(CURRENT_SESSION_KEY, id);
}
```

这段代码做了三件事：

- 把消息和会话统一成标准结构；
- 用固定 key 存到浏览器；
- 读取时做容错，避免坏数据把页面搞崩。

### 截图说明思路
- 截图 1：浏览器开发者工具 Application 面板，展示 `localStorage` 里的 `ai_chat_sessions_v1`。
- 截图 2：刷新页面前后聊天内容仍然存在，突出“记录没丢”。

### 常见坑
- **把整个 React 状态直接塞进去**：会带上很多临时字段，不利于维护。
- **数据结构不统一**：后面升级版本时容易读不出来。
- **只存消息，不存当前会话 ID**：刷新后不知道该打开哪段对话。

---

## 8.2 页面初始化时恢复历史消息，别让 UI 和状态脱节

页面一打开，第一件事就是恢复历史数据。做法很直接：**先读本地存储，再塞回 React state**。

关键是顺序：先恢复会话列表，再恢复当前会话，再决定页面显示哪一组消息。不要先渲染空页面，再慢慢覆盖，不然容易闪一下，或者状态对不上。

### 示例：初始化恢复逻辑

```tsx
import { useEffect, useMemo, useState } from "react";
import {
  loadCurrentSessionId,
  loadSessions,
  saveCurrentSessionId,
  saveSessions,
  ChatSession,
} from "./storage";

export default function App() {
  const [sessions, setSessions] = useState<ChatSession[]>([]);
  const [currentSessionId, setCurrentSessionId] = useState<string>("");

  const currentSession = useMemo(
    () => sessions.find((s) => s.id === currentSessionId),
    [sessions, currentSessionId]
  );

  useEffect(() => {
    const savedSessions = loadSessions();
    const savedCurrentId = loadCurrentSessionId();

    if (savedSessions.length > 0) {
      setSessions(savedSessions);

      const validId =
        savedCurrentId && savedSessions.some((s) => s.id === savedCurrentId)
          ? savedCurrentId
          : savedSessions[0].id;

      setCurrentSessionId(validId);
      saveCurrentSessionId(validId);
      return;
    }

    const firstId = crypto.randomUUID();
    const firstSession: ChatSession = {
      id: firstId,
      title: "新会话",
      messages: [],
      updatedAt: Date.now(),
    };

    setSessions([firstSession]);
    setCurrentSessionId(firstId);
    saveSessions([firstSession]);
    saveCurrentSessionId(firstId);
  }, []);

  useEffect(() => {
    if (sessions.length > 0) saveSessions(sessions);
  }, [sessions]);

  useEffect(() => {
    if (currentSessionId) saveCurrentSessionId(currentSessionId);
  }, [currentSessionId]);

  return <div>{currentSession?.messages.length ?? 0}</div>;
}
```

这套逻辑很简单：

- 页面进来先读历史；
- 有历史就恢复；
- 没历史就创建默认会话；
- 状态变化后再写回本地。

一个细节要注意：如果 `currentSessionId` 找不到对应会话，要自动回退到第一条会话，别让页面卡在空状态。

### 截图说明思路
- 截图 1：刷新前聊天内容完整显示。
- 截图 2：手动刷新后消息仍在。
- 截图 3：Application 面板里能看到当前会话 ID 和会话数据，证明恢复顺序正确。

### 常见坑
- **先渲染空数据再覆盖**：会出现短暂闪烁。
- **currentSessionId 找不到对应会话**：页面会空白或报错。
- **恢复逻辑和保存逻辑互相触发**：状态可能反复刷新，覆盖掉刚输入的内容。

---

## 8.3 新建会话、清空会话、继续会话，界面要联动起来

一个像样的聊天产品，至少要能做三件事：

- **新建会话**：开始一段全新的对话。
- **清空会话**：把当前记录清掉，重新开始。
- **继续会话**：选择旧会话接着聊。

只做单一对话，能演示，但产品感不强。加上会话列表后，整个应用会立刻完整很多。

### 示例：会话操作函数

```tsx
function createNewSession() {
  const id = crypto.randomUUID();
  const newSession = {
    id,
    title: "新会话",
    messages: [],
    updatedAt: Date.now(),
  };

  setSessions((prev) => [newSession, ...prev]);
  setCurrentSessionId(id);
}

function clearCurrentSession() {
  if (!currentSessionId) return;

  setSessions((prev) =>
    prev.map((s) =>
      s.id === currentSessionId
        ? { ...s, messages: [], title: "新会话", updatedAt: Date.now() }
        : s
    )
  );
}

function continueSession(sessionId: string) {
  setCurrentSessionId(sessionId);
}
```

注意：**清空当前会话，不等于删除会话**。  
建议保留会话壳，只清空消息。这样用户不会误以为历史功能坏了，后面继续聊也更方便。等第一条新消息发出后，再自动更新标题，列表会更易读。

### 截图说明思路
- 截图 1：左侧会话列表显示多个历史会话。
- 截图 2：点击不同会话后，右侧聊天区内容切换。
- 截图 3：点击“清空当前会话”前后对比，清空后消息没了，但会话还在。

### 常见坑
- **清空会话时把整条会话删了**：用户会以为历史功能坏了。
- **新建会话后仍写入旧会话**：通常是 `currentSessionId` 没切换成功。
- **会话标题一直叫“新会话”**：列表可读性差，演示效果也差。

---

## 8.4 控制容量，别让 localStorage 越存越大

`localStorage` 不是数据库，不能无限存。聊久了消息多了，数据会越来越大，读写也会变慢。所以最好提前做限制：

1. **限制会话数量**
2. **限制单会话消息数**
3. **控制消息长度**

### 示例：简单裁剪策略

```ts
const MAX_SESSIONS = 20;
const MAX_MESSAGES_PER_SESSION = 100;
const MAX_CONTENT_LENGTH = 4000;

function normalizeSession(session: ChatSession): ChatSession {
  return {
    ...session,
    messages: session.messages
      .slice(-MAX_MESSAGES_PER_SESSION)
      .map((msg) => ({
        ...msg,
        content:
          msg.content.length > MAX_CONTENT_LENGTH
            ? msg.content.slice(0, MAX_CONTENT_LENGTH)
            : msg.content,
      })),
  };
}

function saveSafeSessions(sessions: ChatSession[]) {
  const normalized = sessions
    .map(normalizeSession)
    .sort((a, b) => b.updatedAt - a.updatedAt)
    .slice(0, MAX_SESSIONS);

  saveSessions(normalized);
}
```

这套策略不复杂，但很实用：

- 只保留最近的会话；
- 每个会话只保留最近一部分消息；
- 超长内容截断，避免单条数据过大。

如果后面要升级数据结构，建议保留版本号，比如 `ai_chat_sessions_v1`。结构大改时直接换新 key，旧数据和新数据互不干扰。这样即使未来从单会话升级到多会话，也不会把老用户数据搞坏。

### 截图说明思路
- 截图 1：开发者工具里展示大量历史数据，但实际只保留最近会话。
- 截图 2：长对话被裁剪后，页面依然能正常打开，没有明显卡顿。

### 常见坑
- **把超长模型回复原样存入**：很容易逼近 localStorage 上限。
- **会话列表无限增长**：浏览器越用越慢。
- **修改字段名后直接上线**：旧用户数据可能全部失效。

---

## 8.5 版本升级、JSON 解析失败和空值处理，要提前兜底

有时一打开页面就恢复失败，常见原因就这几个：

1. 用户本地数据损坏，`JSON.parse` 报错。
2. 版本升级后字段结构变了。
3. 浏览器存储被清理或超出限制。

所以读数据时一定要容错。不要因为一条坏数据，让整个聊天页白屏。

### 示例：容错恢复

```ts
export function safeLoadSessions(): ChatSession[] {
  const raw = localStorage.getItem(STORAGE_KEY);
  if (!raw) return [];

  try {
    const data = JSON.parse(raw);
    return Array.isArray(data) ? data : [];
  } catch (err) {
    console.warn("会话数据解析失败，已自动忽略", err);
    return [];
  }
}
```

如果格式不对，宁可回到空会话，也不要让页面崩掉。对新手项目来说，**稳定比完美更重要**。

排查时可以按这个顺序看：

- 先看 localStorage 里有没有数据；
- 再看 JSON 是否完整；
- 再看字段名是否变化；
- 最后看 `currentSessionId` 是否能对应上会话。

如果要升级数据结构，建议先保留旧 key，再写迁移函数慢慢过渡，而不是一次性覆盖所有历史。这样对展示和演示都更稳。比如把旧版本的 `name` 迁移到 `title` 时，先判断字段是否存在，再做兼容转换，就能避免老数据直接报错。

### 截图说明思路
- 截图 1：控制台里出现解析失败警告，但页面仍能正常进入新会话。
- 截图 2：旧版本数据和新版本数据对比，说明为什么要用版本 key。
- 截图 3：空值场景下页面自动回到默认会话，UI 没有白屏。

### 常见坑
- **不写 try/catch**：一条坏数据就能让聊天页白屏。
- **升级字段却不兼容旧数据**：老用户刷新后看不到历史。
- **把报错直接暴露给用户**：体验很差，应该降级为可继续使用的状态。
- **空值没处理**：`null`、`undefined` 直接进 UI，容易报错。

---

## 8.6 本章小结：先把“记住”做对，再谈“更聪明”

这一章你已经把 AI 聊天网页最关键的“记忆能力”补上了。现在你应该具备了这套能力：

- 用 `localStorage` 保存会话和消息
- 页面初始化时自动恢复历史
- 支持新建、切换、清空会话
- 控制存储规模，避免失控
- 处理 JSON 解析失败、空值和版本升级问题

### 本章验收标准
做到下面三条，说明这一章已经跑通：

1. 刷新页面后，上一轮消息还能显示出来。
2. 点击“新建会话”“清空会话”“切换会话”时，右侧聊天区会同步变化。
3. 即使本地数据有异常，页面也不会白屏，仍能进入新会话继续使用。

### 下一步你该做什么
1. 在浏览器里真实刷新一次，确认消息是否恢复。
2. 打开 Application 面板，看看本地数据是否按预期存储。
3. 把会话列表做出来，让“历史记录”真正可见、可切换。

前面的章节是在把聊天功能“跑起来”，这一章是在让它“像个产品”。能记住历史，用户才愿意继续用，你的演示也才更完整。

下一章，我们继续补体验：把错误提示、限流提醒和一些容易让新手卡住的问题一次处理掉。

# 第9章 别让用户等得太慌——错误处理与限流提示

做 AI 聊天网页，最容易翻车的不是“不会调用接口”，而是**接口出问题时页面像死了一样**：按钮一直转、消息丢了、用户分不清是网络断了还是被限流了。真实产品里，这一章很关键，因为它决定你的应用是“能跑”还是“能用”。

这一章直接解决 5 个问题：

1. 请求失败时怎么提示用户；
2. 超时、无响应、接口异常怎么区分；
3. 遇到限流怎么让用户知道“不是你错了”；
4. 失败后怎么保留输入内容，方便重试；
5. 怎么做一个更像产品的重试体验。

**本章验收标准：**
- 请求失败时能看到清楚的中文提示；
- 能区分网络错误、超时、限流和服务端异常；
- 失败后用户输入不会丢；
- 重试按钮能重新发送刚才那条消息；
- 按钮状态能正常恢复，不会卡死。

---

## 9.1 先把失败分清楚：网络错误、接口异常、超时、无响应

先记住一个原则：**不要把所有失败都当成“接口报错”**。用户看到的是“没回话”，但代码里最好拆开处理，这样提示才准，按钮状态也更容易恢复。

常见情况可以分成：

- **网络错误**：断网、DNS 异常，请求根本没发出去；
- **接口异常**：服务端返回 500、502，或者返回内容不对；
- **超时**：请求发出去了，但太久没返回；
- **无响应**：连接建立了，但流式输出中途断掉。

先分类，再决定提示文案和重试逻辑。

先做一个统一的接口请求封装，放到 `api/chat.js`，后面所有发送消息都走它：

```js
// api/chat.js
export async function sendChatMessage(message, signal) {
  const res = await fetch("/api/chat", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ message }),
    signal,
  });

  if (!res.ok) {
    if (res.status === 429) throw new Error("HTTP_429");
    if (res.status === 408) throw new Error("HTTP_408");
    throw new Error(`HTTP_${res.status}`);
  }

  return res;
}
```

再在组件里加超时控制，避免用户一直等：

```jsx
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 30000);

try {
  const res = await sendChatMessage(inputValue, controller.signal);
  // 处理返回
} catch (err) {
  if (err.name === "AbortError") {
    setError("请求超时，请稍后重试");
  } else {
    setError("发送失败，请稍后再试");
  }
} finally {
  clearTimeout(timeoutId);
}
```

如果想更完整，可以继续细分：  
- `res.ok === false`：服务端已响应，但状态不对；  
- `AbortError`：超时；  
- 连接断开、读取流失败：统一按“回复中断，请重试”处理。

这一段的重点不是写得多漂亮，而是把最基本的闭环跑通：**发请求、设超时、捕获错误、恢复按钮状态**。

### 截图说明思路

放一张“失败态截图”：

- 输入框还在；
- 用户消息已经显示；
- 助手回复位置出现红色提示；
- 旁边有“重试”按钮。

重点不是美观，而是让读者一眼看懂：**失败后页面没崩，用户还能继续操作。**

### 常见坑

- 只写 `setTimeout`，却没用 `AbortController`；
- `catch` 里只写“请求失败”，太笼统；
- `finally` 忘了恢复发送按钮，导致按钮一直禁用；
- 失败后直接清空输入框，用户内容丢了。

---

## 9.2 把技术错误翻成人话，用户才看得懂

错误信息不要直接甩给用户，比如：

- `500 Internal Server Error`
- `Failed to fetch`
- `HTTP_429`

这些适合开发者，不适合普通用户。更好的方式，是先统一做一个错误映射函数，前端只负责展示翻译后的提示。

```jsx
function getFriendlyErrorMessage(error) {
  const msg = String(error?.message || error);

  if (msg.includes("AbortError") || msg.includes("HTTP_408")) {
    return "请求超时，请稍后重试";
  }
  if (msg.includes("Failed to fetch")) {
    return "网络不可用，请检查连接";
  }
  if (msg.includes("HTTP_429")) {
    return "当前请求较多，请稍等一下再试";
  }
  if (msg.includes("HTTP_500") || msg.includes("HTTP_502")) {
    return "服务暂时繁忙，请稍后再试";
  }
  return "发送失败，请重试";
}
```

UI 中统一展示：

```jsx
{error && (
  <div className="error-banner">
    <span>{error}</span>
  </div>
)}
```

提示文案最好同时包含三件事：

- **发生了什么**：请求超时、网络断开、服务繁忙；
- **用户该怎么做**：刷新、稍后重试、检查网络；
- **不要责怪用户**：少写“你操作有误”，多写“请稍后再试”。

比如：

- “网络不可用，请检查连接后重试”
- “请求超时，当前响应较慢，请稍后再试”
- “服务暂时繁忙，请过一会儿再发送”

这些话虽然简单，但会让你的聊天页更像真实产品。

### 截图说明思路

准备一张顶部或对话区内的错误提示截图：

- 红色或浅红色提示条；
- 文案是用户能看懂的中文；
- 旁边不要放一堆技术字段。

读者需要看到的是“产品提示”，不是“控制台报错”。

### 常见坑

- 把后台报错原样展示出来，太技术化；
- 所有错误都用同一句话，用户无法判断是不是限流；
- 提示文案太长，移动端看起来很挤。

---

## 9.3 限流要明确说出来，再给重试按钮

AI 接口常见的一种失败是**限流**。这不是页面坏了，而是请求太频繁、额度不足、服务端暂时限制。此时最好的做法不是让用户猜，而是明确告诉他：现在是限流，不是消息没发出去。

你可以在接口返回 429 时单独处理：

```jsx
if (!res.ok) {
  if (res.status === 429) {
    throw new Error("HTTP_429");
  }
  throw new Error(`HTTP_${res.status}`);
}
```

UI 里单独展示限流态：

```jsx
{error === "当前请求较多，请稍等一下再试" && (
  <div className="rate-limit-box">
    <p>当前请求较多，请稍等 10 秒后再试。</p>
    <button onClick={handleRetry} disabled={isSending}>
      {isSending ? "重试中..." : "重试"}
    </button>
  </div>
)}
```

重试按钮的核心不是“重新发一次请求”这么简单，而是要**沿用刚才那条消息**，用户不用重新输入。也就是说，失败后的原始输入要保留下来。

```jsx
const [pendingMessage, setPendingMessage] = useState("");
const [isSending, setIsSending] = useState(false);

async function handleSend(nextMessage) {
  const text = nextMessage ?? inputValue;
  if (!text.trim() || isSending) return;

  setPendingMessage(text);
  setIsSending(true);
  setError("");

  try {
    await sendChatMessage(text);
    setInputValue("");
    setPendingMessage("");
  } catch (err) {
    setError(getFriendlyErrorMessage(err));
  } finally {
    setIsSending(false);
  }
}

function handleRetry() {
  if (!pendingMessage) return;
  handleSend(pendingMessage);
}
```

这里有两个关键点：

1. **失败时保留 `pendingMessage`**，用户不用重新输入；
2. **`isSending` 控制重复点击**，防止连续发多次。

如果你前面已经做了消息列表，还可以把这条消息标记成 `failed`，让用户知道是哪一条没成功。

### 截图说明思路

建议准备两张图：

1. **限流提示态**：提示“当前请求较多，请稍后再试”，下方有重试按钮；
2. **重试中态**：按钮变成 loading，避免用户连点。

这两张图能很好说明你的聊天页已经具备基本容错能力。

### 常见坑

- 429 只当普通失败处理，用户根本不知道是限流；
- 重试按钮只刷新 UI，不会重新发送原消息；
- 用户连续点击导致重复请求，页面出现多条重复回复。

---

## 9.4 失败后别清空输入，还要把消息状态回滚好

这是新手最容易忽略的一点：**失败后不要把用户输入清空**。如果用户打了一大段内容，点发送后失败了，你又把输入框清掉，他会很烦。

推荐做法是：

- 发送前先把输入内容写进消息列表；
- 发送时给这条消息一个 `sending` 状态；
- 失败后改成 `failed`；
- 输入框保留原内容，用户可以直接重试。

示例：

```jsx
const [messages, setMessages] = useState([]);
const [inputValue, setInputValue] = useState("");

async function handleSend(nextMessage) {
  const text = (nextMessage ?? inputValue).trim();
  if (!text || isSending) return;

  const userMsgId = Date.now().toString();
  const userMsg = {
    id: userMsgId,
    role: "user",
    content: text,
    status: "sending",
  };

  setMessages((prev) => [...prev, userMsg]);
  setPendingMessage(text);
  setIsSending(true);
  setError("");

  try {
    await sendChatMessage(text);
    setMessages((prev) =>
      prev.map((m) =>
        m.id === userMsgId ? { ...m, status: "success" } : m
      )
    );
    setInputValue("");
    setPendingMessage("");
  } catch (err) {
    setMessages((prev) =>
      prev.map((m) =>
        m.id === userMsgId ? { ...m, status: "failed" } : m
      )
    );
    setError(getFriendlyErrorMessage(err));
  } finally {
    setIsSending(false);
  }
}
```

如果你前面已经做了消息状态管理，这里只要给消息多加一个状态字段，页面就会立刻丰富起来：

- `sending`：显示“发送中...”；
- `failed`：显示红色失败标记和“重试”；
- `success`：正常显示。

这样做的好处很直接：

- 用户知道自己发过什么；
- 失败的消息不丢；
- 后面做“消息重试”“草稿保存”会很顺手。

### 截图说明思路

建议截图放在对话区中部：

- 用户消息还在；
- 消息旁边显示“发送失败”；
- 下方有“重试”按钮；
- 输入框里内容没有消失。

读者一看就知道：这不是简单弹个错误，而是把失败状态接回到聊天流里了。

### 常见坑

- 发送失败后直接 `setInputValue("")`，用户内容丢失；
- 失败消息和成功消息长得一样，用户不知道哪条没发出去；
- 只在弹窗提示失败，没有留在对话区，体验很弱；
- 失败后按钮状态没恢复，页面像卡死。

---

## 9.5 把错误态、重试态、限流态做成统一样式

界面上建议把错误信息做成**对话区内提示**，而不是只在顶部放横幅。因为用户最关心的是“刚才那条消息怎么了”。

你可以做一个统一的系统提示组件，后面想加“限流”“超时”“重试成功”都能复用。

```jsx
function SystemNotice({ text }) {
  return <div className="system-notice">{text}</div>;
}
```

配合样式：

```css
.system-notice {
  margin: 12px auto;
  padding: 10px 12px;
  max-width: 80%;
  border-radius: 10px;
  background: #fff4f4;
  color: #b42318;
  font-size: 14px;
  line-height: 1.5;
}

.rate-limit-box {
  margin-top: 8px;
  padding: 12px;
  border-radius: 10px;
  background: #fff7ed;
  color: #9a3412;
}

.rate-limit-box button {
  margin-top: 8px;
}
```

如果你愿意更进一步，可以把错误状态分成三种颜色：

- **普通失败**：浅红；
- **限流提醒**：橙色；
- **重试中**：灰色 + loading。

这样一来，聊天页会明显更像一个“能用的产品”，而不是一个“实验页面”。

### 截图说明思路

本节截图建议体现三种状态：

- 普通失败：一条消息下方显示“发送失败，请重试”；
- 限流提示：提示“当前请求较多，请稍后再试”；
- 重试中：按钮 loading，输入框暂时禁用。

读者看到这三张图，就能理解你不是只做了“报错文字”，而是做了完整状态流转。

### 常见坑

- 错误态样式和普通系统消息混在一起，用户看不出重点；
- 重试按钮没有禁用，导致重复提交；
- 限流提示太弱，用户以为只是“没反应”；
- 页面只提示一次错误，没有和消息状态联动。

---

## 9.6 本章最小可完成任务

如果你想快速验收这一章，先别追求花哨，按下面顺序做完就够了：

1. 给发送接口加 `AbortController` 和超时；
2. 把 429、408、500 这几类错误翻译成中文提示；
3. 失败后保留输入框内容；
4. 在消息列表里标记 `sending / failed / success`；
5. 给失败消息加“重试”按钮；
6. 按钮在请求中必须禁用，结束后恢复。

做到这 6 步，你的聊天网页就已经不像“演示程序”，而是更接近一个真正可用的产品了。

---

## 本章小结：别让失败变成“页面死亡”

这一章的重点只有一句话：**AI 聊天网页不是只看成功路径，失败路径才决定它像不像产品。**

你至少要做到这四件事：

- 把网络错误、超时、接口异常、限流区分开；
- 错误提示改成人话，不要直接暴露技术信息；
- 失败后保留用户输入和原始消息；
- 提供重试按钮，并恢复按钮状态。

如果你已经完成前面的页面搭建和接口接入，那么把这一章补上，你的 AI 聊天网页就会从“能跑”变成“更稳、更像真产品”。

下一章我们继续往前走，重点会放到聊天窗口的体验优化上，比如输入框交互、滚动跟随、移动端适配这些更影响“好不好用”的细节。

# 第10章 让聊天更好用——滚动、自动聚焦与体验优化

功能能跑，只说明项目“做出来了”；体验好不好，往往取决于这些细节：新消息来了能不能自动看到底部，发完后能不能立刻继续输入，消息排版清不清楚，AI 生成时有没有明确提示。

这一章不加新概念，直接把前面做好的消息列表、输入框、发送按钮和流式输出整理成更像产品的交互。把滚动、聚焦、状态提示做好，页面完成度会立刻上一个台阶。

## 本章最小可完成任务

先完成一个最小闭环：

- 发送一条消息后，聊天列表自动滚到底部；
- 发送完成后，输入框自动恢复焦点；
- AI 回复时，页面能看到“生成中 / 已完成”的状态；
- 用户消息和 AI 消息样式清晰区分。

做到这四件事，这一章就算过关了。

---

## 一、新消息到达时，如何自动滚动到底部？

### 目标是什么

聊天场景里，用户默认想看最新消息。尤其是 AI 流式输出时，内容不断追加，如果页面不自动滚动，用户看到的可能还是中间那一段，体验会很差。

所以目标很明确：

- 新消息到达时自动滚到最新位置；
- 流式输出时尽量保持视线在底部；
- 但用户手动上翻历史时，不要强行抢回到底部。

### 实现思路

最简单的做法，是在消息列表底部放一个锚点，消息数组变化时让它滚动进入视野。

```tsx
import { useEffect, useRef } from "react";

export default function ChatList({ messages }) {
  const bottomRef = useRef<HTMLDivElement | null>(null);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({
      behavior: "smooth",
      block: "end",
    });
  }, [messages]);

  return (
    <div className="chat-list">
      {messages.map((msg) => (
        <div key={msg.id} className={`msg ${msg.role}`}>
          {msg.content}
        </div>
      ))}
      <div ref={bottomRef} />
    </div>
  );
}
```

如果只是做 demo，这段代码已经够用。

### 更实用的做法：判断用户是否在看底部

真实使用中，用户可能正在回看历史消息。这时每次新消息都强制滚动，会显得很“抢视角”。

更稳妥的方式，是先判断容器是否接近底部，再决定要不要滚动。

```tsx
const isNearBottom = (el: HTMLElement) => {
  const distance = el.scrollHeight - el.scrollTop - el.clientHeight;
  return distance < 80;
};
```

消息变化时，只有在接近底部时才自动滚动。流式输出时也建议稍微节流，不要每个字符都滚一次，页面会更稳定。

### 防闪烁的小技巧

如果你发现滚动时页面有抖动，通常是因为滚动触发太频繁，或者内容高度还没稳定就执行了滚动。可以这样处理：

- 只在消息真正新增时滚动，不要每次状态变化都滚；
- 流式输出时用 `requestAnimationFrame` 或简单节流控制频率；
- 先让 DOM 渲染完成，再执行滚动。

```tsx
useEffect(() => {
  const timer = requestAnimationFrame(() => {
    bottomRef.current?.scrollIntoView({ behavior: "smooth", block: "end" });
  });

  return () => cancelAnimationFrame(timer);
}, [messages]);
```

### 截图说明思路

- 截图 1：消息列表较长，滚动条停在中间，展示历史记录；
- 截图 2：新消息到达后，列表自动滚到最新一条；
- 截图 3：用户上滑查看旧消息时，页面没有被强行拉回底部。

### 常见坑

- `scrollIntoView` 调用太频繁，页面会抖；
- 消息还没渲染完就滚动，位置不准；
- 流式输出每个字符都滚一次，体验很差；
- 容器选错了，滚动的是整页而不是聊天窗口。

**解决建议：** 对流式输出做节流；如果用户正在上翻历史，就暂停自动滚动；滚动目标尽量锁定在聊天容器内部。

---

## 二、发送后如何自动回到输入框并保持聚焦？

### 目标是什么

聊天最顺手的交互就是：输入、回车、继续输入。用户不希望每发一句都重新点一次输入框，连续对话时尤其如此。

### 实现步骤

给输入框加一个 `ref`，发送完成后重新 `focus()`。

```tsx
import { useRef, useState } from "react";

export default function ChatInput({ onSend, isSending }) {
  const [value, setValue] = useState("");
  const inputRef = useRef<HTMLInputElement | null>(null);

  const handleSend = async () => {
    const text = value.trim();
    if (!text || isSending) return;

    await onSend(text);
    setValue("");
    inputRef.current?.focus();
  };

  return (
    <div className="chat-input-wrap">
      <input
        ref={inputRef}
        value={value}
        onChange={(e) => setValue(e.target.value)}
        onKeyDown={(e) => e.key === "Enter" && handleSend()}
        placeholder="输入你的问题..."
      />
      <button onClick={handleSend} disabled={isSending || !value.trim()}>
        {isSending ? "发送中..." : "发送"}
      </button>
    </div>
  );
}
```

核心流程很简单：

1. 用户输入内容；
2. 点击发送或按回车；
3. 请求处理完成；
4. 清空输入框；
5. 重新聚焦输入框。

### 让交互更顺手一点

再补几个细节会更像产品：

- 发送前判断是否为空，避免空消息提交；
- 正在生成时禁用发送按钮，防止重复提交；
- 发送按钮显示加载态；
- 移动端尽量保持输入状态，不要反复收起再弹出键盘。

注意一点：输入框最好常驻，不要因为某个状态切换就卸载掉，不然焦点很容易丢。

### 截图说明思路

- 截图 1：发送前，光标停留在输入框内；
- 截图 2：点击发送后，输入框内容清空，但焦点仍在输入框；
- 截图 3：连续对话两三轮，用户不需要反复点击输入框。

### 常见坑

- `await` 之后组件状态变化，导致 `ref` 短暂失效；
- 输入框被条件渲染卸载了，焦点无法恢复；
- 移动端自动聚焦可能带来页面跳动；
- 回车发送时没有阻止默认行为，导致表单刷新。

**解决建议：** 输入框尽量常驻；聚焦动作放在发送完成后执行，而不是请求刚发出时就抢焦点。

---

## 三、消息列表的视觉细节怎么做得更舒服？

### 目标是什么

消息列表是聊天页最核心的区域。功能即使都对了，只要布局乱、气泡挤、左右不分明，页面也会像半成品。

我们要让用户一眼看懂：

- 谁在说话；
- 哪条是用户消息，哪条是 AI 消息；
- 消息什么时候发送的；
- 当前消息处于什么状态。

### 推荐结构

每条消息建议包含：

- 头像或角色标识；
- 消息内容；
- 时间；
- 状态标签。

```tsx
function MessageItem({ msg }) {
  return (
    <div className={`message-row ${msg.role}`}>
      <div className="avatar">{msg.role === "user" ? "我" : "AI"}</div>
      <div className="bubble">
        <div className="meta">
          <span className="name">{msg.role === "user" ? "你" : "AI 助手"}</span>
          <span className="time">{msg.time}</span>
          <span className={`status ${msg.status}`}>{msg.statusText}</span>
        </div>
        <div className="content">{msg.content}</div>
      </div>
    </div>
  );
}
```

### 样式建议

```css
.message-row {
  display: flex;
  gap: 12px;
  padding: 12px 0;
  align-items: flex-start;
}

.message-row.user {
  flex-direction: row-reverse;
}

.avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: #e5e7eb;
  display: flex;
  align-items: center;
  justify-content: center;
  flex: 0 0 auto;
}

.bubble {
  max-width: 78%;
  padding: 12px 14px;
  border-radius: 12px;
  background: #f8fafc;
  word-break: break-word;
}

.message-row.user .bubble {
  background: #dcfce7;
}
```

消息间距别太小，不然内容会糊成一片；也别太大，否则聊天感会变弱。头像、时间、状态都要弱一点，不要抢正文内容。

如果前面已经做了 Markdown 渲染和代码高亮，这里直接把内容区替换成渲染组件即可，消息外壳保持统一，页面会更像真实产品。

### 截图说明思路

- 截图 1：用户和 AI 左右分布清晰，头像能快速区分角色；
- 截图 2：长消息自动换行，气泡不会顶出屏幕；
- 截图 3：时间和状态位置合理，不抢正文内容。

### 常见坑

- 气泡宽度写死，手机端容易溢出；
- 左右布局不统一，消息排列混乱；
- 时间和状态字体太重，压过正文；
- 长代码块或长链接没有换行处理。

**解决建议：** 气泡宽度用 `max-width` 控制，辅助信息用更弱的颜色和字号，保持层级清晰。

---

## 四、加载中、发送中、已完成状态怎么展示更清楚？

### 目标是什么

用户在聊天时最在意的是“系统有没有在工作”。如果你点了发送，页面一点反馈都没有，他会怀疑是不是没发出去。

所以状态提示一定要有，而且要清楚。通常可以分成三类：

1. `sending`：用户消息已提交，正在等待接口响应；
2. `streaming`：AI 正在逐步生成内容；
3. `done`：回复完成。

### 状态设计

```tsx
const statusMap = {
  sending: "发送中",
  streaming: "生成中...",
  done: "已完成",
};

function StatusTag({ status }) {
  return <span className={`status ${status}`}>{statusMap[status]}</span>;
}
```

### 和流式输出配合

接口开始返回内容时，把最后一条 AI 消息状态设为 `streaming`；流结束后，再改为 `done`。

```tsx
setMessages((prev) =>
  prev.map((m, i) =>
    i === prev.length - 1 ? { ...m, status: "streaming" } : m
  )
);
```

流式输出阶段再加一个闪动光标、转圈图标，或者“正在输入”的提示，用户会更容易理解当前正在发生什么。

按钮状态也要同步：发送中禁用按钮，生成完成再恢复可点。这样页面不会显得卡住。

```tsx
<button disabled={isSending || isStreaming}>
  {isStreaming ? "生成中..." : isSending ? "发送中..." : "发送"}
</button>
```

### 截图说明思路

- 截图 1：点击发送后，按钮进入加载状态；
- 截图 2：AI 正在生成时，消息区域显示“生成中...”；
- 截图 3：回复结束后，状态切换为“已完成”。

### 常见坑

- 只改按钮文案，不改消息状态，用户还是不确定是否成功；
- 流结束后忘了把状态改回 `done`；
- 加载态和禁用态没有区分，按钮看起来像坏掉了；
- 状态更新和滚动逻辑互相打架，导致页面重复跳动。

**解决建议：** 状态展示最好同时覆盖按钮、消息和输入框三个地方，让用户清楚当前发生了什么；状态变化和滚动动作分开处理，别在同一个时机里做太多事。

---

## 五、本章验收标准

做完这一章后，可以按下面清单逐条检查：

- 新消息出现后，页面能自动滚到底部；
- 用户手动上翻历史时，不会被强行拉回；
- 发送消息后，输入框自动聚焦；
- 连续输入和发送时，不需要反复点击输入框；
- 用户消息和 AI 消息样式明显区分；
- 消息气泡宽度合理，手机端不溢出；
- 发送中、生成中、已完成状态都能看出来；
- 流式输出时页面不会频繁抖动；
- 移动端打开后，输入框和消息列表都能正常使用。

---

## 小结：把“能聊天”变成“好聊天”

这一章看起来都是小优化，但它们决定了你的 AI 聊天网页是不是像一个真正可用的产品。自动滚动解决“看不见最新内容”的问题，自动聚焦解决“连续对话不顺手”的问题，状态反馈解决“用户不知道系统有没有在工作”的问题。

如果想继续把项目做得更完整，建议按这个顺序推进：

1. 先做自动滚动和输入框聚焦；
2. 再统一消息气泡和状态展示；
3. 最后检查移动端表现和流式输出时的滚动体验。

下一章，我们就把这些交互体验和消息历史、本地存储连起来，让这个 AI 聊天网页真正变成一个可以持续使用的应用。

# 第11章 手机上也能顺手用——移动端适配与响应式优化

做 AI 聊天网页，最容易被忽略的就是**手机端能不能真正常用**。很多项目电脑上没问题，一到手机上就会遇到：消息区太挤、输入框被键盘挡住、按钮太小点不到、长消息横向溢出、底部栏被安全区吃掉。  
这一章不讲理论，直接把聊天页面改到“手机也能顺手用”为止。

---

## 一、先把整体布局改成桌面和手机都能用

聊天页最核心的就是三块：顶部标题区、消息滚动区、底部输入区。桌面端可以宽一点，但移动端要优先保证：

- 消息区占主要空间
- 输入区始终可见
- 页面不横向滚动
- 文本和按钮不拥挤

推荐直接做成“上中下”三段式：

```jsx
export default function ChatPage() {
  return (
    <div className="chat-page">
      <header className="chat-header">全栈老赵 AI Chat</header>

      <main className="chat-main">
        <div className="message-list">{/* 消息列表 */}</div>
      </main>

      <footer className="chat-input-bar">
        <textarea className="chat-textarea" placeholder="请输入你的问题..." />
        <button className="chat-send-btn">发送</button>
      </footer>
    </div>
  );
}
```

```css
.chat-page {
  height: 100dvh;
  display: flex;
  flex-direction: column;
  background: #f7f7f8;
  overflow: hidden;
}

.chat-header {
  flex: 0 0 auto;
  padding: 12px 16px;
  font-weight: 600;
  border-bottom: 1px solid #e5e7eb;
}

.chat-main {
  flex: 1;
  min-height: 0;
  overflow: hidden;
}

.message-list {
  height: 100%;
  overflow-y: auto;
  padding: 16px;
  -webkit-overflow-scrolling: touch;
}

.chat-input-bar {
  flex: 0 0 auto;
  display: flex;
  gap: 8px;
  padding: 12px;
  border-top: 1px solid #e5e7eb;
  background: #fff;
}
```

关键点有两个：

1. `chat-main` 要加 `min-height: 0`，不然中间滚动区容易被内容撑开。  
2. `height: 100dvh` 比 `100vh` 更适合手机，更贴近真实可视区域。  

### 桌面端到移动端的布局变化规则

你可以把这个页面理解成“桌面版保留完整结构，移动版只做压缩和重排”：

- **顶部标题区**：保留，但缩小内边距和字号
- **消息区**：始终作为主要区域，尽量占满可视高度
- **底部输入区**：固定在底部，按钮更大、输入框更高
- **两侧留白**：桌面端保留更宽边距，手机端缩小边距
- **辅助按钮**：如“清空”“重新生成”，移动端可折叠或收进更多菜单

如果桌面端已经做了复杂布局，手机端不要硬塞，先保证“能聊天”。

### 截图说明思路

准备两张图对比：

- **桌面端截图**：三段布局完整，输入栏固定在底部
- **手机端截图**：消息区占满大部分屏幕，输入栏没有挤压内容

重点看结构是否稳定、底部输入区是否始终可见。

### 常见坑

- 只写 `height: 100vh`，手机地址栏变化时页面会抖动
- `main` 没有 `min-height: 0`，滚动区域可能失效
- 外层没设 `display: flex`，底部输入栏会乱跑

---

## 二、输入框、按钮、消息气泡要适合小屏操作

### 2.1 输入框要够大、好点、能换行

手机上不要把输入框做得像桌面搜索框，建议直接用 `textarea`，长问题和分行输入都更自然。

```jsx
<textarea
  className="chat-textarea"
  rows={1}
  placeholder="请输入内容，支持多行"
/>
```

```css
.chat-textarea {
  flex: 1;
  min-height: 44px;
  max-height: 120px;
  padding: 10px 12px;
  font-size: 16px; /* 避免 iOS 自动放大 */
  line-height: 1.5;
  resize: none;
  border: 1px solid #d1d5db;
  border-radius: 10px;
}
```

按钮也要保证手指好点：

```css
.chat-send-btn {
  min-width: 72px;
  height: 44px;
  padding: 0 16px;
  border-radius: 10px;
  touch-action: manipulation;
}
```

可以顺手支持回车发送、Shift+Enter 换行，电脑和手机体验会更统一。

### 2.2 消息气泡要适配窄屏

移动端最怕长文本把气泡撑爆，尤其是 AI 回复里常有链接、代码和长句子。建议统一这样处理：

```css
.message-item {
  max-width: 85%;
  word-break: break-word;
  overflow-wrap: anywhere;
  white-space: pre-wrap;
  line-height: 1.6;
  margin-bottom: 12px;
}

.message-user {
  margin-left: auto;
  background: #dbeafe;
}

.message-ai {
  margin-right: auto;
  background: #fff;
}
```

核心就是两点：  
一是 `max-width` 别太大；  
二是要允许内容换行，不然一个长链接就能把整页撑歪。

如果页面已经接了 Markdown 渲染和代码高亮，再加一层限制：

```css
.message-content pre,
.message-content code {
  max-width: 100%;
  overflow-x: auto;
}

.message-content {
  overflow-wrap: anywhere;
  word-break: break-word;
}
```

代码块保留横向滚动，不要强行折行，否则手机上反而更难读。

### 截图说明思路

重点看三件事：

- 输入框是否够高，是否容易点
- 按钮是否被压得太小
- 长文本是否会横向溢出

可以拿一条包含网址、代码片段、列表项的 AI 回复来测试，最容易看出适配效果。

### 常见坑

- 字体小于 `16px` 会触发 iPhone 页面缩放
- 气泡没做换行，长链接会把页面撑宽
- 按钮太小，手指点击容易误触

---

## 三、解决移动端键盘弹出遮挡输入区的问题

手机上点输入框后，系统键盘会弹出，视口高度也会变化。如果还死守 `100vh`，底部输入栏很可能被键盘盖住。这个问题不处理，移动端体验会直接翻车。

最实用的办法，是监听 `visualViewport`，在键盘出现时把底部输入栏往上顶一点：

```jsx
import { useEffect, useState } from "react";

export function useKeyboardOffset() {
  const [offset, setOffset] = useState(0);

  useEffect(() => {
    const viewport = window.visualViewport;
    if (!viewport) return;

    const update = () => {
      const keyboardHeight =
        window.innerHeight - viewport.height - viewport.offsetTop;
      setOffset(Math.max(0, keyboardHeight));
    };

    update();
    viewport.addEventListener("resize", update);
    viewport.addEventListener("scroll", update);

    return () => {
      viewport.removeEventListener("resize", update);
      viewport.removeEventListener("scroll", update);
    };
  }, []);

  return offset;
}
```

在输入栏上使用：

```jsx
const keyboardOffset = useKeyboardOffset();

<footer
  className="chat-input-bar"
  style={{ transform: `translateY(-${keyboardOffset}px)` }}
>
  ...
</footer>
```

思路很直接：键盘上来多少，就把输入栏往上抬多少。不同设备表现会有差异，但对实战项目已经够用。

如果想更稳一点，可以在键盘弹出时配合滚动到底部，保证最新消息和输入栏都还在视线里。目标只有一个：**用户点输入框后，输入栏不能消失**。

### iOS 键盘和滚动联动怎么处理

iOS 上最常见的问题不是“键盘弹不出”，而是弹出后页面整体跟着缩、滚动容器位置乱掉。这里有两个实用建议：

1. **不要让外层页面自己滚**，只让消息区滚动。  
2. **键盘弹出后不要强行改很多层 `position: fixed`**，优先用 `transform` 调整输入栏，减少布局重排。  

如果你发现某台 iPhone 上输入栏还是被顶偏了，先检查是不是外层容器、消息区、输入栏三层都在抢滚动。

### 截图说明思路

准备两张状态图：

- 键盘未弹出：底部输入栏贴底显示
- 键盘弹出：输入栏仍可见，消息区被压缩但不乱

这两张图最能说明处理是否有效。

### 常见坑

- 只写 `bottom: 0` 不够，键盘会覆盖底栏
- Android 和 iOS 表现不同，别只在一个设备上测
- 输入栏用 `position: fixed` 时，要注意和滚动容器冲突
- iOS 上外层可滚动容器太多，容易出现“顶上去了但点不到”的问题

---

## 四、别忘了安全区域、滚动容器和长内容处理

### 4.1 安全区域要留出来

刘海屏手机底部有安全区，建议给输入栏加上环境变量内边距：

```css
.chat-input-bar {
  padding-bottom: calc(12px + env(safe-area-inset-bottom));
}
```

也可以给整页容器补一点底部空间：

```css
.chat-page {
  padding-bottom: env(safe-area-inset-bottom);
}
```

这一步很小，但很关键。很多项目在模拟器里正常，上真机后底部按钮却贴得太死，甚至被系统手势区域影响。

### 4.2 消息区单独滚动，别让整页一起滚

聊天页最舒服的方式不是整页滚，而是中间消息区自己滚。这样用户上下滑消息时，不会把输入栏一起拖乱。

推荐结构就是：

- 外层页面：固定布局
- 中间消息区：`overflow-y: auto`
- 底部输入栏：固定在底部

如果消息很多，建议在新消息进入后自动滚到底部，保证用户看到最新回复。

### 4.3 长文本、链接、代码块要兜住

AI 回复里经常有长网址、列表和代码。你要保证它们不会把整个页面撑破：

- 普通文本：允许换行
- 长链接：允许折行
- 代码块：横向滚动，不强行压缩

这样手机上看起来才不会乱。

### 截图说明思路

可以准备一张“长消息测试图”：内容里同时包含链接、代码块、列表。看它是否会撑破布局，最能验证手机端是否稳定。

### 常见坑

- 消息区和整页同时滚动，用户会觉得卡
- 没给安全区留白，底部按钮容易贴边
- 代码块强行换行，反而影响阅读
- 长列表消息没有限制宽度，消息气泡会被撑得太宽

---

## 五、响应式不是只靠一两个媒体查询

很多人一提响应式，就想到写个 `@media (max-width: 768px)`。其实聊天页更重要的是“结构能缩、组件能跟着变”。

你可以这样做：

```css
.chat-page {
  max-width: 960px;
  margin: 0 auto;
}

@media (max-width: 768px) {
  .chat-page {
    max-width: 100%;
  }

  .chat-header {
    padding: 10px 12px;
    font-size: 14px;
  }

  .message-list {
    padding: 12px;
  }

  .chat-input-bar {
    padding: 10px;
    gap: 6px;
  }

  .chat-send-btn {
    min-width: 64px;
  }
}
```

如果你还放了侧边栏、会话列表、设置面板，手机端最好直接收起来：

- 桌面端：左侧会话列表 + 右侧聊天区
- 手机端：默认只展示聊天区
- 会话列表放到抽屉里，通过按钮打开

这样页面不会在小屏上被挤成一团。

### 上线前快速自测一遍

你可以按下面清单过一遍：

- [ ] 小屏下没有横向滚动条
- [ ] 输入框高度合适，字体不被缩放
- [ ] 发送按钮可轻松点击
- [ ] 键盘弹出后输入栏仍可见
- [ ] 消息区可独立滚动
- [ ] 长消息、链接、代码块不会撑破页面
- [ ] 底部安全区留白正常
- [ ] iPhone 和 Android 都测过一遍
- [ ] 桌面端和手机端是同一套代码，不是两套重复页面

建议先用浏览器开发者工具切换手机尺寸，再找一台真手机做最后确认。模拟器能发现大多数布局问题，但键盘、地址栏、安全区这些细节，真机更靠谱。

### 本章验收标准

做到下面几点，就说明这一章基本完成了：

- 手机打开页面后，聊天界面能正常显示
- 输入框不会被键盘挡住
- 消息不会横向溢出
- 底部按钮可点击，安全区不压住内容
- 桌面端和移动端都有正常的聊天体验

---

## 本章小结

移动端适配不是最后顺手调一下，而是聊天网页能不能真正可用的关键。你只要抓住四件事就够了：

1. **布局要自适应**，消息区、输入栏分层明确  
2. **控件要好操作**，按钮大一点、输入框能换行  
3. **键盘弹出要处理**，别让底部输入栏被遮住  
4. **长内容要兜住**，换行、滚动、安全区一起考虑  

### 下一步建议

如果你已经把移动端改好了，下一章就可以继续做两件最实用的事：  
- 让对话体验更顺滑，比如自动滚动、清空、重新生成  
- 把项目部署到 Vercel，让别人真的能在手机上打开你做的 AI 聊天网页

# 第12章 上线给别人看——部署到 Vercel 与常见问题排查

做到这一步，你的 AI 聊天网页就不只是本地练习，而是一个可以发链接给别人看的成品。这一章不追求继续加功能，重点是把**生产环境配置、构建部署、线上验证、问题排查**一次做对。  
你只要跑通这四件事：**能上线、能打开、能聊天、能排错**，就完成了从 0 到 1 的最后闭环。

---

## 12.1 部署前先整理生产环境配置

先把“本地能用、线上也能用”的基础打牢，核心是把接口地址、开关项从代码里拿出来，放进环境变量。

这样做的好处很直接：代码更干净，后面换接口、换模型、换部署环境时，不用到处改代码。

### 先分清这几类配置

- **本地开发配置**：`.env.local`
- **线上生产配置**：Vercel 控制台里的 Environment Variables
- **前端可公开变量**：需要浏览器读取的，才加 `VITE_` 前缀
- **不能暴露的密钥**：不要写进前端代码，也不要提交到仓库

如果你用的是 Vite + React，前端通常只能读取带前缀的环境变量。前缀不对，页面里就拿不到值。

### 示例：前端项目常见配置

```env
# .env.local
VITE_API_BASE_URL=https://api.example.com
VITE_APP_NAME=AI Chat Demo
VITE_ENABLE_STREAM=true
```

请求时统一读取：

```js
const baseURL = import.meta.env.VITE_API_BASE_URL;
```

如果还有模型名称、主题开关、是否启用流式输出，也可以统一放到环境变量里，方便测试和上线切换。

### 部署前检查清单

- [ ] 接口地址没有写死在组件里
- [ ] `.env.local` 已加入 `.gitignore`
- [ ] 线上生产环境变量已经在 Vercel 配好
- [ ] 前端读取变量时带了正确前缀
- [ ] 密钥没有提交到 GitHub

### 截图说明思路

建议截图两张：

1. **本地 `.env.local` 文件**
   - 标出 `VITE_API_BASE_URL`
   - 说明这是本地环境地址
2. **Vercel 环境变量配置页**
   - 标出 Production / Preview 的区别
   - 说明线上部署要单独配置

### 常见坑

- 把 API 密钥直接写进 `src` 目录代码
- 本地用了 `.env.local`，线上却忘了配环境变量
- 变量名写错，比如少了 `VITE_` 前缀
- 改了环境变量后没重新部署，线上还是旧值

**本章验收标准：**你能明确说出哪些配置放本地，哪些配置放 Vercel，页面里读取到的地址不是写死的。

---

## 12.2 用 Vercel 完成首次发布

如果你想最快上线，Vercel 是最省事的选择。它适合 React 静态站点，基本可以做到“连仓库、点一下、自动上线”。

### 操作步骤

1. 把项目推到 GitHub
2. 登录 Vercel
3. 点击 **New Project**
4. 选择你的仓库
5. 确认框架识别正确
6. 填入环境变量
7. 点击部署

第一次做时，建议先只部署前端，先保证网页能打开、能发消息、能显示返回结果，再考虑更多高级能力。

### React 项目初始化后要确认的构建命令

如果是 Vite 项目，通常是：

```bash
npm run build
```

输出目录一般是：

```txt
dist
```

如果你用的是其他脚手架，记得确认 Vercel 的 Build Command 和 Output Directory 是否匹配。很多“部署后白屏”，其实不是代码错，而是平台没找到正确的构建结果。

### 可直接参考的 `package.json` 片段

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

提交前最好先在本地跑一次构建：

```bash
npm run build
npm run preview
```

`preview` 能帮你提前看到生产构建后的页面，尽早发现开发模式下看不出来的问题。

### 截图说明思路

建议截图：

- Vercel 项目创建页
- 首次部署成功页
- 线上访问地址打开后的首页效果

重点是让读者看懂：

- 项目已成功构建
- 地址已经可访问
- 首页能正常渲染聊天界面

### 常见坑

- 本地 `npm run dev` 正常，但 `npm run build` 失败
- 构建命令填错，导致部署卡住
- 输出目录写错，导致页面空白
- 仓库依赖有问题，线上安装时失败

**本章验收标准：**你能从 GitHub 发起部署，并且拿到一个可以打开的线上地址。

---

## 12.3 逐项验证线上接口、路由、静态资源与跨域

部署成功不代表能正常聊天。接下来要做“逐项验收”。不要只看首页能打开，真正要测的是：消息能不能发出去、能不能回来、页面刷新会不会坏。

### 先检查四件事

#### 1）接口是否真的连上了

打开浏览器开发者工具，看 Network：

- 请求有没有发出去
- 返回码是不是 200
- 返回内容是不是你想要的格式

如果是流式响应，还要看是否持续收到分片数据。  
有时页面像“卡住了”，其实是接口根本没返回，或者返回格式和前端解析方式不一致。

#### 2）路由是否正常

如果你用了 React Router，直接刷新某个子路由，看看是否还会 404。

静态站点上需要确认平台支持 SPA 回退，否则刷新二级路由可能报错。聊天应用首页通常问题不大，但后面加了 `/history`、`/settings` 之类路由，就很容易踩坑。

#### 3）静态资源是否加载成功

检查图片、图标、样式文件有没有 404。

很多白屏问题，本质不是代码错，而是资源路径不对。尤其是本地开发时用相对路径，线上构建后路径变化了，就会出现“页面能开，但样式丢了”。

#### 4）跨域是否允许

前端请求后端 API 时，如果接口不在同域，后端必须允许跨域，或者前端通过代理/网关处理。  
本地用了开发代理，线上没对应配置，就会出现“本地没问题，线上请求失败”。

### 接口封装示例

```js
export async function sendChatMessage(messages) {
  const res = await fetch(`${import.meta.env.VITE_API_BASE_URL}/chat`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({ messages })
  });

  if (!res.ok) {
    throw new Error(`请求失败：${res.status}`);
  }

  return res;
}
```

建议把所有请求都封装在统一的 `api` 文件里，不要散在组件中。这样接口地址、请求头、错误提示以后都能集中改。

### 截图说明思路

建议截三类图：

- Network 面板里请求成功的记录
- 页面刷新后路由仍可打开的效果
- 控制台里没有报错的状态

截图时圈出状态码、请求地址、返回结果，让读者知道“看到请求不算成功，拿到正确数据才算成功”。

### 常见坑

- 接口地址写死成本地地址
- 后端没开跨域，浏览器直接拦截
- 路由刷新 404，但点按钮跳转正常
- 静态资源使用了错误相对路径

**本章验收标准：**你能在浏览器里确认消息请求真的发出去了，刷新页面后路由不乱，资源不丢。

---

## 12.4 线上常见问题：白屏、404、环境变量失效、接口报错

上线后最常见的不是“功能不完整”，而是“看起来什么都不对”。这时不要慌，按症状排查。

### 问题一：打开页面白屏

先看这三处：

1. 浏览器控制台有没有红色报错
2. `build` 是否真的成功
3. 组件是否有运行时错误

常见原因：

- 某个空值直接调用方法
- 环境变量是 `undefined`
- Markdown 渲染插件引入错误
- CSS 或 JS 资源路径错误

白屏时先看第一条报错，通常就是线索。若报错在某个组件里，先临时注释可疑代码，逐步缩小范围。

### 问题二：刷新页面 404

这通常是路由问题，不是没部署成功。

解决思路：

- 确认平台已开启 SPA 路由回退
- React Router 使用 `BrowserRouter` 时要特别注意
- 如果平台不方便配置，可临时改用哈希路由

点击能进、刷新就坏，是典型前端路由和服务器回退没配好的表现。

### 问题三：环境变量失效

检查顺序：

- 变量名是否写对
- 是否重新部署
- 是否放在正确的环境选项里
- 前端是否按平台要求加了前缀

刚改完环境变量但页面没变化，先别怀疑代码，先确认有没有重新触发部署。很多平台不会自动把新变量注入到旧构建里。

### 问题四：接口报错

看失败原因：

- 401：鉴权失败，通常是密钥或 token 问题
- 403：权限不足或后端限制
- 404：接口路径错了
- 500：后端内部错误
- CORS：跨域没配好

前端错误提示要清晰，比如“接口暂时不可用，请稍后重试”，不要只在控制台打印 `Failed to fetch`。上线后用户不关心内部日志，只关心还能不能继续用。

### 推荐排查顺序

如果你遇到线上问题，建议按下面顺序查，不要乱跳：

1. 先看 **Vercel 构建日志**
2. 再看 **环境变量是否生效**
3. 再看 **接口地址是否写对**
4. 再看 **浏览器控制台和 Network**
5. 最后再看 **跨域和后端返回值**

这个顺序最省时间，因为很多问题其实都不是代码逻辑错，而是部署配置错。

### 截图说明思路

建议截图：

- 白屏时的控制台报错
- Vercel 构建失败日志
- Network 面板里的失败请求

这样读者能知道：排查不是“猜”，而是按日志和请求一步一步缩小范围。

### 常见坑

- 只看页面不看控制台
- 以为“部署成功”就等于“聊天可用”
- 遇到报错先改代码，没先确认接口和环境变量
- 后端接口升级了，前端请求路径没同步改

**本章验收标准：**你能根据报错类型判断问题大概在构建、环境变量、路由，还是接口本身。

---

## 12.5 上线前最后过一遍检查清单

上线前别凭感觉，按清单扫一遍最稳。这样能避免“已经发链接给别人，结果别人一打开就报错”的尴尬。

### 检查清单

- [ ] `npm run build` 本地能成功
- [ ] 环境变量已在 Vercel 配好
- [ ] 前端请求地址不是本地地址
- [ ] 首屏能正常打开
- [ ] 输入框可以发送消息
- [ ] 流式输出正常显示
- [ ] Markdown 渲染和代码高亮正常
- [ ] 刷新页面不会 404
- [ ] 移动端打开不乱版
- [ ] 控制台没有明显报错

这里特别提醒两项：

- **流式输出**要在线上和真机都看一下，因为开发环境没问题，线上代理或缓存也可能影响分片显示。
- **移动端适配**不要只看浏览器缩放，最好用手机真实打开一次，确认输入框、发送按钮、聊天气泡都不挤压。

### 截图说明思路

最值得保留的三张图：

1. **部署成功页**
   - 证明项目已上线
2. **线上聊天效果图**
   - 证明功能可用
3. **报错排查图**
   - 证明你会定位问题，而不是只会点部署按钮

这些截图不仅能放在文档里，也能用来给别人演示项目完成度。

### 部署成功后的验收建议

如果你想确认“这个项目真的能交给别人看”，可以按下面顺序验收：

1. 复制线上链接到无痕窗口打开
2. 发送一条短消息，确认返回正常
3. 刷新页面，确认路由不丢
4. 手机上打开一次，确认输入框和按钮可用
5. 断开本地开发环境，确认线上仍然可访问

做到这一步，才算真正从“本地能跑”走到了“线上能展示”。

---

## 12.6 本章小结：先上线，再优化

这章的核心不是“把项目发布出去”这么简单，而是让你形成一套最小闭环：

- 本地开发能跑
- 生产配置能分离
- Vercel 能部署
- 线上接口能通
- 路由和资源不出错
- 出问题知道怎么查

如果你已经完成这一步，恭喜你，你的 AI 聊天网页已经具备了“可展示、可分享、可继续迭代”的基础。接下来你可以继续做的方向很明确：

- 加登录与用户会话
- 接入多模型切换
- 增加对话导出
- 做更好的主题和移动端体验

**记住一句话：上线不是终点，而是产品开始被别人使用的起点。**

### 本章行动建议

- 今天先把项目部署到 Vercel
- 明天补一次线上真机测试
- 后天再整理一版报错排查记录

这样你不只是“部署过一次”，而是真的掌握了从本地到上线的完整路径。

---

更多内容请访问：[https://tutor.lao-zhao.com/](https://tutor.lao-zhao.com/)
