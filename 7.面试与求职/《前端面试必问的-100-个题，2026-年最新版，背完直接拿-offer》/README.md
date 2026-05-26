# 《前端面试必问的 100 个题，2026 年最新版，背完直接拿 offer》

<!-- PAGEBREAK -->

![作者介绍图](05_full_book_draft_assets/asset-66469f7644.png)

<!-- PAGEBREAK -->

## 目录

- 第1章 JavaScript 变量、作用域与内存管理
- 第2章 原型链与继承：从原理到实战
- 第3章 数据类型判断与隐式转换
- 第4章 事件循环与异步编程
- 第5章 this、call、apply、bind 全解
- 第6章 防抖、节流与手写实现
- 第7章 深拷贝与 JSON 序列化
- 第8章 盒模型与 box-sizing 解析
- 第9章 Flexbox 布局核心要点
- 第10章 Grid 布局与响应式设计
- 第11章 BFC 原理与清除浮动
- 第12章 Vue3 核心原理与响应式系统
- 第13章 Vue 组件通信与状态管理
- 第14章 React Hooks 原理与实战
- 第15章 React 性能优化与状态管理
- 第16章 浏览器渲染机制与性能优化
- 第17章 缓存机制与存储方案
- 第18章 跨域问题与前端安全
- 第19章 HTTP/HTTPS 与网络协议
- 第20章 前端工程化与构建工具
- 第21章 常见数组操作手写题
- 第22章 Promise 与异步手写题
- 第23章 数据结构与算法应用
- 第24章 前端系统设计思路
- 第25章 项目经验与场景题应对

# 第1章 JavaScript 变量、作用域与内存管理

## 第一章：作用域、闭包与执行上下文

> **覆盖**：8道题 | **核心路径**：变量声明 → 作用域规则 → 闭包原理 → 内存管理

---

## 题目一：什么是作用域？执行上下文又是什么？ 【高频★★】

**作用域**决定变量可访问的范围；**执行上下文**是代码运行时的环境记录。

作用域三种类型：

- **全局作用域**：代码任何位置都能访问
- **函数作用域**：函数内部可访问，外部无法访问
- **块级作用域**：`let/const` 在 `{}` 内创建，仅该块内可用

**执行上下文创建过程**：编译阶段扫描变量声明提升、建立作用域链；执行阶段按顺序执行代码。

**三要素**：变量对象、作用域链、this绑定。

**this绑定规则**：`obj.method()` 指向obj；`fn()` 直接调用指向window/undefined；`new Constructor()` 指向新实例。

**易错提醒**：作用域是静态的编译时规则，执行上下文是动态的运行时实例。

---

## 题目二：var、let、const 有什么区别？ 【高频★★】

| 特性 | var | let/const |
|------|-----|-----------|
| 作用域 | 函数级 | 块级 |
| 可重复声明 | ✓ | ✗ |
| 变量提升 | ✓（undefined） | ✓（TDZ不可访问） |

```javascript
var a = 1; var a = 2; // ✓
let b = 1; let b = 2; // SyntaxError
console.log(x); var x = '提升'; // undefined
console.log(y); let y = '死区'; // ReferenceError
```

**编码建议**：新代码默认用 `const`；需要修改变量用 `let`；循环计数器优先用 `let`。

---

## 题目三：for 循环中 var 和 let 的行为差异 【高频★★】

```javascript
for (var i = 0; i < 5; i++) {
  setTimeout(() => console.log(i), 100); // 6 6 6 6 6
}
for (let i = 0; i < 5; i++) {
  setTimeout(() => console.log(i), 100); // 0 1 2 3 4
}
```

**核心思路**：var 函数作用域导致所有回调共享同一个 i（值为6），let 块级作用域使每次迭代创建独立 i。

**用 IIFE 解决**：`((j) => setTimeout(() => console.log(j), 100))(i)`

---

## 题目四：什么是闭包？有哪些应用场景？ 【高频★★★】

**核心思路**：闭包 = 函数 + 记住外部变量。函数即使在作用域外执行，仍能访问定义时的词法环境。

```javascript
function outer() {
  let count = 0;
  return function inner() {
    count++;
    console.log(count);
  };
}
const counter = outer();
counter(); // 1
counter(); // 2
```

**四大应用场景**：

1. **私有变量**：封装状态，外部无法直接访问
2. **防抖节流**：保存定时器状态，控制执行频率

```javascript
function debounce(fn, delay) {
  let timer = null;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}
function throttle(fn, delay) {
  let timer = null;
  return function(...args) {
    if (timer) return;
    timer = setTimeout(() => {
      fn.apply(this, args);
      timer = null;
    }, delay);
  };
}
```

3. **记忆化**：缓存计算结果，避免重复计算
4. **函数柯里化**：把多参数函数转为逐级调用的单参数函数

---

## 题目五：手写一个闭包计数器 【高频★★】

```javascript
function createCounter() {
  let count = 0;
  return {
    increment() { return ++count; },
    decrement() { return --count; },
    getCount() { return count; }
  };
}
```

限制范围（0-100）：`increment() { if (count >= 100) return count; return ++count; }`

**易错提醒**：私有变量必须用 `let` 声明在函数内部。误用 `this.count` 会成为实例属性，无法实现真正的封装。

---

## 题目六：闭包会引发内存泄漏吗？ 【中频★★★】

**核心思路**：闭包本身不造成泄漏，问题在于闭包"记住"的变量不会被回收。

**常见泄漏场景**：未清理事件监听、未清除定时器、全局变量膨胀、闭包持有大对象引用。

**主动清理方法**：

```javascript
destroy() {
  element.removeEventListener('click', this.handler);
  clearInterval(this.timer);
  this.handler = null;
  this.timer = null;
}
```

**排查方法**：Chrome DevTools Memory 面板堆快照对比。

---

## 题目七：IIFE 与闭包有什么关系？ 【中频★★】

**核心思路**：IIFE 是创建闭包的语法糖。ES6 前通过函数作用域模拟私有空间，配合闭包实现模块化。

```javascript
const module = (function() {
  let privateVar = 0;
  function privateMethod() { return ++privateVar; }
  return { publicMethod: privateMethod };
})();
module.publicMethod(); // 1
console.log(privateVar); // ReferenceError
```

**易错提醒**：Vue 2 响应式原理和 Vuex 都用 IIFE 实现私有状态。

---

## 题目八：JavaScript 垃圾回收机制 【高频★★★】

**核心思路**：两种主要算法——**标记清除**（现代浏览器主用）和**引用计数**（老式浏览器用，无法处理循环引用）。

V8 引擎采用**分代回收**：堆内存分为新生代（存活时间短的对象）和老生代（存活时间长的对象）。新生代使用 Scavenge 算法快速清理，老生代使用标记清除处理大对象。

**WeakMap 实用场景**：适合存储 DOM 节点引用，不阻止垃圾回收。

```javascript
const cache = new WeakMap();
function process(obj) {
  if (cache.has(obj)) return cache.get(obj);
  const result = heavyComputation(obj);
  cache.set(obj, result);
  return result;
}
```

---

## 本章小结

1. **作用域**决定变量可见范围（全局/函数/块级），**执行上下文**是代码运行环境
2. **var** 函数作用域可提升，**let/const** 块级作用域有暂时性死区，循环中优先用 let
3. **闭包**让函数拥有"记忆"和"封装"能力，防抖节流是高频应用
4. **内存问题**：泄漏（持有不释放）、溢出（超出上限）、抖动（频繁回收）
5. **垃圾回收**：V8 分代回收，WeakMap 的弱引用特性有助于避免泄漏

---

## 面试检查清单

**被问到作用域时**：先说定义，再对比 var/let/const，说清提升和暂时性死区，举 for 循环的例子。

**被问到闭包时**：说原理（记住外部变量），举场景（私有变量、模块化、防抖、记忆化），能手写计数器，知道要清理监听器防泄漏。

---

## 追问示例

**Q：闭包和箭头函数有什么区别？**
闭包是一种能力（记住外部变量），箭头函数是一种语法（不绑定this）。箭头函数可能形成闭包，但闭包不一定是箭头函数。

**Q：for...in 和 for...of 的区别？**
for...in 遍历键名（可枚举属性），适合对象；for...of 遍历值（迭代器），适合数组。

---

> **难度标注说明**：本书统一使用 ★ 符号表示题目难度（★=基础/入门级、★★=中等/常考点、★★★=进阶/综合应用），使用【高频】【中频】【低频】表示面试考察频率。题目一~八为本章核心题，建议优先掌握标注为【高频】的题目。

# 第2章 原型链与继承：从原理到实战

> 本章覆盖 5 道高频面试题。学习目标：掌握 prototype 与 __proto__ 的关系、手绘原型链、属性查找规则、四种核心继承方式对比、手写 instanceof。

---

### 【高频】题 1：prototype 与 __proto__ 的关系及 constructor 指向

**题目**：描述 prototype、__proto__、constructor 三者的关系。

**核心要点**：

- `prototype`：构造函数属性，指向实例的原型模板
- `__proto__`：所有对象都有，指向当前对象的原型
- `constructor`：原型对象上，默认指向构造函数

```javascript
function Person(name) { this.name = name; }
const p = new Person('张三');

console.log(p.__proto__ === Person.prototype);           // true
console.log(Person.prototype.constructor === Person);    // true
console.log(p.constructor === Person);                   // true
console.log(Person.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__ === null);              // true
```

> **易错提醒**：`Object.create()` 创建的对象无 `constructor`；`Object.create(null)` 连 `hasOwnProperty` 都没有。手动重写 `Child.prototype = { ... }` 后需修正 `constructor`：`Child.prototype.constructor = Child`。

---

### 【高频】题 2：能手绘完整原型链

**题目**：能手绘普通对象、构造函数实例、数组的完整原型链。

**记住三个固定端点**：`Object.prototype.__proto__ === null`；所有构造函数的 `prototype` 最终指向 `Object.prototype`；函数本身是 `Function` 的实例。

```text
普通对象：obj → Object.prototype → null
构造函数实例：animal → Animal.prototype → Object.prototype → null
数组：arr → Array.prototype → Object.prototype → null
函数实例：MyFunc → Function.prototype → Object.prototype → null
```

**画图三步法**：确定实例类型 → 找 `__proto__` 指向构造函数 `prototype` → 沿 `prototype` 向上追溯直到 `null`。

> **易错点**：函数实例比普通对象多一层 `Function.prototype`，面试最易漏掉。

---

### 【高频】题 3：原型链的属性查找规则及属性遮蔽现象

**题目**：访问属性时 JavaScript 如何查找？什么是属性遮蔽？

**核心原理**：就近原则——先在自身属性查找，找不到沿原型链向上搜索，直到 `Object.prototype`，再找不到返回 `undefined`。

```javascript
function Person() {}
Person.prototype.hobby = '编程';

const p = new Person();
p.name = '张三';

console.log(p.hobby);   // '编程' — 向上找到原型
p.hobby = '钓鱼';       // 写入时在自身创建属性，遮蔽原型
delete p.hobby;         // delete 只能删除自身属性
console.log(p.hobby);   // '编程' — 恢复访问原型属性
```

**遍历规则对比**：

| 方法 | 自身 | 原型 | 说明 |
|------|------|------|------|
| `for...in` | ✓ | ✓ | 需 `hasOwnProperty` 过滤 |
| `Object.keys()` | ✓ | ✗ | 仅自身可枚举属性 |
| `'key' in obj` | ✓ | ✓ | 包含整条原型链 |

---

### 【高频】题 4：四种核心继承方式（寄生组合式最优）

**题目**：列举 JavaScript 的四种核心继承方式，说明最优方案。

| 继承方式 | 核心代码 | 优点 | 缺点 |
|---------|---------|------|------|
| 原型链继承 | `Child.prototype = new Parent()` | 简单 | 引用类型被所有实例共享 |
| 构造函数继承 | `Parent.call(this)` | 属性独立 | 方法无法复用 |
| 组合继承 | 原型链 + 构造函数 | 方法复用 + 属性独立 | 调用两次父构造函数 |
| **寄生组合式** | `inherit(subType, superType)` | 完美继承，性能最优 | 实现稍复杂 |

**最优方案代码**：

```javascript
function inherit(subType, superType) {
  const prototype = Object.create(superType.prototype);
  prototype.constructor = subType;
  subType.prototype = prototype;
}
```

**ES6 class 继承**（本质仍是寄生组合式）：

```javascript
class Child extends Parent {
  constructor(name, age) {
    super(name);   // 必须在 this 之前调用
    this.age = age;
  }
}
```

> **为什么寄生组合式最优**：原型链继承引用类型被共享，构造函数继承方法不可复用，组合继承调用两次父构造函数造成浪费。寄生组合式通过 `Object.create()` 直接克隆父类原型，只调用一次父构造函数，是性能最优方案。

**原型式继承**：`Object.create()` 克隆对象，引用类型属性被共享。

```javascript
const parent = { hobbies: ['篮球'] };
const child = Object.create(parent);
console.log(child.hobbies);  // ['篮球'] — 共享引用
```

---

### 【高频】题 5：手写 instanceof 的实现

**题目**：手写函数实现 instanceof 功能。

**核心原理**：`instanceof` 沿左侧对象的原型链向上查找，检查是否存在与右侧构造函数 `prototype` 相等的节点。

```javascript
function myInstanceOf(left, right) {
  if (left === null || typeof left !== 'object') return false;
  
  let proto = left.__proto__;
  while (proto !== null) {
    if (proto === right.prototype) return true;
    proto = proto.__proto__;
  }
  return false;
}

// 推荐用 Object.getPrototypeOf（ES5 标准）
function instanceOf(obj, Constructor) {
  if (obj === null || typeof obj !== 'object') return false;
  let proto = Object.getPrototypeOf(obj);
  while (proto !== null) {
    if (proto === Constructor.prototype) return true;
    proto = Object.getPrototypeOf(proto);
  }
  return false;
}
```

> **易错提醒**：null 和原始类型（string、number、boolean）没有 `__proto__`，需先排除。

---

### 本章小结

1. **`prototype` vs `__proto__`**：前者是构造函数的模板属性，后者是对象指向原型的通道
2. **原型链结构**：实例 → 各层 `prototype` → `Object.prototype` → `null`
3. **属性查找**：遵循就近原则，自身优先；赋值遮蔽原型，`delete` 可恢复
4. **继承方案**：寄生组合式最优，ES6 `class` 是工程首选语法
5. **instanceof 原理**：沿原型链向上比对 `prototype`

---

### 面试检查清单

| 序号 | 检查项 | 状态 |
|-----|-------|------|
| 1 | 能说清 `prototype` 与 `__proto__` 的区别 | ☐ |
| 2 | 能手绘三种常见对象的原型链 | ☐ |
| 3 | 能解释属性遮蔽与 `delete` 恢复原理 | ☐ |
| 4 | 能对比四种核心继承方式并说明最优方案 | ☐ |
| 5 | 能手写 `instanceof` 并处理边界情况 | ☐ |

> **行动建议**：在纸上默写三个原型链——普通对象、构造函数实例、数组实例，写完用 `console.log` 验证。**常见错误**：漏掉 `Function.prototype` 层；忘记 `Object.prototype.__proto__` 指向 `null`。

# 第3章 数据类型判断与隐式转换

> **本章覆盖【高频】面试题约9道**，掌握 JavaScript 类型系统的核心知识点，避免类型相关面试题失分。

数据类型相关题目面试频率极高，很多人栽在 `typeof null === 'object'` 经典坑上，也搞不清 `==` 和 `===` 的区别。这一章把类型判断和隐式转换讲透，帮你把这类送分题稳稳拿下。

---

## typeof：最常用的类型判断方法

**【高频】题目：typeof 能判断哪些类型？**

```javascript
typeof 123;          // "number"
typeof 'hello';      // "string"
typeof true;         // "boolean"
typeof undefined;    // "undefined"
typeof null;         // "object"  ← 经典坑！
typeof {};           // "object"
typeof [];           // "object"  ← 数组也被当成对象
typeof function(){}  // "function"
```

**核心思路：** `typeof` 是 JS 最常用的类型判断运算符，但它并不完美——`null`、普通对象和数组都返回 `"object"`，无法区分。

**ES6 新增类型：**

```javascript
typeof Symbol('test'); // "symbol"
typeof BigInt(123);    // "bigint"
```

**易错提醒：** `typeof null === 'object'` 是 JavaScript 诞生时的历史 bug，至今未修复。面试时能说出这个背景，说明你对语言理解有深度。

**扩展追问：** 为什么函数返回 `"function"` 而不是 `"object"`？这是 JS 早期设计时的有意为之，方便开发者快速判断可调用对象。

---

## instanceof：判断对象的具体类型

**【高频】题目：instanceof 的原理是什么？**

**核心思路：** `instanceof` 检查构造函数的 `prototype` 属性是否在对象的原型链上。

```javascript
[] instanceof Array;          // true
[] instanceof Object;         // true（数组也是对象）
({}) instanceof Object;       // true
'str' instanceof String;      // false（原始类型不是对象）
new String('str') instanceof String; // true
```

**代码示例：** 手写实现 myInstanceof：

```javascript
function myInstanceof(left, right) {
  let rightProto = right.prototype;
  let leftProto = left.__proto__;
  while (leftProto !== null) {
    if (leftProto === rightProto) return true;
    leftProto = leftProto.__proto__;
  }
  return false;
}
```

**易错提醒：** `instanceof` 不适用于原始类型判断（字符串、数字等），只对对象有效。另外，跨 iframe 创建的对象使用 `instanceof` 会失效，因为不同 iframe 有独立的原型链。

**扩展追问：** 如何用 `myInstanceof` 判断 `null`？答案是会进入死循环，所以在实际代码中要加 `left !== null` 的判断。

---

## Object.prototype.toString：最准确的类型判断

**【高频】题目：如何精确判断一个值的类型？**

**核心思路：** 无论什么类型，`Object.prototype.toString` 都能返回最准确的类型字符串，是类型判断的终极方案。

```javascript
Object.prototype.toString.call(123);        // "[object Number]"
Object.prototype.toString.call(null);        // "[object Null]"
Object.prototype.toString.call([]);         // "[object Array]"
Object.prototype.toString.call(new Date()); // "[object Date]"
Object.prototype.toString.call(function(){}); // "[object Function]"
```

**易错提醒：** 直接调用 `[].toString()` 返回 `"1,2,3"`，不是类型信息。必须用 `Object.prototype.toString.call()` 绑定上下文才能获取准确类型。

**扩展追问：** 手写一个通用类型判断函数：

```javascript
function getType(value) {
  if (value === null) return 'null';
  if (typeof value !== 'object') return typeof value;
  return Object.prototype.toString.call(value).slice(8, -1).toLowerCase();
}
// getType([]) → "array"
```

---

## == 与 ===：隐式转换的陷阱

**【高频】题目：[] == ![] 结果是什么？为什么？**

**核心思路：** `==` 会触发隐式转换，规则复杂；`===` 不转换类型，直接比较。

```javascript
[] == ![];           // true（两者都转成 0）
''; == 0;            // true
''; == false;        // true
null == undefined;   // true（特例）
```

**== 隐式转换规则（优先级从高到低）：**

| 顺序 | 场景 | 转换方式 |
|------|------|----------|
| 1 | null == undefined | 直接返回 true |
| 2 | 类型相同 | 直接比较值 |
| 3 | 字符串 vs 数字 | 字符串转数字 |
| 4 | 布尔 vs 其他 | 布尔先转数字 |
| 5 | 对象 vs 原始类型 | 对象转原始值 |

**应答模板：** 遇到 `==` 比较题，分三步思考：①先看 null/undefined 特例；②再看类型是否相同；③类型不同按规则转换。

**易错提醒：** `![]` 先把数组转布尔得 `true`，再取反得 `false`，所以 `[] == ![]` 等价于 `[] == false`，再等价于 `[] == 0`，最终 `"" == 0` 转成 `0 == 0`，返回 `true`。**实际开发中始终使用 ===**，避免隐式转换。

**扩展追问：** `[] == false` 和 `[] === false` 结果分别是？答案是 `true` 和 `false`。

---

## NaN、undefined、null 的特殊行为

**【高频】题目：如何正确判断一个值是 NaN？**

**核心思路：** `NaN` 是唯一一个不等于自身的值，普通比较无法判断。

```javascript
NaN === NaN;             // false
Number.isNaN(NaN);       // true
Number.isNaN('hello');   // false
isNaN('hello');          // true（会强制转换，不推荐）
```

**易错提醒：** 别用 `isNaN()`，它会强制把参数转成数字导致误判。用 `Number.isNaN()` 更安全。

**undefined vs null 对比：**

| 特征 | undefined | null |
|------|-----------|------|
| typeof | "undefined" | "object" |
| == 比较 | `undefined == null` 为 true | 同左 |
| === 比较 | `undefined === null` 为 false | 同左 |
| 语义 | 未定义/未赋值 | 已赋值为空 |
| 常见场景 | 变量未初始化、函数无返回值 | 主动清空、对象不存在 |

**扩展追问：** `null == 0` 结果是 `false`，但 `Number(null)` 返回 `0`。为什么？这体现了 `==` 比较和类型转换是两套独立的规则。

---

## 本章小结

**核心知识点速查：**

- `typeof` 区分不了 null、对象和数组，但判断原始类型和函数很方便
- `instanceof` 用来判断实例与类的关系，不适用原始类型和跨 iframe 场景
- `Object.prototype.toString` 是最准确的类型判断方法，能区分所有类型
- `==` 触发隐式转换，规则复杂容易踩坑；`===` 不转换类型，优先使用
- `NaN` 是唯一不等于自身的值，必须用 `Number.isNaN()` 判断

**面试检查清单：**

- 能手写 `getType()` 通用类型判断函数
- 能手写 `myInstanceof()` 实现原型链判断
- 理解 `typeof null === 'object'` 的历史原因
- 能用三步模板拆解 `==` 比较题
- 掌握 `Number.isNaN()` 与 `isNaN()` 的区别

**实用建议：**

1. **日常开发**：统一使用 `===`，遇到 `==` 题用三步模板拆解
2. **类型判断优先级**：原始类型用 `typeof`，对象用 `instanceof`，精确区分数组/日期等用 `Object.prototype.toString`
3. **面试加分**：不仅说出结论（如 typeof null 是 'object'），还要解释原因（JS 早期版本遗留的 bug）

