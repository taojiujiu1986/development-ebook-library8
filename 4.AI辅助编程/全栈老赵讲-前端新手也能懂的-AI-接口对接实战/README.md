# 全栈老赵讲 前端新手也能懂的 AI 接口对接实战

<!-- PAGEBREAK -->

![作者介绍图](05_full_book_draft_assets/asset-8534aff2fa.png)

<!-- PAGEBREAK -->

## 目录

- 第1章 先别急着写代码：先看懂 AI 接口文档在讲什么
- 第2章 请求参数与返回格式：前端到底要传什么、拿什么回来
- 第3章 第一次把接口接到页面上：用 fetch 跑通最小闭环
- 第4章 别让请求到处散落：用 axios 和封装把对接变简单
- 第5章 密钥、环境变量与鉴权：前端怎么安全地把门守住
- 第6章 异步流程与 UI 状态管理：让页面知道自己正在等结果
- 第7章 流式输出与非流式输出：AI 回复为什么会一边生成一边显示
- 第8章 错误码、异常和重试：接口失败时前端该怎么稳住
- 第9章 频控、限流与防刷：前端能做的保护措施有哪些
- 第10章 跨域基础与接口调试方法：请求发不出去时先查什么
- 第11章 把前面所有步骤串起来：做一个可上线的 AI 接口接入小项目
- 第12章 收尾与排查手册：新手接 AI 接口时最常见的问题总整理

# 第1章 先别急着写代码：先看懂 AI 接口文档在讲什么

先说清楚：**本书只讲前端如何读文档、发请求、处理返回，不讲后端接口设计。**  
你是前端新手，看到接口文档发怵很正常，但别一上来就复制 URL 开跑。接口对接最容易出问题的，就是顺序反了——文档没看懂，代码先写了，后面不是请求发不出去，就是拿到一堆看不懂的报错。

这章老赵先不写复杂页面，只带你把接口文档拆开看。你会发现，它其实就几块：**地址、方法、请求头、请求体、返回体、错误码**。把这几块看明白，前端接入就有底了。

## 一、先按固定顺序读文档

看文档别凭感觉，按这个顺序来最稳：

1. **接口地址**：请求往哪发  
2. **请求方法**：GET、POST 还是别的  
3. **请求头**：比如 `Content-Type`、`Authorization`  
4. **鉴权方式**：要不要 API Key、Token，放在哪个头里  
5. **请求体**：要传哪些参数，哪些必填  
6. **返回体**：成功结果放哪，字段怎么取  
7. **错误码**：失败时怎么判断原因  
8. **请求限制**：跨域、频控、大小限制、是否流式

### 最小可运行示例

先别做复杂 UI，先用最简单的请求验证接口能不能通：

```js
fetch("https://api.example.com/v1/chat", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer YOUR_KEY"
  },
  body: JSON.stringify({
    prompt: "你好"
  })
})
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

### 错例对比

```js
fetch("https://api.example.com/v1/chat", {
  method: "GET",
  body: { prompt: "你好" }
});
```

这个写法常见问题是：**方法不对、body 格式不对、没带请求头**。  
如果文档要求 `POST`，你却用 `GET`，接口大概率直接拒绝；如果要求 JSON，你却直接塞对象，浏览器也不会自动帮你转。

### 排错清单

- URL 是否抄对
- 方法是否和文档一致
- `Content-Type` 是否正确
- 鉴权信息是否放在正确的请求头
- 请求体是否需要 `JSON.stringify`

## 二、从前端视角判断：能不能直接接

前端最关心的不是“这个接口设计得漂不漂亮”，而是：**我能不能在浏览器里直接调它？**

要重点看三件事：

- **是否需要密钥**：如果需要，不能直接写死在前端源码里
- **是否允许跨域**：浏览器会检查 CORS
- **是否有限流**：按钮连点会不会被封

如果接口要求把密钥放在前端代码里，先别激动，这通常意味着**有安全风险**。浏览器里能看到的代码，用户基本都能看到，真正的密钥不要直接硬编码进去。

### 最小可运行示例

先只做一个按钮，点击后请求接口：

```html
<button id="btn">测试接口</button>
<pre id="out"></pre>
<script>
  const btn = document.getElementById("btn");
  const out = document.getElementById("out");

  btn.onclick = async () => {
    out.textContent = "请求中...";
    try {
      const res = await fetch("https://api.example.com/v1/chat", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ prompt: "你好" })
      });
      out.textContent = JSON.stringify(await res.json(), null, 2);
    } catch (e) {
      out.textContent = "请求失败";
    }
  };
</script>
```

### 错例对比

- 一上来就做复杂表单
- 还没确认跨域就开始联调
- 明知道要鉴权，却没看请求头说明
- 把 API Key 直接写进前端源码里

### 排错清单

- 前端能不能直接请求
- 是否会跨域
- 是否需要后端代转
- 是否有限流
- 是否存在密钥泄露风险

## 三、先看鉴权、返回格式，再决定页面怎么写

很多人喜欢先画 UI，再研究接口。更稳妥的顺序是：**先看接口，再决定页面形态。**

比如返回值如果是这种结构：

```json
{
  "code": 0,
  "data": {
    "reply": "你好，我是 AI"
  }
}
```

前端就知道要展示 `data.reply`。  
如果接口返回的是流式数据，页面就不能只等一次性返回，而要准备**边收边显示**的 UI 状态，也就是 loading、生成中、停止按钮这些交互都得提前想好。这里有个关键差异：**非流式接口一次拿完整结果，流式接口是一段一段到达。**

另外，别只盯着 HTTP 200。很多 AI 接口会把业务错误放在 `code` 里，HTTP 看着成功，实际上内容已经失败了。前端要同时看 **HTTP 状态** 和 **业务字段**。

### 错例对比

- 以为接口返回的是纯文本，结果实际是 JSON
- 以为成功就一定是 200，结果业务错误藏在 `code` 里
- 以为一次请求一次返回，结果文档写的是流式
- 没看清返回字段路径，页面直接取错值

### 排错清单

- 成功标记是 HTTP 状态码，还是业务 `code`
- 返回体有没有固定字段
- 是否需要按字段路径取值
- 是否支持流式
- 页面是否要预留 loading 和错误提示

## 四、把“看不懂文档”拆成固定检查步骤

老赵建议你每次按这个顺序看：

1. **看接口用途**：它是干什么的  
2. **看请求头**：要带哪些鉴权信息  
3. **看请求体**：必填字段有哪些  
4. **看返回体**：成功时怎么取值  
5. **看错误码**：失败时怎么判断  
6. **看限制**：跨域、频控、流式、大小限制

你会发现，“看不懂”往往不是文档难，而是没有固定拆解顺序。前端调接口，最怕凭感觉写，最稳的是先把文档翻成自己的检查清单。

### 最小可运行示例

拿到文档后，先只实现“发出去 + 打印返回”：

```js
async function testApi() {
  const res = await fetch("https://api.example.com/v1/chat", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": "Bearer YOUR_KEY"
    },
    body: JSON.stringify({ prompt: "测试" })
  });
  const data = await res.json();
  console.log("状态码:", res.status);
  console.log("返回体:", data);
}
testApi();
```

### 错例对比

- 先写渲染逻辑，后看接口结构
- 只看成功示例，不看错误码
- 只看请求体，不看请求头
- 连接口是否跨域都没确认，就直接改页面

### 排错清单

- 是否已经明确鉴权方式
- 是否知道成功和失败怎么区分
- 是否知道返回字段路径
- 是否确认这是非流式还是流式接口
- 是否准备好接口调试工具，比如浏览器控制台、Network 面板

## 五、接口调试时，前端具体看什么

别小看浏览器的 **Network 面板**，它就是前端调接口时最实用的放大镜。你要重点看：

- **Request Headers**：有没有带 `Authorization`
- **Request Payload**：请求体有没有按文档传
- **Response**：返回结构是不是你以为的样子
- **Status Code**：是 200、401、429，还是别的
- **跨域报错**：控制台有没有 CORS 提示

如果你连这些都不看，只盯着页面显示，很多问题会被误判成“代码写错了”。

### 最小可运行示例

```js
fetch("https://api.example.com/v1/chat", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer YOUR_KEY"
  },
  body: JSON.stringify({ prompt: "测试调试" })
}).then(async (res) => {
  console.log("status:", res.status);
  console.log("headers:", [...res.headers.entries()]);
  console.log("body:", await res.json());
});
```

### 错例对比

- 只看页面提示，不看 Network
- 401 了还以为是前端渲染问题
- 429 了还在疯狂点按钮
- CORS 报错了却一直怀疑请求参数

### 排错清单

- 控制台有没有跨域错误
- 请求头里有没有鉴权字段
- 返回体是否能被正常解析
- 状态码是否符合预期
- 是否因为频繁点击触发限流

## 六、先验证接口可用，再谈页面联调

新手最容易犯的错误，是一上来就把问题想复杂。其实第一步只要做到：**发一个最简单的请求，确认接口能通。**

这一步的目标不是做得漂亮，而是确认：

- 地址对不对
- 鉴权对不对
- 请求头对不对
- 返回格式是不是你以为的样子
- 有没有跨域问题
- 响应到底是文本、JSON，还是流式数据

如果这一步通了，后面再做 loading、错误提示、流式展示，就顺很多。你只需要把“请求成功”和“请求失败”两种状态，清清楚楚地呈现在 UI 里。

### 结尾小结

记住今天这句话：**先读文档，再写代码；先验证接口，再做页面。**

对前端来说，AI 接口对接不是“会不会写请求”这么简单，而是你能不能先看懂：**请求头、鉴权、请求体、返回体、错误码、跨域限制、流式与非流式差异**。下一章我们就从请求参数和返回格式开始，继续把这个流程拆透。

如果你现在就要动手，先做一件事：打开一份接口文档，按今天这套顺序，把每一项圈出来。只要你能把文档翻译成自己的检查清单，后面接 AI 接口就不会再那么慌了。

# 第2章 请求参数与返回格式：前端到底要传什么、拿什么回来

第一次看 AI 接口文档，新手最容易卡住的不是“怎么写代码”，而是“我到底要传什么，它又会回什么”。  
前端接 AI 服务，本质上就是把**请求参数**和**返回格式**准确对应起来：该放 URL 的别塞进 body，该放请求头的别写到 JSON 里；返回结果也不能只盯着一个 `data`。

这章只站在**前端视角**讲一件事：**怎么读懂接口文档、怎么发对请求、怎么拿对返回值**。你把这一章吃透，后面无论是 `fetch`、`axios`，还是流式输出、错误处理，都会顺很多。

---

## 一、请求参数分三层：URL 参数、请求头、请求体

前端发请求时，先按这三层判断参数放哪：

### 1. URL 参数
放在地址后面，适合表示附加条件。

```txt
/api/chat?model=gpt-4&stream=false
```

常见字段：

- `model`
- `stream`
- `page`
- `limit`

### 2. 请求头
请求头负责“身份”和“格式说明”，不承载业务正文。

AI 接口里最常见的有：

- `Authorization`：鉴权
- `Content-Type`：请求体格式
- `Accept`：期望返回格式

### 3. 请求体
真正的业务内容，比如：

- prompt
- 历史消息
- 图片地址
- 模型参数

判断标准很简单：

- 影响接口行为但不是正文的，优先放请求头或 URL
- 真正要让 AI 处理的内容，通常放请求体

### 最小可运行示例

```js
fetch('/api/chat?stream=false', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: 'Bearer your_token',
    Accept: 'application/json'
  },
  body: JSON.stringify({
    prompt: '帮我总结这段文字',
    model: 'demo-model'
  })
})
```

### 错例对比

**错例：把所有东西都塞进 body**

```js
body: JSON.stringify({
  urlParam: 'stream=false',
  auth: 'Bearer xxx',
  prompt: '你好'
})
```

问题：

- `Authorization` 不该放 body
- `stream=false` 本该按文档放 URL 或指定位置
- 调试混乱，服务端通常也不会这样读

### 排错清单

- URL 是否拼对
- 参数放的位置是否符合文档
- `method` 是否一致
- `body` 是否已 `JSON.stringify`
- 是否漏了 `Content-Type`
- 参数名有没有拼错

---

## 二、请求头里最常见的三个字段：Content-Type、Authorization、Accept

这三个请求头决定了接口能不能正确识别你的请求。

### 1. Content-Type
告诉服务端 body 是什么格式。发 JSON 时通常写：

```js
'Content-Type': 'application/json'
```

### 2. Authorization
鉴权字段，常见写法：

```js
Authorization: 'Bearer your_api_key'
```

注意：

- 这是**鉴权**，不是普通参数
- 前缀必须按文档来
- 最好在 Network 面板里确认它真的发出去了

### 3. Accept
告诉服务端你想接收什么类型的数据：

```js
Accept: 'application/json'
```

流式接口里常见：

```js
Accept: 'text/event-stream'
```

### 最小可运行示例

```js
fetch('/api/chat', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: 'Bearer demo-key',
    Accept: 'application/json'
  },
  body: JSON.stringify({ prompt: '你好' })
})
```

### 错例对比

**错例：少了 Authorization**

```js
headers: {
  'Content-Type': 'application/json'
}
```

常见结果：

- 401
- 403
- “未授权”“缺少令牌”

### 排错清单

- `Authorization` 名字是否拼对
- token 前缀是否符合文档
- `Content-Type` 是否和 body 一致
- `Accept` 是否和返回类型匹配
- Network 里是否真正发出

---

## 三、返回值常见结构：data、message、code、error、usage

很多新手看返回值，只盯着 `data`，一报错就不知道去哪找。实际上，AI 接口返回通常会有这些字段。

### 1. `data`
核心结果，真正要渲染的内容通常在这里。

### 2. `message`
提示信息，可能是成功提示，也可能是错误说明。

### 3. `code`
业务状态码，不一定等于 HTTP 状态码，具体看文档。

### 4. `error`
错误对象，失败时常直接给这里。

### 5. `usage`
用量信息，常见于 token 统计。

### 统一示例返回结构

```json
{
  "code": 0,
  "message": "ok",
  "data": {
    "text": "你好，我可以帮你。"
  },
  "error": null,
  "usage": {
    "input_tokens": 12,
    "output_tokens": 8,
    "total_tokens": 20
  }
}
```

你后面看到别的接口，只要抓住这几个字段的角色就行：

- `code`：先判断成败
- `message`：看提示
- `data`：拿结果
- `error`：看失败原因
- `usage`：看消耗情况

### 最小可运行示例

```js
const res = await fetch('/api/chat', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: 'Bearer demo-key'
  },
  body: JSON.stringify({ prompt: '写一句欢迎语' })
})

const json = await res.json()
console.log('原始返回：', json)
console.log('结果文本：', json.data?.text)
console.log('提示信息：', json.message)
console.log('用量：', json.usage)
```

### 错例对比

**错例：默认认为返回一定有 `data.text`**

```js
const text = json.data.text
```

问题：

- 有些接口是 `result`
- 有些是 `choices[0].message.content`
- 有些失败时只有 `error`
- 不能凭习惯猜字段，必须先看原始返回

### 排错清单

- 先 `console.log` 原始返回
- 查清成功和失败时字段是否一致
- 看 `data` 外还有没有 `choices`、`result`、`output`
- `usage` 是否在成功响应里
- `code` 是业务码还是 HTTP 状态码

---

## 四、非流式与流式返回：字段长得不一样，处理方式也不一样

### 非流式返回
一次请求，一次完整响应，前端直接拿完整 JSON。

```json
{
  "code": 0,
  "message": "ok",
  "data": {
    "text": "你好，我可以帮你。"
  },
  "error": null,
  "usage": {
    "input_tokens": 12,
    "output_tokens": 8
  }
}
```

适合：

- 简单问答
- 不追求实时显示
- UI 先等结果再渲染

### 流式返回
内容一段段回来，前端边收边显示。常见于 SSE 或类似流式协议。

```txt
data: {"code":0,"message":"ok","data":{"delta":"你"}}
data: {"code":0,"message":"ok","data":{"delta":"好"}}
data: {"code":0,"message":"done","data":{"done":true}}
```

这时前端不能再按“一次性解析完整 JSON”的思路写，而要边接收边拼接，UI 也要支持逐步刷新。

### 差异对比表

| 项目 | 非流式 | 流式 |
|---|---|---|
| 返回方式 | 一次性返回完整 JSON | 分段返回增量内容 |
| 前端读取 | `await res.json()` | `reader.read()` 或事件流监听 |
| 页面展示 | 等全部结束再显示 | 边返回边显示 |
| 适合场景 | 简单结果 | 打字机效果、长文本生成 |
| 字段特点 | 常有完整 `data/text` | 常见 `delta/chunk/done` |

### 最小可运行示例

**非流式：**

```js
const res = await fetch('/api/chat', { method: 'POST', headers, body })
const json = await res.json()
console.log(json.data)
```

**流式：**

```js
const res = await fetch('/api/chat?stream=true', { method: 'POST', headers, body })
const reader = res.body.getReader()
const decoder = new TextDecoder()

while (true) {
  const { done, value } = await reader.read()
  if (done) break
  console.log(decoder.decode(value))
}
```

### 错例对比

**错例：把流式接口当普通 JSON 解析**

```js
const json = await res.json()
```

问题：

- 流式响应通常不是一次性 JSON
- 容易直接报错
- 页面会一直 loading，或者抛异常

### 排错清单

- 先确认接口是否声明 `stream=true`
- 看响应类型是 JSON 还是事件流
- 流式接口不要直接 `res.json()`
- UI 是否支持增量更新
- 是否有结束标记可判断

---

## 五、从接口文档里提取前端真正需要的字段

接口文档很长，但前端真正要抓住的只有几件事：**请求地址、请求方法、请求头、请求体和返回结构**。

### 步骤 1：先找请求方式
是 `GET`、`POST` 还是别的？

这一步决定参数放哪里。很多新手一上来就写 `POST`，结果文档明明是 `GET`，请求自然不对。

### 步骤 2：看参数放哪
常见位置：

- query 参数
- headers
- body
- path 参数

比如：

- `stream` 可能是 query
- `Authorization` 一定在 headers
- `prompt` 多半在 body
- `id` 可能在路径里：`/api/chat/:id`

### 步骤 3：只圈出前端必须提供的字段
例如：

- `prompt`
- `model`
- `stream`
- `temperature`

不要把服务端内部字段也照搬进前端。前端只负责按文档把能传的传对。

### 步骤 4：确认返回你要渲染什么
前端通常关心：

- 文本内容
- 是否成功
- 错误信息
- 是否流式
- 是否有用量统计

### 排错清单

- 有没有把可选字段当成必填字段
- 有没有漏掉鉴权字段
- 返回字段有没有和 UI 对上
- 文档示例和正式结构是否一致
- 成功和失败响应是否各读一遍

---

## 六、最小可运行示例：构造完整请求并打印响应

下面这段是前端常用模板，直接改地址和参数就能测。

```js
async function callAI() {
  const res = await fetch('https://example.com/api/chat?stream=false', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Authorization: 'Bearer demo-key',
      Accept: 'application/json'
    },
    body: JSON.stringify({
      prompt: '请用一句话介绍前端',
      model: 'demo-model'
    })
  })

  console.log('HTTP 状态：', res.status)

  const text = await res.text()
  console.log('响应原文：', text)

  let json = null
  try {
    json = JSON.parse(text)
  } catch (e) {
    console.error('响应不是 JSON，请先看是不是报错页或 HTML')
    return
  }

  console.log('原始返回对象：', json)

  if (!res.ok || json.code !== 0) {
    console.error(json.error?.message || json.message || '请求失败')
    return
  }

  console.log('模型回复：', json.data?.text)
  console.log('用量：', json.usage)
}

