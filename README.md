[合集 - 每日一面(9)](https://github.com)

[1.【每日一面】获取文字的真实宽度09-24](https://github.com/keepsmart/p/19109247)[2.【每日一面】React Hooks闭包陷阱09-26](https://github.com/keepsmart/p/19113792)[3.【每日一面】任意 DOM 元素吸顶09-23](https://github.com/keepsmart/p/19106732)[4.【每日一面】setTimeout 延时为 0 的情况09-29](https://github.com/keepsmart/p/19118235):[西部世界加速](https://xibujiasu.com)[5.【每日一面】盒子模型10-09](https://github.com/keepsmart/p/19132053)[6.【每日一面】手写防抖函数10-27](https://github.com/keepsmart/p/19168241)[7.【每日一面】async/await 的原理10-28](https://github.com/keepsmart/p/19172342)[8.【每日一面】对 Promise.race 的理解10-29](https://github.com/keepsmart/p/19173755)

9.【每日一面】你怎么理解 Proxy 的10-30

收起

# 基础问答

问：Proxy 是什么？怎么使用的？

答：Proxy 是用于创建 “对象代理” 的构造函数，它能封装目标对象（target），并通过 “拦截器对象（handler）” 自定义目标对象的基础操作（如属性读取、赋值），实现对对象行为的 “劫持”，手写使用方式。

```
// 语法：new Proxy(target, handler)
// 参数：target-目标对象，handler-拦截器对象
// 返回值：代理实例proxy

const target = { name: '前端面试', age: 2 };
// 定义拦截器对象
const handler = {
  // 拦截“读取属性”操作：参数为target（目标对象）、prop（属性名）、receiver（代理实例）
  get(target, prop, receiver) {
    console.log(`触发get拦截：读取属性${prop}`);
    // 执行原始读取操作（通过Reflect确保this指向正确）
    return Reflect.get(target, prop, receiver);
  },

  // 拦截“赋值属性”操作：参数为target、prop、value（新值）、receiver
  set(target, prop, value, receiver) {
    console.log(`触发set拦截：给属性${prop}赋值${value}`);
    // 自定义逻辑：校验age属性必须为数字
    if (prop === 'age' && typeof value !== 'number') {
      throw new Error('age属性必须是数字');
    }
    // 执行原始赋值操作
    return Reflect.set(target, prop, value, receiver);
  }
};

// 创建代理实例
const proxy = new Proxy(target, handler);

// 操作代理实例，触发拦截
console.log(proxy.name); // 输出：触发get拦截：读取属性name → 前端面试
proxy.age = 3; // 输出：触发set拦截：给属性age赋值3 → 成功
proxy.age = '3'; // 输出：触发set拦截：给属性age赋值3 → 抛出错误：age属性必须是数字

// 直接操作目标对象，不会触发拦截
console.log(target.name); // 输出：前端面试（无拦截日志）
target.age = '4'; // 无拦截日志，且不会触发age的类型校验
```

# 扩展延伸

Proxy API 的核心三要素是：“目标对象（target）”“拦截器对象（handler）”“代理实例（proxy）”。

* **目标对象（target）**：被代理的原始对象（可以是对象、数组、函数，甚至另一个 Proxy 实例）；
* **拦截器（handler）**：包含 “拦截方法” 的对象，每个拦截方法对应一种目标对象的基础操作（如get拦截属性读取，set拦截属性赋值）；
* **代理实例（proxy）**：通过new Proxy(target, handler) 创建的代理对象，所有对目标对象的操作需通过代理实例完成，才能触发拦截器。也就是说，直接操作目标对象，就不会走代理。

Proxy 是一种非侵入性的 API，他不会修改目标对象本身的结构或方法，所有拦截逻辑都封装在拦截器中，实现 “代理行为” 与 “目标对象” 的解耦，相对 `Object.defineProperty` 更加灵活。

拦截器方法除了基础的 get/set 方法，需要注意一些特殊的拦截方法（has/deleteProperty/apply/construct）：

| 拦截方法 | 作用 | 关键参数 | 适用场景 |
| --- | --- | --- | --- |
| get | 拦截属性读取（含 obj.prop、obj [prop]） | target, prop, receiver | 数据劫持（如响应式）、默认值设置 |
| set | 拦截属性赋值 | target, prop, value, receiver | 数据校验、值格式化 |
| has | 拦截 in 运算符（如prop in proxy） | target, prop | 权限控制（隐藏某些属性不被检测） |
| deleteProperty | 拦截 delete 操作（如delete proxy.prop） | target, prop | 禁止删除关键属性 |
| apply | 拦截函数调用（仅当 target 是函数时） | target, thisArg, args | 函数参数校验、调用日志记录 |
| construct | 拦截 new 操作（仅当 target 是构造函数时） | target, args, newTarget | 构造函数参数校验、实例计数 |

如果你使用 Vue3 框架，需要知道的时 Vue3 的响应式设计就是基于 Proxy 的，这是一个简化版的响应式 API 设计：

```
// 存储当前活跃的副作用函数（如组件渲染函数）
let activeEffect = null;

// 依赖映射表：target → { prop → [effect1, effect2,...] }
const targetMap = new WeakMap();

// 1. 依赖收集函数：将副作用函数与target、prop关联
function track(target, prop) {
  if (!activeEffect) return; // 无活跃副作用，不收集
  // 确保target在targetMap中存在映射
  let depsMap = targetMap.get(target);
  if (!depsMap) {
    depsMap = new Map();
    targetMap.set(target, depsMap);
  }
  // 确保prop在depsMap中存在副作用数组
  let deps = depsMap.get(prop);
  if (!deps) {
    deps = new Set(); // 用Set避免重复副作用
    depsMap.set(prop, deps);
  }
  // 添加当前副作用函数
  deps.add(activeEffect);
}

// 2. 依赖触发函数：执行target、prop对应的所有副作用
function trigger(target, prop) {
  const depsMap = targetMap.get(target);
  if (!depsMap) return;
  const deps = depsMap.get(prop);
  if (deps) {
    deps.forEach(effect => effect()); // 执行所有副作用
  }
}

// 3. 响应式函数：创建Proxy代理，实现依赖收集与触发
function reactive(target) {
  return new Proxy(target, {
    get(target, prop, receiver) {
      const value = Reflect.get(target, prop, receiver);
      track(target, prop); // 读取时收集依赖
      // 若value是对象，递归创建响应式（深度响应）
      if (typeof value === 'object' && value !== null) {
        return reactive(value);
      }
      return value;
    },
    set(target, prop, value, receiver) {
      const oldValue = Reflect.get(target, prop, receiver);
      const success = Reflect.set(target, prop, value, receiver);
      if (success && oldValue !== value) {
        trigger(target, prop); // 赋值时触发依赖
      }
      return success;
    }
  });
}

// 4. 副作用函数注册：执行fn并收集其依赖
function effect(fn) {
  activeEffect = fn;
  fn(); // 执行fn，触发get拦截，收集依赖
  activeEffect = null; // 重置，避免后续误收集
}

// 测试响应式
const data = reactive({ count: 0 });
// 注册副作用函数（模拟组件渲染）
effect(() => {
  console.log(`视图更新：count = ${data.count}`);
});

// 修改数据，触发副作用（视图更新）
data.count = 1; // 输出：视图更新：count = 1
data.count = 2; // 输出：视图更新：count = 2
```

# 面试追问

1. Proxy 和 Object.defineProperty 都能实现数据劫持，为什么 Vue3 放弃 Object.defineProperty 改用 Proxy？两者的核心差异是什么？

| 对比维度 | Object.defineProperty | Proxy |
| --- | --- | --- |
| 劫持范围 | 仅能劫持 “对象的单个属性”（需遍历属性逐个定义） | 直接劫持 “整个对象”（无需遍历属性） |
| 数组支持 | 无法劫持数组的原生方法（如 push、splice），需重写数组原型 | 能拦截数组的所有操作（包括索引赋值、原生方法调用） |
| 嵌套对象处理 | 需递归遍历所有嵌套对象，手动为每个属性定义劫持 | 可在 get 拦截中递归创建代理（按需劫持，性能更优） |
| 性能 | 初始化时需遍历所有属性，嵌套层级深时性能差 | 懒加载式劫持（访问嵌套对象时才创建代理），初始化性能更优 |

2. 用 Proxy 拦截对象属性赋值时，若目标对象是冻结对象（Object.freeze），set 拦截器还能生效吗？为什么？
   set 拦截器会触发，但最终赋值会失败，Object.freeze (target) 会让目标对象的属性变为 “不可写、不可配置”，但不会阻止 Proxy 拦截器的触发（拦截器是对操作的劫持，而非直接修改属性）。
3. 若用 Proxy 代理一个频繁修改的大型对象（如包含 1000 个属性的列表），会有性能问题吗？如何优化？
   会有性能问题，需要从两方面考虑：1. 若拦截器逻辑复杂（如每次 get/set 都执行大量校验、日志记录），频繁操作时会累积性能损耗；2. 对嵌套层级极深的大型对象，若初始化时递归创建代理（而非按需劫持），会导致初始化耗时过长。
   优化方案：1. 简化拦截器逻辑：将非必要操作（如详细日志）改为条件触发（如开发环境才执行），避免每次拦截都执行冗余代码；2. 按需劫持（懒加载）：仅在访问嵌套对象时，才为其创建 Proxy（如 Vue3 的响应式实现），避免初始化时递归遍历所有属性；3. 跳过无意义拦截：对不需要劫持的属性（如只读属性、常量），在拦截器中直接返回原始值，不执行额外逻辑；4. 使用 WeakMap 缓存代理实例：避免对同一目标对象重复创建 Proxy，减少内存占用与初始化开销。