# 第4章 事件循环与异步编程

## 本章覆盖 6 道高频面试题

**核心问题**：JavaScript 代码的执行顺序是什么？setTimeout 明明写在前面的代码为什么后面执行？

---

## 一、为什么 JavaScript 需要事件循环

JavaScript 是单线程语言，同一时间只能执行一段代码。遇到耗时操作（如网络请求）时，主线程会被阻塞，页面卡死。

**事件循环**解决此问题：把耗时操作「挂起」，完成后通知主线程继续处理。简单理解，就是让 JavaScript 能够处理异步任务，同时不阻塞主线程。

---

## 二、事件循环的核心组成

```
┌─────────────────────────────────────────┐
│                  调用栈                  │
│         （正在执行的同步代码）            │
└─────────────────┬───────────────────────┘
                  │
        ┌─────────▼─────────┐
        │      任务队列      │
        │   （宏任务队列）    │
        └─────────┬─────────┘
                  │
        ┌─────────▼─────────┐
        │     微任务队列      │
        │  Promise / MutationObserver │
        └───────────────────┘
```

**执行顺序**：调用栈 → 微任务队列 → 宏任务队列 → 重复循环

---

## 三、宏任务与微任务

**宏任务**：需要推迟到下一个事件循环周期执行的任务，包括 setTimeout、setInterval、网络 I/O、UI 渲染等。**微任务**：优先级更高的异步任务，在当前调用栈清空后、但在下一个宏任务之前立即执行，包括 Promise.then、queueMicrotask 等。

JavaScript 区分两者是为了平衡「响应速度」和「公平分配」。宏任务确保浏览器有机会定期执行渲染更新；微任务保证 Promise 回调能尽快执行，不被其他宏任务插队。

**常见任务类型**：

- 微任务：Promise.then/catch/finally、queueMicrotask
- 宏任务：setTimeout/setInterval、网络 I/O、UI 渲染
- Node.js 特殊：process.nextTick 优先级比微任务还高

### 常见误区提醒

**误区 1**：以为 Promise 回调都是微任务。实际上，Promise 构造函数中的代码是同步执行的，只有 then/catch/finally 的回调才会进入微任务队列。

**误区 2**：混淆执行时机。记住：**微任务清空之后，才会执行下一个宏任务**。

---

## 四、经典面试题讲解

### 题目 1：setTimeout 对比 setImmediate

```javascript
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
```

**答案**：Node.js 环境中 setImmediate 可能先执行；浏览器环境中 setTimeout 几乎总是先执行。

---

### 题目 2：Promise 执行顺序

```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

console.log('4');
```

**执行过程**：

1. 同步输出 **1** → **4**
2. 微任务队列 → 输出 **3**
3. 宏任务队列 → 输出 **2**

**最终输出**：1 → 4 → 3 → 2

---

### 题目 3：async/await 与 Promise 的关系

```javascript
async function test() {
  console.log('A');
  await Promise.resolve();
  console.log('B');
}

test();
console.log('C');
```

**答案**：A → C → B

**原因**：await 会阻塞后续代码，但 async 函数本身是同步的。A 和 C 同步执行，B 等待微任务完成。

**高频追问**：

- async 函数永远返回 Promise，即使直接 `return 'hello'`，实际返回的是 `Promise.resolve('hello')`
- await 右边的表达式会立即求值，后面的代码暂停等待
- 连续多个 await 本质是链式调用，前一个完成才会执行下一个

---

## 五、Promise 的三种状态

```
pending（进行中）
    ↓ resolve()
fulfilled（已成功）
    ↓ reject()
rejected（已失败）
```

**特点**：状态一旦改变，不可逆转。

```javascript
const p = new Promise((resolve, reject) => {
  resolve('成功');
  reject('失败'); // 不会生效
});

p.then(res => console.log(res)); // 输出：成功
```

---

## 六、async/await 错误处理

```javascript
async function fetchData() {
  try {
    const res = await fetch('/api/data');
    const data = await res.json();
    return data;
  } catch (err) {
    console.log('捕获错误：', err);
  }
}
```

**技巧**：await 必须放在 try/catch 中才能捕获错误。

---

## 七、综合面试题

### 题目 4：说出输出顺序

```javascript
async function async1() {
  console.log('1');
  await async2();
  console.log('2');
}

async function async2() {
  console.log('3');
}

console.log('4');

setTimeout(() => console.log('5'), 0);

async1();

new Promise(resolve => {
  console.log('6');
  resolve();
}).then(() => console.log('7'));

console.log('8');
```

**答案**：4 → 1 → 3 → 6 → 8 → 2 → 7 → 5

---

## 八、Promise 静态方法

**Promise.all**：所有 Promise 都成功时返回结果数组，任一失败则立即 reject。

```javascript
const p = Promise.all([fetch('/api/users'), fetch('/api/posts')]);
p.then(([users, posts]) => console.log('都成功', users, posts));
p.catch(err => console.log('有失败', err));
```

**Promise.race**：以最快完成的那个 Promise 为准，无论成功还是失败。

```javascript
Promise.race([
  fetch('/api/data'),
  new Promise((_, reject) => setTimeout(() => reject('超时'), 3000))
]).then(res => console.log(res)).catch(err => console.log(err));
```

**Promise.allSettled**：等待所有 Promise 完成，返回每个 Promise 的结果描述，不在乎成功还是失败。

```javascript
Promise.allSettled([fetch('/api/a'), fetch('/api/b')]).then(results => {
  results.forEach((r, i) => {
    console.log(`第${i + 1}个: ${r.status} - ${r.value ?? r.reason}`);
  });
});
```

---

## 九、浏览器与 Node.js 事件循环对比

| 特性 | 浏览器 | Node.js |
|------|--------|---------|
| UI 渲染 | 微任务之后、宏任务之前触发 | 无此概念 |
| setTimeout 最小延迟 | 约 4ms | 约 1ms |
| setImmediate | 不支持 | I/O 回调中优先于 setTimeout |
| process.nextTick | 不存在 | 优先级高于微任务 |

**实战建议**：Node.js 岗位面试时，务必说明两者差异，process.nextTick 是高频追问点。

---

## 本章小结

| 知识点 | 核心要点 |
|--------|----------|
| 事件循环 | 调用栈 → 微任务 → 宏任务 循环执行 |
| 微任务优先 | Promise.then 永远在 setTimeout 之前 |
| async/await | 本质是 Promise 语法糖，await 触发微任务 |
| 状态不可逆 | Promise 三种状态一旦改变不能再变 |
| 静态方法 | all/race/allSettled 应对不同场景 |

**记忆口诀**：同步优先，微任务次之，宏任务殿后。

---

## 面试检查清单

- [ ] 能画出事件循环的执行流程图
- [ ] 能区分宏任务和微任务
- [ ] 能分析 Promise + setTimeout 混合代码的输出顺序
- [ ] 能解释 async/await 与 Promise 的关系
- [ ] 能手写带错误处理的 async 函数
- [ ] 能说出 Promise.all/race/allSettled 的区别

---

## 行动建议

1. **手动画图**：每次遇到异步题，先画调用栈/队列草图，标注执行顺序
2. **背熟经典题**：Promise + setTimeout 组合题出现频率最高，必须练到条件反射
3. **理解本质**：不要死记答案，理解「微任务清空后再执行宏任务」这一核心机制

# 第5章 this、call、apply、bind 全解

本章覆盖【高频】面试题约 8 道，建议配合代码反复实践。this 指向是 JavaScript 面试中出现频率最高的核心考点，很多候选人丢分并非题目多难，而是混淆多种绑定规则、优先级判断失误。

## 一、this 的五种绑定场景

优先级从高到低：**new 绑定 > 显式绑定 > 隐式绑定 > 默认绑定**。

### 1. 默认绑定

独立函数调用时，this 指向全局对象。浏览器环境下是 window，严格模式下是 undefined。

```javascript
'use strict';
function show() { console.log(this); }
show(); // undefined
```

**一句话记住**：单独调用函数 → 非严格模式指向 window，严格模式指向 undefined。

### 2. 隐式绑定

函数作为对象方法调用时，this 指向调用该方法的对象。

```javascript
const person = { name: '张三', say() { console.log(this.name); } };
person.say(); // '张三'
```

**【高频】易错题**：回调函数中 this 丢失。

```javascript
const obj = {
    name: '李四',
    delay() {
        setTimeout(function() {
            console.log(this.name); // undefined，this 指向 window
        }, 100);
    }
};
obj.delay();
```

setTimeout 传入普通函数是独立调用，this 丢失。**三种修复方案**：

```javascript
// 方案一：保存 this 引用
const that = this;
setTimeout(function() { console.log(that.name); }, 100);

// 方案二：箭头函数（推荐）
setTimeout(() => { console.log(this.name); }, 100);

// 方案三：显式绑定
setTimeout(function() { console.log(this.name); }.bind(this), 100);
```

### 3. 显式绑定

使用 call、apply、bind 强制指定 this 指向。

### 4. new 绑定

使用 new 调用构造函数时，this 指向新创建的对象实例。

```javascript
function Person(name) { this.name = name; }
const p = new Person('王五');
console.log(p.name); // '王五'
```

**面试必问**：new 操作符做了四件事——创建空对象、设置原型链指向构造函数的 prototype、执行构造函数并绑定 this、返回新对象。

```javascript
// new Person('王五') 实际执行：
const obj = {};                    // 1. 创建空对象
obj.__proto__ = Person.prototype;  // 2. 设置原型链
Person.call(obj, '王五');          // 3. 执行构造函数，绑定 this
return obj;                        // 4. 返回新对象
```

> **面试追问**：为什么构造函数通常不写 return？因为 new 会自动返回新对象。如果手动 return 一个引用类型，new 的结果就变成你返回的对象。

**【高频】面试题**：箭头函数能否作为构造函数？不能。箭头函数没有 prototype，用 new 调用会报错。

### 5. 箭头函数绑定

箭头函数没有自己的 this，继承外层作用域的 this，且不可被 call/apply/bind 改变。

```javascript
const arrow = () => console.log(this);
arrow.call('test'); // window，call 无法改变箭头函数的 this
```

箭头函数的 this 在定义时已确定，之后任何显式绑定都无法改变。

## 二、call、apply、bind 的区别与实现

这三种方法都是 Function.prototype 上的原型方法，用于显式绑定 this。

| 方法 | 传参方式 | 执行时机 | 返回值 |
|------|----------|----------|--------|
| call | 逐个传参 | 立即执行 | 函数执行结果 |
| apply | 数组传参 | 立即执行 | 函数执行结果 |
| bind | 逐个传参 | 不执行，返回新函数 | 新函数 |

```javascript
function greet(age, city) {
    console.log(`我是${this.name}，${age}岁，来自${city}`);
}
const person = { name: '张三' };
greet.call(person, 25, '北京');
greet.apply(person, [25, '北京']);
greet.bind(person, 25, '北京')();
```

**重要细节**：call 和 apply 第一个参数传入 null 或 undefined，非严格模式下 this 指向 window。

**手写 call 实现**：

```javascript
Function.prototype.myCall = function(context, ...args) {
    const ctx = context || window;
    const fn = Symbol('fn');
    ctx[fn] = this;
    const result = ctx[fn](...args);
    delete ctx[fn];
    return result;
};
```

**【高频】手写 bind（完整版）**：

```javascript
Function.prototype.myBind = function(context, ...args) {
    const fn = this;
    return function F(...innerArgs) {
        if (this instanceof F) {
            return new fn(...args, ...innerArgs);
        }
        return fn.apply(context, [...args, ...innerArgs]);
    };
};
```

完整版需考虑 bind 返回的新函数被 new 调用时的场景，此时 this 应指向实例而非 bind 指定的 context。

## 三、绑定优先级判断

**优先级速记口诀**：new 最强，显式次之，隐式再次，默认最弱。

**快速判断流程**：

```
是否用 new 调用？  → 是 → this = 新实例（优先级最高）
         ↓ 否
是否用 call/apply/bind？ → 是 → this = 指定的对象
         ↓ 否
是否 obj.foo() 形式调用？ → 是 → this = obj
         ↓ 否
默认绑定 → this = window（非严格）/ undefined（严格模式）
```

## 四、面试扩展问题

1. **箭头函数和 bind 都能改变 this，两者同时存在谁说了算？**
   箭头函数。箭头函数的 this 在定义时已绑定完毕，后续显式绑定不起作用。

2. **class 语法中 this 的指向和 ES5 构造函数有何不同？**
   本质相同，class 是 ES6 语法糖。但 class 中的方法默认不绑定 this，需用箭头函数或显式绑定来固定。

3. **bind 返回的函数能二次 bind 吗？**
   不能。已绑定的 this 不会因再次 bind 而改变。

4. **new 操作符和 bind 同时使用会发生什么？**
   当用 new 调用 bind 返回的函数时，bind 指定的 this 会被忽略，this 指向新创建的实例。

## 五、实际应用场景

**类数组转数组**：

```javascript
const likeArr = { 0: 'a', 1: 'b', length: 2 };
const arr = Array.prototype.slice.call(likeArr); // ['a', 'b']
```

**继承实现**：利用 call 在子类构造函数中调用父类构造函数继承属性。

```javascript
function Parent(name) { this.name = name; }
function Child(name, age) {
    Parent.call(this, name);
    this.age = age;
}
```

## 面试检查清单

- [ ] 能说出 this 的五种绑定方式及优先级排序
- [ ] 能解释为什么 setTimeout 回调中 this 丢失，并写出三种解决方案
- [ ] 能默写 new 操作符的四步执行过程
- [ ] 能手写 call 和 bind 的完整实现（含 new 调用场景）
- [ ] 能说清箭头函数 this 与普通函数的本质区别
- [ ] 能回答「箭头函数能否 new」「构造函数为何不写 return」等追问

## 本章小结

1. **this 五种绑定**：默认（严格模式 undefined）、隐式（指向调用对象）、显式（call/apply/bind）、new（指向实例）、箭头函数（继承外层）
2. **优先级**：new > 显式 > 隐式 > 默认
3. **call/apply/bind**：call/apply 立即执行，参数形式不同；bind 返回新函数
4. **new 操作符四步**：创建空对象 → 设置原型 → 执行构造函数 → 返回对象
5. **回调 this 丢失**：箭头函数、that 保存、bind 三种方案，箭头函数最推荐
6. **箭头函数特点**：无 this、无 arguments、无 prototype、不能作构造函数、不可 bind

> **行动建议**：面试前默写 new 操作符四步实现和 bind 完整版手写代码，这是高频考点。

# 第6章 防抖、节流与手写实现

本章围绕防抖（Debounce）和节流（Throttle）这两个前端性能优化必备技能展开，涵盖 5 道高频面试题。

## 一、为什么需要防抖和节流？

高频事件是前端常见问题：搜索框输入每按一次键触发一次 input 事件，窗口 resize 每秒触发几十次，滚动事件更是频繁。这些场景的共同问题是：**事件触发频率远远超过实际需要的处理频率**，导致性能浪费和服务器压力增加。

防抖和节流就是解决这一问题的两把利刃，通过控制函数执行频率来优化性能。

## 二、防抖（Debounce）：等事件停下来再执行

### 2.1 原理

防抖的核心思想：**事件触发后延迟一段时间再执行，如果在延迟期间事件再次被触发，就重新计时**。

### 2.2 手写防抖函数

```javascript
function debounce(fn, delay = 300, immediate = false) {
  let timer = null;

  return function (...args) {
    // 立即执行模式：首次触发直接执行
    if (immediate && !timer) {
      fn.apply(this, args);
    }

    // 清除之前的定时器，重新计时
    if (timer) clearTimeout(timer);

    timer = setTimeout(() => {
      if (!immediate) {
        fn.apply(this, args);
      }
      timer = null;
    }, delay);
  };
}
```

三个关键点：

- **闭包结构**：timer 变量被返回函数捕获，保留函数调用之间的状态
- **clearTimeout(timer)**：这是防抖的核心——只有最后一次触发才能真正执行
- **immediate 参数**：控制函数是「延迟后执行」还是「立即执行后锁定」

### 2.3 面试追问：如何取消已设置的定时器？

```javascript
function debounce(fn, delay = 300, immediate = false) {
  let timer = null;

  const debounced = function (...args) {
    if (timer) clearTimeout(timer);

    if (immediate && !timer) {
      fn.apply(this, args);
    }

    timer = setTimeout(() => {
      if (!immediate) fn.apply(this, args);
      timer = null;
    }, delay);
  };

  debounced.cancel = function () {
    if (timer) {
      clearTimeout(timer);
      timer = null;
    }
  };

  return debounced;
}

// 使用：用户切换路由时取消
onUnmounted(() => debouncedSearch.cancel());
```

### 2.4 防抖的使用场景

- **搜索框输入**：用户停止输入 300ms 后才发送搜索请求
- **窗口 resize**：用户停止调整窗口大小后再重新计算布局
- **表单验证**：用户停止输入后才进行格式校验
- **按钮防重复点击**：设置 1 秒防抖，防止用户快速点击

## 三、节流（Throttle）：固定间隔内只执行一次

### 3.1 原理

节流的核心思想：**在一定时间间隔内，函数只执行一次**，无论事件触发了多少次。

### 3.2 手写节流函数（时间戳版本）

```javascript
function throttle(fn, interval = 300) {
  let lastTime = 0;

  return function (...args) {
    const now = Date.now();
    if (now - lastTime >= interval) {
      lastTime = now;
      fn.apply(this, args);
    }
  };
}
```

时间戳版本的特点：**首次触发立即执行**，但**最后一次触发可能不会执行**（如果事件在间隔中段停止）。

### 3.3 手写节流函数（定时器版本）

```javascript
function throttle(fn, interval = 300) {
  let timer = null;

  return function (...args) {
    if (!timer) {
      timer = setTimeout(() => {
        fn.apply(this, args);
        timer = null;
      }, interval);
    }
  };
}
```

定时器版本的特点：**开始时不立即执行**，但**保证最后一次触发后一定会执行**。

### 3.4 时间戳 vs 定时器：面试必问的对比

| 维度 | 时间戳版本 | 定时器版本 |
|------|-----------|-----------|
| 首次触发 | 立即执行 | 延迟后执行 |
| 最后一次触发 | 可能不执行 | 一定执行 |

实际开发中可以根据业务需求选择或组合使用。

### 3.5 节流的使用场景

- **滚动加载**：每 200ms 检查一次是否到达页面底部
- **鼠标拖拽**：每 16ms（60fps）更新一次元素位置
- **页面滚动埋点**：每 1 秒记录一次滚动位置

## 四、防抖 vs 节流：如何选择？

| 场景 | 选择 | 原因 |
|------|------|------|
| 搜索框输入关键词 | 防抖 | 等用户打完字再搜索 |
| 窗口 resize 调整布局 | 防抖 | 等用户停止调整，只关心最终状态 |
| 滚动加载更多数据 | 节流 | 持续滚动过程中按固定频率加载 |
| 鼠标拖拽元素 | 节流 | 实时响应但控制频率节省性能 |

选择标准：**你需要「最后一次操作的结果」还是「操作期间的持续反馈」**？需要等用户停下来用最终结果 → 防抖；需要跟随用户操作持续响应 → 节流。

## 五、防抖节流在 Vue/React 中的封装使用

### 5.1 Vue 3 组合式 API 封装

```javascript
// useDebounce.js
import { ref, onUnmounted } from 'vue';

export function useDebounce(fn, delay = 300) {
  let timer = null;
  const debounced = (...args) => {
    if (timer) clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
  debounced.cancel = () => timer && clearTimeout(timer);
  onUnmounted(() => debounced.cancel());
  return debounced;
}
```

### 5.2 React Hooks 封装

```javascript
// useDebounce.js
import { useCallback, useRef, useEffect } from 'react';

export function useDebounce(fn, delay = 300) {
  const timer = useRef(null);

  useEffect(() => {
    return () => {
      if (timer.current) clearTimeout(timer.current);
    };
  }, []);

  return useCallback((...args) => {
    if (timer.current) clearTimeout(timer.current);
    timer.current = setTimeout(() => fn(...args), delay);
  }, [fn, delay]);
}
```

封装成 Hook 的好处是逻辑复用、生命周期自动管理、代码整洁。

## 六、高频面试题

### 题目 1【高频】：防抖和节流的区别？各适用什么场景？

**答案**：防抖是延迟执行——事件触发后等待 delay 毫秒，期间如果再次触发则重新计时，只有停下来后才真正执行。节流是固定频率——在 interval 毫秒内无论触发多少次，函数只执行一次。防抖适合搜索框输入、窗口 resize 等"等待最终结果"的场景；节流适合滚动加载、拖拽等"持续响应"的场景。

**扩展追问**：如何实现同时支持 leading（立即执行）和 trailing（延迟执行）？通过参数控制即可，完整实现需要维护定时器和 lastTime 两个状态。

---

### 题目 2【高频】：手写防抖函数，支持 immediate 参数

**易错提醒**：immediate=true 时必须检查 timer 是否为 null，检查逻辑 `if (immediate && !timer)` 必须准确。

**扩展追问**：immediate=true 时函数执行后，下次触发要等 delay 后才能再次执行，如何实现"立即执行后 delay 内可再次立即执行"？这需要额外的标记位。

---

### 题目 3【高频】：手写节流函数（时间戳版本，保证最后一次触发）

**时间戳版本不保证最后一次**：如果用户停止触发时正好在 interval 中间，最后一次调用不会执行。面试官可能会追问如何改进——可以结合定时器版本，在 interval 结束后再执行一次。

---

### 题目 4【高频】：防抖函数如何取消已设置的定时器？

**实战场景**：用户从搜索页面跳转到详情页，如果还保留着上一个页面的防抖定时器，不仅浪费性能，还可能导致页面数据错乱。这时需要主动取消。

---

### 题目 5【高频】：节流的时间戳版本和定时器版本有什么区别？

面试官想听到的是你理解两种实现的 trade-off，能根据场景选择合适的方案。

## 本章小结