callAI()
```

关键点：

1. 先看 `res.status` 和 `res.ok`
2. 再看原始返回 `json`
3. 最后再做字段映射

### 错例对比

**错例：不判断 `res.ok`，也不看原始文本**

```js
const json = await res.json()
console.log(json.data.text)
```

问题：

- 401、500 也会继续往下跑
- 可能直接在 UI 上抛异常
- 错误信息被忽略，排查很痛苦

### 排错清单

- 打印 `res.status`
- 打印完整 `text/json`
- 检查 `res.ok`
- 兼容 `data/result/text` 多种字段
- 先在控制台跑通，再接 UI

---

## 七、前端拿到返回值后，先做一层适配再渲染

接口返回通常不会刚好长成页面想要的样子。更稳妥的做法，是先做一层“适配”。

比如接口可能返回：

```json
{
  "code": 0,
  "message": "ok",
  "data": {
    "choices": [
      { "message": { "content": "你好，前端新手。" } }
    ]
  },
  "error": null,
  "usage": {
    "input_tokens": 12,
    "output_tokens": 8
  }
}
```

页面只想要一个字符串时，可以先统一转换：

```js
function extractReply(json) {
  return (
    json?.data?.text ||
    json?.data?.result ||
    json?.data?.choices?.[0]?.message?.content ||
    json?.result ||
    json?.text ||
    ''
  )
}
```

这样做的好处是：

- 页面组件更干净
- 以后换接口，改适配层就行
- 不用到处写一堆 `?.`

### 最小可运行示例

```js
const reply = extractReply(json)
setMessage(reply)
```

### 错例对比

**错例：在 UI 组件里直接硬取字段**

```js
setMessage(json.data.choices[0].message.content)
```

问题：

- 字段一变就报错
- 失败响应结构不同，页面直接炸
- 组件逻辑太散，不好维护

### 排错清单

- 是否先写统一适配函数
- 页面是否只接收最终可展示文本
- 成功和失败结构是否都兼容
- 是否对空值做了兜底

---

## 八、接口调试时，先盯住真实发出去的请求

接口调试方法，是前端接 AI API 的必修课。很多时候你以为是代码错了，其实是请求根本没按你想的方式发出去。

### 重点看三处

#### 1. 浏览器 Network 面板
确认：

- 请求 URL 对不对
- Method 对不对
- Request Headers 对不对
- Request Payload 对不对
- Response 是什么

#### 2. 控制台打印原始对象
不要一上来就解构字段，先看完整返回：

```js
console.log('res =', res)
console.log('json =', json)
```

#### 3. 用最小请求验证接口
先别塞进复杂组件里，先在一个独立函数中跑通。

### 最小可运行示例

```js
async function debugRequest() {
  const res = await fetch('/api/chat', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Authorization: 'Bearer demo-key'
    },
    body: JSON.stringify({ prompt: 'test' })
  })

  console.log('status:', res.status)
  console.log('ok:', res.ok)
  const text = await res.text()
  console.log('raw response:', text)
}

debugRequest()
```

如果 `res.json()` 失败，先用 `res.text()` 看看返回的是 JSON、HTML，还是纯文本。

### 错例对比

**错例：只写业务代码，不看网络请求**

```js
callAI().then(setReply)
```

问题：

- 发错了也不知道
- 401、403、跨域、500 都混在一起
- 排错只能靠猜

### 排错清单

- Network 里有没有这条请求
- 请求头有没有带上 Authorization
- body 是否与文档一致
- 返回是不是 JSON
- 错误是前端没发出，还是接口返回失败

---

## 九、前端视角下的跨域基础：很多“接口没问题”，其实是浏览器不让你访问

前端接接口时，经常会遇到接口明明通了，浏览器却报跨域错误。

先记住：**跨域不是接口错了，而是浏览器的安全限制。**

比如页面在：

```txt
http://localhost:5173
```

接口在：

```txt
https://api.example.com
```

协议、域名、端口只要有一个不同，就可能触发跨域限制。

### 前端能做什么

- 通过开发服务器代理转发
- 使用同源后端中转
- 按文档要求配置允许跨域

### 最小可运行示例

```js
fetch('/api/chat', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ prompt: 'hello' })
})
```

然后由本地开发服务器把 `/api` 代理到真实接口地址。  
这样浏览器看到的是同源请求，就不会直接拦你。

### 错例对比

**错例：直接请求第三方域名**

```js
fetch('https://api.example.com/api/chat', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ prompt: 'hello' })
})
```

问题：

- 可能被 CORS 拦截
- 即使服务端已收到请求，你的前端也拿不到响应

### 排错清单

- 页面地址和接口地址是否同源
- 开发环境是否配置了代理
- 是否带了不必要的自定义请求头
- 浏览器报错是否明确指向 CORS
- 请求在 Network 中是否真正发出

---

## 十、把请求和返回接到 UI 状态里：loading、error、result 三件套

前端不只是把接口跑通，还要让用户知道发生了什么。

最常见的 UI 状态就三个：

- `loading`：正在请求
- `error`：请求失败
- `result`：请求成功后的内容

### 最小可运行示例

```js
const state = {
  loading: false,
  error: '',
  result: ''
}

async function submit() {
  state.loading = true
  state.error = ''
  state.result = ''

  try {
    const res = await fetch('/api/chat', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: 'Bearer demo-key'
      },
      body: JSON.stringify({ prompt: '帮我写一句产品介绍' })
    })

    const text = await res.text()
    const json = JSON.parse(text)

    if (!res.ok || json.code !== 0) {
      throw new Error(json.error?.message || json.message || '请求失败')
    }

    state.result = extractReply(json)
  } catch (e) {
    state.error = e.message
  } finally {
    state.loading = false
  }
}
```

### 错例对比

**错例：只管发请求，不管状态**

```js
async function submit() {
  const res = await fetch('/api/chat', {...})
  const json = await res.json()
  state.result = json.data
}
```

问题：

- 请求期间页面没反馈
- 失败时用户不知道怎么回事
- 异常时 loading 可能永远不消失

### 排错清单

- 请求前有没有把 loading 置为 `true`
- 成功和失败后有没有恢复 loading
- 错误信息有没有展示给用户
- 成功结果有没有清空旧状态
- UI 是否能区分“空结果”和“请求失败”

---

## 小结：你现在要掌握的是“映射能力”

这一章最核心的事，不是背字段名，而是建立一种前端思维：

- 请求参数要分层
- 请求头负责鉴权和格式
- 请求体负责业务内容
- 返回值先看原始结构，再做适配
- 非流式和流式处理方式不一样
- 调试时先看 Network，再看代码
- 接到页面时，一定配合 loading、error、result 状态

新手最容易犯的错，通常就是把“接口文档”当成“代码题”。其实它更像一张地图：你得先看清入口、补给点和危险区，才能把 AI 服务稳稳接到前端页面里。

下一章，我们继续讲 **fetch/axios 怎么封装**，把这些请求套路整理成你以后能反复复用的前端工具。

# 第3章 第一次把接口接到页面上：用 fetch 跑通最小闭环

如果你是前端新手，第一次看接口文档，最容易卡住的就是三件事：**这个 URL 怎么用、参数往哪放、拿到结果后页面怎么显示**。  
这一章不绕弯，直接用原生 `fetch` 跑通一个最小闭环：**准备参数 → 发请求 → 读响应 → 转成 JSON → 渲染到页面**。先把这条链路跑通，后面再学 `axios`、封装、错误处理、流式输出，都会顺很多。

> 先记住一句话：**这一章只讲前端在浏览器里怎么把接口接进页面，不讲后端怎么写接口。**

---

## 一、先看懂文档：这次请求到底要准备什么？

先别写代码，先确认接口文档里的四项：

1. **请求地址 URL**
2. **请求方法**：GET 还是 POST
3. **请求头**：尤其是 `Content-Type` 和 `Authorization`
4. **请求体**：你要传给接口的参数

如果文档写了“需要带鉴权”，前端一般要在请求头里放 token：

```js
'Authorization': 'Bearer YOUR_TOKEN'
```

如果文档写的是“请求体为 JSON”，就要先把对象转成字符串再发。

### 最小可运行示例

```js
const url = 'https://api.example.com/chat';
const options = {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer YOUR_TOKEN'
  },
  body: JSON.stringify({
    prompt: '你好，介绍一下前端 fetch'
  })
};
```

### 错例对比

**错例 1：漏写请求头**
```js
fetch(url, {
  method: 'POST',
  body: JSON.stringify({ prompt: '你好' })
});
```

**问题：**接口可能不认识 JSON，或者不认你的身份。

**错例 2：漏写 `JSON.stringify`**
```js
fetch(url, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: { prompt: '你好' }
});
```

**问题：**`body` 不是合法字符串，接口解析会失败。

### 排错清单

- URL 是否写对？
- 方法是否和文档一致？
- 是否需要 `Content-Type: application/json`？
- 是否需要 `Authorization`？
- 参数名是否和文档一致？
- `body` 是否已经 `JSON.stringify`？

---

## 二、用 fetch 把请求发出去：最小可运行闭环

`fetch` 是浏览器原生能力，适合先验证接口能不能通。它返回 Promise，所以通常配合 `async/await` 使用。你可以把它理解成：**浏览器帮你把请求发出去，再把结果拿回来**，但这个过程是异步的。

### 最小可运行示例

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <title>AI 接口测试</title>
</head>
<body>
  <button id="btn">发送请求</button>
  <pre id="output">等待结果...</pre>

  <script>
    const btn = document.querySelector('#btn');
    const output = document.querySelector('#output');

    btn.addEventListener('click', async () => {
      output.textContent = '请求中...';

      try {
        const res = await fetch('https://api.example.com/chat', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': 'Bearer YOUR_TOKEN'
          },
          body: JSON.stringify({
            prompt: '你好，介绍一下前端 fetch'
          })
        });

        const data = await res.json();
        output.textContent = JSON.stringify(data, null, 2);
      } catch (err) {
        output.textContent = `请求失败：${err.message}`;
      }
    });
  </script>
</body>
</html>
```

这个闭环已经成立了：点击按钮、发请求、等待响应、转成 JSON、显示到页面。

### 错例对比

**错例 1：忘了 `await`**
```js
const res = fetch('https://api.example.com/chat');
const data = res.json();
```

**问题：**`res` 还是 Promise，后面的解析接不上。

**错例 2：把响应对象当数据用**
```js
const res = await fetch(url);
output.textContent = res;
```

**问题：**页面拿到的是响应对象信息，不是业务数据。

### 排错清单

- 有没有写 `async`？
- 有没有漏写 `await fetch(...)`？
- URL 是否可访问？
- 请求是否被跨域拦住？
- 控制台有没有报错？

---

## 三、先读响应，再转 JSON：不要急着直接渲染

请求发出去后，先看**返回值长什么样**。接口文档通常会写 `code`、`message`、`data`、`result` 之类字段。你要判断两层：**HTTP 是否成功，业务是否成功**。

### 最小可运行示例

```js
const res = await fetch(url, options);

if (!res.ok) {
  throw new Error(`HTTP 错误：${res.status}`);
}

const data = await res.json();
console.log(data);
```

这里先看 `res.ok`，再看业务字段。即使 HTTP 200，也可能是参数错、鉴权失败、余额不足，所以不能只看状态码。

### 错例对比

**错例：不看响应直接渲染**
```js
const data = await res.json();
output.textContent = data.answer;
```

**问题：**如果字段不是 `answer`，页面就会空白或显示 `undefined`。

### 排错清单

- 状态码是多少？
- `res.ok` 是 `true` 还是 `false`？
- 返回体是不是 JSON？
- 成功字段名是 `code`、`result` 还是别的？
- 页面取值路径是否写对了？

---

## 四、把结果渲染到页面：别只停留在 console

前端接接口，最终不是“控制台能看到”，而是“页面能用”。所以拿到数据后，要更新 UI 状态：请求前显示加载中，请求成功展示结果，请求失败提示错误。

### 最小可运行示例

```html
<pre id="output">等待结果...</pre>
<script>
  const output = document.querySelector('#output');

  async function loadData() {
    output.textContent = '加载中...';

    try {
      const res = await fetch('https://api.example.com/chat', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': 'Bearer YOUR_TOKEN'
        },
        body: JSON.stringify({ prompt: '你好' })
      });

      if (!res.ok) {
        throw new Error(`HTTP 错误：${res.status}`);
      }

      const data = await res.json();
      output.textContent = data.answer || '没有返回 answer 字段';
    } catch (err) {
      output.textContent = `请求失败：${err.message}`;
    }
  }

  loadData();
</script>
```

这里已经包含最基础的 **UI 状态管理**：

- 请求前：`加载中...`
- 请求成功：展示内容
- 请求失败：展示错误信息

### 错例对比

**错例：只打印 console，不更新页面**
```js
const data = await res.json();
console.log(data);
```

**问题：**用户看不到结果。

### 排错清单

- 页面元素是否选对？
- `textContent` 是否赋值成功？
- 返回字段路径是否正确？
- 请求结束后有没有切回正常状态？
- 出错时有没有提示用户？

---

## 五、开发时怎么调试：先看 Network，再看跨域

前端第一次接接口，最常见的问题不是写法，而是**不会看调试信息**。  
先学会用浏览器开发者工具看三件事：**请求有没有发出去、状态码是什么、响应内容是什么**。

### 接口调试方法

1. 打开 DevTools 的 Network 面板
2. 点击按钮发请求
3. 看请求有没有出现
4. 看请求头、请求体、响应体是否符合预期
5. 观察状态码：200、400、401、403、500 分别意味着什么

如果请求根本没出现，先查按钮事件、JS 报错、代码是否真的执行到了那一步。

### 跨域基础

如果本地页面访问另一个域名的接口，浏览器可能因为跨域拦截请求。  
这不是语法错，而是浏览器的安全机制。前端要记住：**请求能不能通，除了代码，还要看接口是否允许当前来源访问**。开发环境没配好跨域时，Network 里可能看到失败，控制台也会报跨域错误。

### 排错清单

- Network 里有没有这条请求？
- 状态码是多少？
- 是浏览器拦截，还是接口返回错误？
- 响应内容是不是预期的 JSON？
- 本地开发环境是否存在跨域限制？

---

## 六、这三个高频错误，先记牢

### 1）漏写 headers
结果：服务端不认识你的数据格式，或者不认你的身份。  
尤其是 `Content-Type` 和 `Authorization`，这是高频检查项。

### 2）漏写 JSON.stringify
结果：`body` 不是合法 JSON，接口解析失败。

### 3）忘记 await
结果：你拿到的是 Promise，不是真正的数据，后续逻辑会乱掉。

这三个错误，基本可以作为你的第一版排错清单。以后接口一报错，先查它们，再看请求头、鉴权、跨域、状态码和返回结构。

### 排错清单

- 是否带了正确的请求头？
- 是否正确处理了鉴权信息？
- `body` 是否序列化成 JSON？
- 是否正确使用了 `await`？
- 是否按返回结构取值？

---

## 七、如果页面没显示，按这个顺序查

这是新手最容易慌的地方：代码看起来没报错，但页面就是没结果。别乱猜，按顺序查：

1. **请求是否真的发出去了**
   - 看 Network 里有没有这条请求

2. **响应是否成功**
   - 看状态码
   - 看 `res.ok`

3. **JSON 是否解析成功**
   - 响应体是不是标准 JSON
   - 有没有在 `res.json()` 这里报错

4. **DOM 是否更新成功**
   - 是否正确选中了元素
   - 是否真的给 `textContent` 赋值了

5. **返回字段是否取对**
   - `answer`、`data`、`result`、`message` 别写错

### 最小可运行检查模板

```js
console.log('1. 准备发请求');

const res = await fetch(url, options);
console.log('2. 收到响应', res.status);

const data = await res.json();
console.log('3. 解析结果', data);

output.textContent = data.answer || '没有内容';
console.log('4. 已更新页面');
```

### 排错清单

- 按钮点击事件有没有触发？
- 请求有没有发出去？
- 状态码是否正常？
- JSON 是否成功解析？
- 页面元素是否存在？
- 赋值的字段是否真的有值？

---

## 八、这一章你应该带走什么？

你现在不用把所有 AI 接口知识一次学完，只要先完成一个最小闭环：

1. 看懂接口文档里的方法、请求头、请求体  
2. 用 `fetch + async/await` 发出请求  
3. 把返回值转成 JSON  
4. 将数据写进页面  
5. 会用 Network 面板做基础调试  
6. 遇到报错时先看状态码、头部、参数和渲染路径  

只要你已经能把“按钮点击 → 发请求 → 页面显示结果”跑通，恭喜你，你已经迈过了前端接 AI 接口最难的第一步。下一章，我们再把这个最小闭环升级成更稳、更好维护的封装。

# 第4章 别让请求到处散落：用 axios 和封装把对接变简单

前面我们已经知道，AI 接口对接不是“会发一个请求”就结束了。真正进项目后，你会很快遇到这些问题：每个页面都写一遍请求、请求头忘了带、超时不统一、错误提示到处散落、改个接口地址要改十几个文件。  
所以这一步，老赵建议你先别急着做页面，先把**请求层**收拢起来。前端对接 AI 接口，最重要的不是“写得快”，而是“改得动、查得到、复用得上”。这章只站在**前端视角**讲：你怎么在浏览器里把请求发对、把返回值接住、把错误收拢，不讲后端代理，也不讲服务治理。

---

## 一、为什么要封装请求：统一超时、统一错误处理、统一请求头？

AI 接口通常会涉及：

- **请求头**：如 `Authorization`、`Content-Type`
- **鉴权**：token、API Key
- **超时**：AI 推理往往比普通接口更慢
- **错误处理**：网络错、401、429、500
- **UI 状态管理**：loading、disabled、错误提示

如果每个页面都单独写 `fetch` 或 `axios`，这些逻辑会重复散落。后面改 token 规则、改超时时间、改错误提示，都会变成全项目搜改。

### 最小可运行示例：不封装时的重复写法
```js
// 页面 A
axios.post('/api/chat', { prompt: '你好' }, {
  headers: { Authorization: 'Bearer xxx' },
  timeout: 30000
})

// 页面 B
axios.post('/api/chat', { prompt: '你是谁' }, {
  headers: { Authorization: 'Bearer xxx' },
  timeout: 30000
})
```

### 错例对比
**错例：**
- 每个页面自己写 headers
- 每个页面自己写 timeout
- 每个页面自己写错误提示

**问题：**
- 改 token 规则时全项目替换
- 某个页面忘了带鉴权，直接 401
- 有的页面 10 秒超时，有的 30 秒，体验不一致

### 排错清单
- 请求头是否每次都带上了？
- 鉴权字段是否和接口文档一致？
- timeout 是否过短，导致 AI 还没返回就中断？
- 是否已经接入统一错误提示？

---

## 二、axios 和 fetch 的差别：前端项目里该怎么选？

**fetch 是浏览器原生能力，axios 更适合工程化项目的请求库。**

对 AI 接口接入来说，你更需要的是统一处理请求头、超时、错误、返回值，而不是追求代码最短。

### 重点差别
- **fetch**：原生、轻量，但错误处理要自己写
- **axios**：默认把非 2xx 当成错误，更适合统一拦截
- **axios**：支持拦截器、baseURL、timeout、请求响应统一处理
- **fetch**：要自己处理 JSON、超时、错误状态

### 最小可运行示例：axios 和 fetch
```js
// axios
import axios from 'axios'

axios.get('/api/profile').then(res => {
  console.log(res.data)
}).catch(err => {
  console.error(err)
})

// fetch
fetch('/api/profile')
  .then(async res => {
    if (!res.ok) throw new Error('请求失败')
    return res.json()
  })
  .then(data => console.log(data))
  .catch(err => console.error(err))
```

### 错例对比
**错例：**
```js
fetch('/api/chat').then(res => console.log(res))
```

你只拿到了 Response 对象，没解析 JSON，也没判断状态码。

**正确做法：**
- 判断 `res.ok`
- 再 `res.json()`
- 再进入业务逻辑

### 排错清单
- 是不是把 `fetch` 的 Response 当成数据用了？
- 是否统一处理了非 2xx 状态？
- 是否需要拦截器、统一配置？如果需要，更适合 axios

---

## 三、如何封装一个适合前端项目的 request 工具？

封装不是为了“高级”，而是统一三件事：

- **请求头**
- **错误处理**
- **超时和返回格式**

### 最小可运行示例：创建一个 request 实例
```js
// request.js
import axios from 'axios'

const request = axios.create({
  baseURL: '/api',
  timeout: 30000
})

request.interceptors.request.use(config => {
  const token = localStorage.getItem('token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  config.headers['Content-Type'] = 'application/json'
  return config
})

request.interceptors.response.use(
  res => res.data,
  err => {
    if (err.response?.status === 401) {
      alert('登录已过期，请重新登录')
    }
    return Promise.reject(err)
  }
)

export default request
```