防抖和节流的核心区别在于「等停下来再做」还是「按固定节奏做」。手写实现的关键是理解闭包保存状态、clearTimeout 重置计时器、setTimeout 延迟执行这三个核心要素。面试中能写出版本支持 immediate/cancel 参数、能说清时间戳和定时器版本的区别、能封装 Vue/React Hook，说明你对原理理解透彻。

**面试检查清单**：

- ☐ 能说清防抖和节流的原理区别
- ☐ 能手写防抖函数（支持 immediate）
- ☐ 能手写节流函数（时间戳版本）
- ☐ 能说出时间戳和定时器版本的区别
- ☐ 能为防抖函数添加 cancel 方法
- ☐ 能封装 Vue/React 中的防抖 Hook

# 第7章 深拷贝与 JSON 序列化

# 深拷贝与 JSON 序列化：引用类型拷贝的核心问题

> **本章覆盖 6 道高频面试题**，建议学习时间 30 分钟。

JavaScript 中，数据分为**基本类型**（string、number、boolean、null、undefined、symbol）和**引用类型**（object、array、function、Date、RegExp 等）。基本类型存放在栈中，赋值时传递值的副本；引用类型存放在堆中，变量名只是一个"指针"，赋值时传递的是地址引用。

```javascript
// 基本类型——赋值后互不影响
let a = 1;
let b = a;
b = 2;
console.log(a); // 1

// 引用类型——赋值后共享同一份数据
let obj1 = { name: '张三' };
let obj2 = obj1; // obj2 只是拿到了地址
obj2.name = '李四';
console.log(obj1.name); // "李四"
```

---

## 【基础★】题目一：浅拷贝与深拷贝的区别

### 核心思路

浅拷贝只复制第一层属性，嵌套的引用类型复制的是内存地址。深拷贝递归遍历所有层级，为每一层创建全新对象。

### 代码示例

```javascript
const obj1 = { name: '张三', info: { age: 25 } };

// 常见浅拷贝实现
const copy1 = Object.assign({}, obj1);
copy1.name = '王五'; // 不影响 obj1

// 嵌套对象仍共享引用
copy1.info.age = 40;
console.log(obj1.info.age); // 40 —— 原对象被改了
```

### 易错提醒

`Object.assign()` 会**覆盖**同名属性。用它合并 `{a: {x: 1}}` 和 `{a: {y: 2}}`，结果只有 `{a: {y: 2}}`，x 会丢失。

---

## 【基础★★】题目二：JSON.stringify 的三个致命坑

### 坑一：无法拷贝函数和 undefined

```javascript
const obj = { name: '测试', fn: function() {}, val: undefined };
const copy = JSON.parse(JSON.stringify(obj));
console.log(copy); // { name: '测试' } —— fn 和 val 消失了
```

### 坑二：无法处理循环引用

```javascript
const circular = { name: '循环' };
circular.self = circular;
JSON.stringify(circular); // TypeError: Converting circular structure to JSON
```

### 坑三：无法拷贝特殊对象

`Date` 变成字符串，`RegExp`、`Map`、`Set` 变成空对象，数据全丢。

```javascript
const obj = { date: new Date(), reg: /test/i };
const copy = JSON.parse(JSON.stringify(obj));
console.log(copy.date); // "2024-01-01T00:00:00.000Z" —— 字符串，不是 Date
console.log(copy.reg); // {} —— 正则信息全丢了
```

### 适用场景

适合"纯数据对象"的拷贝，不包含函数和特殊类型时优先考虑。

---

## 【进阶★★★】题目三：手写深拷贝函数

### 核心思路

三个要点：1. `typeof` 判断基本类型直接返回；2. `instanceof` 判断特殊对象（Date、RegExp、Map、Set）单独处理；3. `WeakMap` 记录已拷贝对象，遇到循环引用直接返回已存在的副本。

```javascript
function deepClone(target, map = new WeakMap()) {
  // 处理基本类型和 null
  if (typeof target !== 'object' || target === null) {
    return target;
  }

  // 处理循环引用
  if (map.has(target)) {
    return map.get(target);
  }

  // 处理特殊对象
  if (target instanceof Date) return new Date(target);
  if (target instanceof RegExp) return new RegExp(target.source, target.flags);

  if (target instanceof Map) {
    const cloneMap = new Map();
    map.set(target, cloneMap);
    target.forEach((v, k) => cloneMap.set(deepClone(k, map), deepClone(v, map)));
    return cloneMap;
  }

  if (target instanceof Set) {
    const cloneSet = new Set();
    map.set(target, cloneSet);
    target.forEach(v => cloneSet.add(deepClone(v, map)));
    return cloneSet;
  }

  // 处理普通对象和数组
  const cloneTarget = Array.isArray(target) ? [] : {};
  map.set(target, cloneTarget); // 先记录，再递归

  for (const key in target) {
    if (Object.prototype.hasOwnProperty.call(target, key)) {
      cloneTarget[key] = deepClone(target[key], map);
    }
  }
  return cloneTarget;
}
```

### 循环引用场景

循环引用指对象引用自身或两个对象互相引用（如树结构父子节点互相引用）。没有 WeakMap 处理会导致无限递归栈溢出。

### 为什么用 WeakMap？

面试高频追问点：WeakMap 的 key 是**弱引用**，不阻止垃圾回收。用普通 Map 会导致已拷贝对象无法被回收，造成内存泄漏。

---

## 【进阶★★】题目四：lodash cloneDeep 源码思路

```javascript
function cloneDeep(obj) {
  const type = Object.prototype.toString.call(obj);

  // 特殊类型单独处理
  if (type === '[object Date]') return new Date(obj);
  if (type === '[object RegExp]') return new RegExp(obj.source, obj.flags);
  if (type === '[object Map]') return new Map([...obj].map(([k, v]) => [cloneDeep(k), cloneDeep(v)]));
  if (type === '[object Set]') return new Set([...obj].map(v => cloneDeep(v)));

  // 数组和普通对象递归处理
  return Array.isArray(obj) ? obj.map(cloneDeep) : Object.fromEntries(
    Object.entries(obj).map(([k, v]) => [k, cloneDeep(v)])
  );
}
```

### 方案取舍

| 场景 | 推荐方案 |
|------|----------|
| 简单对象（不含函数） | JSON.parse/stringify |
| 生产环境复杂场景 | lodash.cloneDeep |
| 面试手写环节 | 手写递归函数 |

---

## 【基础★★】题目五：structuredClone 内置方法

> ⚠️ **时效性说明**：Chrome 97+、Edge 97+、Firefox 94+、Safari 15.4+、Node.js 17+ 已支持。

### 核心思路

现代浏览器原生深拷贝 API，能处理循环引用，性能优于 JSON 方法（大型数据差距明显），但不支持函数、Symbol、RegExp、Error、DOM 节点。

```javascript
const original = { name: '测试', data: [1, 2, 3] };
const cloned = structuredClone(original);
cloned.data.push(4);
console.log(original.data); // [1, 2, 3]

// 处理循环引用
const circular = { name: '循环' };
circular.self = circular;
const clonedCircular = structuredClone(circular); // 不会报错
```

### 局限性

```javascript
const obj = { fn: () => {}, reg: /test/ };
structuredClone(obj); // DOMException: () => {} could not be cloned
```

### 性能对比

| 方案 | 循环引用 | 函数 | 性能 |
|------|----------|------|------|
| JSON 方法 | ❌ | ❌ | 基准 |
| structuredClone | ✅ | ❌ | 最快 |
| 手写递归 | ✅ | ✅ | 中等 |

---

## 【进阶★★★】题目六：项目中实际用过哪些拷贝方案？

### 回答示例

> "做过配置合并功能，用户个性化配置需与默认配置合并。最初用 `Object.assign` 导致嵌套属性污染默认配置，改用 lodash `cloneDeep` 解决，但也遇到过 Date 被转成字符串的坑。
>
> 现在我的选择：接口数据转存用 JSON 方法；可能有循环引用的复杂对象树用 `structuredClone`；面试手写递归是加分项。"

### 方案对比总结

| 方案 | 循环引用 | 函数 | 特殊类型 | 性能 | 适用场景 |
|------|----------|------|----------|------|----------|
| JSON.parse/stringify | ❌ | ❌ | ❌ | 快 | 纯数据、无循环引用 |
| lodash.cloneDeep | ✅ | ✅ | ✅ | 中等 | 生产环境复杂场景 |
| 手写递归 | ✅ | ✅ | ✅ | 视实现 | 面试、定制化需求 |
| structuredClone | ✅ | ❌ | 部分 | 最快 | 现代浏览器、纯数据 |

---

## 本章小结

| 知识点 | 核心要点 |
|--------|----------|
| 浅拷贝 vs 深拷贝 | 浅复制第一层引用，深递归复制所有层 |
| JSON 方法 | 简单但有三大坑：函数、循环引用、特殊对象 |
| 手写深拷贝 | 递归 + WeakMap 处理循环引用 |
| structuredClone | 现代浏览器原生方案，不支持函数 |

## 面试检查清单

- ☐ 能说清楚浅拷贝和深拷贝的区别
- ☐ 能列举 JSON 方法的三个坑
- ☐ 能手写一个基础的深拷贝函数
- ☐ 能解释为什么用 WeakMap 而不是 Map
- ☐ 能处理循环引用，说清 WeakMap 作用原理
- ☐ 能对比四种拷贝方案的优缺点和适用场景

**面试行动建议**：回答这类题目**先讲思路再写代码**。把上面的手写深拷贝函数自己跑一遍测试，改改边界情况加深理解。

# 第8章 盒模型与 box-sizing 解析

# Grid 布局与响应式设计：搞定复杂布局不再难

**本章覆盖 10 道高频面试题**，掌握 Grid 网格布局核心概念，理解响应式设计原理与四大单位，能独立完成 Grid + 响应式联合布局。

---

## 一、盒模型基础回顾

> **核心口诀**：「content-box 加法算总宽，border-box 减法算内容；margin 会合并，Flex/Grid 能阻断」

- **标准盒模型（content-box）**：`width` = content，padding/border 额外叠加
- **IE 盒模型（border-box）**：`width` = content + padding + border
- **box-sizing 属性**：推荐 `border-box`，`inherit` 可强制继承，`padding-box` 已废弃
- **外边距合并**：仅在垂直块级元素间发生，Flexbox/Grid 容器内不会合并

---

## 二、Grid 布局：二维网格的利器

### 【高频★★★】面试题 1：什么是 Grid 布局？核心属性有哪些？

Grid 是 CSS 提供的二维布局系统，能同时控制行和列。Flexbox 处理单方向，Grid 处理行列交叉。

**核心属性**：

```css
.container {
  display: grid;
  grid-template-columns: 100px 1fr 2fr;  /* 固定100px + 剩余空间1:2分配 */
  grid-template-rows: auto;
  gap: 20px;  /* 或 gap: 20px 30px 行列间距 */
}
```

### 【高频★★★】面试题 2：`fr` 单位与 `auto-fill`/`auto-fit` 的区别？

**`fr`**：fraction 的缩写，表示剩余空间的等分。`1fr 2fr` 表示第一列占 1/3，第二列占 2/3。

**`auto-fill` vs `auto-fit`**：当列数不足时，`auto-fill` 保留空列占用空间，`auto-fit` 折叠空列让项目扩展填满。

| 场景 | `auto-fill` | `auto-fit` |
|------|-------------|------------|
| 项目填满容器 | 行为相同 | 行为相同 |
| 列数不足 | 空列占位 | 空列消失 |

### 【高频★★★】面试题 3：`grid-area` 如何简化复杂布局？

用命名区域替代行列序号：

```css
.container {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 200px 1fr;
}
.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.footer { grid-area: footer; }
```

### 【高频★★★】面试题 4：Grid 与 Flexbox 的区别及选型？

| 维度 | Grid | Flexbox |
|------|------|---------|
| 维度 | 二维（行+列） | 一维（单行或单列） |
| 适用场景 | 页面整体布局、相册 | 导航栏、卡片内对齐 |
| 项目位置 | 容器统一控制 | 项目自己决定 |

**原则**：需要同时控制行列选 Grid，单方向流动选 Flexbox。常见「外层 Grid 布局框架，内层 Flexbox 处理细节」。

---

## 三、响应式设计核心

### 【高频★★★】面试题 5：viewport 设置与响应式布局的关系？

移动端必须正确设置，否则响应式失效：

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

- `width=device-width`：页面宽度等于设备宽度
- `initial-scale=1.0`：初始缩放比例不放大不缩小

### 【高频★★★】面试题 6：媒体查询的断点如何设计？

**断点基于内容而非设备**：

```css
/* 默认样式（手机优先） */
.column { flex-direction: column; }

/* 大屏幕（>= 768px） */
@media (min-width: 768px) {
  .column { flex-direction: row; }
}
```

常用断点：`768px`（平板及以下）、`480px`（手机）。CSS 规则「后面覆盖前面」。

### 【高频★★★】面试题 7：rem/em/vw/vh 四大单位怎么选？

| 单位 | 相对参照 | 典型应用 |
|------|---------|---------|
| rem | 根元素字体 | **整体缩放** |
| em | 当前元素字体 | 组件内部相对尺寸 |
| vw/vh | 视口宽/高 | 全屏布局 |

**rem 换算技巧**：设计稿 750px，约定 1rem = 50px → 750px ÷ 50 = 15rem。实际开发用 `font-size: 62.5%` 让 1rem ≈ 10px，配合 PostCSS 自动转换。

### 【高频★★】面试题 8：响应式图片 `srcset` 怎么用？

根据设备像素比加载不同图片：

```html
<img src="photo-400.jpg"
     srcset="photo-400.jpg 1x, photo-800.jpg 2x"
     alt="响应式图片">
```

- `1x`：标准屏幕加载 400px 图片
- `2x`：Retina 屏幕加载 800px 图片

---

## 四、实战场景：Grid + 响应式联合应用

### 【高频★★★】面试题 9：如何用 Grid 实现响应式卡片布局？

```css
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}
```

- `auto-fit`：自动计算列数，不足时项目扩展填满
- `minmax(250px, 1fr)`：最小 250px，最大平分剩余空间
- 浏览器自动调整列数，无需多个媒体查询

### 【高频★★】面试题 10：Flexbox 与 Grid 如何协同使用？

**「外层 Grid、内层 Flex」黄金法则**：

```css
/* 外层：页面两栏布局 */
.page {
  display: grid;
  grid-template-columns: 1fr 250px;
}
/* 内层：卡片内部元素对齐 */
.card {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
```

---

## 本章小结

| 知识点 | 核心要点 |
|--------|----------|
| Grid vs Flexbox | Grid 处理二维布局，Flexbox 处理一维对齐 |
| fr 单位 | 比例分配剩余空间，配合 `auto-fit` 实现自适应列数 |
| viewport | 必须正确设置，否则响应式失效 |
| 媒体查询 | 断点基于内容而非设备，推荐 1-3 个断点 |
| 四大单位 | rem 整体缩放、em 组件内部、vw/vh 全屏布局 |

---

## 面试检查清单

- [ ] 能说清 Grid 与 Flexbox 的核心区别和选型原则
- [ ] 能手写 `repeat(auto-fit, minmax())` 响应式网格
- [ ] 能写出正确的 viewport meta 标签
- [ ] 能解释 rem/em/vw/vh 的适用场景
- [ ] 能用 Grid + Flexbox 组合实现复杂布局

---

## 行动建议

1. **动手实验**：在 CodePen 写 `repeat(auto-fit, minmax(200px, 1fr))` 拖动看列数变化
2. **对比差异**：分别用 `auto-fill` 和 `auto-fit` 做同一布局，观察空列行为
3. **模拟断点**：Chrome DevTools 切换设备模式，看媒体查询生效效果

---

**学完 Grid 与响应式设计，你已经掌握了布局的核心能力。下一章我们来学习 Flexbox 布局核心要点，掌握「一维对齐」的精髓。**

# 第9章 Flexbox 布局核心要点

Flexbox 是现代前端布局的核心技术，从导航栏到卡片列表，从居中对齐到自适应排列，几乎承包了 80% 的日常布局需求。很多候选人能写 flex: 1，但被问到 flex: auto 和 flex: 1 的本质区别时就卡壳。本章彻底拿下这个核心技能。

## 核心概念：主轴与交叉轴

Flexbox 两条轴：**主轴（Main Axis）** 由 flex-direction 决定方向，子元素默认沿主轴排列；**交叉轴（Cross Axis）** 永远垂直于主轴。主轴横向时交叉轴纵向，反之亦然。

## 容器属性

**flex-direction**：row（默认从左到右）、row-reverse、column、column-reverse。

**flex-wrap**：nowrap（默认压缩子元素）、wrap（允许换行）。

**flex-flow** 是 direction 和 wrap 的简写，例如 `flex-flow: row wrap`。

**justify-content**（主轴对齐）：flex-start、center、space-between、space-around、space-evenly。

**align-items**（交叉轴对齐）：flex-start、center、stretch（默认）、baseline。

> **【高频】justify-content:center 和 align-items:center**：前者管主轴方向，后者管交叉轴方向，两者组合实现真正居中。

## 项目属性

**flex-basis**：定义基准尺寸，优先级高于 width。

**flex-grow**：空间充足时按权重分配。容器 800px，三个子元素 flex-basis 各 200px，剩余 200px 按权重分配。

**flex-shrink**：空间不足时按权重收缩。flex-shrink: 0 表示不收缩。

**flex 简写**：

| 简写 | 等价形式 | grow | shrink | basis |
|------|----------|------|--------|-------|
| flex: 1 | flex: 1 1 0% | 1 | 1 | 0% |
| flex: auto | flex: 1 1 auto | 1 | 1 | auto |
| flex: none | flex: 0 0 auto | 0 | 0 | auto |

**【高频】flex: 1 vs flex: auto**：flex: 1 以 0 为基准平分，元素会尽可能占据所有可用空间；flex: auto 以内容为基准扩展，先计算内容固有尺寸，剩余空间再按比例分配。侧边栏常用 flex: 1，主内容区用 flex: auto 保持内容原始尺寸。

## 常用布局实战

**水平垂直居中**：

```css
.parent { display: flex; justify-content: center; align-items: center; }
```

**等高列**：`display: flex; gap: 16px; .card { flex: 1; }`

**Sticky Footer**：

```css
.page { display: flex; flex-direction: column; min-height: 100vh; }
.header, .footer { flex-shrink: 0; }
.main { flex: 1; }
```

核心原理：min-height: 100vh 占满视口，flex-shrink: 0 保证页眉页脚不被压缩，flex: 1 让主内容区占据剩余空间。

## 高频面试题精讲

**题目 1：Flexbox 和 Grid 的区别？**

Flexbox 处理一维布局（单行或单列），Grid 处理二维布局（行和列同时控制）。导航栏、表单用 Flexbox；页面框架、相册用 Grid。简单来说，Flexbox 适合「排队」，Grid 适合「排兵布阵」。

**题目 2：flex: 1 和 flex-grow: 1 效果相同吗？**

不同。flex: 1 会设置 flex-shrink: 1 和 flex-basis: 0%；flex-grow: 1 则 shrink 默认为 0、basis 默认为 auto。空间不足时，flex: 1 的元素会收缩，flex-grow: 1 的元素保持原尺寸。

**题目 3：align-content 和 align-items 的区别？**

align-items 调整每个子元素在交叉轴上的位置；align-content 调整行与行之间的间距分布。当 flex-wrap: wrap 且存在多行时，align-content 才会生效。

**题目 4：子元素 flex-basis 为 0 和为 auto 有什么区别？**

flex-basis: 0 表示元素初始尺寸为 0，完全由 flex-grow 决定最终宽度；flex-basis: auto 表示先按内容或 width 确定初始尺寸，剩余空间再由 flex-grow 分配。

**题目 5：如何实现圣杯布局？**

```css
.layout { display: flex; flex-direction: column; min-height: 100vh; }
.header, .footer { flex-shrink: 0; }
.body { display: flex; flex: 1; }
.aside-left, .aside-right { flex: 0 0 200px; }
.main { flex: 1; }
```

**题目 6：如何实现平均分布且首尾两端留白？**

使用 justify-content: space-between（两端对齐、中间等间距，适合导航栏）或 space-evenly（间距和两端间距都相等，适合卡片列表）。

## 本章小结

**容器属性**：display: flex 开启布局；flex-direction 决定主轴方向；flex-wrap 控制换行策略；justify-content 负责主轴对齐；align-items 负责交叉轴对齐；gap 设置项目间距。

**项目属性**：flex-grow 定义扩展比例；flex-shrink 定义收缩比例；flex-basis 设置基准尺寸，优先级高于 width；flex 是三者的简写；align-self 覆盖对齐方式；order 改变排列顺序。

**flex 简写对比**：flex: 1 = flex: 1 1 0%，以零为起点平分空间；flex: auto = flex: 1 1 auto，先满足内容尺寸再分配剩余；flex: none = flex: 0 0 auto，固定尺寸不参与弹性计算。

**常用布局**：水平垂直居中（justify-content + align-items 均为 center）、Sticky Footer（flex-direction: column + min-height: 100vh + flex: 1）、等高列（flex: 1 等宽）。

**2026 年备考提示**：Flexbox 与 CSS Grid 形成互补组合。面试中除掌握基础属性外，建议理解 flex 容器与项目间的尺寸计算机制。Flexbox 与 Grid 的选型决策、rem/em 配合 flex 实现自适应布局，也是近年高频考点。

学完 Flexbox，下一章我们来掌握 CSS Grid。与 Flexbox 的一维布局不同，Grid 能同时控制行和列，是实现复杂页面结构的不二之选。

# 第10章 Grid 布局与响应式设计

> **本章覆盖题目**：约 6 道
>
> **学习目标**：掌握 Grid 布局核心概念，能区分 Flexbox 与 Grid 的适用场景，理解现代响应式设计的实现方法

CSS Grid 能同时处理行和列，让复杂布局简单可控。Grid 和 Flexbox 的选择策略是高频考点，这章帮你彻底搞懂。

---

## 一、Grid 布局核心概念

### 1.1 三个关键术语

**网格线**（Grid Lines）：组成网格的水平和垂直分隔线，从 1 开始编号。放置网格项时通过指定开始和结束的线号来控制位置。

**轨道**（Tracks）：两条相邻网格线之间的区域，代表一行或一列。通过 `grid-template-columns` 和 `grid-template-rows` 定义。

**单元格**（Cells）：行轨道和列轨道交叉形成的最小区域，一个网格项可以跨越一个或多个单元格。

### 1.2 创建网格容器

```css
.container {
  display: grid;
  grid-template-columns: 200px 1fr auto;
  grid-template-rows: 100px 100px;
  gap: 20px;
}
```

> **面试常问**：`grid-gap`、`grid-row-gap`、`grid-column-gap` 已是 `gap`、`row-gap`、`column-gap` 的别名，新代码直接用 `gap`。

---

## 二、fr 单位与轨道计算

### 2.1 fr 的含义

`fr` 表示网格容器的**可用空间比例**。剩余空间 = 容器宽度 - 固定轨道宽度，再按 `fr` 比例分配。

```css
.container {
  width: 1000px;
  grid-template-columns: 200px 1fr 2fr;
}
```

计算：固定列 200px，剩余 800px。按 1:2 分配，第二列 ≈ 267px，第三列 ≈ 533px。

### 2.2 repeat() 语法

```css
/* 5 列等宽 */
grid-template-columns: repeat(5, 1fr);

/* auto-fill：根据容器宽度自动填充尽可能多的列，最小 200px */
grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
```

> **面试加分**：`auto-fill` 会在没有元素时保留空轨道，`auto-fit` 会让已有元素展开填满空白。

---

## 三、网格项对齐

| 属性 | 作用范围 | 对齐方向 |
|------|----------|----------|
| `justify-items` | 容器内所有项 | 水平方向（列内对齐） |
| `align-items` | 容器内所有项 | 垂直方向（行内对齐） |
| `place-items` | 同时设置上述两者 | 两个方向 |
| `justify-content` | 整个网格在容器中的位置 | 水平方向 |
| `align-content` | 整个网格在容器中的位置 | 垂直方向 |
| `justify-self` / `align-self` | 单个网格项自身 | 单独控制 |

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  height: 300px;
  place-items: center center; /* 单元格内居中 */
}
```

> **易错提醒**：`justify-items` 控制列内对齐，`justify-content` 控制整个网格在容器的水平位置，别搞混。

---

## 四、Flexbox vs Grid：什么时候选谁？

| 场景 | 推荐方案 | 原因 |
|------|----------|------|
| 一行/一列均匀分布 | Flexbox 或 Grid 均可 | 简单场景用 Flexbox 更轻量 |
| 复杂二维布局（行+列同时控制） | **Grid** | Flexbox 嵌套多层才能实现 |
| 导航菜单（内容自适应） | **Flexbox** | Flex item 根据内容自动伸缩 |
| 整体页面布局（Header/Sidebar/Main/Footer） | **Grid** | 一套定义搞定所有区域 |
| 卡片流自动排列 | **Grid**（配合 auto-fill） | Flexbox 实现等高卡片较困难 |

**面试经典回答**：Flexbox 是「内容决定布局」，适合组件内部一维排列；Grid 是「容器决定布局」，适合页面级别二维结构。两者互补，实际项目经常同时使用。

---

## 五、响应式布局方案

### 5.1 媒体查询 + 断点

```css
/* 移动优先 */
.card { width: 100%; }

@media (min-width: 768px) {
  .card { width: 50%; }
}