### 为什么这样写
- `axios.create()`：创建独立实例，避免污染全局
- 请求拦截器：统一加鉴权头
- 响应拦截器：统一处理返回值和错误

接口文档里通常会明确要求请求头，比如：

- `Authorization: Bearer xxx`
- `Content-Type: application/json`

统一放在封装里，页面就不用管这些细节了。

### 错例对比
**错例：**
```js
axios.defaults.baseURL = '/api'
axios.defaults.headers.common.Authorization = 'Bearer xxx'
```

看似省事，但项目大了之后，不同业务线容易互相污染。

**正确做法：**
- 重要业务单独实例
- AI 接口单独配置
- 普通接口和 AI 接口分开管理

### 排错清单
- 是否返回了 `res.data`，避免页面到处写 `.data`
- 是否在拦截器里处理了 401/403
- 是否避免了全局默认配置互相影响

---

## 四、如何为 AI 接口单独配置 baseURL、headers、timeout？

AI 服务和普通业务接口不一样：

- 返回更慢，需要更长 timeout
- 鉴权方式可能不同
- 可能有独立的 baseURL
- 可能有流式和非流式两种请求

### 最小可运行示例：AI 专用实例
```js
// aiRequest.js
import axios from 'axios'

const aiRequest = axios.create({
  baseURL: 'https://ai.example.com',
  timeout: 60000
})

aiRequest.interceptors.request.use(config => {
  config.headers.Authorization = `Bearer ${import.meta.env.VITE_AI_API_KEY}`
  config.headers['Content-Type'] = 'application/json'
  return config
})

export default aiRequest
```

### 这里要注意环境变量
前端项目里，密钥不要直接硬编码，应该放进**环境变量**，比如 `VITE_AI_API_KEY`。  
这样能区分开发、测试、生产环境，也方便统一管理。

> 提醒一句：如果这是直接暴露给浏览器的高权限密钥，风险很高。前端更常见的做法是后端代签或转发。这里先从前端对接视角理解结构。前端能做的是：通过环境变量管理不同环境的配置、把不该散落的值集中起来、在浏览器里确认当前环境注入是否正确。

### 错例对比
**错例：**
```js
const API_KEY = 'sk-xxxxxxx'
```

直接写死在代码里，等于把密钥贴在页面上。

**更合理的做法：**
- 用环境变量管理
- 不把高权限密钥放进公开前端代码
- 用最小权限策略或中转方案

### 排错清单
- baseURL 是否指向正确环境？
- timeout 是否适合 AI 接口？
- headers 是否包含文档要求的鉴权字段？
- 环境变量是否已经正确注入？
- 浏览器控制台里能否确认当前拿到的配置值？

---

## 五、页面只管业务：别让请求逻辑继续散在各处

重复写请求时，常见后果是：

- 这里用 `fetch`，那里用 `axios`
- 这个页面带 token，那个页面忘了带
- 一个页面处理 401，另一个页面直接白屏
- 请求失败时提示文案不一致

这就是“请求散落”。一开始不明显，等接口一多，维护成本就会暴涨。

### 最小可运行示例：错例
```js
// 页面 A
axios.post('/chat', { prompt })

// 页面 B
fetch('/chat', {
  method: 'POST',
  body: JSON.stringify({ prompt })
})
```

一旦接口改成需要 `Authorization`、需要统一 `timeout`、需要统一错误码处理，你就得两边同时改。

### 正确思路：页面只关心业务和 UI 状态
```js
import aiRequest from './aiRequest'

async function sendPrompt(prompt) {
  const data = await aiRequest.post('/chat', { prompt })
  return data
}
```

页面只负责：
- 触发请求
- 显示 loading
- 展示结果
- 处理错误

请求细节交给封装层。这样前端对接才算真正可维护。

### 排错清单
- 页面里是否出现大量重复请求代码？
- 是否能做到“改一次封装，全局生效”？
- 业务层是否还在处理 headers、baseURL 这类低层细节？

---

## 六、最小可运行示例：封装一个可复用的请求函数

### 完整示例
```js
// request.js
import axios from 'axios'

const request = axios.create({
  baseURL: '/api',
  timeout: 30000
})

request.interceptors.request.use(config => {
  const token = localStorage.getItem('token')
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})

request.interceptors.response.use(
  res => res.data,
  err => {
    const status = err.response?.status
    let message = '请求失败，请稍后重试'
    if (status === 401) message = '未登录或登录过期'
    if (status === 429) message = '请求太频繁，请稍后再试'
    if (status >= 500) message = '服务端异常'
    return Promise.reject(new Error(message))
  }
)

export default request
```

```js
// api.js
import request from './request'

export function askAI(prompt) {
  return request.post('/chat', { prompt })
}
```

```js
// 页面中
import { askAI } from './api'

let loading = false

async function handleSend() {
  loading = true
  try {
    const data = await askAI('你好')
    console.log('AI 回复：', data)
  } catch (e) {
    console.error(e.message)
  } finally {
    loading = false
  }
}
```

### 这段代码解决了什么
- 请求统一走一个入口
- 鉴权统一加
- 错误统一转成可读消息
- 页面只关心 loading 和展示

这里的 `loading` 很关键。因为请求是**异步流程**，按钮点下去后，页面不能“装作没事”。  
你需要用 `loading`、`disabled`、骨架屏或者转圈图标，让用户知道“请求正在进行中”。

### 错例对比
**错例：**
- 每个按钮单独写请求
- 每个 catch 都写一遍提示
- 每个页面自己维护 token

**正确：**
- 请求层封装
- 页面层只做 UI 状态管理

### 排错清单
- 组件里是否忘了 `finally`，导致 loading 不关闭？
- 错误是否被吞掉，页面无提示？
- `res.data` 是否已经统一处理，避免重复解包？

---

## 七、接口调试和跨域：前端先把问题看清楚

### 怎么调试请求
你在浏览器里最常用的调试方法不是猜，而是打开开发者工具：

- **Network 面板**：看请求有没有发出去
- **Headers**：看请求头里有没有 `Authorization`
- **Response**：看后端到底回了什么
- **Status**：确认是 200、401、429 还是 500

如果接口文档说“需要鉴权”，但你在 Network 里看不到 `Authorization`，问题就已经定位了。

### 跨域基础先知道一点
前端请求不同域名的接口时，浏览器可能会拦截，这就是**跨域**。  
它不是请求没发出去，而是浏览器的同源策略在拦你。常见表现是：

- 控制台报跨域错误
- Network 里能看到请求，但页面拿不到响应
- 本地开发环境需要代理或后端允许跨域

你先记住：**跨域是浏览器层的问题，不是你 request 封装写错了就一定能解决。**

### 最小可运行示例：调试时先打印关键信息
```js
async function testRequest() {
  try {
    const res = await request.post('/chat', { prompt: '测试' })
    console.log('成功返回：', res)
  } catch (err) {
    console.error('请求失败：', err.message)
  }
}
```

### 错例对比
**错例：**
- 只看页面报错，不看 Network
- 认为“没返回就是接口挂了”
- 遇到跨域就疯狂改请求代码

**正确：**
- 先看状态码
- 再看请求头
- 再看响应内容
- 最后再判断是跨域、鉴权还是业务错误

### 排错清单
- Network 里能看到请求吗？
- 请求头里鉴权字段带上了吗？
- 状态码是不是 401/429/500？
- 如果是跨域，开发环境配置是否需要代理或服务端放行？

---

## 本节小结：先把请求收拢，后面的 AI 接口才好接

你现在要记住的不是某个 API 细节，而是一个前端实战原则：

1. **先封装请求，再做业务页面**
2. **axios 更适合统一管理**
3. **AI 接口最好单独配置 baseURL、headers、timeout**
4. **环境变量管理密钥，别把敏感信息写死**
5. **页面只负责 UI 状态管理，请求细节交给工具层**
6. **调试先看 Network，跨域先分清是浏览器问题还是请求配置问题**

下一步，你就可以在这个封装基础上继续处理：  
**异步流程、loading 状态、流式与非流式返回、错误码分支、重试与频控**。  
把请求层搭稳了，AI 接口接入就不再是“到处散落的补丁活”，而是能持续维护的前端工程。

# 第5章 密钥、环境变量与鉴权：前端怎么安全地把门守住

## 开篇引入：先把话说透，前端能“减风险”，不能“永绝后患”

新手接 AI 接口最常见的错，不是 `fetch` 写错，而是把 **API Key 直接写进前端代码**。先记住：**前端只能降低暴露风险，不能把真正敏感的密钥长期、安全地藏在浏览器里。** 只要代码跑到浏览器，用户就可能通过开发者工具、网络请求或打包产物看到它。

所以这章只讲前端该做的事：**尽量安全地读取配置、正确带上鉴权请求头、处理未授权状态，并明确前端能做什么、不能做什么。**  
你可以把前端理解成：**负责把门票带到门口，但不能负责打开保险柜。**

---

## 一、为什么不能把密钥硬编码在前端源码里？

### 1. 这把“钥匙”本来就不该公开

前端代码最终会到浏览器里，用户能看到：

- 打包后的 JS
- 网络请求里的请求头
- 页面中写死的字符串
- 构建产物中的常量

把 `API_KEY` 直接写进源码，等于把钥匙贴在门上。

### 2. 硬编码的风险

- **被复制盗用**：别人拿到 key，就能冒充你调用接口。
- **额度被刷爆**：AI 接口通常按量计费，泄露后很容易被盗刷。
- **难以轮换**：改代码不等于安全，旧包里可能还留着。
- **仓库污染**：一旦进了 Git 历史，删当前文件也不代表彻底消失。

### 3. 前端视角下的正确思路

前端不负责制造密钥，只负责使用配置。真正敏感的长期密钥，最好放后端代理；如果必须前端直连，也要尽量使用**受限、短期、可轮换**的凭证，并配合域名限制、额度控制和请求限制。前端要做的是把风险降到最低，而不是假装“藏起来了就安全”。

### 最小可运行示例

```js
// ❌ 错例：不要这样写
const apiKey = "sk-xxxxxxxxxxxx";
```

```js
// ✅ 正确思路：通过环境变量注入
const apiKey = import.meta.env.VITE_AI_API_KEY;
```

### 错例对比

- 错：把 key 写进组件、常量文件或源码
- 错：把 key 提交到可公开访问的仓库
- 对：把 key 放到环境变量，由构建工具注入，并尽量避免长期敏感密钥直接出现在前端

### 排错清单

- [ ] 密钥是不是直接写在 JS 文件里了？
- [ ] 是否已经提交到 Git 仓库？
- [ ] 打开浏览器后能不能在源码或请求里看到它？
- [ ] 这个 key 是否需要立刻轮换？

---

## 二、环境变量在前端项目中的常见用法与边界

### 1. 前端环境变量是什么？

前端环境变量，是在开发、测试、生产等不同环境里，给前端构建过程提供不同配置值的方式。它解决的是“不要把配置写死”，但不是“把秘密藏起来”的万能办法。

你可以把它理解成配置清单：开发环境连测试接口，生产环境连正式接口；本地调试一套值，上线后切另一套。

### 2. 常见用法

以 Vite 为例：

```bash
# .env.development
VITE_AI_API_BASE=https://api.example.com
VITE_AI_API_KEY=demo_key
```

代码中读取：

```js
const baseUrl = import.meta.env.VITE_AI_API_BASE;
const key = import.meta.env.VITE_AI_API_KEY;
```

### 3. 它的边界在哪里？

要记住：**前端环境变量不是保密柜，而是配置入口。**

只要变量进入前端构建产物，用户理论上就可能看到。  
所以：

- **适合放**：接口地址、功能开关、非敏感配置
- **谨慎放**：会暴露权限能力的鉴权信息
- **不适合放**：真正不能公开的长期密钥

如果你是在做前端页面接入 AI 接口，环境变量更多是“方便切换环境”，不是“把 key 变得看不见”。

### 最小可运行示例

```js
const baseUrl = import.meta.env.VITE_AI_API_BASE;

async function ping() {
  const res = await fetch(`${baseUrl}/health`);
  console.log(await res.text());
}
```

### 错例对比

- 错：把所有配置写死在 `config.js`
- 错：把敏感值当成“只要放进 .env 就绝对安全”
- 对：用 `.env` 管理不同环境配置，代码只读变量，并知道它仍可能在浏览器里被看见

### 排错清单

- [ ] 变量名前缀是否符合工具要求？例如 Vite 需要 `VITE_`
- [ ] `.env` 文件是否放对目录？
- [ ] 修改后是否重新启动开发服务器？
- [ ] 是否误以为环境变量天然保密？

---

## 三、Authorization 请求头：前端发请求时怎么把“门票”带上？

### 1. 什么是鉴权请求头？

前端调用 AI 接口时，服务端通常要确认“你是谁、有没有权限”。最常见的方式之一，就是通过请求头传递认证信息，例如：

```http
Authorization: Bearer xxxxxxx
```

它就像一张门票：格式对了，才能进场；格式不对，就会被拦下。

### 2. 常见写法

```js
fetch("https://api.example.com/v1/chat", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${token}`,
  },
  body: JSON.stringify({ prompt: "你好" }),
});
```

如果接口文档要求的是别的格式，就按文档来，例如：

- `Authorization: Bearer xxx`
- `X-API-Key: xxx`
- `Authorization: Basic xxx`

**前端要做的不是猜，而是读文档。**

### 3. 请求头不是“写上就行”，而是要“写对”

新手常见问题通常不是“有没有写请求头”，而是：

- 字段名写错
- 少了前缀
- 值拼接错
- 传到 URL 里去了
- 被封装函数覆盖了

### 最小可运行示例

```js
const token = import.meta.env.VITE_AI_TOKEN;