@media (min-width: 1024px) {
  .card { width: 25%; }
}
```

### 5.2 rem/em/vw 单位选择

| 单位 | 含义 | 适用场景 |
|------|------|----------|
| `rem` | 相对根元素字体大小 | 全局缩放 |
| `em` | 相对当前元素字体大小 | 组件内局部缩放 |
| `vw/vh` | 相对视口宽/高度 | 横幅、Hero 区域全屏适配 |

```css
/* vw 实现流式字体大小 */
.title {
  font-size: clamp(1.5rem, 5vw, 3rem);
}
```

> ⚠️ **2026 备考提示**：CSS 容器查询（Container Queries）已获主流浏览器支持，可在组件级别实现响应式，不再依赖视口宽度，有兴趣可以了解 `@container` 规则。

---

## 本章小结

| 知识点 | 核心要点 |
|--------|----------|
| Grid 三元素 | 网格线（Lines）、轨道（Tracks）、单元格（Cells） |
| fr 单位 | 分配剩余空间的比例，计算时先减去固定值 |
| 对齐属性 | place-items 控制单元格内对齐，place-content 控制网格在容器内位置 |
| Flex vs Grid | 组件一维排列用 Flex，页面整体布局用 Grid |
| 响应式 | 媒体查询配合 rem/vw，配合 clamp() 实现流式缩放 |

---

## 面试检查清单

- [ ] 能手写一个简单的 Grid 布局（定义列、行、gap）
- [ ] 能解释 `fr` 单位如何计算，配合 `repeat()` 写出响应式列定义
- [ ] 能区分 `justify-items`、`justify-content`、`justify-self` 三个属性的作用范围
- [ ] 能针对具体场景说明应该用 Flexbox 还是 Grid，并给出理由
- [ ] 能说出 rem、em、vw 的区别和各自适用场景

---

学完 Grid 布局后，下一章我们进入「BFC 原理与清除浮动」，理解块级格式化上下文如何解决父元素塌陷和边距重叠问题——这和布局息息相关，是你布局能力的重要补充。

# 第11章 BFC 原理与清除浮动

# 第 11 章 BFC 原理与清除浮动

**本章覆盖【高频】面试题约 6 道**

BFC（Block Formatting Context，块级格式化上下文）是 CSS 布局的高频考点，理解它能解决外边距合并、浮动清除等常见问题。

---

## 一、为什么要学 BFC

新手常遇父子元素上边距重叠、设置 margin-top 但父元素跟着掉、右边内容被左边浮动压住等问题。这些都和 BFC 有关，掌握它能让你从"瞎试"升级到"精准定位"。

---

## 二、什么是 BFC

BFC 是独立的渲染区域，容器内部的布局规则与外部隔离，互不影响——就像给元素套了个"隔板"。

**触发条件（满足任意一条）：**

- `display` 为 `flow-root`、`inline-block`、`table` 等
- `position` 为 `absolute` 或 `fixed`
- `float` 不为 `none`
- `overflow` 不为 `visible`（常用 `hidden`、`auto`、`scroll`）

其中 `flow-root` 是专门触发 BFC 的属性，无副作用，现代浏览器全面支持。

---

## 三、外边距合并

**【高频】面试题 1：两个 div 上下排列，上面的下边距 20px，下面的上边距 10px，实际间距是多少？**

垂直相邻的块级元素，外边距会"融合"，取较大值而非相加。

**答案：** 20px。

**扩展追问：** 外边距合并的三条规则：
1. **同级合并**：相邻块级元素垂直 margin 会合并
2. **父子合并**：父元素无 `border`、`padding` 隔开时，子元素 margin 会合并到父元素外部
3. **自身合并**：空块级元素上下 margin 会合并

解决方式：给父元素加 `overflow:hidden`、`border` 或 `padding`。

---

## 四、清除浮动

**【高频】面试题 2：子元素 float 后父容器高度塌陷，怎么解决？**

浮动元素脱离文档流，但仍在父容器内部，导致父容器高度计算不包含浮动子元素。

**答案：** 让父容器触发 BFC，用 `overflow:hidden` 或 `display:flow-root`。

---

## 五、清除浮动的多种方法对比

**【高频】面试题 3：清除浮动有哪几种方式？各自优缺点是什么？**

| 方法 | 优点 | 缺点 | 场景 |
|-----|------|------|------|
| 额外标签法 | 兼容性好 | 增加无意义标签 | 老项目 |
| `overflow:hidden/auto` | 简单 | 可能裁剪内容 | 单纯解决塌陷 |
| `display:inline-block` | 触发 BFC | 改变布局特性 | 不推荐 |
| **clearfix（::after）** | 无副作用 | IE7 以下不支持 | **现代项目首选** |

---

## 六、overflow:hidden 与 clearfix 技巧

**【高频】面试题 4：overflow:hidden 为什么能清除浮动？**

它触发 BFC，BFC 会自动计算并包含浮动子元素的高度。

### clearfix 写法

```css
.clearfix::after {
  content: '';
  display: block;
  clear: both;
}
```

**注意：** `overflow:hidden` 可能裁剪超出内容，适合无滚动条场景；`overflow:auto` 会在内容溢出时显示滚动条。

---

## 七、实战场景

**【高频】面试题 5：如何用浮动实现左固定、右自适应的两栏布局？**

```html
<style>
  .container { overflow: hidden; }
  .sidebar { float: left; width: 200px; background: #eee; }
  .content { overflow: hidden; }
</style>
<div class="container">
  <div class="sidebar">侧边栏 200px</div>
  <div class="content">自适应内容区</div>
</div>
```

左边栏浮动，右边内容区触发 BFC 形成格式化隔离，自动避开浮动区域。

**防止文字环绕：** 让文字在浮动元素下方重新开始，而非环绕：

```html
<style>
  .box { float: left; width: 150px; height: 150px; background: #ddd; }
  .content { overflow: hidden; }
</style>
<div class="box">浮动元素</div>
<p class="content">这段文字会被推到浮动元素的下方。</p>
```

**多栏等高布局：** 给每栏设置 `overflow:hidden` 触发 BFC，各栏自动等高：

```html
<style>
  .row { overflow: hidden; }
  .col { float: left; width: 33.33%; overflow: hidden; }
</style>
```

---

## 八、BFC 与 IFC 的区别

**【高频】面试题 6：BFC 和 IFC 有什么区别？**

| 维度 | BFC | IFC |
|------|-----|-----|
| 布局方向 | 垂直，从上到下 | 水平，从左到右 |
| 排列元素 | 块级盒子 | 行内盒子 |
| 宽度计算 | 独占一行 | 由内容决定 |

**口诀：** BFC 是"一列一列"的，IFC 是"一行一行"的。

**实际应用：** IFC 用于文字换行、inline-block 垂直对齐、单行文字垂直居中（用 line-height）。

---

## 九、知识点延伸：为什么 overflow 不为 visible 会触发 BFC

CSS 规范规定，当 `overflow` 值不为 `visible` 时，元素会创建一个新的 BFC。

**原理：** 设置 `overflow:hidden`、`auto` 或 `scroll` 时，元素必须建立裁剪边界来控制内容显示——这个裁剪边界要求元素成为独立的格式化上下文，否则裁剪规则无法明确作用范围。

简单说：**需要控制内容溢出的元素，必须先成为独立的渲染区域，才能正确实施裁剪。**

---

## 十、IE 兼容性（了解即可）

IE6/7 没有 BFC，但有类似机制 `hasLayout`，触发方式有 `zoom:1`、`position:relative`、`设置宽高` 等。现代项目不需考虑。

---

## 本章小结

BFC 核心三个特性：

1. **独立渲染区域** — 内部布局不影响外部
2. **外边距不合并** — 触发 BFC 后垂直 margin 不会折叠
3. **包含浮动元素** — 计算高度时包含浮动子元素

| 常见问题 | 核心答案 |
|---------|---------|
| 什么是 BFC | 独立的块级渲染区域 |
| 怎么触发 BFC | `overflow:hidden`、`display:flow-root` 等 |
| 外边距合并原因 | 同一 BFC 内块级元素垂直 margin 会合并 |
| 清除浮动最优方案 | clearfix 或 `display:flow-root` |
| overflow:hidden 与 auto 的区别 | hidden 可能裁剪内容，auto 允许滚动 |
| hasLayout 是什么 | IE6/7 的类似 BFC 的机制 |

---

## 面试检查清单

面试前用这些问题自检：

- [ ] 能说出 4 种以上触发 BFC 的方式
- [ ] 能解释外边距合并的三条规则及解决方案
- [ ] 能对比 4 种清除浮动方法的优缺点
- [ ] 能解释 clearfix 的原理
- [ ] 能说出 BFC 和 IFC 的区别
- [ ] 能手写两栏浮动布局并解释原理

**行动建议：** 动手写一个外边距合并 demo 和浮动清除 demo。如果面试官追问"为什么 overflow:hidden 能清除浮动"，可以回答："因为它触发了 BFC，而 BFC 的计算规则要求父容器必须包含所有浮动子元素的高度，这是一条 CSS 规范。具体来说，当 overflow 不为 visible 时，元素需要建立裁剪边界来控制内容显示，而裁剪边界的前提是元素成为独立的格式化上下文。"

# 第12章 Vue3 核心原理与响应式系统

> **本章导学**：覆盖 6 道高频面试题，聚焦 Vue3 响应式核心机制：Proxy 如何取代 Object.defineProperty、ref 与 reactive 的本质区别、computed 与 watch 的实现原理、Vue3 生命周期变化以及 Composition API 的设计初衷。读完本章，你能从源码级别理解「Vue3 的响应式是怎么工作的」。

---

## 一、为什么 Vue3 抛弃了 Object.defineProperty

### 1.1 Vue2 响应式的局限性

Vue2 的响应式系统基于 `Object.defineProperty` 实现，这带来了几个难以克服的问题：

**无法监听新增/删除属性**：`Object.defineProperty` 只能在对象创建时定义属性。后续动态添加的属性（如 `obj.newProp = 'value'`）是响应不到的，必须用 `Vue.set` 或 `this.$set` 手动处理。

**无法监听数组下标变化**：直接通过下标修改数组元素（如 `arr[0] = 'newValue'`）不会触发响应式更新，Vue2 不得不hack 数组的 7 个方法（push、pop、splice 等）来实现监听。

**性能开销大**：每个属性都要调用 `Object.defineProperty`，对于嵌套层级深的对象，初始化时要递归遍历所有层级，性能损耗明显。

### 1.2 Proxy：更强大的拦截器

Vue3 选择了 `Proxy` 作为响应式系统的基石。Proxy 可以在对象级别拦截所有操作，而不需要对每个属性单独定义：

```javascript
const raw = { name: '张三', age: 25 }
const proxy = new Proxy(raw, {
  get(target, key) {
    console.log(`读取 ${key}`)
    return Reflect.get(target, key)
  },
  set(target, key, value) {
    console.log(`设置 ${key} = ${value}`)
    return Reflect.set(target, key, value)
  }
})

proxy.name        // 触发 get，输出：读取 name
proxy.age = 30    // 触发 set，输出：设置 age = 30
```

**核心优势**：

- **自动监听新增属性**：给代理对象添加新属性时，`set` 陷阱自动触发
- **天然支持数组**：通过下标访问或修改数组，`get` 和 `set` 陷阱都会被触发
- **性能更好**：Proxy 在运行时才创建代理，不会在初始化时深度遍历所有属性

### 1.3 Proxy 的陷阱与应对

Proxy 看似完美，但有两个经典「坑」需要避开：

**深嵌套对象的响应式丢失**：

```javascript
const state = reactive({ user: { profile: { name: '张三' } } })
// 当你解构时
const { profile } = state.user
// profile 脱离代理范围，修改它不会触发更新
```

Vue3 提供了 `toRef` 和 `toRefs` 来保持解构后的响应性。

**⚠️ 时效性说明**：Vue3.3+ 对 reactive 解构做了改进，可以直接解构基本类型。但在解构对象属性时，仍建议使用 `toRefs` 或 `toRef` 保持响应性。

**数组性能问题**：直接用下标修改大数组（如 `arr[10000] = 'value'`）虽然会触发响应式，但 Vue3 无法像 Vue2 一样「记住」这类操作，需要配合 `push` 或 `splice` 使用。

---

## 二、ref 与 reactive：两种响应式定义方式

### 2.1 ref：包装基本类型

`ref` 本质上是用一个对象包裹基本类型，通过 `Object.defineProperty` 的 `get`/`set` 实现响应式：

```javascript
// ref 内部实现简化
function ref(value) {
  return {
    get value() {
      track()  // 收集依赖
      return value
    },
    set value(newVal) {
      value = newVal
      trigger()  // 触发更新
    }
  }
}

const count = ref(0)
console.log(count.value)  // 读取要加 .value
count.value++             // 修改要加 .value
```

为什么基本类型需要 `.value`？因为基本类型本身不是引用，无法被代理。ref 通过创建一个包含 `value` 属性的响应式对象来解决这个问题。

**模板中使用 ref 时，Vue 会自动解包**，不需要写 `.value`：

```vue
<template>
  <!-- Vue 自动解包，count 等价于 count.value -->
  <span>{{ count }}</span>
</template>
```

### 2.2 reactive：深度代理对象

`reactive` 直接返回一个 Proxy，适用于对象和数组：

```javascript
const state = reactive({ name: '张三', skills: ['JavaScript', 'Vue'] })
state.name = '李四'     // 响应式更新
state.skills.push('CSS') // 数组操作也响应式
```

### 2.3 何时用 ref，何时用 reactive

| 场景 | 推荐 | 原因 |
|------|------|------|
| 基本类型（number、string、boolean） | `ref` | 基本类型不是引用，无法被 Proxy 代理 |
| 对象/数组 | `reactive` | Proxy 天然支持复杂类型 |
| 组合式函数返回响应式数据 | `ref` | 返回值更清晰，避免解构丢失响应式 |
| 需要解构响应式对象 | `reactive` + `toRefs` | 保持解构后每个属性的响应性 |

---

## 三、computed 与 watch：两种监听机制

### 3.1 computed：计算属性的缓存机制

`computed` 的核心是「懒计算 + 缓存」。它只在依赖变化时才重新计算，依赖未变时直接返回缓存结果：

```javascript
const firstName = ref('张')
const lastName = ref('三')

const fullName = computed(() => {
  // 只有 firstName 或 lastName 变化时才重新执行
  return lastName.value + firstName.value
})
```

**原理**：computed 内部维护一个「脏标记」——依赖变化时标记为脏，下次访问时重新计算并清除脏标记。

### 3.2 watch：主动监听变化

`watch` 用于执行副作用操作（如 API 调用、DOM 操作）：

```javascript
watch(count, (newVal, oldVal) => {
  console.log(`count 从 ${oldVal} 变成了 ${newVal}`)
})

// 监听多个依赖
watch([count, name], ([newCount, newName], [oldCount, oldName]) => {
  // 两个依赖任一变化都会触发
})
```

**与 computed 的区别**：computed 适合「根据响应式数据计算新值」，watch 适合「响应式数据变化时执行副作用」。

---

## 四、Vue3 生命周期：从 Options 到 setup

### 4.1 生命周期对照表

| Vue2 生命周期 | Vue3 生命周期 | 执行时机 |
|--------------|--------------|---------|
| beforeCreate | — | setup() 执行之前 |
| created | — | setup() 执行之后 |
| beforeMount | onBeforeMount | 挂载前 |
| mounted | onMounted | 挂载完成 |
| beforeUpdate | onBeforeUpdate | 更新前 |
| updated | onUpdated | 更新完成 |
| beforeDestroy | onBeforeUnmount | 卸载前 |
| destroyed | onUnmounted | 卸载完成 |

### 4.2 setup 的执行时机

`setup` 在 `beforeCreate` 和 `created` 之间执行，且只执行一次。这意味着：

```javascript
export default {
  beforeCreate() {
    console.log('beforeCreate')
  },
  setup() {
    console.log('setup - 此时组件还未创建完成')
    // 可以直接使用响应式数据和 methods
  },
  created() {
    console.log('created')
  }
}
// 输出顺序：beforeCreate → setup → created
```

**⚠️ 时效性说明**：Vue3.0 起完全支持 Composition API，但 Vue3.2+ 引入了 `<script setup>` 语法糖，setup 函数会自动执行，代码更简洁。

---

## 五、Composition API 的设计初衷

### 5.1 逻辑复用：比 Mixin 更清晰

Vue2 的 Mixin 存在命名冲突、来源不明等问题。Composition API 通过「组合式函数」（composables）实现逻辑复用：

```javascript
// useMousePosition.js
import { ref, onMounted, onUnmounted } from 'vue'

export function useMousePosition() {
  const x = ref(0)
  const y = ref(0)
  
  const update = (e) => {
    x.value = e.clientX
    y.value = e.clientY
  }
  
  onMounted(() => window.addEventListener('mousemove', update))
  onUnmounted(() => window.removeEventListener('mousemove', update))
  
  return { x, y }
}
```

使用时，组件清晰知道「鼠标位置」这个逻辑来自哪里：

```vue
<script setup>
import { useMousePosition } from './useMousePosition'
const { x, y } = useMousePosition()
</script>
```

### 5.2 更好的 TypeScript 支持

`ref` 和 `reactive` 可以自动推断类型，不需要像 Vue2 那样写繁琐的泛型。

---

## 本章小结

| 概念 | 核心要点 |
|------|----------|
| Proxy vs Object.defineProperty | Proxy 拦截所有操作，自动监听新增属性和数组下标 |
| ref vs reactive | ref 包装基本类型（需要 .value），reactive 直接代理对象 |
| computed | 懒计算 + 缓存，依赖不变不重算 |
| watch | 主动监听，执行副作用 |
| setup 执行时机 | 在 beforeCreate 和 created 之间执行一次 |
| Composition API | 逻辑复用更清晰，TS 支持更好 |

**面试检查清单**：

- 能说出 Proxy 相比 Object.defineProperty 的三个优势
- 能解释为何 ref 需要 `.value` 而 reactive 不需要
- 能说明 computed 和 watch 的适用场景
- 能画图表示 Vue3 生命周期的执行顺序
- 能手写一个简单的 composable 函数

**复习行动计划**：

1. **20 分钟**：用 `new Proxy` 手写一个简易响应式函数，验证 get/set 触发
2. **25 分钟**：对比 `ref` 和 `reactive` 在基本类型和对象类型上的表现差异
3. **20 分钟**：梳理 Vue3 生命周期图，对照 Vue2 理解变化点
4. **面试前**：用自己的话解释「Vue3 如何实现响应式」，从 Proxy 讲到依赖收集和触发

> **面试技巧**：Vue3 响应式几乎是必问的原理题。回答时先说「Vue3 选择了 Proxy 作为响应式基础」，然后分别讲 Proxy 的优势、ref/reactive 的区别、依赖收集机制，最后可以加一句「Vue3 的响应式系统解决了 Vue2 中数组下标无法监听的问题」。结构清晰、细节到位，面试官会认为你对原理有深入理解。

# 第13章 Vue 组件通信与状态管理

**本章覆盖 5 道 Vue 组件通信与状态管理高频面试题**

**本章目标**：掌握 Vue 组件间通信方式，理解 Vuex 和 Pinia 状态管理，能根据业务场景选择最优方案。

---

## 第 1 题：Vue 中父子组件如何通信？

父子组件通信的**标准方案是 props + emit**，遵循单向数据流原则：父组件通过 props 向下传数据，子组件通过 emit 向上传事件。

```vue
<!-- 父组件 -->
<template>
  <Child :title="pageTitle" :count="num" @update="handleUpdate" @reset="handleReset" />
</template>

<script setup>
import { ref } from 'vue';
import Child from './Child.vue';

const pageTitle = ref('面试题列表');
const num = ref(10);

const handleUpdate = (val) => { num.value = val; };
const handleReset = () => { num.value = 0; };
</script>
```

```vue
<!-- 子组件 -->
<script setup>
defineProps({ title: { type: String, required: true }, count: { type: Number, default: 0 } });
const emit = defineEmits(['update', 'reset']);
</script>

<template>
  <div>
    <h1>{{ title }}</h1>
    <p>计数：{{ count }}</p>
    <button @click="emit('update', count + 1)">增加</button>
    <button @click="emit('reset')">重置</button>
  </div>
</template>
```

**易错提醒**：子组件绝对不能直接修改 props。Vue 遵循单向数据流——数据从上往下流动，子组件通过 `$emit` 触发事件，由父组件决定是否修改。

**面试追问**：为什么 Vue 要设计单向数据流？

让数据流向可追踪。出现 bug 时只需沿组件树向上追溯父组件，不用在任意子组件中排查谁改了值。

---

## 第 2 题：v-model 的原理是什么？它和 .sync 修饰符有什么区别？

v-model 是**双向绑定的语法糖**，本质是 **props + emit 的组合**：父组件传递 `modelValue`，子组件通过 `update:modelValue` 事件回传。

```vue
<!-- 父组件 -->
<template>
  <Input v-model="username" />
  <!-- 等价于 :modelValue="username" @update:modelValue="username = $event" -->
</template>
```

```vue
<!-- 子组件 -->
<script setup>
defineProps({ modelValue: { type: String, default: '' } });
const emit = defineEmits(['update:modelValue']);
const onInput = (e) => emit('update:modelValue', e.target.value);
</script>

<template>
  <input :value="modelValue" @input="onInput" placeholder="请输入用户名" />
</template>
```

Vue 3 支持多 v-model 绑定：`v-model:name="form.name" v-model:age="form.age"`。

**注意**：原生表单（input、textarea、select）的 v-model 是 Vue 内置处理的，**自定义组件的 v-model 需要开发者手动接收 props 并 emit 事件**。

---

## 第 3 题：跨级组件通信有哪些方案？如何避免 prop drilling？

prop drilling 是数据穿过很多层无关组件，代码臃肿难以维护。provide/inject 就是来解决这个问题，允许祖先组件向所有后代组件"注入"数据，跨越任意层级。

| 特性 | Vue 2 | Vue 3 |
|-----|-------|-------|
| API | provide/inject 选项式 | setup 中使用 Composition API |
| 响应式 | 传递对象本身 | 传递 ref/reactive 更灵活 |

```vue
<!-- 祖先组件 -->
<script setup>
import { provide, ref } from 'vue';

const userInfo = ref({ name: '张三', role: 'admin' });
provide('user', userInfo);
</script>
```

```vue
<!-- 后代组件（任意层级） -->
<script setup>
const user = inject('user');
console.log(user.value.name); // 张三
</script>
```

**响应式规则**：

| 传递方式 | 是否响应式 |
|---------|-----------|
| ref/reactive/computed | ✅ |
| 普通值（字符串、数字） | ❌ |

**后代能否修改 provide 的值？** 技术层面可以，但强烈不推荐。provide/inject 是单向的，正确做法是让祖先提供更新方法：

```vue
<!-- 祖先组件 -->
<script setup>
const userInfo = ref({ name: '张三' });
const updateUser = (newInfo) => { userInfo.value = { ...userInfo.value, ...newInfo }; };
provide('user', { data: userInfo, update: updateUser });
</script>
```

**$attrs**：透传未声明的属性（class、style、事件监听等），用于包装组件：

```vue
<!-- 父组件 -->
<Button type="primary" size="large" class="custom-btn" @click="handleClick" />
```

```vue
<!-- Button 组件 -->
<template>
  <el-button v-bind="$attrs">内容</el-button>
  <!-- 自动获得 type、size、class、@click -->
</template>
```

---

## 第 4 题：非父子组件之间怎么通信？

非父子组件通信有两种思路：**事件总线（mitt）** 适合零散事件，**Pinia** 适合共享状态。

```js
// eventBus.js
import mitt from 'mitt';
export const emitter = mitt();
```

```vue
<!-- 组件A：发送事件 -->
<script setup>
import { emitter } from './eventBus';
const sendData = () => emitter.emit('data-updated', { id: 1 });
</script>
```

```vue
<!-- 组件B：监听事件 -->
<script setup>
import { onMounted, onUnmounted } from 'vue';
import { emitter } from './eventBus';

onMounted(() => emitter.on('data-updated', (data) => console.log('收到:', data)));
onUnmounted(() => emitter.off('data-updated')); // ⚠️ 卸载时必须清理
</script>
```

**典型使用场景**：页面刷新提示、夜间模式切换、全局 loading 状态、WebSocket 消息推送。

**易错提醒**：
- **内存泄漏风险**：组件卸载时没有调用 `emitter.off()` 移除监听，监听函数仍在内存中。
- **维护性下降**：事件满天飞后数据流向不可追踪。

**为什么推荐用 Pinia 替代 mitt**：

| 维度 | mitt 事件总线 | Pinia |
|------|-------------|-------|
| 数据流向 | 隐式 | 显式 |
| 调试难度 | 困难 | 简单（Vue DevTools 可视化） |
| 类型支持 | 弱 | 强（TS 友好） |
| 适用场景 | 2个组件以下 | 无限制 |

---

## 第 5 题：Vuex 和 Pinia 怎么选？

**Vue 3 项目选 Pinia，Vue 2 项目选 Vuex。** Pinia 是 Vue 官方推荐的状态管理方案，API 更简洁、TypeScript 支持更好。

```js
// Pinia（Vue 3 推荐）
import { defineStore } from 'pinia';

export const useUserStore = defineStore('user', () => {
  const userInfo = ref(null);
  const isLoggedIn = computed(() => !!userInfo.value);
  
  const login = async (credentials) => {
    const user = await api.login(credentials);
    userInfo.value = user;
  };
  
  const logout = () => { userInfo.value = null; };
  
  return { userInfo, isLoggedIn, login, logout };
});
```

**Pinia 进阶功能**（Vuex 不支持）：

```js
import { storeToRefs } from 'pinia';
const { userInfo } = storeToRefs(useUserStore());

// 监听状态变化
userStore.$subscribe((mutation, state) => {
  localStorage.setItem('user', JSON.stringify(state.userInfo));
});

// 监听 actions 执行
userStore.$onAction(({ name, args, after }) => {
  console.log(`调用 action: ${name}`, args);
});
```

```js
// Vuex（Vue 2 方案）
import { createStore } from 'vuex';

export default createStore({
  state: { count: 0, user: null },
  getters: { doubleCount: (state) => state.count * 2 },
  mutations: {
    INCREMENT(state) { state.count++; },
    SET_USER(state, user) { state.user = user; }
  },
  actions: {
    async fetchUser({ commit }) {
      const user = await api.getUser();
      commit('SET_USER', user);
    }
  }
});
```

**关键区别**：mutation 同步修改 state，action 可包含异步操作，最终调用 mutation 修改状态。

---

## 面试检查清单：通信方式选型决策树

```
需要通信吗？
├── 父子通信
│   ├── 父→子单向传值 → props
│   ├── 子→父事件通知 → emit
│   └── 双向绑定 → v-model
├── 跨级通信
│   ├── 祖先→后代 → provide
│   ├── 后代→祖先 → inject + 方法
│   └── 透传属性 → $attrs + v-bind
└── 非父子通信
    ├── 零散事件（2个组件以下） → mitt
    ├── 共享状态（2个组件以上） → Pinia
    └── Vue 2 项目 → Vuex
```

| 场景 | 推荐方案 |
|-----|---------|
| 父组件给子组件传配置参数，两层 | **props**（不要过度设计） |
| 10 个组件共享用户登录态 | **Pinia**（useAuthStore） |
| 一个组件触发，另一个无关组件响应 | 偶尔一次用 `mitt`；超过2次考虑 **Pinia 重构** |

---

## 本章小结

1. **父子通信**：props + emit 或 v-model，defineProps/defineEmits 是 Vue 3 面试高频考点
2. **跨级通信**：provide/inject 避免 prop drilling，传递响应式对象；后代修改值不推荐
3. **任意组件**：mitt 事件总线（慎用，卸载清理）或 Pinia（推荐）
4. **$attrs**：透传未声明的属性和事件，用于包装组件
5. **Pinia**：Vue 3 推荐，API 简洁、TS 友好、$subscribe/storeToRefs 等进阶功能
6. **Vuex**：Vue 2 官方方案，mutation/action 职责分离

**行动建议**：在项目中找一个三层以上嵌套的组件，用 provide/inject 重构透传逻辑；把一个共享状态从事件总线迁移到 Pinia，亲自感受代码量差异和调试体验提升。这些实践经历整理成项目经验，面试时更有说服力。

# 第14章 React Hooks 原理与实战

> **本章覆盖 React Hooks 高频面试题 12 道**，重点掌握 useState/useReducer 原理、useEffect 依赖管理、useLayoutEffect 执行时机、useCallback/useMemo 优化策略、useRef 进阶用法，以及 React 18 并发模式新 Hooks 用法。

---

## 题目一：useState 与 useReducer 的原理与选择

**面试场景**：「你负责的状态管理方式有哪些？什么情况下用 useState，什么情况下用 useReducer？」

React 内部通过链表存储 Hooks 状态，每次渲染按调用顺序读取。Hooks 不能写在条件语句或循环中，因为调用顺序决定状态与 Hook 的对应关系。

| Hook | 适用场景 |
|-----|---------|
| useState | 简单状态（布尔、字符串、数字） |
| useReducer | 复杂状态逻辑、需要状态溯源或可测试性高的场景 |

```javascript
const reducer = (state, action) => {
  switch (action.type) {
    case 'FETCH_SUCCESS': return { ...state, loading: false, data: action.payload }
    case 'FETCH_ERROR': return { ...state, loading: false, error: action.payload }
    default: return state
  }
}
const [state, dispatch] = useReducer(reducer, { loading: false, data: null, error: null })
```

**追问**：什么时候反而用 useState 更好？
> 状态逻辑非常简单时（如只有一个布尔开关），useState 更直观。只有当涉及多个子状态或需要可追溯的 actions 时，useReducer 的优势才明显。

---

## 题目二：useEffect 与 useLayoutEffect 的执行时机差异

**面试场景**：「useEffect 和 useLayoutEffect 有什么区别？分别在什么时候用？」

| Hook | 执行时机 | 适用场景 |
|-----|---------|---------|
| useEffect | 异步，浏览器paint后 | 数据请求、订阅、事件监听 |
| useLayoutEffect | 同步，DOM变更后、paint前 | 立即操作 DOM（测量尺寸、聚焦） |

**执行顺序**：React 更新 → commit 阶段 → useLayoutEffect 同步执行 → 浏览器 paint → useEffect 异步执行。

```javascript
// useLayoutEffect：DOM 变化后立即执行，可能阻塞 paint
useLayoutEffect(() => {
  inputRef.current?.focus()
}, [])

// useEffect：浏览器 paint 后执行，不阻塞渲染
useEffect(() => {
  document.title = `计数：${count}`
}, [count])
```

> ⚠️ **易错提醒**：SSR 中 useLayoutEffect 会在服务端执行，若依赖浏览器 API 会报错。用 Suspense 包裹或检测环境解决。

---

## 题目三：useEffect 依赖管理与清理机制

**面试场景**：「有同事说他写的 useEffect 导致了无限循环，怎么排查？」

| 依赖写法 | 执行时机 |
|---------|---------|
| `[]` | 仅首次渲染后 |
| `[dep1, dep2]` | 指定依赖变化时 |
| 无依赖数组 | 每次渲染后 ⚠️极易导致无限循环 |

**定时器闭包陷阱**：

```javascript
// ❌ 陷阱：定时器闭包捕获的 count 永远是初始值
useEffect(() => {
  const timer = setInterval(() => setCount(count + 1), 1000)
  return () => clearInterval(timer)
}, []) // count 未在依赖中 → 永远只计数到 1

// ✅ 函数式更新
useEffect(() => {
  const timer = setInterval(() => setCount(prev => prev + 1), 1000)
  return () => clearInterval(timer)
}, [])

// ✅ 用 useRef 存储最新值
const countRef = useRef(0)
useEffect(() => {
  const timer = setInterval(() => {
    countRef.current += 1
  }, 1000)
  return () => clearInterval(timer)
}, [])
```

> ⚠️ **易错提醒**：依赖对象或数组时，每次引用都是新的，导致 Effect 死循环。用 useRef 存储或用 useMemo 稳定引用。

---

## 题目四：useCallback 与 useMemo 的使用场景

**面试场景**：「你在项目中用过 useMemo 或 useCallback 吗？什么情况下会用？」

真正的优化场景是**子组件被 React.memo 包裹，且 props 引用需要稳定**。

| Hook | 缓存内容 | 适用场景 |
|-----|---------|---------|
| useMemo | 缓存计算结果 | 复杂计算、派生数据 |
| useCallback | 缓存函数引用 | 传递给子组件的回调 |

```javascript
const ChildComponent = React.memo(({ onClick, data }) => (
  <button onClick={onClick}>{data}</button>
))

const Parent = () => {
  const [count, setCount] = useState(0)
  const handleClickMemo = useCallback(() => console.log('点击了'), [])
  const expensiveData = useMemo(() => count * 2, [count])
  return <ChildComponent onClick={handleClickMemo} data={expensiveData} />
}
```

> ⚠️ **易错提醒**：滥用 useMemo/useCallback，每次创建新缓存的开销可能更大。先 profile 再优化。

---

## 题目五：useRef 的深度解析

**面试场景**：「useRef 有什么用途？它和 createRef 有什么区别？」

useRef 有两个核心用途：**操作 DOM** 和**跨渲染周期存储可变值**。

```javascript
// 用途一：DOM 操作
const inputRef = useRef<HTMLInputElement>(null)
useEffect(() => { inputRef.current?.focus() }, [])

// 用途二：跨渲染周期存储值（不触发重渲染）
const timerRef = useRef(0)
const handleClick = () => {
  timerRef.current += 1
}
```

**createRef vs useRef**：createRef 每次渲染都创建新引用；useRef 在整个生命周期内返回同一引用。

**forwardRef + useImperativeHandle 组合**：

```javascript
const CustomInput = forwardRef<HTMLInputElement, Props>((props, ref) => {
  const innerRef = useRef<HTMLInputElement>(null)
  useImperativeHandle(ref, () => ({
    focus: () => innerRef.current?.focus(),
    reset: () => { innerRef.current.value = '' }
  }))
  return <input ref={innerRef} {...props} />
})
```

---

## 题目六：自定义 Hook 的设计模式

**面试场景**：「你在项目中封装过自定义 Hook 吗？用来解决什么问题？」

自定义 Hook 是「以 `use` 开头的函数」，核心价值是**逻辑复用**和**关注点分离**。

```javascript
const useFetch = (url: string) => {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)

  useEffect(() => {
    let cancelled = false
    fetch(url).then(res => res.json())
      .then(data => !cancelled && setData(data))
      .catch(err => !cancelled && setError(err))
    return () => { cancelled = true }
  }, [url])

  return { data, loading, error }
}
```

**三大规范**：命名必须以 `use` 开头、单一职责（一个 Hook 只做一件事）、返回格式一致。

---

## 题目七：useContext 性能陷阱与优化

**面试场景**：「用 Context 时有什么性能问题？怎么解决？」

Context 的值变化时，所有使用该 Context 的组件都会重渲染。

```javascript
// ❌ 问题：每次 render 都创建新对象
const AppContext = createContext()
const AppProvider = ({ children }) => (
  <AppContext.Provider value={{ state, dispatch }}>
    {children}
  </AppContext.Provider>
)

// ✅ 优化：用 useMemo 稳定 value 引用
const AppProvider = ({ children }) => {
  const [state, dispatch] = useReducer(reducer, initialState)
  const value = useMemo(() => ({ state, dispatch }), [state])
  return <AppContext.Provider value={value}>{children}</AppContext.Provider>
}
```

**进阶**：按功能拆分为多个小 Context，避免一个值变化导致整个子树重渲染。

---

## 题目八：useTransition 与 useDeferredValue

**面试场景**：「React 18 的并发模式有什么新 Hook？能解决什么问题？」

> ⚠️ **时效性标注**：以下均为 **React 18+** 特性。

| API | 作用 | 适用场景 |
|-----|------|---------|
| useTransition | 标记哪些更新是「可延迟的」 | 组件内部控制特定状态更新优先级 |
| useDeferredValue | 让某个值「延迟响应」变化 | 只延迟某值，不关心来源 |

```javascript
// useTransition 示例
const [isPending, startTransition] = useTransition()
const handleSearch = (e) => {
  setQuery(e.target.value)
  startTransition(() => setResults(heavySearch(e.target.value)))
}

// useDeferredValue 示例
const deferredQuery = useDeferredValue(query)
return <SlowList text={deferredQuery} />
```

**追问**：两者如何选择？
> 能改源码、用 startTransition 包裹延迟操作，选 useTransition。只能改消费端（第三方组件），用 useDeferredValue。

---

## 题目九：React 18 新 Hooks —— useSyncExternalStore 与 useId

**面试场景**：「用过 React 18 的新 Hook 吗？useSyncExternalStore 是什么？」

**useSyncExternalStore**：用于订阅外部 store（如 Redux、Zustand），是 SSR/同构应用的必备 API。三个参数：subscribe（订阅函数）、getSnapshot（获取状态）、getServerSnapshot（服务端渲染时的快照）。

**useId**：为 SSR 生成唯一 ID，解决水合不匹配问题：

```javascript
const id = useId() // 生成形如 :r1: 的唯一字符串
return <input id={id} aria-labelledby={`${id}-label`} />
```

---

## 题目十：Hooks 的规则与闭包陷阱

**面试场景**：「React 为什么要规定 Hooks 只能在顶层调用？条件语句里不能用吗？」

**两大核心规则**：只在顶层调用（不要在循环、条件中调用）、只在 React 函数中调用（组件或自定义 Hook 中）。

React 依赖调用顺序来确定每个 Hook 对应的 state。如果在条件语句中调用，某些渲染中 Hook 数量不同，React 就无法将 state 与正确的 Hook 关联。

---

## 题目十一：Vue3 Composition API 与 React Hooks 对比

**面试场景**：「Vue3 的 Composition API 和 React Hooks 有什么本质区别？」

| 对比维度 | React Hooks | Vue3 Composition API |
|---------|-------------|---------------------|
| 条件调用 | ❌ 不支持 | ✅ 支持 |
| 清理机制 | useEffect 中 return | onUnmounted |
| 响应式原理 | 手动依赖数组 | 自动追踪依赖（Proxy） |

**核心差异**：Vue3 的 setup 只执行一次，响应式系统不依赖调用顺序；React Hooks 在每次渲染时重建调用链，必须保证顺序稳定。

---

## 题目十二：React Compiler 的作用与局限性【中频】

> ⚠️ **时效性标注**：截至 2026 年仍处于**实验性阶段**。

React Compiler 能自动分析代码并插入 memoization，但依赖代码规范、复杂状态逻辑的手动优化仍不可替代。**面试应答**：Compiler 是辅助工具，理解 Hooks 原理才能写出「可被优化」的代码。

---

## 本章小结

| 核心知识点 | 关键要点 |
|-----------|---------|
| useState vs useReducer | 简单状态用 useState，复杂逻辑用 useReducer |
| useEffect vs useLayoutEffect | 前者异步不阻塞paint，后者同步阻塞paint |
| useEffect 依赖管理 | 依赖数组决定执行时机，清理函数防止内存泄漏 |
| useCallback/useMemo | 配合 React.memo 稳定 props 引用才有意义 |
| useRef | DOM 操作 + 跨渲染周期存储 + forwardRef 组合用法 |
| 自定义 Hook | 以 use 开头，遵循单一职责，返回值结构稳定 |
| React 18 新 Hooks | useTransition/useDeferredValue 区分、useSyncExternalStore、useId |
| Hooks 规则 | 顶层调用是铁律，闭包陷阱用函数式更新或 useRef 解决 |

---

## 面试检查清单

| 红线 | 错误做法 | 正确做法 |
|-----|---------|---------|
| ☐ 条件调用 | 在 if/循环中调用 useState | 统一在组件顶层调用 |
| ☐ 遗漏清理 | useEffect 不写 return 清理函数 | 订阅/定时器必须有清理 |
| ☐ 依赖错误 | 依赖对象/数组每次新引用 | 用 useRef 或 useMemo 稳定引用 |
| ☐ 滥用优化 | 任何地方都用 useMemo/useCallback | 仅在子组件被 memo 包裹时使用 |
| ☐ 闭包陈旧 | 定时器/回调中直接使用 state | 用函数式更新或 useRef |
| ☐ 执行时机混用 | useLayoutEffect 写异步逻辑 | 只在需要同步 DOM 操作时使用 |

---

## 行动建议

1. **手写验证**：尝试手写一个简化版 useState，感受链表存储机制
2. **场景练习**：找一个带搜索过滤的组件，实践 useTransition
3. **对比学习**：阅读 Vue3 Composition API 文档，对比理解两者设计差异

> **本章完结**

# 第15章 React 性能优化与状态管理

> 本章覆盖【高频】面试题约 9 道

React 需要开发者主动控制渲染时机，不像 Vue 有自动依赖追踪。掌握这套优化体系，是初中级工程师进阶的标志。

## 一、React.memo 与 PureComponent：避免重复渲染

### 【高频】React.memo 与 PureComponent 的区别

两者都实现「props 未变化则不渲染」，但原理不同：

| 对比维度 | React.memo | PureComponent |
|---------|------------|---------------|
| 适用组件 | 函数组件（⚠️ React 16.3+） | 类组件 |
| 比较方式 | 浅比较 props（可自定义） | 浅比较 props 和 state |
| 灵活性 | 支持第二参数自定义比较 | 不支持 |

```jsx
// React.memo（函数组件）
const UserCard = React.memo(({ name, age }) => {
  return <div>{name}: {age}</div>
}, (prevProps, nextProps) => prevProps.age === nextProps.age)

// PureComponent（类组件）
class UserCard extends React.PureComponent {
  render() {
    return <div>{this.props.name}: {this.props.age}</div>
  }
}
```

⚠️ **易错点：** 比较开销可能大于渲染收益时慎用；props 传入新对象/函数会致优化失效。

**面试追问：** 组件频繁更新、父组件每次都传新 props、或组件本身很轻量时不用。

## 二、useMemo 与 useCallback：缓存与稳定引用

### 【高频】useMemo 与 useCallback 的使用场景

useMemo 缓存计算结果，避免重复计算：

```javascript
const sortedList = useMemo(() => {
  return items.filter(item => item.active).sort((a, b) => a.name.localeCompare(b.name))
}, [items])
```

useCallback 稳定函数引用，常配合 React.memo 使用：

```javascript
const handleSubmit = useCallback((data) => {
  setList(prev => [...prev, data])
  onSave(data)
}, [onSave])
```

**常见依赖陷阱：**

- 依赖数组为空 → 函数永远不变，可能读取旧闭包数据
- 漏写依赖 → 闭包陷阱，函数内使用旧值
- 依赖对象/数组 → 每次渲染引用都变，缓存失效

⚠️ useCallback 只在函数作为其他 useCallback 依赖、传给 memo 子组件、或 useEffect 精确依赖时才值得使用。

## 三、列表渲染的 key 原则与性能陷阱

### 【高频】为什么不能用 index 作为 key

key 帮助 React 识别元素变化，引导 Diff 算法高效复用 DOM。

```jsx
// ✅ 正确
{items.map(item => <UserCard key={item.id} user={item} />)}
// ❌ 错误
{items.map((item, index) => <UserCard key={index} user={item} />)}
```

**index 作为 key 的性能灾难：** 当列表头部插入新项时：

```jsx
// 初始：[A, B]，index: 0→"0", 1→"1"
// 头部插入 C：[C, A, B]，index: 0→"0"→对应 A，1→"1"→对应 B
// 结果：A、B 组件被判定"没变"，C 被创建
```

实际 DOM 操作变成：保留 A 和 B DOM，在头部插入 C DOM——既没复用，又制造不必要更新。

**常见 key 误区：**

- 随机数作 key：每次渲染都变，等于禁用 Diff
- index + 列表重排：会导致组件状态错位（如输入框内容跑到错误行）

**面试追问：** 数据没有唯一 ID 时，用 `nanoid` 或 `crypto.randomUUID()`（⚠️ 现代浏览器均支持）。

## 四、React 性能优化的演进：从类组件到并发模式

| 阶段 | 优化手段 | 适用场景 |
|-----|---------|---------|
| 类组件时代 | shouldComponentUpdate、PureComponent | props/state 比较 |
| 函数组件时代 | React.memo、useMemo、useCallback | 引用稳定化 |
| 并发模式时代 | useTransition、useDeferredValue（⚠️ React 18+） | 耗时操作不阻塞 UI |

### 【高频】useTransition 与 useDeferredValue 的并发优化（⚠️ React 18+）

useTransition 将更新标记为非紧急：

```javascript
const [isPending, startTransition] = useTransition()

const handleSearch = (query) => {
  startTransition(() => setSearchResults(filterData(query)))
}
```

useDeferredValue 延迟更新次要值：

```javascript
const deferredQuery = useDeferredValue(query)
const results = useMemo(() => searchData(deferredQuery), [deferredQuery])
```

**适用场景：** 搜索框输入立即响应 UI、结果延迟显示。

**演进对比：** shouldComponentUpdate/PureComponent「阻止渲染」，React.memo「跳过不必要渲染」，useTransition/useDeferredValue「让必要渲染不阻塞用户操作」。

## 五、虚拟滚动：海量列表的终极优化

### 【高频】虚拟滚动的实现原理

只渲染可视区域列表项，大幅减少 DOM 节点：

```javascript
const rowHeight = 50
const visibleCount = Math.ceil(containerHeight / rowHeight)
const startIndex = Math.floor(scrollTop / rowHeight)
const visibleData = data.slice(startIndex, startIndex + visibleCount)
```

实际项目推荐 `react-window`，支持动态高度、无限滚动。

⚠️ 列表超过 100 条且每行高度可估算时效果显著。

## 六、状态管理方案对比

### 【高频】Context vs Zustand vs Redux vs Jotai

| 方案 | 适用场景 | 优点 | 缺点 |
|-----|---------|------|------|
| Context + useReducer | 简单全局状态 | 无需额外依赖 | 频繁更新卡顿 |
| Zustand | 中小型应用 | 极简 API、TS 友好、v5 支持 middleware | 生态不如 Redux |
| Redux Toolkit | 大型复杂应用 | 成熟生态、时间旅行调试、RTK Query | 样板代码仍较多 |
| Jotai | 原子化状态 | 细粒度更新、SSR 友好 | 学习曲线较陡 |
| React Query/SWR | 服务端状态 | 自动缓存、乐观更新 | 需配合本地状态 |

**Zustand v5 写法：**

```javascript
const useStore = create(
  devtools(
    persist(
      (set, get) => ({
        count: 0,
        increment: () => set(state => ({ count: state.count + 1 })),
      }),
      { name: 'app-storage' }
    )
  )
)
```

相比 Redux：无需 Provider 包裹、直接组件调用、slice 模式解耦、`immer` 开箱即用。

### React Query 与传统状态管理的区别

React Query 管理**服务端状态**（API 数据），传统状态管理管理**客户端状态**（UI 状态）：

| 维度 | 客户端状态 | 服务端状态 |
|-----|-----------|-----------|
| 数据来源 | 组件内部、用户操作 | 后端 API |
| 缓存策略 | 无自动缓存 | 内置缓存、后台刷新 |
| 一致性要求 | 强一致性 | 可接受短期不一致 |

```javascript
const { data, isLoading, refetch } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
  staleTime: 5 * 60 * 1000,
  refetchOnWindowFocus: true,
})
```

⚠️ 状态更新频繁、需要时间旅行调试、项目规模大时选 Redux；数据来自 API 且需要缓存时选 React Query。

## 本章小结

React 性能优化核心是「减少不必要的渲染」。从类组件的 shouldComponentUpdate/PureComponent，到函数组件的 React.memo 和 useMemo/useCallback，再到 React 18 的并发模式，手段不断演进。key 属性优化列表 Diff 效率，用 index 作 key 会导致头部插入性能灾难。状态管理按复杂度选择 Context → Zustand → Redux → Jotai，服务端数据交给 React Query/SWR。

| 核心知识点 | 关键要点 |
|-----------|---------|
| React.memo vs PureComponent | 函数 vs 类组件，memo 支持自定义比较 |
| useMemo/useCallback | 依赖陷阱：空数组、旧闭包、对象/数组引用变化 |
| key 原则 | 唯一 ID 优先，index 导致 Diff 性能灾难 |
| 并发优化 | useTransition/useDeferredValue（⚠️ React 18+） |
| 状态管理选型 | 客户端状态用 Zustand/Redux，服务端状态用 React Query |

**【高频】面试检查清单：**

1. ☐ React.memo 与 PureComponent 的核心区别与选型原则
2. ☐ useMemo 与 useCallback 的依赖陷阱与正确用法
3. ☐ 列表渲染 key 的最佳实践与 index 误区
4. ☐ useTransition 与 useDeferredValue 的并发优化场景
5. ☐ 虚拟滚动的实现原理与适用场景
6. ☐ Context、Zustand、Redux、Jotai 的对比与选型
7. ☐ React Query 与传统状态管理的区别
8. ☐ React 性能优化常见手段与演进脉络
9. ☐ React.memo 失效的常见原因

**行动建议：** 使用 React DevTools Profiler 观察频繁渲染的组件。尝试用 React.memo 包装纯展示组件，用 useMemo 缓存计算密集型派生数据，用虚拟滚动改造超过 200 条数据的列表。重点练习 key 使用场景——在列表头部插入元素时，观察 DOM 更新数量变化。

# 第16章 浏览器渲染机制与性能优化

# 第十六章：前端工程化：Webpack/Vite 构建原理与性能优化

> **本章覆盖【高频】面试题 6 道**

Webpack 和 Vite 的底层原理、Tree shaking、Code Splitting、热更新机制是 2026 年面试高频考点。

---

## 【高频】面试题 1：Webpack 的打包流程是怎样的？

Webpack 打包分为四阶段：

1. **初始化**：读取配置，初始化 Compiler
2. **编译**：从入口递归解析依赖，构建依赖图谱
3. **生成**：组装成多个 Chunk，生成资产
4. **输出**：写入文件系统，生成最终产物

> **易错点**：Tree shaking 和 Scope Hoisting 在生成阶段进行。

### 面试扩展

**Q：Webpack 如何构建依赖图？**

A：通过 Parser 将代码解析成 AST，遍历识别 `import`、`require` 等依赖声明，递归处理所有依赖模块。

---

## 【高频】面试题 2：Loader 和 Plugin 的区别是什么？

| 维度 | Loader | Plugin |
|------|--------|--------|
| **作用对象** | 单个文件内容 | 整个构建过程 |
| **执行时机** | 模块转换时 | 特定生命周期钩子 |
| **使用方式** | 链式调用 | 注册到 plugins 数组 |

```javascript
// Loader：转换单个文件
module.exports = function(source) {
  return `module.exports = ${JSON.stringify(marked(source))}`
}

// Plugin：在钩子上增强构建过程
class BuildInfoPlugin {
  apply(compiler) {
    compiler.hooks.done.tap('BuildInfoPlugin', () => {
      console.log('构建完成')
    })
  }
}
```

> **口诀**：Loader 专注"转换"，Plugin 专注"增强"。

### 面试扩展

**Q：如何开发自定义 Loader 或 Plugin？**

A：Loader 导出函数返回转换后的代码；Plugin 实现 `apply` 方法，通过 `compiler.hooks.xxx.tap` 注册钩子。

---

## 【高频】面试题 3：什么是 Tree shaking？如何让它生效？

Tree shaking 基于 **ES Module 静态特性**，在编译阶段分析导入导出，移除未使用的代码。

### 生效条件

```javascript
// ✅ 使用 ES Module + package.json 设置 sideEffects: false + production 模式

// ❌ 失效情况
import { cloneDeep } from 'lodash-es'  // 整体导入
// 正确做法：
import cloneDeep from 'lodash-es/clonedeep'
```

`sideEffects` 告诉 Webpack 哪些文件可以安全移除未使用导出。

### 面试扩展

**Q：为什么 CommonJS 无法 Tree shaking？**

A：`require` 是动态的，可在条件语句中调用；`import` 是静态的，Webpack 可准确分析依赖关系。

---

## 【高频】面试题 4：Webpack 的 Code Splitting 有几种实现方式？

Code Splitting 将代码按需加载，减少首屏体积。

### 三种方式

```javascript
// 方式一：配置多个入口
// 方式二：动态 import（最常用）
const Dashboard = () => import('./Dashboard.vue')

// 方式三：splitChunks 配置
optimization: { splitChunks: { chunks: 'all' } }
```

> `webpackPrefetch` 可实现预取，用户导航时直接读缓存。

### 面试扩展

**Q：splitChunks 如何让第三方库单独打包？**

A：配置 `cacheGroups.vendor`，将 `node_modules` 模块打包到 vendor chunk。

---

## 【高频】面试题 5：Vite 为什么比 Webpack 快？

| 维度 | Webpack | Vite |
|------|---------|------|
| 开发模式 | 全量打包，先构建再启动 | 按需编译 + native ESM |
| 依赖预构建 | 无 | esbuild 预构建 |
| 热更新 | 重新编译整个模块图 | 只更新变化模块 |
| 生产构建 | terser 压缩 | Rollup 打包 |

### Vite 工作原理

启动开发服务器 → 返回含 `<script type="module">` 的 HTML → 浏览器请求模块 → Vite 实时编译返回（esbuild）。

> **适用场景**：Vite 适合中小型项目；Webpack 适合大型复杂项目。

### 面试扩展

**Q：依赖预构建解决什么问题？**

A：1）CJS 转 ESM；2）合并模块减少 HTTP 请求；3）缓存加快二次启动。

---

## 【高频】面试题 6：如何优化构建性能？

```javascript
// Webpack 优化
module.exports = {
  optimization: {
    usedExports: true,
    splitChunks: { chunks: 'all' }
  },
  cache: { type: 'filesystem' }  // 持久化缓存，二次构建缩短 80%
}
```

### 通用优化策略

1. 减少 `resolve.extensions`，用 `resolve.alias` 简化路径
2. 开启持久化缓存
3. 用 `esbuild-loader` 替代 `babel-loader`（速度提升 10-100 倍）
4. 使用 `thread-loader` 多进程并行

### 面试扩展

**Q：Tree shaking 不生效的常见原因？**

A：使用 CommonJS、未设置 `sideEffects`、使用动态导入、配置低效 Source Map。

---

## 本章小结

**核心概念**：Webpack 四阶段打包流程、Loader 转换文件、Plugin 增强过程、Tree shaking 基于 ES Module 静态分析、Code Splitting 三种方式、Vite 按需编译原理。

**行动建议**：执行 `webpack --analyze` 查看产物结构、配置动态 import 改写路由、开启持久化缓存、用 `npx vite` 对比构建速度。

---

## 面试检查清单

- ☐ 能完整描述 Webpack 四阶段打包流程
- ☐ 能说清 Loader 和 Plugin 的区别
- ☐ 知道 Tree shaking 基于 ES Module 实现
- ☐ 能列举三种 Code Splitting 方式
- ☐ 知道 Vite 用 ESM 实现按需编译
- ☐ 能说出至少三种构建性能优化手段
- ☐ 知道 Webpack 5 持久化缓存配置

---

理解浏览器如何将代码变成页面是前端面试重点——DOM 树构建、回流重绘、GPU 加速、长任务优化等知识能帮助你从根本上解决页面卡顿问题。

# 第17章 缓存机制与存储方案

> **覆盖题目**：约 8 道高频面试题
> **学习目标**：掌握浏览器缓存策略完整链路，理解强缓存与协商缓存协作机制，熟悉前端本地存储技术适用场景。

---

## 一、强缓存：Cache-Control 与 Expires

强缓存命中时，浏览器**完全不发起请求**，直接使用本地资源。

### Cache-Control（现代主流）

通过 HTTP 响应头设置常用指令：

- **`max-age=31536000`**：资源有效期（秒），期间直接使用本地缓存
- **`no-cache`**：每次询问服务器确认是否最新
- **`no-store`**：彻底禁止缓存，登录态等敏感数据必须加
- **`private` vs `public`**：private 仅终端用户可缓存，public 允许任何节点缓存

### Expires（历史兼容）

```http
Expires: Wed, 21 Oct 2026 07:28:00 GMT
```

HTTP/1.0 方案，用时间戳标记过期。缺陷：依赖客户端时钟，精度差。**最佳实践**：`Cache-Control: max-age` 为主，`Expires` 仅作兼容兜底。

---

## 二、协商缓存：Last-Modified 与 Etag

强缓存失效后，浏览器携带标识向服务器确认资源能否继续使用——返回 304 继续用缓存，返回 200 使用新资源并更新缓存。

### Last-Modified + If-Modified-Since

服务器返回 `Last-Modified: 时间戳`，浏览器下次请求带上 `If-Modified-Since`。**两个坑**：精度只到秒级，1秒内多次修改无法区分；文件属性变化也会导致误判。

### Etag + If-None-Match

Etag 是服务器根据内容生成的唯一标识符（文件哈希或版本号）。比 Last-Modified 更精确，能捕获任何内容变化，代价是额外的服务端计算开销。

**实际项目通常两者配合**：Etag 为主、Last-Modified 为辅。

---

## 三、缓存完整生命周期

强缓存和协商缓存串联工作：

1. 检查强缓存（Cache-Control/Expires）
2. 命中 → 直接使用，**流程结束**
3. 未命中 → 发起请求，检查协商缓存（带 If-None-Match / If-Modified-Since）
4. 命中 → 返回 304，继续用本地缓存
5. 未命中 → 返回 200，使用新资源并更新本地缓存

**DevTools 观察**：状态 `304` 是协商缓存命中，`(from memory/disk cache)` 是强缓存命中。

---

## 四、前端本地存储全家桶

| 特性 | Cookie | localStorage | sessionStorage | IndexedDB |
|------|--------|--------------|----------------|-----------|
| **容量** | 约 4KB | 约 5MB | 约 5MB | 约 50MB+ |
| **生命周期** | 可设置过期 | 永久 | 标签页关闭清除 | 永久 |
| **与服务器交互** | 自动随请求发送 | 不参与 | 不参与 | 不参与 |
| **API 风格** | 字符串 | 键值对 | 键值对 | 异步类数据库 |
| **适用场景** | 会话识别、Token | 持久配置 | 临时表单数据 | 大数据、结构化存储 |

**Cookie 的不可替代性**：Cookie 自动携带在同源请求头中，适合登录态等必须与服务器交互的数据。但要警惕 XSS——永远不要存明文敏感信息。

**IndexedDB 实战场景**：离线笔记应用、大文件分片上传断点续传、游戏存档、用户行为数据本地聚合。

---

## 五、Service Worker 与离线缓存

Service Worker 拦截网络请求，实现精细化离线缓存策略，是 **PWA（渐进式 Web 应用）** 的核心技术。通过 `install` 事件预缓存静态资源，通过 `fetch` 事件决定走缓存还是网络：

```javascript
// 优先返回缓存，缓存没有再走网络
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(response => response || fetch(event.request))
  );
});
```

结合 Cache-Control 策略，可实现"缓存优先、网络其次"的离线体验。

---

## 本章小结

| 知识点 | 核心要点 |
|--------|----------|
| 强缓存 | Cache-Control max-age 为主，Expires 为兼容兜底 |
| 协商缓存 | Etag 为主、Last-Modified 降级，配合链路工作 |
| 存储方案 | Cookie 负责会话、Storage 负责持久/临时配置、IndexedDB 负责大数据 |
| 离线缓存 | Service Worker 拦截请求 + PWA 实现离线可用 |

**面试检查清单** ☐ 能画出缓存完整生命周期图并解释每一步；☐ 能对比 Cache-Control 各项指令含义；☐ 能说明强缓存和协商缓存的优先级关系；☐ 能根据业务场景推荐合适的存储方案。

**行动建议**：打开成熟项目的 DevTools Network 面板，查看 CSS/JS 资源的缓存命中情况，分析背后的策略设计。

# 第18章 跨域问题与前端安全

> **高频面试题 6 道**：同源策略、CORS 预检、XSS 防御、CSRF 防御、CSP 配置。

## 一、同源策略：浏览器安全的基石

**【高频】题目：什么是同源策略？它限制了什么操作？**

同源指协议、域名、端口三者完全相同，任一项不同即为**跨域**。同源策略限制一个源的文档或脚本**读写**另一个源的资源，这是浏览器的核心安全机制。

**注意**：同源策略只限制浏览器端，服务端之间的 HTTP 请求完全不受影响。跨域报错信息是 `Access-Control-Allow-Origin` missing。

---

## 二、跨域三大解决方案

### 2.1 JSONP：历史遗留方案

**【高频】题目：JSONP 的实现原理是什么？有什么局限？**

利用 script 标签不受同源限制的特性，服务端返回函数调用代码。

```javascript
function handleData(data) { console.log(data); }
const script = document.createElement('script');
script.src = 'https://api.example.com/data?callback=handleData';
document.body.appendChild(script);
```

服务端返回 `handleData({ name: '张三', age: 25 });`，前端接收到后自动执行回调。

**局限**：只支持 GET，无法获取状态码，存在注入风险。2019 年后新项目一律用 CORS。

### 2.2 CORS：官方标准方案

**【高频】题目：什么是 CORS 预检请求？为什么需要预检？**

CORS 通过 HTTP 响应头告知浏览器跨域请求是否合法，核心是 `Access-Control-*` 系列头部。

**预检请求判断**：

- GET/POST/HEAD 请求 + `application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain` → 简单请求，直接发
- 其他方法（PUT/DELETE/OPTIONS 等）或自定义请求头 → 非简单请求，先发 OPTIONS 预检

**简单请求**：浏览器自动携带 Origin 头，服务端返回 `Access-Control-Allow-Origin` 即可。

**非简单请求**：先发 OPTIONS 预检，服务端确认后再发真实请求。

**常见 CORS 响应头**：

| 头部 | 含义 | 注意 |
|------|------|------|
| `Access-Control-Allow-Origin` | 允许的源 | `*` 不能配合凭证使用 |
| `Access-Control-Allow-Credentials` | 是否允许携带 Cookie | 为 true 时 Origin 不能是 `*` |
| `Access-Control-Max-Age` | 预检缓存时间（秒） | 可减少预检开销 |

### 2.3 代理服务器：开发环境常用方案

**题目：开发环境如何解决跨域问题？生产环境呢？**

**开发环境**：前端请求同域地址（如 `/api`），由 devServer 代理转发。

```javascript
// vite.config.js
export default {
  server: {
    proxy: {
      '/api': {
        target: 'https://api.example.com',
        changeOrigin: true
      }
    }
  }
};
```

**生产环境**：Nginx 反向代理。

```nginx
location /api {
    proxy_pass https://api.example.com;
}
```

**注意**：代理方案只适用于开发环境，生产环境必须用 CORS。

---

## 三、XSS 与 CSRF：两大前端攻击

### 3.1 XSS：跨站脚本攻击

**【高频】题目：XSS 有哪几种类型？如何防御？**

XSS 核心是将恶意脚本注入页面执行，窃取 Cookie、监听输入、篡改页面。

**三种类型**：

- **存储型**：恶意代码存入数据库，所有访问者都会中招，危害最大
- **反射型**：恶意代码作为 URL 参数，用户点击即触发
- **DOM 型**：纯前端漏洞，不经过服务端

**防御手段**：

1. **输出转义**（主力）：HTML 特殊字符转义（`<` → `&lt;`、`>` → `&gt;`）
2. **CSP 内容安全策略**：限制脚本来源
3. **HTTP-only Cookie**：保护敏感 Cookie

**注意**：后端必须做输出转义。React/Vue 默认转义，但使用 `dangerouslySetInnerHTML` / `v-html` 时要小心。DOM 型 XSS 完全由前端代码负责防护。

### 3.2 CSRF：跨站请求伪造

**【高频】题目：CSRF 的攻击原理是什么？如何防御？**

攻击利用用户已登录的身份，诱导用户访问恶意页面，浏览器自动带上目标站点的 Cookie 发起请求，服务端无法区分是否为用户本人操作。

**防御手段**：

| 手段 | 原理 | 前端改动 |
|------|------|----------|
| **CSRF Token** | 随机 Token 验证请求来源 | 表单/请求中携带 |
| **SameSite Cookie** | 限制 Cookie 随跨域请求发送 | 服务端设置 |
| **验证码/密码确认** | 二次验证身份 | 需要，体验略差 |

**注意**：GET 请求不应有副作用（删除、转账等）。SameSite Cookie 三种值：`Strict` 完全禁止跨站携带，`Lax` 允许导航跳转时携带，`None` 不限制（需配合 Secure）。

---

## 四、CSP 安全头配置

**【高频】题目：什么是 CSP？如何配置？**

CSP 通过 HTTP 响应头指定页面允许加载的资源来源，从源头防止 XSS 注入。脚本不在白名单内，浏览器拒绝执行。

```http
Content-Security-Policy: 
  default-src 'self'; 
  script-src 'self' https://trusted.com; 
  img-src *; 
  report-uri https://report.example.com/csp-violation