fetch("/api/chat", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${token}`,
  },
  body: JSON.stringify({ prompt: "写一段自我介绍" }),
});
```

### 错例对比

- 错：把 token 放到 URL 参数里
- 错：请求头名字写错，比如文档要求 `Authorization`，你写成 `Auth`
- 错：少了 `Bearer ` 前缀
- 对：按文档在请求头里传正确字段

### 排错清单

- [ ] 接口文档要求的是哪个请求头？
- [ ] `Bearer` 前缀是否需要？
- [ ] 请求头名字有没有写错？
- [ ] `Content-Type` 是否和 body 格式一致？
- [ ] token 是否真的出现在 Network 里？

---

## 四、前端如何理解鉴权：不把自己写成后端

### 1. 前端的职责是什么？

前端不是负责“发明安全体系”，而是负责：

1. 从配置里读取密钥或 token
2. 按文档带上请求头
3. 展示登录态、权限态
4. 处理未授权、过期、无权限等结果

前端做的是配合鉴权，不是替代鉴权。

### 2. 哪些事情前端能做？

- 控制按钮是否可点
- 未登录时提示跳转
- 请求前检查 token 是否存在
- 接到 401 时引导重新登录
- 在 UI 上区分“没权限”和“请求失败”

### 3. 哪些事情前端不能做？

- 不能把真正的密钥长期暴露给所有用户
- 不能指望隐藏按钮就等于安全
- 不能用前端代码防住所有盗刷
- 不能自己伪造服务端权限判断

### 最小可运行示例

```js
async function callAI() {
  const token = import.meta.env.VITE_AI_TOKEN;
  if (!token) {
    alert("未配置鉴权信息");
    return;
  }

  const res = await fetch("/api/chat", {
    method: "POST",
    headers: { Authorization: `Bearer ${token}` },
  });

  if (res.status === 401) alert("未授权，请重新登录");
}
```

### 错例对比

- 错：认为“按钮隐藏了，用户就无法调用接口”
- 错：把前端判断当成最终权限校验
- 对：前端只做展示与流程控制，真正权限仍要靠服务端校验

### 排错清单

- [ ] 我是不是把鉴权当成纯前端问题了？
- [ ] UI 是否明确提示登录态、权限态？
- [ ] 收到 401 时是否有处理逻辑？
- [ ] 我是不是把“安全”误解成“隐藏代码”？

---

## 五、前端防刷怎么理解：能做的是“体验保护”，不是“绝对拦截”

这一点要特别说清楚：**前端可以做节流、按钮禁用、重复提交提示、加载中锁定等体验保护，但真正的防刷必须依赖服务端。**  
前端只能尽量减少误触和重复请求，不能阻止有意攻击者绕过页面直接发请求。

### 最小可运行示例

```js
let loading = false;

async function submitPrompt() {
  if (loading) return;
  loading = true;

  try {
    await fetch("/api/chat", { method: "POST" });
  } finally {
    loading = false;
  }
}
```

### 错例对比

- 错：以为按钮置灰就等于防刷成功
- 错：把所有防护都压在前端
- 对：前端做限频体验，服务端做真正限流与校验

### 排错清单

- [ ] 是否有重复点击保护？
- [ ] 是否在加载中禁用提交按钮？
- [ ] 是否明白前端防刷只是辅助？
- [ ] 是否把真正安全交给了服务端？

---

## 六、错例对比：把 key 暴露在浏览器、仓库或构建产物中

### 错例 1：直接写在代码里

```js
const key = "sk-xxxxx";
```

后果：任何人都能在浏览器或打包文件里找到。

### 错例 2：提交到仓库

```bash
git add .env
git commit -m "init"
```

后果：即使后来删除文件，Git 历史里仍可能留痕。

### 错例 3：只检查源码，不检查构建产物

开发时以为源码没了就安全，但上线后的 JS 包里可能仍包含变量值。  
所以要检查：**最终上线文件里有没有敏感信息。**

### 正确做法

- 密钥尽量不要直接放前端
- 能放后端代理的，优先放后端
- 前端只拿短期 token 或受限令牌
- 环境变量只用于管理配置，不等于保密工具
- 发布前检查浏览器 Network、构建产物和仓库历史

### 排错清单

- [ ] 仓库历史里是否已有泄露？
- [ ] 构建包里是否还能搜到 key？
- [ ] 线上页面源码是否可见？
- [ ] 是否需要立即轮换密钥？

---

## 七、排错清单：环境变量没生效、请求头没带上、接口返回未授权怎么办？

### 1. 环境变量没生效

检查顺序：

1. 文件名是否正确：`.env.development`、`.env.production`
2. 变量前缀是否符合工具要求
3. 是否重启开发服务器
4. 是否在代码中用对读取方式
5. 是否被打包时替换成空值

如果在浏览器里看到的是 `undefined`，多半不是接口问题，而是变量根本没读到。

### 2. 请求头没带上

检查顺序：

1. 打开浏览器 Network 面板
2. 看请求详情里的 Request Headers
3. 确认 `Authorization` 是否存在
4. 确认值是否为空、拼接是否正确
5. 确认是否被拦截器或封装函数覆盖

前端调试时，最直接的方法不是猜，而是看 Network 里真实发出去的内容。

### 3. 接口返回未授权

常见原因：

- token 过期
- token 格式不对
- 少了 `Bearer`
- 请求头字段名不对
- 服务端要求的鉴权方式与前端写法不一致

处理建议：

```js
if (res.status === 401) {
  // 清理本地登录态
  // 提示用户重新登录
  // 必要时跳转登录页
}
```

### 4. 接口调试方法

- 先看浏览器 Network，确认真实请求是否发出
- 再用 Postman / curl 复现同样请求
- 对照接口文档逐项检查请求头、参数、返回值
- 如果本地和线上表现不同，优先排查环境变量和域名配置
- 如果是跨域报错，先确认是不是浏览器拦截，而不是接口本身坏了

### 最小可运行示例

```js
async function testAuth() {
  const res = await fetch("/api/me", {
    headers: { Authorization: `Bearer ${import.meta.env.VITE_AI_TOKEN}` },
  });
  console.log(res.status, await res.text());
}
```

### 错例对比

- 错：只看控制台报错，不看 Network
- 错：看到 401 就立刻改业务逻辑，不先检查请求头
- 对：先看请求有没有发出去，再看请求头对不对，再看响应码

### 排错清单

- [ ] 环境变量是否真的读到了值？
- [ ] 请求头是否在 Network 里可见？
- [ ] 是否正确处理了 401 / 403？
- [ ] 是否用工具复现过同样请求？
- [ ] 这是前端问题，还是配置问题？

---

## 结尾小结：前端守门，守的是“边界”，不是“绝对安全”

这章你只要记住三句话：

1. **密钥不要硬编码到前端源码里。**
2. **环境变量是配置工具，不是保密保险箱。**
3. **鉴权要按接口文档带请求头，前端负责配合，不负责发明后端安全。**

再补一句更重要的：**前端防刷只能做体验保护，真正的安全边界还得靠服务端。**

以后做 AI 接口接入时，先问自己四个问题：

- 密钥放哪了？
- 请求头带对了吗？
- 环境变量生效了吗？
- 401 / 未授权有没有处理？

把这四个问题跑顺，你的前端接入就会稳很多。

# 第6章 异步流程与 UI 状态管理：让页面知道自己正在等结果

很多新手接 AI 接口时，最常见的问题不是“请求失败”，而是“页面像没反应”：按钮点了没动静、结果区空白、用户连点把请求打乱。  
问题通常不在接口本身，而在 **异步流程没有被前端正确管理**。这一章只站在前端视角，把“点击触发—进入 loading—等待响应—成功渲染—失败提示—状态复位”这条链路理顺。你不用先懂后端，先把页面自己的状态管明白，接 AI 接口就稳一半。

---

## 一、先记住一句话：只要在等接口，页面就不能假装结果已经来了

AI 接口通常不是立刻返回，而是先发出请求，再过一会儿拿到结果。  
`Promise` 可以理解为“一个未来会交答案的盒子”，`async/await` 则是“写法更像同步代码，但本质还是异步”。

对新手来说，关键不是背概念，而是知道：**请求发出去以后，页面状态必须先切到“正在等待”**，否则用户会以为按钮坏了。

### 最小可运行示例

```html
<button id="btn">发送请求</button>
<pre id="out"></pre>

<script>
  const btn = document.getElementById('btn');
  const out = document.getElementById('out');

  function mockAIRequest() {
    return new Promise((resolve) => {
      setTimeout(() => resolve('AI 回复：你好，前端新手！'), 1500);
    });
  }

  btn.onclick = async () => {
    out.textContent = '请求中...';
    const result = await mockAIRequest();
    out.textContent = result;
  };
</script>
```

### 错例对比

```js
const result = mockAIRequest();
out.textContent = result;
```

**问题：** 没等返回就渲染，页面拿到的是 Promise，不是 AI 回复。

### 排错清单

- 是否把异步函数写成了 `async`
- 是否在需要结果的地方用了 `await`
- 是否误把 Promise 当成字符串或对象直接渲染
- 是否考虑了失败分支，而不只是成功分支

---

## 二、请求发出、响应回来、页面更新：完整流程到底长什么样？

前端调用 AI 接口，本质是一个状态变化过程：

1. 用户点击按钮或提交表单  
2. 页面进入 `loading`  
3. 发起请求  
4. 接收到响应  
5. 把结果渲染到页面  
6. 结束 `loading`  
7. 出错时进入 `error`  
8. 没有内容时回到 `empty`

少一步，体验就会差。新手最常见的问题，就是只写“发请求”，没写“收尾”。  
尤其要注意：**成功、失败、空结果，都要有明确的页面反馈**。

### 最小可运行示例

```js
let loading = false;
let result = '';
let error = '';

async function send() {
  loading = true;
  error = '';
  result = '';
  render();

  try {
    const res = await fetch('/api/ai', {
      headers: {
        'Content-Type': 'application/json'
      }
    });
    const data = await res.json();
    result = data.text || '';
  } catch (e) {
    error = '请求失败，请稍后再试';
  } finally {
    loading = false;
    render();
  }
}
```

这里的重点是 `try...catch...finally`：`try` 处理成功，`catch` 处理错误，`finally` 统一收尾，比如关闭 loading。  
如果你只写成功，不写失败，页面就会在异常时“假死”。

### 错例对比

```js
loading = true;
const res = await fetch('/api/ai');
result = await res.json();
// 忘了 loading = false
```

**问题：** 页面会一直显示加载中，像卡死了一样。

### 排错清单

- 请求前是否清空了旧结果
- 请求结束后是否恢复 loading
- 是否用 `finally` 统一收尾
- 是否区分了成功、失败、空结果三种情况
- 是否把异常真正显示给用户

---

## 三、loading、error、success、empty：页面至少要有四种状态

做 AI 页面时，不能只写“有结果”和“没结果”。更合理的是四种状态：

- **loading**：请求进行中
- **error**：请求失败或接口报错
- **success**：拿到有效结果
- **empty**：还没有内容，或结果为空

这四种状态能让页面行为更稳定，也更符合用户预期。第一次打开页面时是空态；点击生成后进入加载态；成功后显示结果；失败后展示错误信息。

### 最小可运行示例

```html
<button id="send">生成回答</button>
<div id="status"></div>
<pre id="result"></pre>

<script>
  let state = 'empty'; // empty | loading | success | error
  let text = '';

  function render() {
    document.getElementById('status').textContent = state;
    document.getElementById('result').textContent = text;
  }

  document.getElementById('send').onclick = async () => {
    state = 'loading';
    text = '';
    render();

    try {
      const res = await fetch('/api/ai', {
        headers: {
          'Content-Type': 'application/json'
        }
      });
      const data = await res.json();
      text = data.text || '';
      state = text ? 'success' : 'empty';
    } catch {
      state = 'error';
      text = '接口请求失败';
    } finally {
      render();
    }
  };
</script>
```

### 错例对比

- 只写一个 `isLoading`
- 成功后不区分空返回和真实结果
- 失败后页面仍显示旧内容

**问题：** 用户不知道是接口没回、返回为空，还是渲染没成功。

### 排错清单

- 是否把“无内容”也当成一种状态处理
- 错误提示是否覆盖了旧结果
- 成功态是否真的有可展示的数据
- 空态是否给了用户下一步提示

---

## 四、按钮、表单、结果区：状态切换怎么做才顺手？

前端页面通常至少有三个区域要联动：

- **按钮**：控制是否可点击
- **表单**：控制是否可再次提交
- **结果区**：展示返回内容、错误信息、加载状态

原则很简单：**请求未完成时，禁止重复触发；请求完成后，再恢复交互。**  
这不仅是体验问题，也是避免接口被误刷的第一道前端防线。顺手把请求头也写对，尤其是 `Content-Type`，这样接口才能正确识别你发过去的是 JSON。

### 最小可运行示例

```html
<textarea id="prompt"></textarea>
<button id="go">发送</button>
<div id="tip"></div>
<pre id="box"></pre>

<script>
  let loading = false;

  async function submit() {
    if (loading) return;

    loading = true;
    go.disabled = true;
    tip.textContent = '生成中...';
    box.textContent = '';

    try {
      const res = await fetch('/api/ai', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': 'Bearer <token>'
        },
        body: JSON.stringify({ prompt: prompt.value })
      });
      const data = await res.json();
      box.textContent = data.text || '无结果';
      tip.textContent = '生成成功';
    } catch (e) {
      tip.textContent = '生成失败';
    } finally {
      loading = false;
      go.disabled = false;
    }
  }

  go.onclick = submit;
</script>
```

这里已经能看到前端接 AI 接口的几个关键点：`method`、`headers`、`body`，以及请求过程中的 UI 联动。`Authorization` 头也是常见的鉴权方式之一，前端通常通过它把登录态或临时令牌带给接口。  
注意：真正的密钥不应该硬编码在前端页面里，正式项目里要放到环境变量或后端代理里处理，前端只拿自己能暴露的那部分配置。

### 错例对比

- 按钮不禁用，用户连续点击
- 结果区和提示区同时显示旧数据
- 表单提交后没有清空状态
- 请求头没写对，接口拿不到 JSON 或鉴权信息

**问题：** 容易重复请求、结果互相覆盖、页面状态混乱，甚至直接 401/400。

### 排错清单

- 按钮是否在请求中禁用
- 是否有防重复提交逻辑
- 提示区是否跟随状态变化
- 表单提交后是否恢复可用
- 请求头是否写对，尤其是 `Content-Type` 和 `Authorization`
- 前端是否把不该暴露的密钥直接写进了代码

---

## 五、重复点击、多次请求覆盖、状态没复位：新手最容易踩的坑

### 1. 重复点击

用户连点两下，可能发出两次请求。后回来的请求，反而覆盖先回来的结果。  
这不是接口错了，而是前端没有控制好触发频率。

**处理办法：**
- 请求中禁用按钮
- 或记录请求编号，只接收最新一次

### 2. 多次请求覆盖

如果用户快速改了输入框并再次提交，旧请求回来后可能覆盖新结果。  
这在 AI 页面里特别常见，因为生成时间往往不固定。

### 最小可运行示例

```js
let requestId = 0;

async function send() {
  const currentId = ++requestId;
  const res = await fetch('/api/ai');
  const data = await res.json();

  if (currentId !== requestId) return;
  box.textContent = data.text;
}
```

意思是：谁最新，谁才算数。网络不稳定时，这个办法很实用。

### 3. 状态没复位

错误最常见：失败后按钮一直禁用，或者 loading 一直转。  
所以无论成功还是失败，收尾都要做，`finally` 是你的好朋友。

### 排错清单

- 是否允许用户重复触发
- 是否会被旧请求覆盖新请求
- 是否用了请求序号或取消机制
- 是否在异常时恢复 UI
- 是否在每次提交前清空旧提示
- 是否检查了输入为空时的空态处理

---

## 六、流式与非流式差异：UI 状态为什么更重要？

非流式接口是“等全部结果回来再显示”；流式接口是“边生成边显示”。  
对前端来说，二者的 UI 重点不同：

- **非流式**：重点在 loading 和最终展示
- **流式**：重点在逐步追加文本，同时维护连接状态

但无论哪种，前端都要明确：

- 什么时候开始等
- 什么时候结束
- 出错时怎么提示
- 页面是否允许继续操作

### 最小可运行示例

**非流式：**

```js
async function run() {
  status.textContent = '生成中...';
  const res = await fetch('/api/ai');
  const data = await res.json();
  result.textContent = data.text;
  status.textContent = '完成';
}
```

**流式：**

```js
async function runStream() {
  status.textContent = '持续生成中...';
  result.textContent = '';
  // 核心是“边到边追加”，而不是一次性等完再显示
}
```

流式和非流式最大的区别，不只是返回快慢，而是 UI 处理方式不同。  
非流式通常一次性更新结果；流式则要一段一段追加，不能每来一点就覆盖旧内容。  
如果你要继续往下做，记住：流式页面更需要“停止生成”“继续生成中”的状态提示，否则用户不知道连接是不是还活着。

### 排错清单

- 非流式是否在结束后一次性渲染
- 流式是否正确追加内容而不是覆盖
- 流式中断后是否清理 loading
- 用户是否能明确知道“正在生成中”
- 页面是否保留了终止/重试入口

---

## 七、接口调试时，前端先看哪里？

页面状态乱，不一定是代码逻辑错，也可能是接口根本没按预期回来。前端调试时，先看浏览器开发者工具里的这几项：

- **Network**：请求有没有发出去，状态码是多少
- **Response**：返回结构是不是你想要的
- **Console**：有没有报错
- **Headers**：请求头有没有带齐，尤其是 `Content-Type` 和 `Authorization`

如果接口返回 401，先想鉴权是否过期；如果返回 400，先想参数格式对不对；如果页面没反应，先看是不是 Promise 没 `await`，或者 loading 状态没改。跨域报错时，也要先确认前端请求地址、接口域名和浏览器控制台提示，不要一上来就怀疑页面渲染。

### 最小可运行示例

```js
async function debugSend() {
  console.log('开始请求');
  try {
    const res = await fetch('/api/ai', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer <token>'
      },
      body: JSON.stringify({ prompt: '你好' })
    });

    console.log('状态码', res.status);
    const data = await res.json();
    console.log('返回内容', data);
  } catch (e) {
    console.error('请求异常', e);
  }
}
```

### 错例对比

- 只看页面，不看 Network
- 只看成功案例，不看错误码
- 只改 UI，不查请求头和返回体

**问题：** 你会一直在页面上猜，排查效率很低。

### 排错清单

- 请求是否真的发出
- 状态码是否正常
- 返回字段名是否和代码一致
- 请求头是否缺少鉴权信息
- 是否有跨域报错或预检失败
- 控制台是否有异步异常

---

## 章末小结：先管好状态，再谈 AI 接口体验

如果你只记住一句话，那就是：**AI 接口接进前端，不只是“发请求”，更是“管理页面状态”。**

你今天至少要做到这几件事：

1. 会用 `async/await` 理解异步等待  
2. 会按流程处理 `loading / success / error / empty`  
3. 会让按钮、表单、结果区联动  
4. 会防重复点击、处理覆盖、恢复状态  
5. 会在浏览器里用 Network、Response、Console 做基础调试  
6. 会认识请求头、鉴权、跨域这些前端必须面对的问题  

另外别忘了，真正落地时，你还要同时看好 **请求头**、**鉴权**、**环境变量**、**跨域基础** 和 **接口调试方法**。这些不是为了增加难度，而是为了让你在前端页面里把请求发稳、把结果接稳、把用户体验做稳。  
特别是环境变量：正式项目里，前端页面不要直接写死敏感密钥；能放到环境变量的放环境变量，能交给后端代理的交给后端代理，前端只负责拿到安全可用的那部分配置。

下一步你可以先检查页面里有没有这四个状态：`loading`、`error`、`success`、`empty`。  
有了它们，页面才会真正“知道自己正在等结果”。

# 第7章 流式输出与非流式输出：AI 回复为什么会一边生成一边显示

## 开篇引入：为什么有的 AI 回答是“整段出现”，有的却是“边打字边出来”

前端接 AI 接口时，最常见的问题就是：  
**为什么有的回答要等很久才一次性返回，有的却能一边生成、一边显示？**

答案不在页面，而在接口设计：

- **非流式输出**：服务端把完整结果生成完，再一次性返回。
- **流式输出**：服务端把结果切成多段，生成一段就发一段，前端边收边更新。

对前端新手来说，重点不是背概念，而是知道：

> **怎么从接口文档判断它是哪一种，怎么发请求，怎么接收返回，怎么更新 UI，怎么避免把流式接口当普通 JSON 接口处理。**

你可以先把它理解成两种收快递方式：

- 非流式：包裹打包好再一次送来。
- 流式：边打包边送来，你边收边看。

本章你要掌握的，就是：**看懂流式接口、正确读取分段数据、把内容逐步显示到页面上。**

---

## 一、先分清：非流式一次性返回，流式逐段返回

### 1. 一句话区分

- **非流式**：适合“我等你一次性给我完整答案”
- **流式**：适合“我想尽快看到内容，哪怕还没生成完”

### 2. 前端视角下的体验差别

#### 非流式
点击发送后：

1. 页面进入 loading
2. 等接口完成
3. 一次性拿到完整文本
4. 直接渲染

优点：简单。  
缺点：等待感强，长文本体验差。

#### 流式
点击发送后：

1. 页面进入 loading
2. 接口持续返回片段
3. 前端不断拼接内容
4. 文本逐步显示，直到结束

优点：体验好、响应快。  
缺点：要处理状态、拼接、结束和错误。

### 3. 简短对比表

| 项目 | 非流式 | 流式 |
|---|---|---|
| 请求方式 | 一次请求，等完整结果 | 一次请求，持续接收片段 |
| 响应方式 | 一次性返回 JSON | 分段返回文本或事件流 |
| UI 更新方式 | 最后一次性更新 | 每收到一段就更新一次 |

### 4. 最小可运行示例

#### 非流式
```js
const res = await fetch("/api/ai");
const data = await res.json();
message.value = data.reply;
```

#### 流式
```js
message.value = "";
message.value += chunkText;
```

### 5. 错例对比

**错例：**
```js
const data = await res.json();
console.log(data.reply);
```

如果接口其实是流式返回，这里会直接失败，或拿不到完整数据。

**对例：**
```js
// 先确认文档是否说明流式
// 再选择对应的读取方式
```

### 6. 排错清单

- 看接口文档是否写了 `stream=true`
- 看返回格式是否说明“分段返回”
- 不要默认所有 AI 接口都能 `res.json()`
- 先确认拿到的是完整 JSON，还是一段一段的数据

---

## 二、SSE 是什么？你只要先记住“服务端持续推消息”

### 1. 最简概念

在本书里，你可以把 **SSE** 理解成一种“服务端持续向前端推消息”的方式。  
它的特点很简单：

- 连接不立刻结束
- 服务端不断发内容
- 前端边收边显示

很多 AI 接口会用它，因为它特别适合“边生成边展示”。

### 2. 前端为什么会遇到它

你不需要先学完整协议，只要记住：

> **SSE 场景下，响应不是一次性完整 JSON，而是连续到来的数据片段。**

所以你不能只等最终结果，还要处理过程中收到的每一段。

前端通常关心三件事：

- 连接有没有建立成功
- 片段有没有持续到达
- 结束信号有没有正确识别

### 3. 最小可运行示例

```js
const response = await fetch("/api/ai?stream=1");
const reader = response.body.getReader();
const decoder = new TextDecoder();

let text = "";
while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  text += decoder.decode(value, { stream: true });
}
console.log(text);
```

关键不是 `fetch`，而是 `response.body.getReader()`：它让你按块读取响应体。

### 4. 错例对比

**错例：**
```js
const data = await response.json();
```

问题：把连续流当普通 JSON 解析，通常会失败。

**对例：**
```js
const reader = response.body.getReader();
```

### 5. 排错清单

- 响应体是否存在 `response.body`
- 文档是否说明是流式 / SSE
- 浏览器是否支持当前读取方式
- 服务端是否真的在持续推数据
- 是否被代理、缓存或拦截打断

---

## 三、先跑通：前端如何把分段内容显示到页面上

### 1. 核心思路

收到一段，显示一段：

1. 准备一个空字符串存放“正在生成中的回复”
2. 每读到一段就追加进去
3. 追加后立刻更新页面

前端的关键，不是“会不会读”，而是**会不会把读到的内容正确同步到 UI 状态里**。

### 2. 最小可运行示例

```js
async function sendMessage() {
  isLoading.value = true;
  answer.value = "";

  const res = await fetch("/api/ai?stream=1");
  const reader = res.body.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value, { stream: true });
    answer.value += chunk;
  }

  isLoading.value = false;
  isFinished.value = true;
}
```

### 3. 这段代码的前端思维

这里最重要的三个状态是：

- `answer`：当前已显示内容
- `isLoading`：是否还在生成
- `isFinished`：是否完成

这就是流式场景里的基本 **UI 状态管理**。

如果你要让用户更清楚当前状态，还可以加上：

- `isStreaming`：当前是否处于流式接收中
- `isStopped`：是否被用户主动停止

### 4. 错例对比

**错例：**
```js
answer.value = chunk;
```

每次收到新片段都覆盖旧内容，最后只剩最后一小段。

**对例：**
```js
answer.value += chunk;
```

这才是逐步拼接。

### 5. 排错清单

- 是否用 `+=` 拼接，而不是覆盖
- 文本编码是否正常
- UI 是否在数据到来后重新渲染
- 是否有防抖、节流挡住实时更新
- 是否忘了在开始前清空旧内容

---

## 四、流式场景下 loading、停止、结束、失败怎么分

### 1. 为什么不能只用一个 loading

非流式接口里，一个 `loading` 常常够用；  
但流式接口里，至少要分清：

- **loading / streaming**：正在接收内容
- **stopped**：用户主动停止
- **finished**：正常结束
- **error**：异常中断

如果只写一个 `loading = false`，用户分不清是结束了、卡住了还是报错了。

### 2. 推荐的状态设计

```js
const state = {
  loading: false,
  streaming: false,
  stopped: false,
  finished: false,
  error: ""
};
```

### 3. 最小可运行示例

```js
async function start() {
  state.loading = true;
  state.streaming = true;
  state.stopped = false;
  state.finished = false;
  state.error = "";
  answer.value = "";

  try {
    const res = await fetch("/api/ai?stream=1");
    const reader = res.body.getReader();
    const decoder = new TextDecoder();

    while (true) {
      if (state.stopped) {
        await reader.cancel();
        break;
      }

      const { done, value } = await reader.read();
      if (done) {
        state.finished = true;
        break;
      }

      answer.value += decoder.decode(value, { stream: true });
    }
  } catch (e) {
    state.error = "请求失败";
  } finally {
    state.loading = false;
    state.streaming = false;
  }
}

function stop() {
  state.stopped = true;
}
```

这里的重点是：  
**流式请求不是一直“等结果”，而是持续读取、持续更新、必要时可中止。**

### 4. 错例对比

**错例：**
```js
state.loading = true;
// 只管开始，不管结束、不管中断
```

**问题：** 停不下来，错误也无法收尾。

**对例：**
```js
state.loading = false;
state.finished = true;
state.stopped = true;
```

### 5. 排错清单

- 点击“停止”后是否真的取消读取
- 结束后是否把 loading 关掉
- 出错后是否显示错误提示
- 重试前是否清空旧状态
- 页面离开时是否清理未完成请求

---

## 五、别把流式接口当普通 JSON 接口处理

### 1. 最常见错误

新手最容易犯的错，就是**把流式接口当普通 JSON 接口处理**。

### 2. 错例

```js
const res = await fetch("/api/ai?stream=1");
const data = await res.json();
answer.value = data.reply;
```

这个写法默认服务端一次性返回完整 JSON。  
但如果它是流式，这样写通常会失败。

### 3. 正确思路

先看文档，判断返回类型：

- 写的是 `application/json` 且一次性返回：用 `res.json()`
- 写的是分段推送、SSE、stream：用 `reader.read()`

别凭感觉写，**一定先看接口文档**。尤其要注意请求参数、响应格式、请求头、鉴权方式里有没有 `stream` 开关。

这里也别忘了前端常见的基础问题：

- **请求头**里是否要带 `Authorization`
- **鉴权**是否需要 token
- **跨域**时是否允许浏览器访问这个接口

这些问题不解决，流式代码写得再对也收不到数据。

### 4. 最小可运行示例

#### 非流式
```js
const res = await fetch("/api/ai");
const data = await res.json();
answer.value = data.reply;
```

#### 流式
```js
const res = await fetch("/api/ai?stream=1");
const reader = res.body.getReader();
```

### 5. 排错清单

- 接口文档有没有标明流式
- 请求参数里有没有 `stream`
- 响应是不是分块而非完整对象
- 页面报错是不是因为解析方式错了
- 调试时先打印原始响应，再决定怎么解析

---

## 六、接口调试时，前端该怎么一步步确认

### 1. 调试顺序建议

1. 先看接口文档，确认是否流式
2. 用浏览器 Network 看响应是否持续到达
3. 先打印原始文本，不急着解析
4. 再决定拼接、分隔、渲染方式

前端接 AI 接口，最怕的就是代码写了一堆，方向却错了。调试时先确认：

- 请求发出去了没有
- 鉴权对不对
- 请求头对不对
- 跨域有没有拦住

再考虑业务逻辑。

### 2. 标准调试 SOP

你可以按这个顺序查：

- **Network**：看请求是否成功、响应是否持续返回
- **Response**：看返回内容是完整 JSON，还是一段一段的数据
- **Console**：打印每个 chunk，确认前端是否真的收到了
- **Postman / curl 对照**：先在工具里验证，再回到页面里对接

### 3. 最小调试示例

```js
const res = await fetch("/api/ai?stream=1");
console.log(res.headers.get("content-type"));

const reader = res.body.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  console.log(decoder.decode(value, { stream: true }));
}
```

这段代码能先帮你看清“服务端到底吐了什么”。

### 4. 排错清单

- 看响应头 `content-type`
- 看 Network 面板是否持续有数据
- 控制台是否能打印出片段
- 是否被跨域或代理拦截
- 是否请求成功但前端解析方式不对

---

## 七、前端视角下，流式接口还要注意什么

流式接口看起来像“不断吐字”，但前端真正要管的是浏览器里的这些动作：

- **显示中**：文字要能持续追加
- **滚动到底部**：新内容来了，用户能看到最后一行
- **停止按钮可用**：用户可以中断生成
- **异常可恢复**：失败后能重新发起请求

如果你的页面是聊天框，建议在每次追加内容后，把滚动条拉到底部，否则用户会看到内容在增长，却停在旧位置。

### 1. 最小可运行示例

```js
answer.value += chunk;
await nextTick();
chatBoxRef.value?.scrollTo({
  top: chatBoxRef.value.scrollHeight,
  behavior: "smooth"
});
```

### 2. 错例对比

**错例：**
```js
answer.value += chunk;
// 不处理滚动
```

结果就是：内容更新了，用户却看不到最新一段。

**对例：**
```js
answer.value += chunk;
// 更新后滚动到底部
```

### 3. 排错清单

- 文字是否追加成功
- 是否等待 DOM 更新后再滚动
- 容器是否设置了可滚动高度
- 是否被父级样式挡住
- 是否移动端键盘弹出后遮住了内容

---

## 结尾小结：先分清接口类型，再谈页面体验

这章你只要记住一件事：

> **流式接口不是更高级的 JSON，而是数据分段到达的另一种接收方式。**

前端要做的不是猜，而是：

- 先看文档判断流式 / 非流式
- 再选择 `res.json()` 还是 `reader.read()`
- 再设计 `loading / streaming / stopped / finished / error`
- 再把片段逐步拼接到 UI 上
- 最后处理滚动、结束信号、失败重试

### 你现在应该能做的事

- 看懂流式和非流式的区别
- 用前端代码接收分段数据
- 在页面里逐步显示 AI 回复
- 设计停止、结束、错误状态
- 避免把流式接口当普通 JSON 接口处理
- 在浏览器里通过 Network 和控制台做基本调试

### 下一步该怎么练

先找一个明确支持流式返回的 AI 接口，按这个顺序练一遍：

1. 先确认文档里有没有 `stream`
2. 再在页面里做最小可运行版
3. 然后加上 loading、停止、finished
4. 最后再补跨域、请求头、鉴权和错误提示

本章你已经掌握了流式输出的前端接法。下一章就可以把它和**请求头、鉴权、环境变量、错误处理**串起来，做成一个真正完整的前端 AI 接入流程。

# 第8章 错误码、异常和重试：接口失败时前端该怎么稳住

前端接 AI 接口，最怕的不是“没请求成功”，而是**失败了却不知道失败在哪一层**。有时是网络断了，有时是接口返回 401/429，有时是业务层提示“余额不足”，还有时是你把返回体解析错了。新手一旦把这些都混成一句“请求失败”，就会导致两个问题：**用户不知道发生了什么**，**你自己也不知道该怎么修**。

这一章我们按前端视角，把错误处理拆开看清楚，并学会：失败时怎么提示用户、什么时候能重试、什么时候绝对不能重试。  
先记住一句话：**前端处理错误，不是只会 `console.error()`，而是要能看懂错误层级、更新 UI、决定是否重试。**

---

## 一、先分清：网络错误、HTTP 错误、业务错误、解析错误

这是错误处理的第一步。你要先知道自己到底失败在哪一层，而不是一上来就 `catch` 然后统一提示“请求异常”。

### 1. 网络错误：请求根本没到服务端
常见表现：

- 断网
- DNS 解析失败
- 跨域被浏览器拦截
- 服务器没开
- 请求地址写错

这类错误通常连响应都拿不到，`fetch` 会直接进 `catch`。排查时先看 DevTools 的 Network 面板：请求有没有发出去、有没有被跨域拦下、地址是不是写错了。

### 2. HTTP 错误：服务端收到了，但返回了非 2xx
比如：

- `401`：没登录、鉴权失败
- `403`：没权限
- `404`：接口不存在
- `429`：请求太频繁
- `500`：服务端内部错误

注意：`fetch` 默认**不会**因为 401/500 自动抛错，你要自己判断 `response.ok`。否则会把失败请求当成成功返回。  
这里还要结合**请求头**来看：很多接口要求你在 `Authorization` 里带 token，或者带 `x-api-key` 之类的鉴权字段。前端如果没按文档把请求头拼对，拿到的就是 401，而不是“接口坏了”。

### 3. 业务错误：HTTP 成功，但接口告诉你失败
例如：

```json
{
  "code": 4001,
  "message": "余额不足",
  "data": null
}
```

这里 HTTP 可能是 200，但业务上失败了。前端不能只看状态码，还要看接口文档里的 `code`、`message` 规则。看文档时，先找到“成功条件”和“失败字段”，别只盯着请求参数。

### 4. 解析错误：拿到数据了，但你读错了
比如：

- 以为返回的是 JSON，结果其实是文本
- 以为字段叫 `result`，实际叫 `data.answer`
- 流式接口还没结束，你却把它当成完整 JSON 解析

这类错误最容易出现在“接口已经通了但页面不显示”的场景。尤其是**流式与非流式差异**要分清：非流式接口通常一次性返回完整 JSON；流式接口可能分片返回或 SSE 推送，不能用普通 `await res.json()` 一把梭。

### 最小可运行示例

```js
async function requestAI() {
  try {
    const res = await fetch('/api/chat', {
      headers: {
        Authorization: 'Bearer YOUR_TOKEN'
      }
    });

    if (!res.ok) {
      throw new Error(`HTTP错误：${res.status}`);
    }

    const json = await res.json();

    if (json.code !== 0) {
      throw new Error(`业务错误：${json.message}`);
    }

    return json.data;
  } catch (err) {
    console.error('请求失败', err);
    throw err;
  }
}
```

### 错例对比

**错例：**
```js
const res = await fetch('/api/chat');
const data = await res.json();
console.log('成功了', data);
```

问题：

- 不判断 `res.ok`
- 不判断业务 `code`
- 没带鉴权请求头
- 出错时只会在控制台看到一团乱

### 排错清单

- 先看 Network 里有没有发出去
- 看请求头是否带了鉴权信息
- 看是否被跨域拦截
- 看响应状态码是不是 401/429/500
- 看返回体是不是业务失败
- 看字段路径是否写错
- 流式接口是否误用普通 JSON 解析

---

## 二、常见错误码怎么处理，前端别一刀切

错误码不是“报错字符串”，而是前端的**分流信号**。不同码，处理方式不同。

### 1. 401：鉴权失败
常见原因：

- 请求头没带 token
- token 过期
- API Key 配错

处理方式：

- 提示“登录已过期，请重新登录”
- 用户态系统可跳转登录页
- 如果是 AI 服务 API Key 问题，提示“服务暂不可用，请稍后重试”

这里要特别注意请求头里的 `Authorization`。接口文档要求你带鉴权信息，你却没带，服务端就会直接拒绝。前端不是“发出去就完事”，而是要按文档把请求头拼正确。

### 2. 403：无权限
说明身份可能对，但权限不够。

处理方式：

- 提示“当前账号无权限使用该功能”
- 不要盲目重试
- 必要时隐藏按钮或禁用功能入口

### 3. 429：限流
这是 AI 接口非常常见的错误。

处理方式：

- 提示“请求过于频繁，请稍后再试”
- 按 `Retry-After` 头或文档建议等待
- 前端加节流、防连点、按钮禁用

### 4. 500/502/503：服务端异常
处理方式：

- 提示“服务繁忙，请稍后再试”
- 可在有限次数内重试
- 不要无限循环

### 最小可运行示例

```js
function handleStatus(status) {
  switch (status) {
    case 401:
      return '登录过期，请重新登录';
    case 403:
      return '没有权限使用该功能';
    case 429:
      return '请求太频繁，请稍后再试';
    default:
      return '服务暂时异常，请稍后再试';
  }
}
```

### 错例对比

**错例：**
```js
if (!res.ok) {
  alert('出错了');
}
```

问题：

- 所有错误都同一种提示
- 用户不知道该等、该登录，还是该刷新
- 你也无法定位问题

### 排错清单

- 看响应状态码
- 看接口文档是否说明错误码
- 看是否有 `Retry-After`
- 看是否是鉴权头缺失
- 看是否是额度、频率、权限问题

---

## 三、如何把错误信息展示给用户，而不是只在控制台报错

新手最容易犯的错，就是只写 `console.error()`，以为问题解决了。其实控制台只是给开发者看的，**用户需要页面反馈**。

### 前端展示错误的基本原则

1. **可理解**：别把原始堆栈直接丢给用户  
2. **可操作**：告诉用户下一步怎么办  
3. **不吓人**：不要把技术细节全暴露出来  
4. **可恢复**：能重试就给重试按钮

### 推荐的 UI 状态设计

- `loading`：正在请求
- `success`：请求成功
- `error`：请求失败
- `empty`：没内容
- `retrying`：正在重试

这就是前端的 **UI 状态管理**。接口一旦失败，页面不能卡在加载中，也不能什么都不显示。你要明确告诉用户当前发生了什么。  
比如：`loading` 时禁用按钮，`error` 时显示一条可读提示，`retrying` 时显示“正在重试，请稍候”，这样用户才不会误以为页面卡死。

### 最小可运行示例

```html
<button id="sendBtn">发送</button>
<div id="msg"></div>
<script>
const btn = document.getElementById('sendBtn');
const msg = document.getElementById('msg');

btn.onclick = async () => {
  msg.textContent = '加载中...';
  btn.disabled = true;

  try {
    const res = await fetch('/api/chat', {
      headers: {
        Authorization: 'Bearer YOUR_TOKEN'
      }
    });

    if (!res.ok) throw new Error('HTTP错误');

    const data = await res.json();
    msg.textContent = data.message || '完成';
  } catch (e) {
    msg.textContent = '请求失败，请稍后重试';
  } finally {
    btn.disabled = false;
  }
};
</script>
```

### 错例对比

**错例：**
```js
catch (e) {
  console.error(e);
}
```

问题：页面没反应，用户以为按钮失灵；同时 `loading` 也没有关闭。

### 排错清单

- 错误信息有没有渲染到页面
- loading 状态有没有及时关闭
- 按钮失败后是否可再次点击
- 是否给出了明确的用户提示
- 是否保留了开发者日志

---

## 四、重试什么时候能用，什么时候不能乱用

重试不是“失败就再来一次”。AI 接口里，**错误重试要非常克制**。

### 适合重试的场景

- 短暂网络抖动
- 5xx 服务端临时异常
- 429 限流后等待一段时间再试

### 不适合重试的场景

- 401 鉴权失败
- 403 无权限
- 参数校验失败
- 用户已经提交成功但前端没收到结果
- 可能产生重复扣费、重复生成、重复提交的操作

你尤其要警惕“看起来失败，其实已经成功”的情况。比如某些 AI 生成任务，第一次请求虽然超时了，但服务端可能已经开始处理。这个时候你如果无脑重试，可能会重复消耗额度，甚至产生重复结果。  
所以重试前先问自己一句：**这个请求能不能重复发？发两次会不会出事？**

### 重试要控制什么

- 次数：通常 1~3 次足够
- 间隔：指数退避更稳妥，比如 1s、2s、4s
- 条件：只对可恢复错误重试
- 幂等性：能否重复执行不出问题

### 最小可运行示例

```js
async function fetchWithRetry(url, times = 2) {
  for (let i = 0; i <= times; i++) {
    try {
      const res = await fetch(url, {
        headers: {
          Authorization: 'Bearer YOUR_TOKEN'
        }
      });

      if (res.ok) return await res.json();

      if (![429, 500, 502, 503].includes(res.status)) {
        throw new Error(`不可重试错误：${res.status}`);
      }
    } catch (e) {
      if (i === times) throw e;
    }

    if (i < times) {
      await new Promise(r => setTimeout(r, 1000 * (i + 1)));
    }
  }
}
```

### 错例对比

**错例：**
```js
while (true) {
  await fetch('/api/pay');
}
```

问题：

- 无限重试
- 可能重复扣费
- 可能把服务打爆
- 用户根本停不下来

### 排错清单

- 这个接口能不能重复发
- 会不会产生副作用
- 是否只对 429/5xx 重试
- 是否设置最大次数
- 是否加了等待间隔
- 是否有“正在重试”的 UI 提示
- 是否在失败时停掉 loading 并恢复按钮

---

## 五、接口调试时，前端还要注意什么

当你怀疑接口有问题时，不要只盯代码。建议按这个顺序排：

- 看 Network 里请求是否发出
- 看请求头是否带了鉴权信息
- 看跨域是否报错
- 看状态码和返回体
- 看字段是否和文档一致
- 如果是流式接口，要确认是不是按 SSE 或分片方式接收，而不是当普通 JSON 读

这里的“跨域基础”你只要先记住一点：**浏览器会拦截不符合跨域规则的前端请求**。所以你在本地开发时，可能会遇到“接口明明后端通了，页面却报跨域”的情况。这个时候先看控制台和 Network，不要先怀疑业务代码写错。  
另外，调试时一定记得检查**请求头**和**鉴权**是否一致：很多所谓“接口失败”，其实就是你少带了 token，或者环境变量里的地址配错了。

### 最小可运行示例：带调试日志的请求

```js
async function debugRequest() {
  try {
    const res = await fetch('/api/chat', {
      headers: {
        Authorization: 'Bearer YOUR_TOKEN'
      }
    });

    console.log('status:', res.status);

    const text = await res.text();
    console.log('raw response:', text);
  } catch (err) {
    console.error('network fail:', err);
  }
}
```

### 错例对比

**错例：**
```js
fetch('/api/chat')
  .then(res => res.json())
  .then(console.log);
```

问题：

- 没有日志，难排查
- 不看状态码
- 不看返回原文
- 失败后不知道卡在哪一步

### 排错清单

- 浏览器控制台有没有跨域报错
- 请求头是否带上鉴权信息
- 接口地址是否和环境变量一致
- 返回体是 JSON 还是文本
- 流式接口是不是误按普通接口处理
- 状态码正常但业务失败时，是否忽略了 `code`

---

## 六、接口失败时，前端怎么稳住：一套实战思路

你可以把错误处理记成四步：

1. **识别错误类型**：网络 / HTTP / 业务 / 解析  
2. **映射用户提示**：登录过期、服务繁忙、请求过快  
3. **更新 UI 状态**：关闭 loading，显示 error，提供重试  
4. **决定是否重试**：只对可恢复错误、有限次数重试

### 你在项目里至少要做的事

- 请求前设置 loading
- 请求后关闭 loading
- 判断 HTTP 状态
- 判断业务 code
- 统一错误提示函数
- 记录日志方便排查
- 对 429 和 5xx 做有限重试
- 对 401/403 不重试，改为引导用户处理
- 请求头里把鉴权信息按文档带齐
- 用环境变量管理接口地址和密钥来源，避免在代码里硬写敏感信息

### 结尾小结

接口失败不是“坏消息”，而是前端必须处理的正常分支。真正成熟的前端，不是永远不报错，而是**出错时不崩、不错乱、能提示、能恢复**。  
你只要记住一句话：**先分清错误层级，再决定提示、状态和重试策略。** 这样接 AI 接口时，页面才会稳，用户才会信任你。  
如果你是第一次做这类接入，先把本章的最小示例跑通，再按排错清单逐项检查；能稳定识别错误、展示错误、控制重试，你就已经跨过了新手最容易摔跤的那一步。

# 第9章 频控、限流与防刷：前端能做的保护措施有哪些

## 开篇引入：前端先管住“别点太快”
很多新手以为 AI 接口就是“点一下，等结果”，但真实项目里，连点、重复提交、网络抖动，都会导致**重复生成、重复扣费、页面状态错乱**。

先划边界：**前端能做的是按钮禁用、节流、防抖、重复提交拦截和体验提示**；真正的安全、限流和防刷，还是要靠**服务端**。前端不是安全兜底，只是第一道体验防线。我们的目标是：让一次用户操作，尽量只对应一次有效请求。

这也和前面讲过的**请求头、鉴权、环境变量、跨域**有关。前端要按接口文档正确发请求；密钥不要写死在页面里；请求发不出去时，先看是不是跨域或鉴权问题，再看是不是点太快。

### 最小可运行示例
```html
<button id="sendBtn">发送请求</button>
<script>
const btn = document.getElementById('sendBtn');
btn.addEventListener('click', () => {
  console.log('发起一次请求');
});
</script>
```

### 错例对比
**错例**：点击后不做任何保护，用户连点就会多次触发。  
**对的做法**：请求进行中先禁用按钮，避免重复提交。

### 排错清单
- 请求未完成时是否还能再次点击？
- 按钮是否有 loading 状态？
- 是否出现重复生成、重复扣费？
- 失败后按钮状态是否正确恢复？

---

## 1. 频控、限流、防刷分别是什么？
- **频控**：限制短时间内的操作频率，比如 1 秒内只允许点一次。
- **限流**：接口整体访问上限，通常由服务端控制。
- **防刷**：更广义的保护措施，减少恶意或无意义的高频请求。

前端最关心的不是“把流量全拦住”，而是**避免用户无意中制造重复请求**。比如 AI 生成按钮被连续点击 3 次，如果每次都发请求，就会出现 3 次响应。  
记住：前端频控是体验保护，服务端限流是系统保护，职责不同，但要配合。

### 最小可运行示例
```js
let lastTime = 0;
function canClick() {
  const now = Date.now();
  if (now - lastTime < 1000) return false;
  lastTime = now;
  return true;
}
```

### 错例对比
**错例**：以为前端防刷能独立防住所有攻击。  
**对的做法**：前端只挡误触和重复操作，真正安全还要服务端兜底。

### 排错清单
- 你要解决的是误触，还是高频请求？
- 是否把前端体验保护和服务端限流混为一谈？
- 是否明确了职责边界？

---

## 2. 按钮禁用、节流、防抖、重复提交拦截怎么选？
这几个方法别混着用，要按场景选。

- **按钮禁用**：最直接，适合“提交一次就等结果”的 AI 请求。
- **节流**：一段时间内最多触发一次，适合快速点击。
- **防抖**：等用户停下来再发，适合搜索建议、输入联想。
- **重复提交拦截**：请求未结束前，直接拒绝新的提交。

对“生成文本”“生成图片”这类操作，最推荐的是：**loading + 按钮禁用 + 重复提交拦截**。  
如果是**非流式接口**，请求结束后一次性返回结果；如果是**流式接口**，内容会一段一段回来，按钮恢复时机要更谨慎，必须等流结束后再恢复。

### 最小可运行示例
```html
<button id="sendBtn">生成内容</button>
<script>
let loading = false;
let lastTime = 0;

document.getElementById('sendBtn').onclick = async () => {
  const now = Date.now();
  if (loading || now - lastTime < 1000) return;

  loading = true;
  lastTime = now;
  const btn = document.getElementById('sendBtn');
  btn.disabled = true;
  btn.innerText = '生成中...';

  try {
    await fetch('/api/chat');
  } finally {
    loading = false;
    btn.disabled = false;
    btn.innerText = '生成内容';
  }
};
</script>
```

### 错例对比
**错例**：只改按钮文案，不禁用按钮。  
**对的做法**：文案、`disabled`、`loading` 一起做，用户才知道当前状态。

### 排错清单
- 是否同时处理了 `loading` 和 `disabled`？
- 是否在 `finally` 里恢复状态？
- 节流时间是否过长，影响体验？
- 防抖是否会让用户误以为没点上？

---

## 3. UI 上怎么提示用户“稍后再试”？
前端不仅要拦，还要告诉用户为什么被拦，不然用户会一直点。

推荐三种提示：

1. 按钮旁显示“正在生成，请稍后”
2. 超过频率时提示“操作太快，请 3 秒后再试”
3. 接口返回 429 时，明确告诉用户这是**限流**，不是程序坏了

这里尤其要注意异步流程：请求还没结束时，就别让用户误以为可以继续提交。UI 状态管理要和请求状态一一对应：`loading` 时禁用按钮，`成功` 时恢复按钮，`失败` 时提示原因并决定是否允许重试。

### 最小可运行示例
```js
if (tooFast) {
  alert('操作太频繁，请稍后再试');
  return;
}
```

### 错例对比
**错例**：直接 `return`，页面没有任何反馈。  
**对的做法**：明确告知用户当前状态，减少重复点击。

### 排错清单
- 提示文案是否清楚？
- 是否区分“请求中”和“被限流”？
- 是否给出大致等待时间？
- UI 状态是否和真实请求状态一致？

---

## 4. 前端怎么配合服务端限流，而不是对抗它？
前端不要想着绕过限流，而是要**顺着规则做体验优化**。  
服务端如果返回 429，前端应该：

- 停止继续重试
- 显示“请稍后再试”
- 根据返回信息决定是否延迟重试

如果接口需要请求头鉴权、会话标识或用户标识，前端要正确携带，不要因为漏掉请求头导致服务端误判成异常流量。  
另外，跨域基础也别忽略：如果浏览器直接请求 AI 接口，而接口没放行跨域，连最基础的请求都发不出去，更谈不上限流处理。  
所以在调试时，先确认**请求头、鉴权、跨域**都正常，再去看是不是触发了频控或限流。

### 最小可运行示例
```js
fetch('/api/chat', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token'
  },
  body: JSON.stringify({ prompt: '你好' })
});
```

### 错例对比
**错例**：没带鉴权请求头，服务端可能把你当成异常请求。  
**对的做法**：按文档正确带上 `Authorization` 等请求头，并处理 429 状态。

### 排错清单
- 是否正确带上鉴权请求头？
- 是否对 429 做了单独处理？
- 是否在短时间失败后还在自动狂重试？
- 是否确认接口允许跨域访问？

---

## 5. 连续点击为什么会导致重复生成？
因为前端每点一次按钮，通常就会发起一次异步请求。  
如果你没有阻止重复点击，页面上可能同时存在多个 `fetch` 请求，返回顺序还可能乱掉，最后显示的不是用户最新一次操作的结果。

这就是典型的异步流程问题：**请求先后顺序不等于返回顺序**。  
所以前端要做两件事：一是阻止重复提交，二是只接受“当前这次操作”的结果。  
如果是流式输出，问题更明显：上一条流还在推数据，下一条又开始了，UI 很容易出现两段内容交错、按钮状态提前恢复等问题。

### 错例对比
**错例**
```js
btn.onclick = () => fetch('/api/generate');
```

**对的做法**
```js
let loading = false;
btn.onclick = async () => {
  if (loading) return;
  loading = true;
  try {
    await fetch('/api/generate');
  } finally {
    loading = false;
  }
};
```

### 排错清单
- 是否允许同一个操作并发发出多个请求？
- 是否会出现旧请求覆盖新请求结果？
- 是否在异步流程里维护了状态？

---

## 6. 前端防刷的边界：能做什么，不能做什么？
前端能做的，主要是**减少误操作**、**降低低成本重复请求**、**优化用户体验**。  
但前端做不到真正的安全防护，因为代码跑在浏览器里，用户随时可以打开 DevTools、改代码、重放请求。

所以你要把前端保护理解成三件事：

- 挡住正常用户的误触
- 降低无意识重复提交
- 配合服务端风控做提示

真正的安全控制，还是要落在服务端鉴权、限流、签名、黑白名单等策略上。前端的职责是把请求发对、把状态管好、把异常展示清楚。

### 最小可运行示例
```js
async function submitOnce() {
  if (window.__sending) return;
  window.__sending = true;
  try {
    await fetch('/api/chat', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer token'
      },
      body: JSON.stringify({ prompt: '你好' })
    });
  } finally {
    window.__sending = false;
  }
}
```

### 错例对比
**错例**：以为前端加个变量就能防住所有刷接口行为。  
**对的做法**：前端只做第一道体验防线，真正的防刷要靠后端配合。

### 排错清单
- 是否把前端保护当成了安全方案？
- 是否明确了前后端职责边界？
- 是否保留了服务端鉴权与风控？

---

## 7. 在 AI 接口里，最常见的三个场景怎么处理？
如果你对接的是 AI 服务，最容易出问题的不是“不会发请求”，而是这三个场景：

1. **发送生成请求**：用户点一次，发一次，结果回来一次。
2. **流式输出中重复点击**：还没输出完，又点了一次，容易造成两条流并行。
3. **返回 429**：接口告诉你“太频繁了”，前端要停止继续发，不要硬重试。

对于非流式接口，重点是禁用按钮和防重复提交。  
对于流式接口，除了禁用按钮，还要保证当前 SSE 或流式连接没结束前，不允许再次发起新任务，否则 UI 很容易乱。流式场景里，`loading` 不只是“请求发送中”，而是“整条连接还活着”。

### 最小可运行示例
```js
let loading = false;

async function sendPrompt() {
  if (loading) return;
  loading = true;
  try {
    const res = await fetch('/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ prompt: '帮我写一段文案' })
    });
    console.log(await res.json());
  } finally {
    loading = false;
  }
}
```

### 错例对比
**错例**：流式输出还没结束，又允许用户再次点击发送。  
**对的做法**：把“连接未结束”视为忙碌状态，直到完整结束再恢复按钮。

### 排错清单
- 当前接口是流式还是非流式？
- 是否在流式未结束时允许再次提交？
- 是否对 429 单独提示并停止自动重试？

---

## 8. 频控、限流和防刷的实战组合拳
只靠一个 `disabled` 按钮，其实不够稳。实战里更常见的是几种措施叠起来用：

- **第一层：UI 禁用**
  - 请求开始后，按钮变灰
  - 文案切成“生成中”
- **第二层：前端节流**
  - 例如 1 秒内重复点击直接拦掉
- **第三层：请求去重**
  - 同一参数、同一场景下，只保留一个进行中的请求
- **第四层：错误兜底**
  - 429 时提示用户稍后再试
  - 网络错误时允许手动重试

这样即使用户习惯性连点，页面也不会一口气发出一堆重复请求。  
如果是流式接口，还要注意“恢复按钮”的时机：不是收到第一段数据就放开，而是要等流结束、连接关闭、状态确认完成后再恢复，否则用户会误触发第二次请求。

### 最小可运行示例
```js
let loading = false;
let lastClick = 0;

async function submitPrompt(prompt) {
  const now = Date.now();
  if (loading) return;
  if (now - lastClick < 800) return;

  loading = true;
  lastClick = now;

  try {
    const res = await fetch('/api/chat', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer token'
      },
      body: JSON.stringify({ prompt })
    });

    if (res.status === 429) {
      alert('请求太频繁，请稍后再试');
      return;
    }

    console.log(await res.json());
  } finally {
    loading = false;
  }
}
```

### 错例对比
**错例**：只做节流，不做 loading 管理。  
**问题**：请求还没结束时仍可能并发。  
**对的做法**：节流负责“别太快”，loading 负责“没结束别再来”。

### 排错清单
- 是否同时做了 UI 禁用和频率限制？
- 是否区分“重复点击”和“请求未完成”？
- 是否对同参数请求进行了去重？
- 是否处理了 429 与网络错误？

---

## 9. 接口调试时怎么确认到底是“点太快”还是“接口本身有问题”？
新手很容易把所有失败都归因成“AI 接口坏了”，其实很多时候只是前端状态没管好。  
你可以按这个顺序排查：

1. 看按钮状态：是不是没禁用？
2. 看 Network 面板：是不是发了多个请求？
3. 看请求头：鉴权是否正确？
4. 看响应状态码：是 200、401、403，还是 429？
5. 看返回体：接口是不是已经告诉你“过于频繁”？

如果是跨域问题，浏览器控制台通常会有明显报错。这时根本不是“点太快”，而是请求压根没真正打到接口。  
所以接口调试方法很关键：不要只盯着页面，要同时看 DevTools 里的 Network 和 Console。前端排障的顺序永远是：**先确认请求有没有发出去，再确认服务端怎么回，再看页面状态有没有正确恢复**。

### 最小可运行示例
```js
fetch('https://example.com/api/chat', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token'
  },
  body: JSON.stringify({ prompt: '测试' })
}).then(res => {
  console.log('status:', res.status);
  return res.text();
}).then(console.log).catch(console.error);
```

### 错例对比
**错例**：只看页面有没有报错，不看网络请求。  
**对的做法**：Network、Console、响应状态码一起看，先定位是前端状态问题还是接口返回问题。

### 排错清单
- 是否确认按钮点击后只发出一次请求？
- 是否检查过请求头是否完整？
- 是否区分了 401/403/429/跨域错误？
- 是否在浏览器调试工具里确认过真实请求行为？

---

## 10. 真实项目里，前端防刷还要补哪几块？
如果只是演示页，上面的措施基本够用；放到真实项目里，还可以再加几层更稳的保护。

### 10.1 请求去重
同一个输入、同一个按钮，在短时间内只允许一个任务存在。  
这对 AI 对话、图片生成、摘要生成都很重要，因为这些场景的结果通常不该重复发起。

### 10.2 取消上一次请求
如果用户又发了新内容，老请求的结果就可能过时。  
这时可以用 `AbortController` 取消上一条请求，避免旧结果覆盖新结果。

### 最小可运行示例
```js
let controller = null;

async function sendLatest(prompt) {
  if (controller) controller.abort();
  controller = new AbortController();

  try {
    const res = await fetch('/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ prompt }),
      signal: controller.signal
    });
    return await res.json();
  } catch (err) {
    if (err.name === 'AbortError') return;
    throw err;
  }
}
```

### 错例对比
**错例**：新请求发出后，老请求还在跑，最后旧结果反而覆盖新结果。  
**对的做法**：必要时取消旧请求，只保留最新一次操作。

### 排错清单
- 是否需要取消旧请求？
- 是否会出现返回顺序错乱？
- 是否处理了 `AbortError`？

---

## 11. 把频控、限流、防刷和接口规范一起看
前面一直在说“别点太快”，但实际项目里，频控不是单独存在的，它和接口规范绑在一起。

你在前端对接 AI 接口时，至少要同时确认这些信息：

- **请求方法**：`GET`、`POST` 还是别的
- **请求头**：有没有 `Content-Type`、`Authorization`
- **鉴权方式**：Bearer Token、API Key 还是 Cookie 会话
- **返回码**：成功、参数错误、未授权、频率过高分别怎么处理
- **是否支持流式**：非流式一次性返回，流式则要实时接收
- **是否允许跨域**：本地调试时尤其重要

如果这些基础没搞明白，前端的频控策略也会失效。比如你按钮禁用了，但请求头漏了，服务端返回 401；或者你以为是点太快，实际是 CORS 没放行。  
所以别把频控看成一个孤立功能，它其实是“请求发起、状态管理、接口响应、错误处理”这一整套流程的一部分。

### 最小可运行示例
```js
async function callAI(prompt) {
  const res = await fetch('/api/chat', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer token'
    },
    body: JSON.stringify({ prompt })
  });

  if (!res.ok) {
    throw new Error(`请求失败：${res.status}`);
  }

  return res.json();
}
```

### 错例对比
**错例**：只关注“点了几次”，不看请求头、鉴权和返回码。  
**对的做法**：把频控放进完整的接口对接流程里一起检查。

### 排错清单
- 是否确认了请求方法和请求头？
- 是否知道不同状态码该怎么处理？
- 是否区分了接口限流和浏览器跨域报错？
- 是否明确了流式和非流式的恢复时机？

---

## 12. 最后给新手的实战建议
如果你刚开始接 AI 接口，建议按这个顺序做：

1. 先把按钮禁用和 `loading` 做对
2. 再加节流，防止连点
3. 再处理 429 和错误提示
4. 再补请求去重和取消旧请求
5. 最后再考虑流式输出和更细的 UI 状态管理

别一上来就想“完美防刷”，先把最常见的重复提交问题解决掉。  
记住一个很实用的判断标准：**用户点一次，页面最好只发一次；请求没回来，最好别再让他点第二次。**

### 最小可运行示例
```html
<button id="genBtn">开始生成</button>
<div id="msg"></div>
<script>
let loading = false;

document.getElementById('genBtn').onclick = async () => {
  if (loading) return;

  loading = true;
  const btn = document.getElementById('genBtn');
  const msg = document.getElementById('msg');
  btn.disabled = true;
  msg.textContent = '生成中，请稍后...';

  try {
    const res = await fetch('/api/chat', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer token'
      },
      body: JSON.stringify({ prompt: '写一段产品介绍' })
    });

    if (res.status === 429) {
      msg.textContent = '请求太频繁，请稍后再试';
      return;
    }

    msg.textContent = '生成完成';
  } catch (e) {
    msg.textContent = '网络或接口异常，请检查请求头、鉴权、跨域配置';
  } finally {
    loading = false;
    btn.disabled = false;
  }
};
</script>
```

### 错例对比
**错例**：只在失败时提示错误，成功和加载状态都没有。  
**对的做法**：请求前、请求中、请求后都要有明确状态。

### 排错清单
- 是否做到一次点击只发一次请求？
- 是否有清晰的 loading 和恢复逻辑？
- 是否正确处理 429、401、403 和跨域问题？
- 是否区分了前端体验保护与服务端安全防护？

---

## 结尾小结：前端的职责是“稳住体验”
这一节你只要记住一句话：**前端能防的是误触和重复提交，不能把自己当成最终安全墙。**

把它落实到页面上，就是四个动作：

- **别乱点**：按钮禁用、节流、重复提交拦截
- **别乱发**：请求去重、取消旧请求、正确处理异步
- **别乱猜**：看 Network、Console、状态码、返回体
- **别越界**：前端做体验保护，真正防刷靠服务端配合

如果你的页面已经做到**请求前有判断、请求中有 loading、请求后能恢复、异常时能提示**，再配合正确的请求头、鉴权、环境变量和跨域排查思路，你就已经把“重复请求”和“误操作”这类常见问题压下去了。

### 最后排错清单
- 按钮是否在请求期间禁用？
- 是否有 loading 状态提示？
- 是否正确处理异步返回顺序？
- 是否带齐请求头与鉴权信息？
- 是否区分了频控、限流和 429 错误？
- 是否在跨域环境下验证过接口可用性？
- 是否需要取消旧请求并避免结果覆盖？

# 第10章 跨域基础与接口调试方法：请求发不出去时先查什么

## 开篇引入：接口明明写对了，为什么浏览器就是不理你？

新手前端最常见的挫败感，不是不会写代码，而是“代码看起来没错，页面就是没结果”。你在 Postman 里能通，接口文档也照着抄了，`fetch` 也发了，控制台却一片红。

先别急着改代码。**前端调接口，第一步不是猜，而是查。**  
而且要按前端视角来查：先看浏览器有没有发请求，再看有没有被浏览器拦，再看接口有没有正常回，最后才看页面状态有没有更新。

本章你要会的，就是把问题分成四层：

1. 请求到底有没有发出去
2. 浏览器有没有拦
3. 服务器有没有回
4. 返回字段对不对，UI 状态有没有变

学会这套思路，以后遇到 AI API、普通业务接口、流式输出接口，你都能先定位问题，不至于一上来就乱撞。

---

## 一、跨域是什么，为什么前端会遇到它？

### 1. 什么叫跨域

前端里，**页面地址和接口地址的协议、域名、端口有任意一个不同**，就可能跨域。比如：

- 页面：`http://localhost:5173`
- 接口：`https://api.xxx.com`

这不是同源。

再比如：

- 页面：`http://localhost:3000`
- 接口：`http://localhost:8080`

虽然都在本机，但端口不同，也算不同源。

### 2. 浏览器为什么会管这件事

浏览器有同源策略，核心目的是防止网页随便读取别的网站数据。  
所以很多时候不是请求没发出去，而是**浏览器发了，但不让你直接拿到结果**。

这也是新手最容易误判的地方：你看到控制台报错，第一反应可能是“接口挂了”，其实接口可能是活的，只是浏览器没放行。

### 3. 前端为什么特别容易碰到

前端开发常见场景是：

- 本地开发用 `localhost`
- 线上接口用真实域名
- 测试环境和生产环境地址不同
- AI 接口常带 `Authorization`，更容易触发预检和跨域问题

所以接 AI 接口时，跨域几乎绕不过去。你要学的不是“消灭跨域”，而是识别它、看懂它、定位它。

### 最小可运行示例

```js
fetch("https://api.example.com/v1/chat", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer YOUR_TOKEN"
  },
  body: JSON.stringify({ prompt: "你好" })
})
  .then(res => res.json())
  .then(console.log)
  .catch(console.error);
```

### 错例对比

**错例 1：**
```js
fetch("api.example.com/v1/chat")
```
少了协议，浏览器会把它当相对路径，不是你要的绝对地址。

**错例 2：**
页面在 `http://localhost:3000`，接口在 `http://127.0.0.1:8000`。  
看起来都是本机，其实域名不同，也可能出问题。

### 排错清单

- 页面地址和接口地址是否同源
- 接口 URL 是否写全协议 `http/https`
- 是否把不同端口当成“同一个地址”
- 是请求没发出，还是发出了但被浏览器拦了

---

## 二、CORS 与预检请求：站在浏览器视角理解放行流程

### 1. CORS 是什么

CORS 可以理解成：**服务器告诉浏览器，哪些前端页面可以访问我**。  
它不是前端语法，而是浏览器和服务器之间的一套放行规则。

对前端来说，你不用背服务端配置细节，但一定要知道：**CORS 的成败，决定了浏览器能不能把响应交给你。**

### 2. 预检请求是什么

当请求比较“特殊”时，浏览器会先发一个 `OPTIONS` 检查请求。你可以把它理解成浏览器先问一句：

> “我等会儿要发正式请求，你允不允许？”

常见触发预检的情况包括：

- 使用了 `Authorization`
- 请求头比较复杂
- `Content-Type` 不是简单类型
- 某些非简单请求方法

所以你会在 Network 里看到：先 `OPTIONS`，再正式请求。**如果预检失败，正式请求通常不会继续。**

### 3. 前端该怎么理解

记住三句话就够用：

- 预检失败，正式请求通常不会继续
- CORS 错误不一定是接口坏了
- Postman 能通，不代表浏览器也能通

### 最小可运行示例

```js
fetch("https://api.example.com/v1/chat", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer YOUR_TOKEN"
  },
  body: JSON.stringify({ prompt: "你好" })
});
```

这个请求很可能触发预检，因为带了 `Authorization` 和 JSON 请求头。

### 错例对比

**错例：**  
你以为接口没返回，其实是 `OPTIONS` 预检阶段就失败了，正式请求根本没机会发。

**错例：**  
你在代码里随便加很多请求头，却没意识到它会让请求变复杂，触发浏览器额外检查。

### 排错清单

- Network 里有没有 `OPTIONS`
- `OPTIONS` 的状态码是否异常
- 响应头里有没有允许跨域相关信息
- 是否带了会触发预检的请求头
- 接口文档是否说明支持浏览器跨域调用

---

## 三、浏览器 Network 面板怎么看请求、响应和错误？

### 1. 先看哪里

打开开发者工具，重点看 **Network** 面板。  
养成习惯：**不要只看 Console，真正的证据在 Network。**

Console 里的报错往往只是结果，Network 里才有过程。你要找的是：请求有没有出去、服务器回了什么、浏览器为什么没放行。

### 2. 标准化调试 SOP：先看 Network，再看 Response/Headers，再看 Console，最后对照 Postman 或 curl

你可以把调试顺序固定成一套 SOP，这样每次都不会乱：

1. **先看 Network**：有没有请求、有没有 `OPTIONS`、状态码是什么  
2. **再看 Headers / Response**：请求头、响应头、返回体是否符合文档  
3. **再看 Console**：有没有跨域、解析、代码异常  
4. **最后用 Postman 或 curl 复现**：确认接口本身是否正常，再判断是不是浏览器环境问题

这个顺序很重要，因为它能帮你快速区分：  
**是前端代码没发出去，还是浏览器拦了，还是接口本身回错了。**

### 3. 关键字段怎么读

- **Headers**：看请求地址、方法、请求头、响应头
- **Payload / Request**：看你发出去的参数是否正确
- **Response**：看接口真实返回内容
- **Status**：看 200、400、401、403、500
- **Timing**：看请求是不是卡住、超时、被重试

### 最小可运行示例

```js
async function testApi() {
  const res = await fetch("/api/test", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ name: "赵同学" })
  });

  const data = await res.json();
  console.log("返回：", data);
}
testApi();
```

配合 Network 看这个请求是否真的发出、响应是否正常。

### 错例对比

**错例：**  
控制台只打印 `TypeError: Failed to fetch`，你就以为是接口崩了。  
实际上很可能是跨域、URL 错误、代理没配好。

**错例：**  
Network 里有请求，但你只看 Console，不看状态码和响应体，最后一直猜。

### 排错清单

- Network 里有没有对应请求
- 请求路径是否拼错
- 请求方法是否和文档一致
- 请求头是否缺失鉴权信息
- 状态码是否提示未授权、跨域、参数错误

---

## 四、怎么区分前端代码问题、跨域问题和接口本身问题？

### 1. 先做三个判断

#### 判断一：请求有没有到浏览器层
如果连 Network 都没有请求，先查代码有没有执行、按钮有没有点到、事件有没有触发、URL 有没有拼错。

#### 判断二：请求有没有被浏览器拦
如果有请求但状态异常，特别是 CORS、preflight、blocked 这类字样，优先考虑跨域。

#### 判断三：服务器有没有正常返回
如果状态码已经出来了，比如 400、401、403、500，那更像是接口参数、鉴权或服务端逻辑问题。

### 2. 三类问题，一条最短判断线索

- **前端代码问题**：Network 里根本没有请求，或请求地址明显不对
- **跨域问题**：Network 里有 `OPTIONS`，并出现 CORS / preflight / blocked
- **接口本身问题**：请求已发出，状态码明确返回 4xx/5xx，响应体里有错误信息

### 3. 前端常见误判

- 以为“接口没通”，其实是参数格式错了
- 以为“跨域了”，其实是 token 失效
- 以为“后端挂了”，其实只是返回字段和预期不一致

前端接 AI 接口时，这类误判尤其多。因为 AI 接口常返回包装结构，比如 `code / message / data`，如果你直接当纯文本去读，很容易拿错字段。

### 最小可运行示例

```js
fetch("/api/chat", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer demo-token"
  },
  body: JSON.stringify({ prompt: "给我一个标题" })
})
  .then(async res => {
    console.log("状态码：", res.status);
    const data = await res.json();
    console.log("数据：", data);
  });
```

### 错例对比

**错例：**
```js
const data = await res.json();
setTitle(data.title);
```

如果真实返回是 `{ code: 0, data: { title: "..." } }`，你就会取空。

**错例：**  
只判断“有没有报错”，不判断 `status` 和业务 `code`，错误会被吞掉。

### 排错清单

- 是代码没运行，还是请求被拦
- 是跨域失败，还是返回了错误状态码
- 是接口不通，还是字段结构不匹配
- 是网络失败，还是业务层失败
- 是真实接口问题，还是你用错了测试环境地址

---

## 五、错例对比：接口明明能用，但浏览器里就是失败

### 场景一：Postman 能用，浏览器失败

**原因可能是：**

- Postman 不受浏览器 CORS 限制
- 浏览器会发预检，Postman 不一定走同样流程
- 前端请求头带了 `Authorization`，触发预检失败

**正确思路：**  
不要拿 Postman 成功直接等于前端成功。它只能说明接口本身大概率可用，不能证明浏览器环境也没问题。

### 场景二：接口返回了，但页面没更新

**原因可能是：**

- 异步流程没处理好
- `await` 漏了
- 状态更新写错位置
- 取值字段和返回结构不一致

所以你不仅要会发请求，还要会管理 UI 状态：请求中显示 loading，成功后展示结果，失败后提示错误。

### 场景三：看起来像跨域，其实是 401

**原因可能是：**

- 鉴权 token 缺失
- 请求头没带 `Authorization`
- 过期 token 被接口拒绝

这种情况很容易误判成 CORS。实际上，服务器已经回你了，只是回的是“未授权”。

### 最小可运行示例

```js
async function sendPrompt() {
  try {
    const res = await fetch("https://api.example.com/v1/chat", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer YOUR_TOKEN"
      },
      body: JSON.stringify({ prompt: "你好" })
    });

    if (!res.ok) {
      console.log("HTTP错误：", res.status);
      return;
    }

    const data = await res.json();
    console.log("成功：", data);
  } catch (e) {
    console.log("网络/CORS/代码异常：", e);
  }
}
```

### 错例对比

**错例：**
```js
fetch(url).then(res => console.log(res))
```

只看到了 `Response` 对象，没有真正读 `json()`，也没有检查 `ok`。

### 排错清单

- 是否只验证了工具，不验证浏览器
- 是否遗漏了请求头里的鉴权信息
- 是否只看状态码，不看返回体
- 是否把业务失败当成网络失败
- 是否忽略了异步流程中的异常

---

## 六、接口调试的最小闭环：从点击到渲染，一步一步确认

前端调接口，最好养成一个小闭环：**触发、发送、接收、解析、渲染**。

### 1. 先确认事件真的触发了

有时候不是接口问题，而是按钮事件根本没进来。  
最简单的办法就是先打一条日志：

```js
button.addEventListener("click", async () => {
  console.log("按钮点到了");
});
```

如果这条日志都没有，先别看跨域，先查 DOM 绑定、事件写法和函数执行位置。

### 2. 再确认请求真的发出去了

```js
async function queryAI() {
  console.log("准备发请求");
  const res = await fetch("/api/chat", { method: "POST" });
  console.log("请求已返回", res.status);
}
```

这一步能帮你判断：是代码没执行，还是请求发出后出了别的问题。

### 3. 最后确认渲染逻辑没写错

很多新手以为拿到数据就结束了，其实还要更新 UI 状态。

```js
async function queryAI() {
  loading.value = true;
  error.value = "";
  try {
    const res = await fetch("/api/chat");
    const data = await res.json();
    answer.value = data.data?.text || "";
  } catch (e) {
    error.value = "请求失败，请检查网络或跨域配置";
  } finally {
    loading.value = false;
  }
}
```

这就是前端视角的完整流程：**请求、状态、结果、异常**都要管。

### 错例对比

**错例：**
```js
const data = await res.json();
answer.value = data.text;
```

如果接口返回的是 `data.data.text`，页面就会空白，但你还以为接口没返回。

### 排错清单

- 点击事件有没有触发
- 请求有没有真正发出
- `await` 有没有漏掉
- 返回字段是否和页面绑定一致
- loading、error、success 状态有没有正确切换

---

## 七、前端安全与防刷：能做什么，不能做什么

这里顺手提醒一句：**前端只能做体验层面的保护，真正的安全和防刷，必须依赖服务端配合。**

前端能做的主要是：

- 隐藏密钥，不把真实密钥直接写进页面代码
- 通过环境变量区分开发、测试、生产环境
- 按钮禁用、节流、防连点，减少用户误操作
- 对请求频率做体验限制，避免页面一顿乱点

前端不能做的事情是：

- 不能真正阻止别人抓包拿到请求
- 不能单靠页面代码保护接口不被滥用
- 不能代替服务端鉴权、限流和风控

所以你在浏览器侧要做的是“尽量少暴露、尽量少误触发”，而不是幻想前端能独立完成安全防护。

### 最小可运行示例

```js
let pending = false;

async function submit() {
  if (pending) return;
  pending = true;

  try {
    await fetch("/api/chat", { method: "POST" });
  } finally {
    pending = false;
  }
}
```

### 错例对比

**错例：**
```js
const token = "sk-xxxxxx";
```

把真实密钥直接写死在前端代码里，刷新页面、打包发布、抓包查看，都很危险。

### 排错清单

- 密钥是否写死在前端源码里
- 是否只做了前端按钮禁用，却没做服务端限流
- 是否把“防刷”理解成前端能单独完成
- 是否在开发环境和生产环境使用了同一套配置

---

## 结尾小结：先查顺序，再查原因，最后再改代码

前端接口调试，核心不是猜，而是分层定位。

你可以按这个顺序走：

1. **看 URL 和触发条件**：请求到底发没发
2. **看 Network**：有没有 `OPTIONS`、有没有状态码
3. **看请求头和响应头**：鉴权、`Content-Type`、CORS 是否匹配
4. **看 Response**：字段结构是不是你预期的
5. **看 Console**：有没有代码异常
6. **用 Postman 或 curl 对照**：确认接口本身是否正常
7. **看 UI 状态**：是否正确处理 loading、成功、失败

如果你只记住一句话，那就是：**浏览器里调接口，先看 Network，再看 Response/Headers，再看 Console，最后用 Postman 或 curl 复现。**

本章你已经掌握了跨域、预检、浏览器调试路径，以及如何区分前端问题、跨域问题和接口本身问题。下一步，当你面对更复杂的 AI 接口返回、错误码和流式输出时，就不会再一头雾水了。

# 第11章 把前面所有步骤串起来：做一个可上线的 AI 接口接入小项目

前面我们已经讲过怎么读接口文档、怎么发请求、怎么处理返回值、怎么管错误、怎么理解流式输出。到了这一章，咱们把这些知识串成一个能跑、能调、能扩展的小项目。

你可以把它理解成一个“AI 问答页面”：

- 页面上有输入框，用户输入问题
- 点击发送后，请求 AI 接口
- 页面展示返回结果
- 出错时给出清晰提示
- 保存历史结果
- 同时兼容非流式和流式两种接口

这一章重点不是“功能多”，而是“结构对”。前端接 AI 接口，最怕页面能跑，但代码一团乱；今天能用，明天换个接口就全崩。

**本章你要会什么：**看懂文档、拼出请求、管理状态、处理错误、区分流式与非流式，并把这一套做成能复用的小项目。  
**提醒一句：**这一章始终站在前端视角讲，只讲你在浏览器里怎么把页面接起来。

---

## 一、项目需求拆解：先想清楚要做什么

做接口接入项目，第一步不是写代码，而是拆需求。新手常见问题是一上来就写 `fetch`，写到一半才发现 loading 怎么管、错误怎么显示、流式结果往哪放。

### 1.1 这个页面至少要有五块能力

**输入区**
- 用户输入问题
- 可选：选择“普通回答 / 流式输出”

**发送区**
- 点击按钮触发请求
- 请求前禁用按钮，防止重复提交

**展示区**
- 展示 AI 返回内容
- 流式时边到边显示

**辅助区**
- 错误提示
- 历史结果列表
- loading 状态

**调试区**
- 能看请求状态
- 能看返回结果是否符合文档
- 能快速定位是“没发出去”还是“返回读错了”

这五块能力对应前端接 AI 接口的五个核心问题：**怎么发、怎么等、怎么显示、怎么兜底、怎么查错**。

### 1.2 最小可运行示例

```html
<div id="app">
  <textarea id="prompt" placeholder="请输入你的问题"></textarea>
  <div>
    <label><input type="radio" name="mode" value="normal" checked /> 普通回答</label>
    <label><input type="radio" name="mode" value="stream" /> 流式输出</label>
  </div>
  <button id="sendBtn">发送</button>
  <pre id="result"></pre>
  <div id="error"></div>
  <ul id="history"></ul>
</div>
```

```js
const state = {
  loading: false,
  error: '',
  result: '',
  history: []
};

function render() {
  resultEl.textContent = state.result;
  errorEl.textContent = state.error;
  historyEl.innerHTML = state.history
    .map(item => `<li>${item.prompt} => ${item.answer}</li>`)
    .join('');
  sendBtn.disabled = state.loading;
}
```

先把页面骨架和状态骨架搭起来。很多新手的问题不是不会发请求，而是页面状态没设计好，后面越写越乱。

### 1.3 错例对比

**错例：先写请求，后补 UI**
```js
async function send() {
  const res = await fetch('/api/chat');
}
```

问题是：没有输入校验、没有 loading、没有错误提示，用户体验差，也不好排查问题。

**正例：先定义页面状态**
```js
let state = {
  input: '',
  loading: false,
  error: '',
  result: '',
  history: []
};
```

先有状态，再有请求，再有展示，逻辑才完整。

### 1.4 排错清单

- 页面是否有输入、发送、展示、错误、历史这些区域？
- 是否提前设计了 loading 和 error 状态？
- 是否考虑了重复点击发送？
- 是否需要保存历史记录？
- 是否能一眼看出当前是普通模式还是流式模式？

---

## 二、从接口文档到页面代码：一步一步落地

接 AI 接口，真正难的不是请求本身，而是**看文档**。前端要关心的通常是：

- 接口地址
- 请求方法
- 请求头
- 鉴权方式
- 请求参数
- 返回格式
- 是否流式
- 错误码含义
- 是否跨域

### 2.1 先读懂接口文档的关键字段

假设文档写的是：

- `POST /v1/chat`
- 请求头：`Content-Type: application/json`
- 鉴权：`Authorization: Bearer xxx`
- 请求体：`{ "prompt": "你好", "stream": false }`
- 返回：`{ "data": { "text": "..." } }`

你要立刻翻译成前端动作：

1. 构造 JSON 请求体
2. 把 token 放进请求头
3. 发 POST 请求
4. 解析返回 JSON
5. 渲染 `data.text`

接口文档不是背书用的，而是给你“翻译成代码”的。每看到一个字段，都要问：**放请求头、请求体，还是响应体？**

### 2.2 最小可运行示例

```js
const API_BASE = 'https://api.example.com';
const API_KEY = import.meta.env.VITE_AI_KEY;

async function chat(prompt, stream = false) {
  const res = await fetch(`${API_BASE}/v1/chat`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${API_KEY}`
    },
    body: JSON.stringify({ prompt, stream })
  });

  if (!res.ok) throw new Error(`请求失败：${res.status}`);
  return await res.json();
}
```

关键点：

- `Content-Type` 告诉服务端你发的是 JSON
- `Authorization` 放鉴权信息
- 密钥从环境变量读取，不要硬编码
- `res.ok` 用来判断 HTTP 状态是否正常
- `stream` 明确告诉接口你要普通模式还是流式模式

### 2.3 错例对比

**错例：把鉴权写进请求体**
```js
body: JSON.stringify({
  prompt,
  apiKey: 'xxx'
})
```

问题：密钥不该放在请求体里，更不该直接写死在前端代码中。接口文档要求放请求头，就必须放请求头。

**错例：忽略请求头**
```js
fetch(url, { method: 'POST', body: JSON.stringify({ prompt }) })
```

如果接口要求 `Content-Type` 和 `Authorization`，少一个都可能 401 或 415。

### 2.4 排错清单

- 请求方法是否和文档一致？
- 请求头是否包含 `Content-Type`？
- 鉴权是否按文档要求放在 `Authorization`？
- 返回字段名是不是 `data.text`，还是别的结构？
- 是否检查了 `res.ok` 或错误码？
- 本地环境变量是否真的生效了？

---

## 三、非流式与流式接口：同一个项目里怎么同时支持

很多 AI 接口都提供两种模式：

- **非流式**：一次性返回完整答案
- **流式**：内容一段一段回来，页面边收边显示

前端要做的不是二选一，而是都能接。

### 3.1 非流式：简单直接，适合先跑通

```js
async function sendNormal(prompt) {
  const data = await chat(prompt, false);
  return data.data.text;
}
```

非流式结构清楚、容易调试，适合先把整条链路跑通。建议先做这个版本，确认接口、请求头、鉴权、返回结构都没问题，再去碰流式。

### 3.2 流式：体验更好，但代码更复杂

流式通常会用 SSE 或类 SSE 的方式。你可以先把 SSE 理解成一句话：**服务端不是一次性把结果都吐给你，而是分多次推给前端，前端边接边显示。**

前端最关键的边界是：**你负责接收、解析、拼接和渲染，不负责定义服务端怎么发。**

```js
async function sendStream(prompt, onChunk) {
  const res = await fetch(`${API_BASE}/v1/chat`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${API_KEY}`
    },
    body: JSON.stringify({ prompt, stream: true })
  });

  if (!res.ok) throw new Error(`HTTP ${res.status}`);

  const reader = res.body.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { value, done } = await reader.read();
    if (done) break;
    onChunk(decoder.decode(value, { stream: true }));
  }
}
```

### 3.3 两者差异怎么理解

**非流式**
- 简单
- 适合快速验证
- 用户需要等完整结果

**流式**
- 体验更好
- 代码更复杂
- 要处理分片、拼接、中断
- 前端要持续更新 UI，而不是等最后一次性渲染

新手建议先把非流式调通，再改成流式，理解会更稳。

### 3.4 错例对比

**错例：把流式当普通 JSON 读**
```js
const data = await res.json();
```

如果返回的是分片内容，这样会直接报错。

**错例：流式时不更新 UI**
```js
onChunk(chunk);
```

但没有把 chunk 拼到页面上，用户看不到变化。

**正例：边收边显示**
```js
state.result += chunk;
render();
```

### 3.5 排错清单

- 接口是否明确支持 stream？
- 流式返回是不是要用 `reader` 读取？
- 页面是否有“正在生成中”的状态？
- 中断时是否能停止请求？
- 非流式和流式是否共用同一套状态管理？
- 返回内容是整段 JSON，还是一段段文本流？

---

## 四、如何整理代码结构：请求层、状态层、UI 层

一个能上线的小项目，最重要的是分层。别把请求、状态、页面渲染全写在一个函数里，否则后面维护会很痛苦。

### 4.1 推荐结构

- **请求层**：只负责和接口打交道
- **状态层**：只负责保存页面状态
- **UI 层**：只负责展示和交互

### 4.2 最小可运行示例

```js
// 请求层
async function requestChat(prompt, stream = false) {
  const res = await fetch(`${API_BASE}/v1/chat`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${API_KEY}`
    },
    body: JSON.stringify({ prompt, stream })
  });
  return res;
}