```

**nonce 机制**：服务端每次生成随机 nonce，script 必须携带匹配值才能执行。动态脚本每次 nonce 不同，无法被预先植入。

**其他安全响应头**：
- `X-Frame-Options: DENY` — 防点击劫持
- `X-Content-Type-Options: nosniff` — 防 MIME 嗅探
- `Strict-Transport-Security` — 强制 HTTPS

**注意**：CSP 不能完全替代转义，只是纵深防御。初始配置建议用 `report-only` 模式观察。

---

## 本章小结

跨域本质是**浏览器的同源策略限制**。三种方案适用场景不同：JSONP 已过时只留作了解、CORS 是标准需要服务端配合、代理适合开发环境。CORS 预检机制是高频考点，务必讲清触发条件。安全核心是**防御 XSS 和 CSRF**——输出转义配合 CSP 防 XSS，CSRF Token + SameSite Cookie 防 CSRF。

---

## 面试检查清单：安全考点速记

- ☐ 同源策略三要素：协议、域名、端口
- ☐ JSONP 利用 script 标签不受同源限制，缺点是只支持 GET
- ☐ CORS 预检请求（OPTIONS）触发条件：非简单请求
- ☐ 凭证模式：`withCredentials: true` 时 Origin 不能是 `*`
- ☐ XSS 三类型：存储型、反射型、DOM 型
- ☐ XSS 防御：输出转义是主力，CSP 是纵深防御
- ☐ CSRF 利用用户已登录 Cookie，Token 验证是核心
- ☐ SameSite Cookie：Strict/Lax/None 三种模式
- ☐ CSP nonce 机制：每次请求生成随机数白名单

**下一步行动建议**：检查项目跨域配置，开发环境用代理，生产环境用 CORS；打开 Chrome DevTools Network 面板观察预检流程；确保所有输出点都做了 HTML 转义。

# 第19章 HTTP/HTTPS 与网络协议

**【高频】本章覆盖约8道网络协议面试题**

## 一、HTTP 请求方法与状态码

### 【高频】题目：HTTP 常见请求方法及区别

| 方法 | 语义 | 幂等 | 典型场景 |
|------|------|------|----------|
| GET | 获取资源 | ✅ | 查询列表、商品详情 |
| POST | 提交数据 | ❌ | 登录、注册、创建订单 |
| PUT | 完整替换 | ✅ | 更新用户信息（全量字段） |
| PATCH | 部分修改 | ❌ | 修改用户名 |
| DELETE | 删除资源 | ✅ | 删除商品 |

**易错提醒**：PUT 与 PATCH 最易混淆——PUT 全量替换，PATCH 打补丁。POST 不幂等，多次调用创建多条记录。

---

### 【高频】题目：HTTP 状态码分类与易混淆对比

| 分类 | 范围 | 含义 | 典型状态码 |
|------|------|------|----------|
| 1xx | 100-199 | 信息性 | 101 切换协议 |
| 2xx | 200-299 | 成功 | 200、201、204 |
| 3xx | 300-399 | 重定向 | 301、302、304 |
| 4xx | 400-499 | 客户端错误 | 400、401、403、404 |
| 5xx | 500-599 | 服务端错误 | 500、502、504 |

**易混淆对比**：

| | 301 | 302 | 307 |
|--|-----|-----|-----|
| 性质 | 永久 | 临时 | 临时 |
| SEO | 传递权重 | 不传递 | 不传递 |
| 方法保持 | ❌可能变GET | ❌可能变GET | ✅强制保持 |

| | 200 OK | 204 No Content | 304 Not Modified |
|--|--------|----------------|------------------|
| 响应体 | 有数据 | 无 | 无（用缓存） |

**易错提醒**：301/302 可能改请求方法（POST 变 GET），敏感操作用 307 或 308。

---

## 二、HTTP 版本演进

### 【高频】题目：HTTP/1.0、HTTP/1.1、HTTP/2、HTTP/3 的区别

**HTTP/1.0**（1996）：每次请求单独 TCP 连接。

**HTTP/1.1**（1999，主流）：默认持久连接，支持管道化；新增 PUT、DELETE、PATCH、OPTIONS；支持断点续传和缓存控制。

**HTTP/2**（2015）：多路复用解决应用层队头阻塞，二进制分帧更高效，HPACK 压缩 Header，支持服务端推送。但基于 TCP，仍受 TCP 层队头阻塞影响。

**HTTP/3**（2022+）：基于 QUIC/UDP，彻底解决 TCP 队头阻塞。支持 0-RTT/1-RTT 快速握手，网络切换不丢连接。

> ⚠️ **时效性说明**：截至 2025 年主流浏览器 HTTP/3 支持率超 90%，2026 年面试需掌握其与 HTTP/2 的性能对比。

### 【知识点延伸】QUIC 如何解决 TCP 的队头阻塞？

TCP 队头阻塞发生在传输层：某报文段丢失后，后续所有报文段必须等待重传成功才能交付上层。

QUIC 在用户态实现可靠传输，每个 Stream 独立管理序列号。丢包只影响对应 Stream，其他 Stream 继续传输，从根本上消除队头阻塞。

---

## 三、HTTPS 加密原理与 TLS 握手

### 【高频】题目：HTTPS 为什么安全？TLS 握手流程是什么？

HTTPS = HTTP + TLS/SSL，目标：防窃听、防篡改、防冒充。混合加密：非对称加密传输对称密钥，后续用对称密钥加密通信。

**TLS 1.3 握手流程（1-RTT）**：
1. Client Hello：发送 TLS 版本、加密套件、随机数 A
2. Server Hello：选择加密套件、发送证书、随机数 B
3. 证书验证：验证 CA 签名、有效期、域名
4. 预主密钥交换：用公钥加密「预主密钥」发给服务端
5. 生成会话密钥：随机数 A + B + 预主密钥 → 对称密钥
6. 切换加密通信

> ⚠️ **时效性说明**：TLS 1.3 握手从 2-RTT 简化为 1-RTT；废弃 RSA 密钥交换；前向保密成为默认。2026 年主流网站已全面支持。

**追问**：为什么需要证书？防止中间人攻击——攻击者伪造公钥窃取通信。证书由 CA 机构签发，证明「公钥属于某某网站」。

---

## 四、TCP 三次握手与四次挥手

### 【高频】题目：TCP 为什么需要三次握手？四次挥手为什么要等待？

**三次握手**：验证双方收发能力都正常。
1. 客户端发 SYN=1, seq=x
2. 服务端回 SYN+ACK, ack=x+1
3. 客户端再发 ACK, ack=y+1

**四次挥手**：确保数据完整关闭。
1. 客户端发 FIN（请求关闭写方向）
2. 服务端回 ACK（可能还有数据要发）
3. 服务端发 FIN（双向都发完了）
4. 客户端回 ACK

**TIME_WAIT 等待 2MSL**：确保服务端收到最后的 ACK（未收到会重发 FIN）；让旧连接报文消散，防止影响新连接。

**易错提醒**：TCP 全双工，每方向需单独关闭，所以 FIN 和 ACK 分开发送。

---

## 五、TCP 与 UDP 的区别

### 【高频】题目：TCP 和 UDP 各有什么特点？如何选择？

| 对比项 | TCP | UDP |
|--------|-----|-----|
| 连接方式 | 面向连接 | 无连接 |
| 可靠性 | 可靠（确认、重传、排序） | 不可靠 |
| 有序性 | 保证按序到达 | 不保证顺序 |
| 速度 | 较慢 | 快（头部仅8字节） |
| 流量/拥塞控制 | 有 | 无 |

**适用场景**：
- **TCP**：网页访问、文件传输、邮件、数据库
- **UDP**：视频直播、实时游戏、DNS 查询、QUIC

**追问**：QUIC 基于 UDP 但在用户态实现可靠性。丢包率高的网络中，TCP 拥塞控制反而更稳定。

### 【知识点延伸】选择场景速记

- **DNS 用 UDP**：请求小（几十字节）、频率高，TCP 握手延迟从 1ms 飙升到几十毫秒。丢包后重试即可，换服务器也不影响。
- **视频用 UDP**：实时性优先，偶尔丢帧可接受（画面轻微卡顿），等重传导致停滞体验更差。
- **网页用 TCP**：数据必须完整无误——少一个字符可能导致 JS 报错、页面崩溃。

---

## 本章小结

| 知识点 | 核心要点 |
|--------|----------|
| 状态码 | 2xx成功、3xx重定向、4xx客户端错、5xx服务端错 |
| HTTP版本 | 1.1主流、2.0多路复用、3.0基于QUIC/UDP |
| HTTPS | 混合加密 + TLS 1.3 握手 + 证书验证 |
| TCP三次握手 | 验证双方收发能力都正常 |
| TCP vs UDP | TCP可靠有序、UDP快但不保证 |

**☐ 面试检查清单**：

- ☐ 能说清 GET/POST/PUT/PATCH/DELETE 的语义区别和幂等性
- ☐ 能对比 301/302/307 的 SEO 影响和方法保持特性
- ☐ 能解释 HTTP/1.1 持久连接 vs HTTP/2 多路复用的区别
- ☐ 能说明 HTTP/3 基于 QUIC 彻底解决队头阻塞的原因
- ☐ 能描述 TLS 1.3 的 1-RTT 握手流程
- ☐ 能解释 DNS 用 UDP、视频用 UDP、网页用 TCP 的原因

**行动建议**：
1. 用 Chrome DevTools 观察 HTTPS 握手耗时
2. 用 curl 测试不同 HTTP 方法的响应差异
3. 对比 HTTP/2 和 HTTP/3 在弱网环境下的表现

# 第20章 前端工程化与构建工具

Webpack 和 Vite 是大厂必考点。

## 模块化发展历程

前端模块化经历三个阶段：**CommonJS** 是 Node.js 运行时同步加载，模块是对象；**ES Module** 是 ES6+ 编译时静态分析，导出是绑定（只读），天然支持 Tree-shaking；**AMD** 通过 `define()` 异步加载，多见于早期浏览器项目。

> **面试题 1：CommonJS 和 ES Module 有什么区别？**
>
> 加载时机不同：CommonJS 运行时同步加载，ES Module 编译时静态分析。导出形式不同：CommonJS 值拷贝，ES Module 值引用。Tree-shaking 只支持 ES Module。循环依赖处理也不同。

## loader 与 plugin

**loader** 处理单个文件的格式转换，是链式管道调用；**plugin** 介入构建生命周期钩子，可自定义行为、干预输出产物。

> **面试题 2：Webpack 的 loader 和 plugin 有什么区别？**
>
> loader 作用于单个文件，plugin 作用于整个构建过程。loader 是管道式链式调用，plugin 是生命周期扩展。

## Tree-shaking 原理与失效排查

Tree-shaking 依赖 ES Module 静态结构，编译时分析依赖图谱，标记未使用导出并剔除。CSS 导入、全局变量修改等副作用会使 Tree-shaking 失效。

```javascript
// ✅ 可 Tree-shaking
import { join } from 'lodash';
// ❌ 不可 Tree-shaking（整库导入）
import _ from 'lodash';
```

五步法排查失效：① 确认 ES Module → ② 检查 `sideEffects` 配置 → ③ 确认 terser 开启压缩 → ④ 动态 `import()` 不参与 Tree-shaking → ⑤ import 方式差异（按名导入 vs 默认导入整库）。

> **面试题 3：为什么没有 Tree-shaking 效果？**
>
> 标准排查路径：确认 ES Module → 检查 `sideEffects` → 确认 terser 压缩 → 检查 import 方式。

## Webpack 5 Module Federation

**Module Federation** 允许子应用按需加载其他子应用模块，共享依赖避免重复打包，是微前端架构的核心能力。相比 qiankun 沙箱隔离方案，Module Federation 更适合多团队协作独立部署的场景。

> **面试题 4：Module Federation 解决了什么问题？**
>
> 解决微前端场景下各子应用重复打包相同依赖的问题，实现运行时共享模块。

## Vite Dev Server 原理

Vite 开发时基于原生 ESM，浏览器直接请求模块，Server 按需编译，启动秒级。生产环境用 Rollup 打包。Webpack 需全量打包，冷启动 10-30 秒；Vite 适合中小型项目和 Vue 3/React 新项目。

| 维度 | Webpack | Vite |
|------|---------|------|
| 冷启动 | 全量打包 | 原生 ESM，秒级 |
| HMR | 模块越多越慢 | 变化模块即时生效 |
| 生产 | 自研打包 | Rollup |
| 适用 | 大型、微前端 | 中小型、新项目 |

> **面试题 5：Webpack 和 Vite 怎么选？**
>
> 选 Vite：中小型项目、Vue 3/React 新项目、追求开发体验。选 Webpack：超大型项目、多页面、需要 Module Federation。

## CI/CD 流水线

完整流程：**push → Webhook 触发 → 安装依赖 → ESLint 检查 → 单元测试 → 构建打包 → 推送 CDN → 通知**。

**排错场景**：CI 成功但页面白屏？排查链路：检查构建产物完整性 → CDN 刷新延迟 → 控制台报错定位 → 确认 `publicPath` 配置正确。

## 本章小结

**核心检查清单**：能说清 CommonJS 和 ES Module 区别 ✅；能解释 Tree-shaking 原理和失效原因 ✅；能对比 Webpack 和 Vite 适用场景 ✅；了解 Module Federation 微前端应用 ✅。

**行动建议**：动手从 0 配置 Webpack loader/plugin，用 Vite 对比冷启动速度，用 `webpack-bundle-analyzer` 分析自己的 bundle。面试时说"自己搭过完整 CI/CD 流水线、写过自定义 plugin"，比背概念更有说服力。

# 第21章 常见数组操作手写题

> **本章覆盖 5 道高频手写题**：数组去重、扁平化与去重、深拷贝与循环引用、手写 map/filter/reduce、洗牌算法 Fisher-Yates。

## 浅拷贝与深拷贝的区别

浅拷贝只复制第一层属性，嵌套对象共享引用；深拷贝递归复制所有层级，对象完全独立。

```javascript
// 浅拷贝：嵌套对象共享引用
const shallow = [...[1, [2, 3]]];
shallow[1][0] = 99; // 原数组被修改

// 深拷贝：对象完全独立
const deep = JSON.parse(JSON.stringify([1, [2, 3]]));
```

## 数组去重

**核心思路**：Set（O(n)）优先，filter+indexOf（O(n²)）次之，复杂类型用对象键值对。

```javascript
// Set去重（推荐）
const unique = arr => [...new Set(arr)];

// filter+indexOf（保持顺序但O(n²)）
const unique2 = arr => arr.filter((item, i) => arr.indexOf(item) === i);

// 对象键值对（适合对象数组）
const unique3 = arr => {
  const seen = {};
  return arr.filter(item => {
    const key = typeof item + JSON.stringify(item);
    return seen.hasOwnProperty(key) ? false : (seen[key] = true);
  });
};
```

| 方案 | 时间复杂度 | 空间复杂度 | 适用场景 |
|------|-----------|-----------|---------|
| Set | O(n) | O(n) | 基本类型去重 |
| filter + indexOf | O(n²) | O(1) | 需保持原顺序 |
| 对象键值对 | O(n) | O(n) | 对象数组去重 |

> **易错点**：filter+indexOf 是嵌套循环，大数据量时容易超时。面试优先说 Set 解法，再补充 filter 方案展示思维广度。

**面试追问**：

- 对象如何去重？自定义序列化 key
- Set 和 Map 去重有什么区别？Set 只存值，Map 可存额外信息
- 稀疏数组怎么去重？空位不是 undefined，需用 `in` 或 `hasOwnProperty` 判断

## 扁平化与去重

```javascript
const flat = arr => arr.reduce((res, item) =>
  res.concat(Array.isArray(item) ? flat(item) : item), []);

const flatUnique = arr => [...new Set(flat(arr))];
```

> `flat(Infinity)` 能一行搞定，但手写更能体现递归思维。

**面试追问**：递归和 reduce 实现的性能对比？本质相同，reduce 版本更函数式；递归需注意栈溢出风险。

## 深拷贝与循环引用

**核心思路**：递归拷贝，WeakMap 处理循环引用，单独处理 Date/RegExp/Map/Set 等特殊类型。

```javascript
const deepClone = (obj, hash = new WeakMap()) => {
  if (obj === null || typeof obj !== 'object') return obj;
  if (hash.has(obj)) return hash.get(obj); // 处理循环引用
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  if (obj instanceof Map) {
    const clone = new Map();
    hash.set(obj, clone);
    obj.forEach((v, k) => clone.set(deepClone(k, hash), deepClone(v, hash)));
    return clone;
  }
  if (obj instanceof Set) {
    const clone = new Set();
    hash.set(obj, clone);
    obj.forEach(v => clone.add(deepClone(v, hash)));
    return clone;
  }
  const clone = Array.isArray(obj) ? [] : {};
  hash.set(obj, clone);
  for (const key in obj) {
    if (Object.hasOwn(obj, key)) clone[key] = deepClone(obj[key], hash);
  }
  return clone;
};
```

> **易错点**：`JSON.parse(JSON.stringify())` 无法处理函数、undefined、Symbol、正则、循环引用。WeakMap 的 key 是弱引用，不阻止垃圾回收，适合处理循环引用。

**面试追问**：WeakMap 和普通 Map 有什么区别？WeakMap 的 key 只能是对象，且是弱引用，GC 时自动清除。

## 手写 map / filter / reduce

回调函数签名：`fn(item, index, array)`

```javascript
Array.prototype.myMap = function(fn, thisArg) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (i in this) result.push(fn.call(thisArg, this[i], i, this));
    else result.push(undefined); // map 保持长度一致
  }
  return result;
};

Array.prototype.myFilter = function(fn, thisArg) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (fn.call(thisArg, this[i], i, this)) result.push(this[i]);
  }
  return result;
};

Array.prototype.myReduce = function(fn, initValue) {
  let result = initValue, startIndex = 0;
  if (initValue === undefined) { result = this[0]; startIndex = 1; }
  for (let i = startIndex; i < this.length; i++) {
    result = fn(result, this[i], i, this);
  }
  return result;
};
```

> 手写时用 `for` 而不是 `forEach`——手写实现需要自己控制遍历逻辑。`thisArg` 用于绑定回调函数的 `this` 上下文。

**面试追问**：reduce 和 map 的区别？map 返回新数组，reduce 聚合成单个值。

## 洗牌算法 Fisher-Yates

```javascript
const shuffle = arr => {
  const result = [...arr];
  for (let i = result.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [result[i], result[j]] = [result[j], result[i]];
  }
  return result;
};
```

时间复杂度 O(n)，空间复杂度 O(1)。**从后往前遍历**确保每个位置只交换一次。

> **常见错误**：用 `Math.random() * length` 会导致概率不均等。Fisher-Yates 保证每个元素出现在每个位置的概率均为 1/n。

**应用场景**：抽奖系统、随机点名、卡牌游戏、随机推荐算法。

---

## 面试变形：组合考察

真实面试常把多道题揉进一道。常见组合：

- 数组扁平化 + 去重 + 深拷贝
- reduce 实现 map/filter
- 洗牌 + 抽奖（洗牌后取前 k 个为中奖名单）

应对策略是**拆解**——把复合问题拆成单个小问题依次解决。

---

## 本章小结

| 题型 | 推荐解法 | 关键点 |
|------|---------|--------|
| 去重 | Set | O(n) 时间复杂度 |
| 扁平化 | reduce + 递归 | concat 合并新数组 |
| 深拷贝 | WeakMap 处理循环引用 | hasOwnProperty 防继承 |
| 高阶函数 | 手写 myMap/filter/reduce | 回调参数 (item, index, array) |
| 洗牌 | Fisher-Yates | 从后往前交换 |

---

## 面试检查清单

☐ **是否需要考虑空数组？** —— 返回 `[]` 还是报错

☐ **是否需要考虑稀疏数组？** —— 空位是否视为 `undefined`，map 需保持长度

☐ **是否包含 `NaN` 或 `null`？** —— Set 能正确处理 NaN，`indexOf` 不能

☐ **是否要求保持原顺序？** —— 影响选择哪种算法

☐ **是否有性能要求？** —— 大数据量场景优先选 O(n) 解法

---

## 速查模板

```javascript
// 去重
const unique = arr => [...new Set(arr)];