// 状态层
const state = {
  loading: false,
  error: '',
  result: '',
  history: [],
  mode: 'normal'
};

// UI 层
function render() {
  document.querySelector('#result').textContent = state.result;
  document.querySelector('#error').textContent = state.error;
  document.querySelector('#sendBtn').disabled = state.loading;
}
```

### 4.3 错例对比

**错例：所有逻辑混在一起**
```js
sendBtn.onclick = async () => {
  const res = await fetch(...);
  const data = await res.json();
  resultEl.textContent = data.data.text;
};
```

问题很明显：

- 不能复用
- 不能测试
- 不能扩展流式
- 不方便处理错误和历史
- 一旦接口字段变了，整段代码都要翻

**正例：先封装，再调用**
- 请求层封装接口变化
- 状态层管理 loading、error、history、mode
- UI 层只关心展示

### 4.4 排错清单

- 接口地址是不是只写在请求层？
- 密钥是不是只从环境变量读取？
- 页面状态是否统一管理？
- UI 是否只做渲染，不混业务？
- 流式和非流式是否都走同一套状态更新入口？

---

## 五、异步流程、错误处理、历史结果和接口调试：让项目真的能上线

能跑不等于能用。前端接 AI 接口，最常见的线上问题就是：报错不清楚、用户反复点、结果丢失、调试困难。

### 5.1 异步流程要有完整闭环

前端最常见的异步流程就是这五步：

1. 点击发送，设置 `loading = true`
2. 清空旧错误
3. 发起请求
4. 成功后更新结果和历史
5. 失败后显示错误，最后恢复按钮状态

```js
async function safeSend(prompt) {
  if (state.loading) return;

  try {
    state.loading = true;
    state.error = '';
    state.result = '';
    render();

    const res = await requestChat(prompt, state.mode === 'stream');
    if (!res.ok) throw new Error(`HTTP ${res.status}`);

    const data = await res.json();
    state.result = data.data.text || '无返回内容';
    state.history.unshift({ prompt, answer: state.result });
  } catch (err) {
    state.error = err.message || '请求失败';
  } finally {
    state.loading = false;
    render();
  }
}
```

这段代码把关键动作串起来了。注意这里的 `loading` 不只是“显示一个字”，它还负责：

- 禁止重复点击
- 告诉用户“当前正在请求”
- 防止状态被二次覆盖

这就是前端做接口接入时常见的异步流程：**开始、执行、成功、失败、结束**。

### 5.2 接口调试方法：按固定 SOP 来排查

前端调接口，建议按这个顺序排查：

1. 先看浏览器 DevTools 的 Network，确认请求有没有发出去
2. 再看 Request Headers，确认请求头里有没有鉴权和 `Content-Type`
3. 再看 Request Payload，确认请求体是不是文档要求的 JSON
4. 再看 Response，确认返回字段名和文档一致
5. 再看 Status Code，区分 401、403、429、500
6. 流式接口再看 `reader` 是否持续拿到 chunk
7. 如果是跨域，再看 Console 和 Network 里的预检请求是否失败
8. 必要时用 Postman 或 `curl` 对照复现，确认是不是前端代码问题

这个 SOP 很适合新手：先看浏览器，再对照文档，最后再和 Postman、`curl` 比一下，能少走很多弯路。

### 5.3 跨域基础：前端为什么会“明明地址对了却请求失败”

如果接口不是同源，浏览器可能会拦截请求，这就是跨域问题。前端要记住三件事：

- 不是所有“请求失败”都是代码写错
- 浏览器会基于同源策略做限制
- 你在 DevTools 里看到的报错信息，往往比接口文档更直接

前端排查跨域时，先看：

- 请求地址是不是拼错了
- 是否触发了预检请求
- 响应头里有没有允许跨域
- 本地开发时是否用了代理配置

这里不展开后端配置细节，只提醒你：**跨域是前端调接口最常见的障碍之一，必须会看浏览器报错。**

### 5.4 错例对比

**错例：只提示“请求失败”**
```js
catch (e) {
  state.error = '请求失败';
}
```

这样用户和开发者都不知道原因。前端做错误提示，不是为了“有字就行”，而是为了让问题可定位。

**正例：保留状态码和关键信息**
```js
catch (e) {
  state.error = e.message;
}
```

必要时再进一步区分 401、429、500。

### 5.5 排错清单

- 错误信息是否具体？
- 是否区分网络错误和接口错误？
- 是否记录历史结果？
- 是否在 DevTools 里检查了请求头、请求体、响应体？
- 是否处理了 401、429、500？
- 跨域报错时是否先确认前端请求地址写对了？
- 是否用 Postman / `curl` 复现过同样的请求？

---

## 六、前端安全、鉴权、环境变量、跨域与防刷：上线前必须想明白

最后这部分很重要。很多新手把 AI 密钥直接写进前端代码，结果页面一上线就泄露了。

### 6.1 密钥不要硬编码

```js
// 错误示例
const API_KEY = 'sk-xxx';
```

应该使用环境变量：

```js
const API_KEY = import.meta.env.VITE_AI_KEY;
```

注意：**环境变量不是万能安全**。它只是让你不要把密钥写死在源码里。真正上线时，更稳妥的方式通常是由你自己的服务层转发请求，但在前端阶段，先要学会：**别直接裸写密钥**。

### 6.2 鉴权放在请求头

```js
headers: {
  'Content-Type': 'application/json',
  'Authorization': `Bearer ${API_KEY}`
}
```

这比把 token 放在 URL 里更合理，也更符合接口文档常见要求。

### 6.3 防刷与频控思路

前端能做的防刷有限，但可以先做基础保护。这里要把边界讲清楚：**前端只能做体验层和基础拦截，真正的防刷、配额控制、风控和权限限制，还是要依赖服务端。**

前端可以先做这些：

- 按钮发送后立即禁用
- 请求未完成前不允许再次提交
- 对同一输入做简单去重
- 对 429 错误做提示和退避重试

```js
if (state.loading) return;
state.loading = true;
sendBtn.disabled = true;
```

这类保护看起来简单，但很有用。它能避免用户连点导致重复请求，也能减少页面状态混乱。

### 6.4 错例对比

**错例：不禁用按钮**
用户连点三次，发出三次请求，既费钱又混乱。

**正例：请求中锁定 UI**
```js
sendBtn.disabled = true;
try {
  // 请求逻辑
} finally {
  sendBtn.disabled = false;
}
```

### 6.5 排错清单

- 密钥是否写在环境变量中？
- 请求头是否正确携带鉴权信息？
- 是否禁用了重复提交？
- 是否考虑 429 频控？
- 页面是否明确告知“正在请求中”？
- 是否能区分“跨域失败”和“接口返回错误”？
- 是否明确知道前端只做基础防刷，没有误把自己当成安全最终防线？

---

## 七、把所有模块串起来：一个完整的小项目骨架

下面把请求层、状态层、UI 层串成一个最小闭环。这个版本的目标很明确：**能跑、能看、能改、能扩展**。

### 7.1 完整示例

```js
const API_BASE = 'https://api.example.com';
const API_KEY = import.meta.env.VITE_AI_KEY;

const state = {
  loading: false,
  error: '',
  result: '',
  history: [],
  mode: 'normal'
};

function render() {
  document.querySelector('#result').textContent = state.result;
  document.querySelector('#error').textContent = state.error;
  document.querySelector('#sendBtn').disabled = state.loading;
  document.querySelector('#history').innerHTML = state.history
    .map(item => `<li>${item.prompt} => ${item.answer}</li>`)
    .join('');
}

async function requestChat(prompt, stream = false) {
  const res = await fetch(`${API_BASE}/v1/chat`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${API_KEY}`
    },
    body: JSON.stringify({ prompt, stream })
  });
  return res;
}

async function send(prompt) {
  if (state.loading) return;

  state.loading = true;
  state.error = '';
  state.result = '';
  render();

  try {
    const stream = state.mode === 'stream';
    const res = await requestChat(prompt, stream);

    if (!res.ok) throw new Error(`HTTP ${res.status}`);

    if (!stream) {
      const data = await res.json();
      state.result = data.data.text || '无返回内容';
    } else {
      const reader = res.body.getReader();
      const decoder = new TextDecoder();
      while (true) {
        const { value, done } = await reader.read();
        if (done) break;
        state.result += decoder.decode(value, { stream: true });
        render();
      }
    }

    state.history.unshift({ prompt, answer: state.result });
  } catch (err) {
    state.error = err.message || '请求失败';
  } finally {
    state.loading = false;
    render();
  }
}

document.querySelector('#sendBtn').onclick = () => {
  const prompt = document.querySelector('#prompt').value.trim();
  const mode = document.querySelector('input[name="mode"]:checked').value;
  state.mode = mode;

  if (!prompt) {
    state.error = '请输入内容';
    render();
    return;
  }

  send(prompt);
};

render();
```

### 7.2 这段代码的结构边界

- **请求层**：`requestChat`
- **状态层**：`state`
- **UI 层**：`render` 和按钮点击事件
- **异步流程**：`send`
- **鉴权与请求头**：`Authorization` 和 `Content-Type`
- **环境变量**：`import.meta.env.VITE_AI_KEY`
- **流式与非流式差异**：`stream` 参数和 `reader` 分支
- **错误与重试入口**：`catch` 和 `state.error`

这就是一个前端接 AI 接口的基本骨架。以后你接别的 AI 服务，只要把接口地址、请求参数、返回字段按文档替换掉，主体结构基本都能复用。

### 7.3 错例对比：功能能跑，但后续难维护

**错例：所有东西堆在一个函数里**
```js
sendBtn.onclick = async () => {
  sendBtn.disabled = true;
  resultEl.textContent = '';
  errorEl.textContent = '';

  try {
    const res = await fetch('/v1/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json', 'Authorization': 'Bearer xxx' },
      body: JSON.stringify({ prompt: promptEl.value, stream: false })
    });

    const data = await res.json();
    resultEl.textContent = data.data.text;
    historyEl.innerHTML += `<li>${promptEl.value}: ${data.data.text}</li>`;
  } catch (e) {
    errorEl.textContent = e.message;
  }

  sendBtn.disabled = false;
};
```

这段能跑，但问题不少：

- 密钥写死
- 请求和渲染混在一起
- 没有统一状态
- 流式时要重写一遍
- 接口变动时改动范围大

**正例：先拆层，再拼装**
- 请求层只管接口
- 状态层只管数据
- UI 层只管显示
- 异步流程统一入口处理

---

## 结尾小结：把“会写请求”变成“能接上线”

这一章你要带走的，不只是一个页面，而是一套接入思路：

1. **先拆需求**：输入、发送、展示、错误、历史  
2. **再读文档**：地址、方法、请求头、鉴权、返回格式、流式与非流式  
3. **再写请求层**：把 fetch/axios 封装好  
4. **再做状态层**：管理 loading、error、result、history、mode  
5. **再接 UI 层**：按钮、输入框、结果区、错误提示  
6. **最后做上线思维**：环境变量、鉴权、防刷、调试、跨域、错误分级  

如果你能独立做出这一页，说明你已经不是“看接口发怵”的新手了。下一步，你就可以把这个模式复用到更多 AI 服务上：翻译、摘要、问答、改写、图片生成，底层思路都是一样的。

**行动建议：**
- 先用非流式接口跑通整个页面
- 再把结果改成流式
- 再把请求层、状态层、UI 层拆开
- 最后补上错误处理、环境变量、重复提交保护和跨域排查
- 调试时固定走一遍：Network → Response → 文档 → Postman / `curl`

**排错总清单：**
- 请求头是否带齐？
- 鉴权是否正确？
- 环境变量是否生效？
- `loading` 是否能锁住按钮？
- 普通模式和流式模式是否都能跑？
- DevTools 里能否看到真实请求和响应？
- 跨域报错时是否先确认地址和预检请求？
- 前端是否只做体验层防刷，没有误把自己当成安全最终防线？

做到这一步，你就真正具备了独立完成 AI 服务接入的能力。  
**本章你已掌握：从文档到页面的完整接入闭环、非流式与流式的统一处理、请求层/状态层/UI 层的拆分方法。下一步，你可以把这套骨架迁移到任意新的 AI 接口，只需要替换文档里的地址、参数和返回字段即可。**

# 第12章 收尾与排查手册：新手接 AI 接口时最常见的问题总整理

前面几章，我们已经把“看文档、发请求、收响应、做状态、管错误”走了一遍。到了收尾这章，老赵不再加新知识，而是把整套方法收成一份**前端实战排查手册**。以后你再接一个新的 AI 服务，先按这套顺序查，基本就能把问题定位出来。先记住一句话：**前端先读文档，再发请求，再处理状态，再看错误，再做调试。**

## 一、前端接入 AI 接口，先按什么顺序查？

最稳的顺序不是先看代码，而是先看**文档 → 请求头 → 鉴权 → 参数 → 返回值 → 页面状态 → 调试结果**。  
很多新手一上来就盯着 `fetch` 写法，其实真正出问题的地方，往往是文档没看细、字段没对上，或者请求头少了一项。

### 最小可运行示例
```js
async function askAI() {
  const res = await fetch('/api/ai/chat', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer xxx'
    },
    body: JSON.stringify({ prompt: '你好' })
  });
  return res.json();
}
```

### 错例对比
```js
fetch('/api/ai/chat', {
  method: 'POST',
  body: { prompt: '你好' } // 错：没有 JSON.stringify
});
```

### 排错清单
- 文档里要求的是 `POST` 还是 `GET`
- 请求头有没有 `Content-Type`
- 鉴权字段是 `Authorization` 还是别的
- 参数名是否写错
- 返回值是数组、对象，还是嵌套字段
- 页面是否处理了 `loading`、成功、失败三种状态

## 二、请求头、鉴权、环境变量、跨域、状态管理怎么复盘？

### 1）请求头与鉴权

很多 AI 接口都要带鉴权信息，但**不要把密钥直接写死在前端代码里**。前端更常见的做法，是把它放到环境变量里做开发配置，或者通过自己的中转接口去请求。  
凡是会出现在浏览器里的内容，都不要默认它是“绝对安全”的。

### 最小可运行示例
```js
const token = import.meta.env.VITE_AI_TOKEN;

fetch('/api/ai/chat', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  }
});
```

### 错例对比
```js
const token = 'sk-xxxx'; // 错：直接写死
```

### 排错清单
- 请求头里是否真的带上了 `Authorization`
- 文档要求的鉴权格式是不是 `Bearer xxx`
- 环境变量名是否写对
- 前端里有没有把敏感密钥直接暴露
- 开发环境和线上环境是否用了不同配置

### 2）环境变量

环境变量适合存开发配置，比如接口地址、调试开关、代理地址等，但它不是“万能保险箱”。前端里能被打包进浏览器的变量，本质上都可能被看到，所以**真正敏感的密钥，不该直接暴露在前端**。  
你可以把“接口地址、是否开启调试、是否走本地代理”放进环境变量，但别把核心密钥当成前端秘密。

### 3）跨域基础

如果你直接请求第三方 AI 域名，浏览器可能拦截，这不是代码写坏了，而是**跨域限制**。常见处理方式有三个：
- 本地开发时配代理
- 让自己的服务端做中转
- 按接口文档要求配置允许来源

### 4）UI 状态管理

前端至少要同时管理：
- `loading`：正在请求
- `data`：成功结果
- `error`：失败信息

### 最小可运行示例
```js
const state = { loading: false, data: null, error: '' };

async function submit() {
  state.loading = true;
  state.error = '';
  try {
    const res = await askAI();
    state.data = res;
  } catch (e) {
    state.error = '请求失败';
  } finally {
    state.loading = false;
  }
}
```

### 错例对比
```js
async function submit() {
  state.loading = true;
  const res = await askAI();
  state.data = res;
} // 错：失败时没有 finally，loading 可能收不回来
```

### 排错清单
- 按钮点了没反应，是不是 `loading` 把按钮禁用了却没恢复
- 返回成功了，页面没显示，是不是字段取错
- 报 401/403，是不是鉴权失效
- 报跨域，是不是请求地址直接打到了第三方域名
- 控制台里是否能看到真实报错，而不是只看到“请求失败”

## 三、流式与非流式接口，怎么选？

这是前端接 AI 时很关键的一步。先别急着追求“高级”，先分清楚接口类型。

### 判断标准
- **非流式**：等全部结果返回后一次性显示，适合短回复、实现简单
- **流式**：边生成边显示，体验更好，适合长回答、聊天场景

### 最小可运行示例
非流式：
```js
const res = await fetch('/api/ai/chat');
const data = await res.json();
```

流式：
```js
const res = await fetch('/api/ai/chat-stream');
const reader = res.body.getReader();
```

### 错例对比
```js
const data = await res.json(); // 错：把流式接口当普通 JSON 读
```

### 排错清单
- 文档有没有写 `stream: true`
- 返回的是 JSON，还是分片数据
- 页面是否准备了“逐步追加文本”的逻辑
- 流式时是否要处理断开、重连、结束标记
- 如果只是做简单问答，先用非流式更稳

## 四、常见踩坑：格式错误、权限错误、状态错误、调试盲点

### 1）格式错误
最常见的是参数名、请求体结构、返回字段名不一致。比如文档写的是 `prompt`，你写成了 `content`；文档返回的是 `result.text`，你却去读 `data.message`。这类问题不会报语法错，但页面就是不对。

### 2）权限错误
401 通常是没登录或 token 失效；403 通常是有身份但没权限。看到这两个状态码，先别急着改页面，先检查请求头里是否真的带上了鉴权信息。

### 3）状态错误
请求发出去了，但界面一直 loading，往往是 `finally` 没写，或者异常没被捕获。前端处理异步流程时，最怕“出错后状态没收回来”，按钮就一直灰着。

### 4）调试盲点
不要只看页面，记得打开浏览器 DevTools：
- Network 看请求有没有发出去
- Response 看返回内容
- Console 看前端报错
- 对照文档检查请求头和参数

### 排错清单
- 请求是否成功进入 Network
- 状态码是多少
- 返回体是不是你以为的格式
- 前端是否把错误吞掉了
- 是否用了错误的接口地址
- 页面提示是否和真实失败原因一致

## 五、怎样把这套方法迁移到新的 AI 服务？

记住一个简单套路：**先对照文档，再做最小请求，再补状态，再补错误，再补流式**。  
前端真正要掌握的，不是某一家接口的固定写法，而是这套排查顺序。换了新服务，也照样能用。

### 迁移步骤
1. 读文档，确认请求方式、请求头、鉴权
2. 先用最小请求打通
3. 再接入 UI 和 `loading`
4. 再处理错误码和提示
5. 最后再做流式、重试、频控

### 排错清单
- 新服务的鉴权方式是否和旧服务一致
- 返回字段是否不同
- 是否需要单独的流式接口
- 是否有调用频率限制
- 是否需要额外的安全策略
- 是否能先在本地用一个最小示例跑通

## 结尾：把排查顺序记成一句话

老赵建议你以后每次接新 AI 接口，都按这句口诀来查：

**看文档、验请求头、查鉴权、对参数、看跨域、试状态、分流式与非流式，最后再看页面。**

你只要把这套闭环养成习惯，就不再是“看到接口文档就发怵”的新手了。接下来无论换什么 AI 服务，你都能先跑通最小示例，再一点点扩展成完整功能。

---

更多内容请访问：[https://tutor.lao-zhao.com/](https://tutor.lao-zhao.com/)