// 扁平化
const flat = arr => arr.reduce((res, item) =>
  res.concat(Array.isArray(item) ? flat(item) : item), []);

// 深拷贝（含 Map/Set/循环引用）
const deepClone = (obj, hash = new WeakMap()) => {
  if (obj === null || typeof obj !== 'object') return obj;
  if (hash.has(obj)) return hash.get(obj);
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj.source, obj.flags);
  if (obj instanceof Map) {
    const clone = new Map();
    hash.set(obj, clone);
    obj.forEach((v, k) => clone.set(deepClone(k, hash), deepClone(v, hash)));
    return clone;
  }
  if (obj instanceof Set) {
    const clone = new Set();
    hash.set(obj, clone);
    obj.forEach(v => clone.add(deepClone(v, hash)));
    return clone;
  }
  const clone = Array.isArray(obj) ? [] : {};
  hash.set(obj, clone);
  for (const key in obj) {
    if (Object.hasOwn(obj, key)) clone[key] = deepClone(obj[key], hash);
  }
  return clone;
};

// reduce
Array.prototype.myReduce = function(fn, initValue) {
  let result = initValue, startIndex = 0;
  if (initValue === undefined) { result = this[0]; startIndex = 1; }
  for (let i = startIndex; i < this.length; i++) {
    result = fn(result, this[i], i, this);
  }
  return result;
};

// 洗牌
const shuffle = arr => {
  const result = [...arr];
  for (let i = result.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [result[i], result[j]] = [result[j], result[i]];
  }
  return result;
};
```

# 第22章 Promise 与异步手写题

> **本章覆盖**：5 道 Promise 与异步手写题  
> **学习目标**：手写实现 Promise/A+ 规范核心逻辑、理解链式调用原理、掌握 async/await 与并发控制技巧

Promise 是现代 JavaScript 异步编程的基础，手写 Promise/A+ 规范是高频考点。

---

## Promise/A+ 规范核心要点

1. **状态机机制**：三种状态 `pending`（初始）、`fulfilled`（成功）、`rejected`（失败），状态一旦确定不可改变。
2. **then 方法契约**：`.then(onFulfilled, onRejected)` 返回新 Promise，回调在微任务队列中异步执行。
3. **链式调用与值穿透**：`.then()` 永远返回新 Promise。`onFulfilled` 默认 `v => v`，`onRejected` 默认 `err => { throw err }`。

---

## 题目一：Promise 基本结构与状态管理

### 题目描述

实现一个简化版 MyPromise，支持状态管理和基本的 resolve/reject 功能。

### 核心思路

用 `state` 记录状态，`value` 和 `reason` 存储结果。**关键**：resolve/reject 必须先判断状态是否为 pending，确保不可逆。

### 代码示例

```javascript
class MyPromise {
  constructor(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    this.callbacks = [];

    const resolve = (value) => {
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
        this.callbacks.forEach(cb => cb.onFulfilled(value));
      }
    };

    const reject = (reason) => {
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.reason = reason;
        this.callbacks.forEach(cb => cb.onRejected(reason));
      }
    };

    try { executor(resolve, reject); }
    catch (e) { reject(e); }
  }
}
```

### 易错提醒

`if (this.state === 'pending')` 判断是状态不可逆的核心保证，面试务必加上。

### 面试扩展

- **追问**：executor 执行报错时为什么能自动 reject？
  - 答：构造函数用 try-catch 包裹 executor，任何同步错误都会触发 reject。

---

## 题目二：then、catch、finally 的实现

### 题目描述

为 MyPromise 实现 then、catch、finally 方法，支持链式调用。

### 核心思路

每次 `then` 返回新 Promise，回调函数需要默认值处理实现「值穿透」，异步执行用 setTimeout 模拟微任务。

### 代码示例

```javascript
then(onFulfilled, onRejected) {
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v;
  onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err; };

  const promise = new MyPromise((resolve, reject) => {
    if (this.state === 'fulfilled') {
      setTimeout(() => {
        try { resolve(onFulfilled(this.value)); }
        catch (e) { reject(e); }
      }, 0);
    } else if (this.state === 'rejected') {
      setTimeout(() => {
        try { resolve(onRejected(this.reason)); }
        catch (e) { reject(e); }
      }, 0);
    } else {
      this.callbacks.push({
        onFulfilled: () => { try { resolve(onFulfilled(this.value)); } catch (e) { reject(e); } },
        onRejected: () => { try { resolve(onRejected(this.reason)); } catch (e) { reject(e); } }
      });
    }
  });
  return promise;
}

catch(onRejected) { return this.then(null, onRejected); }

finally(onFinally) {
  return this.then(
    value => { onFinally(); return value; },
    err => { onFinally(); throw err; }
  );
}
```

### 易错提醒

- then 返回新 Promise 而非 this，这是链式调用的基础
- `onRejected` 默认 `err => { throw err }` 让错误在链中「冒泡」，直到被 catch 捕获

---

## 题目三：Promise.all 与 Promise.race

### 题目描述

实现 Promise.all、Promise.race、Promise.allSettled 静态方法。

### 核心思路

Promise.all 用计数器跟踪完成数量，全部成功才 resolve，任一失败立即 reject。Promise.race 返回第一个 settled 的结果。Promise.allSettled 收集全部结果不受失败影响。

### 代码示例

```javascript
static all(promises) {
  return new MyPromise((resolve, reject) => {
    const results = new Array(promises.length);
    let completed = 0;
    if (promises.length === 0) return resolve([]);

    promises.forEach((promise, index) => {
      MyPromise.resolve(promise).then(
        value => { results[index] = value; if (++completed === promises.length) resolve(results); },
        reason => reject(reason)
      );
    });
  });
}

static race(promises) {
  return new MyPromise((resolve, reject) => {
    promises.forEach(promise => MyPromise.resolve(promise).then(resolve, reject));
  });
}

static allSettled(promises) {
  return new MyPromise(resolve => {
    const results = new Array(promises.length);
    let completed = 0;
    if (promises.length === 0) return resolve([]);

    promises.forEach((promise, index) => {
      MyPromise.resolve(promise).then(
        value => { results[index] = { status: 'fulfilled', value }; },
        reason => { results[index] = { status: 'rejected', reason }; }
      ).finally(() => { if (++completed === promises.length) resolve(results); });
    });
  });
}
```

### 方法对比

| 方法 | 场景 | 行为特点 |
|------|------|----------|
| Promise.all | 全部成功才有用 | 任一失败立即 reject |
| Promise.allSettled | 收集全部结果 | 不因失败中断 |
| Promise.race | 快速响应首个结果 | 返回第一个 settled |

---

## 题目四：async/await 原理与错误处理

### 题目描述

实现 asyncToGenerator 函数，理解 async/await 本质是 Generator 语法糖。

### 核心思路

`await` 驱动 Generator 执行，通过递归调用 `.next()` 或 `.throw()` 处理返回值或错误。

### 代码示例

```javascript
function asyncToGenerator(generatorFn) {
  return function(...args) {
    const gen = generatorFn.apply(this, args);
    return new Promise((resolve, reject) => {
      function step(key, arg) {
        try {
          const info = gen[key](arg);
          const { value, done } = info;
          if (done) resolve(value);
          else Promise.resolve(value).then(val => step('next', val), err => step('throw', err));
        } catch (e) { reject(e); }
      }
      step('next');
    });
  };
}
```

### 易错提醒

async 函数自动返回 Promise：return 普通值等价于 `Promise.resolve(value)`，抛出异常等价于 `Promise.reject(error)`。

---

## 题目五：并发控制与限制异步并发数

### 题目描述

实现一个函数，限制同时运行的异步任务数量，适用于批量请求场景。

### 核心思路

维护运行中的任务计数器 `running`，初始启动 n 个任务，每个任务完成时减少计数器并启动下一个（「信号量控制」）。

### 代码示例

```javascript
function limitConcurrency(tasks, limit) {
  return new Promise((resolve, reject) => {
    let running = 0, index = 0;
    const results = new Array(tasks.length);

    function runTask() {
      if (index >= tasks.length) {
        if (running === 0) resolve(results);
        return;
      }
      const i = index++;
      running++;
      tasks[i]().then(
        val => { results[i] = val; running--; runTask(); },
        err => reject(err)
      );
    }

    for (let i = 0; i < Math.min(limit, tasks.length); i++) runTask();
  });
}
```

### 易错提醒

当批量请求 100 个接口且服务器有限流时，必须用并发控制。

---

## 本章小结

手写 Promise 考察三点：**状态不可逆**、**链式调用返回新 Promise**、**异步回调的收集与执行**。Promise.all/race 考并发处理，async/await 考 Generator 理解，并发控制考工程思维。

### 行动建议

1. 动手实现完整版 MyPromise（含 then、catch、finally、all、race、allSettled）
2. 理解 async/await 与 Generator 的关系
3. 练习用 Promise 实现并发控制

---

## 面试检查清单

| 要点 | 关键词 |
|------|--------|
| 三种状态 | pending → fulfilled / rejected，不可逆 |
| then 必须异步 | 微任务或 setTimeout 模拟 |
| 返回新 Promise | 链式调用的基础 |
| 值穿透 | 默认 `v => v` |
| 错误冒泡 | 默认 `err => { throw err }` |
| Promise.all | 全成功才 resolve，任一失败立即 reject |
| Promise.race | 第一个 settled 的结果 |
| Promise.allSettled | 全部完成，不因失败中断 |
| async/await | Generator 语法糖，async 永远返回 Promise |
| 并发控制 | 信号量模式，running 计数器 |

# 第23章 数据结构与算法应用

# 第 23 章 数据结构与算法应用

## 一、算法解题四步法

**第一步——理解题意**：确认输入范围、边界条件、返回值要求，面试时复述一遍避免理解偏差。

**第二步——暴力尝试**：先想最直接的办法，O(n²) 也可以，暴力解能帮你发现规律。

**第三步——找规律优化**：观察有没有重复计算（动态规划）、有没有单调性（双指针/滑动窗口）、能不能分割（分治）。这是面试最看重的部分。

**第四步——写代码验证**：注意边界处理、空值判断、循环条件，写完用小样本手动跑一遍。

> 口诀：**暴力起步，找规律优化，写代码验证，最后检查复杂度**

## 二、栈、队列应用

**【题目】用两个栈实现队列。**

栈 FILO，队列 FIFO。stackIn 入队，stackOut 出队——stackOut 为空时才将 stackIn 数据倒入，保证均摊 O(1)。

```javascript
class MyQueue {
  constructor() {
    this.stackIn = [];
    this.stackOut = [];
  }
  push(x) { this.stackIn.push(x); }
  pop() {
    if (this.stackOut.length === 0) {
      while (this.stackIn.length) this.stackOut.push(this.stackIn.pop());
    }
    return this.stackOut.pop();
  }
  peek() {
    if (this.stackOut.length === 0) {
      while (this.stackIn.length) this.stackOut.push(this.stackIn.pop());
    }
    return this.stackOut[this.stackOut.length - 1];
  }
  empty() { return this.stackIn.length === 0 && this.stackOut.length === 0; }
}
```

> 易错点：只在 stackOut 为空时倒数据，每 pop 一次倒一次会退化为 O(n)。

## 三、链表反转与双指针

**【题目】反转单向链表，并用快慢指针判断链表是否有环。**

反转需要三个指针：prev、cur、next。快指针一次两步，慢指针一次一步，相遇即有环。

```javascript
// 反转链表
function reverseList(head) {
  let prev = null, cur = head;
  while (cur) {
    const next = cur.next;
    cur.next = prev;
    prev = cur;
    cur = next;
  }
  return prev;
}

// 快慢指针判环并找入口
function detectCycle(head) {
  let fast = head, slow = head;
  while (fast && fast.next) {
    fast = fast.next.next;
    slow = slow.next;
    if (fast === slow) {
      fast = head;
      while (fast !== slow) { fast = fast.next; slow = slow.next; }
      return fast;
    }
  }
  return null;
}
```

> 易错点：反转前先保存 next；快慢指针条件用 `while (fast && fast.next)`。

## 四、二叉树遍历与最近公共祖先

**【题目】二叉树前序、中序、后序、层序遍历（递归），求两节点最近公共祖先。**

前序"根-左-右"、中序"左-根-右"、后序"左右-根"、层序用队列。LCA：BST 利用值的大小关系；普通二叉树用递归——若 root 是 p 或 q 直接返回，否则递归左右子树找结果。

```javascript
function preOrder(root, res = []) {
  if (!root) return res;
  res.push(root.val);
  preOrder(root.left, res);
  preOrder(root.right, res);
  return res;
}

function levelOrder(root) {
  if (!root) return [];
  const queue = [root], res = [];
  while (queue.length) {
    const level = [], size = queue.length;
    for (let i = 0; i < size; i++) {
      const node = queue.shift();
      level.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    res.push(level);
  }
  return res;
}

function lowestCommonAncestor(root, p, q) {
  if (!root || root === p || root === q) return root;
  const left = lowestCommonAncestor(root.left, p, q);
  const right = lowestCommonAncestor(root.right, p, q);
  if (!left) return right;
  if (!right) return left;
  return root;
}
```

> 易错点：层序遍历每轮开始前记录当前队列长度 `size`。

## 五、LRU 缓存

**【题目】用双向链表和哈希表实现 LRU，要求 get/put 都是 O(1)。**

哈希表提供 O(1) 查找，双向链表维护访问顺序——最近使用在头部，最久未用在尾部。

```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }
  get(key) {
    if (!this.cache.has(key)) return -1;
    const val = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, val);
    return val;
  }
  put(key, value) {
    if (this.cache.has(key)) this.cache.delete(key);
    this.cache.set(key, value);
    if (this.cache.size > this.capacity) {
      this.cache.delete(this.cache.keys().next().value);
    }
  }
}
```

> 易错点：`get` 后必须 `delete` 再 `set`，仅 `set` 不会改变 Map 内部顺序。

## 六、排序算法

**【题目】手写快速排序，并说明与归并排序的区别。**

快排原地分割——选基准分割数组，递归处理两边。归并"先分后合"——递归拆分到单个元素，再逐步合并排序。快排原地分割 O(1) 空间，但有序数组可能退化 O(n²)；归并稳定 O(n log n)，但需 O(n) 额外空间。

```javascript
function quickSort(arr, left = 0, right = arr.length - 1) {
  if (left >= right) return arr;
  const pivot = arr[Math.floor((left + right) / 2)];
  let i = left, j = right;
  while (i <= j) {
    while (arr[i] < pivot) i++;
    while (arr[j] > pivot) j--;
    if (i <= j) { [arr[i], arr[j]] = [arr[j], arr[i]]; i++; j--; }
  }
  quickSort(arr, left, j);
  quickSort(arr, i, right);
  return arr;
}

function mergeSort(arr) {
  if (arr.length <= 1) return arr;
  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));
  return merge(left, right);
}

function merge(left, right) {
  const res = [];
  let i = 0, j = 0;
  while (i < left.length && j < right.length) res.push(left[i] <= right[j] ? left[i++] : right[j++]);
  return res.concat(left.slice(i)).concat(right.slice(j));
}
```

> 易错点：分区循环方向不要搞反；递归边界参数 `left, j` 和 `i, right` 不要写错。

## 七、复杂度速查

| 复杂度 | 场景 | 口诀 |
|---|---|---|
| O(1) | 哈希表查找 | 一步到位 |
| O(log n) | 二分查找 | 砍一半 |
| O(n) | 单链表遍历 | 一个一个来 |
| O(n log n) | 快排、归并平均 | 分而治之 |
| O(n²) | 冒泡排序 | 双重遍历 |
| O(2ⁿ) | 全排列 | 指数爆炸 |

**判断方法**：看几层循环 → 单层 O(n)；循环减半 → O(log n)；递归树深度 × 每层工作量 → 乘积即复杂度。

## 本章小结

- **栈/队列**：FILO/FIFO，两栈实现队列只在出队栈空时倒数据，均摊 O(1)
- **链表**：反转三变量 prev/cur/next；快慢指针万能钥匙（判环、找中点、删倒数第 N 个）
- **二叉树**：四种遍历要都能写；LCA 分 BST 和普通二叉树两种解法
- **LRU**：Map 实现最简洁，delete 再 set 移末尾，超过容量删第一个 key
- **排序**：快排原地分割最坏 O(n²)，归并稳定 O(n log n) 但需额外空间

# 第24章 前端系统设计思路

## 开篇引入

前端系统设计面试没有标准答案，考的是**思考过程、权衡取舍、沟通能力**。

## 系统设计基本流程

核心框架：**需求澄清→高层设计→核心方案→权衡取舍→可优化点**。以大文件上传为例，先追问文件大小、是否断点续传、秒传需求，再画架构草图明确职责划分。面试官不在意方案多完美，关键能否**把问题想清楚、把 trade-off 说明白**。

## 场景一：大文件上传与断点续传

分片上传策略，每片 2-5MB，并发控制限制 5 个分片。Web Worker 计算 hash 实现秒传，本地记录已上传分片索引实现断点续传。

## 场景二：前端权限系统

RBAC 模型：**用户→角色→权限**三级映射。前端做第一道防线，动态路由过滤菜单，hasPermission 指令控制按钮；后端 API 做最终校验。**前端权限的本质是防君子不防小人**。

## 场景三：图片懒加载方案

IntersectionObserver 监听图片进入视口，提前 200px 触发加载，占位图防止空白。旧浏览器降级 scroll 事件，大量图片注意内存管理避免 OOM。

## 场景四：前端监控系统

SDK 控制在 10KB 以内，异步加载不阻塞主线程。JS 异常 100% 上报，PerformanceObserver 监听 LCP/FID/CLS，sendBeacon 保证页面关闭时数据送达。采样率 0.1% 即可，基准：LCP < 2.5s、FID < 100ms、CLS < 0.1。

## 场景五：消息通知系统

WebSocket 长连接 + 心跳保活，消息 ack 确认可靠性，IndexedDB 持久化离线消息。指数退避重连，降级到轮询保障基本可用。

## 场景六：低代码编辑器架构

**组件树（Schema）→渲染引擎→画布** 三层分离。拖拽仅更新 transform，drop 才更新 Schema。Immer 不可变数据结构管理操作栈，支持 100 步 Undo/Redo。

## 场景七：SSR 渲染方案选型

| 方案 | 上手难度 | 适用场景 |
|------|---------|---------|
| Next.js | 低 | 快速 MVP |
| Remix | 中 | 复杂数据流 |
| RSC | 高 | 极致性能 |

## 小结

核心框架：**需求澄清→高层设计→核心方案→权衡取舍→可优化点**。心法：**MVP 先跑通再优化**。

**关键要点**：

- 大文件上传：分片 + 并发控制 + hash 秒传
- 权限系统：RBAC 模型，前端防君子、后端保安全
- 懒加载：IntersectionObserver，降级 scroll 兜底
- 监控系统：SDK 轻量、异步加载、采样控制
- 消息推送：WebSocket 心跳 + ack 确认 + 离线存储

## 面试自测问题

- ☐ 能画出系统架构拓扑图，标注数据流向
- ☐ 能说出每个方案的 trade-off 和选择理由
- ☐ 能扩展到异常场景（断网、重试、超时）
- ☐ 能提出优化方向（用户增长、数据量增长）

学完系统设计思维，下一章我们来学习**项目经验与场景题应对**——用 STAR 法则讲好项目故事。

# 第25章 项目经验与场景题应对

项目经验是面试官考察技术能力、问题解决和协作沟通的关键环节。通过 STAR 法则把项目经历组织成有条理的故事，能让你在有限时间内讲得既专业又打动人心。

## STAR法则：结构化描述项目

- **S（Situation 情境）**：项目背景、团队规模、你的角色。两句话点明即可。
- **T（Task 任务）**：核心挑战或目标。说清"面临什么问题"或"承担什么责任"。
- **A（Action 行动）**：按时间顺序或重要性分点描述，用"首先、其次、最后"让逻辑清晰，每个行动一句话说清。
- **R（Result 结果）**：用数据量化成果，突出个人贡献。要写"我的贡献"而非"我们团队"，数据要提前演练。

> **易错点**：T 是目标、A 是手段，两者不能颠倒。R 量化要准确（如"减少60%维护成本"），不能说"项目顺利上线"。

> **完整范例**：我主导了电商中台权限重构（S：8人中台团队负责用户模块）。老系统权限逻辑耦合，每次需求变更涉及7个文件改动（T：目标解耦并引入RBAC模型）。我设计三张核心表、编写双写脚本实现平滑迁移、分批次灰度切换（A）。最终维护成本降低60%，开发周期从5天缩至2天（R）。

## 技术选型理由的表达方式

回答框架：**问题背景 → 备选方案对比 → 决策理由 → 最终效果**

> "项目约20页面、5人团队都有 Vue 基础。对比 React（学习成本高）和 Angular（过于重），选 Vue 因学习曲线平缓、文档中文友好、生态完善。最终上线提前3天，维护效率提升明显。"

关键：选"最适合团队"而非"最好"的方案，理由要有数据或事实支撑。

## 常见追问应对技巧

**Q：遇到的最大技术挑战？**

> "移动端首屏加载慢，Lighthouse 定位到主包 2.3MB。路由按需分割、图片转 WebP、合并高频接口请求，最终首屏从 4.5 秒降至 1.9 秒，用户流失率下降 12%。"

**Q：和后端/产品经理意见分歧？**

明确分歧类型：方案问题调整方案，认知偏差用数据说服，无法达成一致则找上级协调。不要情绪化，不要背后抱怨。

**Q：怎么推动技术方案落地？**

> "先在小范围验证，产出数据后在周会分享，得到认可再推广全组。关键是先跑通、再推广，让数据说话。"

## 本章小结

- **STAR法则**：情境→任务→行动→成果，每点用数据说话，R 要写个人贡献
- **回答框架**：结论先行→逻辑展开→数据支撑→主动复盘
- **行动建议**：选2-3个核心项目，各准备60秒概述版和3分钟详细版，对着镜子或录音复述三遍以上，数字提前演练准确

---

更多内容请访问：[https://tutor.lao-zhao.com/](https://tutor.lao-zhao.com/)
