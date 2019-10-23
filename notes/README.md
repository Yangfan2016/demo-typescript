# TypeScript
> 文档：https://github.com/zhongsp/TypeScript


## 基本类型

- boolean
- number
- string
- 数组 T[]
- 元组 [T,U]
- 枚举 enum
- any
- 空值 void 与 any 相反，表示没有任何类型
- null
- undefined
- never 表示永远不存在的类型，如抛出异常
- object

## 泛型
> 在像c#和Java这样的语言中，创建 **可重用组件** 的工具箱中的一个主要工具是泛型，也就是说，能够创建可以在多种类型上工作的组件，而不是单个类型。这允许用户使用这些组件并使用自己的类型

### 泛型例子

- 函数返回值和输入值的类型相同
    - 不使用泛型
    ```ts
    function clone(a:any):any{
        return a;
    }
    ```
    - 使用泛型
    ```ts
    function clone<T>(a:T):T{
        return a;
    }
    
    let r1=clone<string>("haha"); //  r1:string
    let r2=clone<number>(1); // r2:number
    // 可以不明确传入类型，编译器会自动推断
    let r3=clone(true); // r3:boolean
    ```


### 泛型变量
> 把泛型当做变量作为类型的一部分使用

```ts
function logger<T>(args:Array<T>):Array<T>{
    console.log(args.length);
    return args;
}
```
也可以如下这么写：
```ts
function logger<T>(args:T[]):T[]{
    console.log(args.length);
    return args;
}
```

### 泛型类型

- 泛型函数类型

```ts
function clone<T>(arg: T): T {
    return arg;
}

let f1: <T>(arg: T) => T = clone;
// 也可以使用带有调用签名的对象字面量定义
let f2: { <T>(arg: T): T } = clone;

// 泛型接口
interface GenericFn{
    <T>(arg:T):T
}

interface GenericFn2<T>{
    (arg:T):T
}



let f3: GenericFn = clone;
let f3: GenericFn2<string> = clone;

```

### 泛型类

```ts
class Queue<T> {
  private data: T[] = [];
  push = (item: T) => this.data.push(item);
  pop = (): T | undefined => this.data.shift();
}

// 简单的使用
const queue = new Queue<number>();
queue.push(0);
queue.push("1"); // Err: 接受的始类型应为 number
```

### 泛型约束
- `extends` 关键字

```ts
interface ILength {
    length: number;
}

function logger<T extends ILength>(n: T) {
    return n.length;
}
```
- 约束类型参数

```ts
function getProp<T, K extends keyof T>(obj: T, prop: K) {
    return obj[prop];
}

let o = {
    name: "aaa",
    age: 111,
}

getProp(o,"name");
getProp(o,"age");
getProp(o,"b"); // Err 对象 "o" 上不存在属性 "b"
```
- 使用类类型

```ts
function createInstance<T>(Ctor: { new(): T }):T {
    return new Ctor;
}
```

## 枚举

### 数字枚举
- 默认从 0 开始递增
```ts
enum Direction {
  Up,    // 0
  Right, // 1
  Down,  // 2
  Left   // 3
}
```
- 自定义递增初始值
```ts
enum ServerCode {
  BadRequest = 400,   // 400
  NoAuth,             // 401
  BanAccess = 403,    // 403
  NotFound,           // 404
  ServerError = 500   // 500
}
```


### 字符串枚举

```ts
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT",
}
```

### 异构枚举

> 混入数字和字符串成员的枚举

```ts
enum BooleanLikeHeterogeneousEnum {
    No = 0,
    Yes = "YES",
}
```

### 运行时枚举
> 枚举是运行时真正存在的对象

```ts
enum E {
    X,
    Y,
    Z
}

function f(obj: { X: number }) {
    return obj.X;
}

f(E)
```


### 编译时枚举

```ts

enum LogLevel {
    ERROR,
    WARN,
    INFO,
    DEBUG
}

type LogLevelStrings = keyof typeof LogLevel;

let status: LogLevelStrings = "ERROR";
status="666"; // Err: "666" 不能赋值给 "ERROR" | "WARN" | "INFO" | "DEBUG"

```

### `const` 枚举
> 常量枚举不允许包含计算成员（只允许使用常量枚举表达式）

```ts
const enum Enum {
    A = 1,
    B = 2 * A, // ok 常量枚举表达式
    C = Math.random(), // Err: 计算成员
    D = (() => 1)() // Err: 计算成员
}
```

### 外部枚举 ?
> 在正常的枚举里，没有初始化方法的成员被当成常量成员。 对于非常量的外部枚举而言，没有初始化方法时被当做需要经过计算的

```ts
declare enum Enum {
    A = 1,
    B,
    C = 2
}
```

## 高级类型

### 交叉类型
> 同时拥有所有类型的全部成员

```ts
interface IA {
  name: string;
}

interface IB {
  age: number;
}

let d: IA & IB = {
  name: "",
  age: 1
};

```

### 联合类型
> 只能是所有类型之一

```ts
function printSth(input: string | number) {
  return input;
}

printSth(1);
printSth("");

interface IC {
  name: string;
  c: number;
}
```

> 如果值是联合类型，只能访问所有类型的公有部分


```ts
interface IC {
  name: string;
  c: number;
}

interface ID {
  name: string;
  d: string;
}

function f3(): IC | ID {
  return {
    name: "",
    c: 1
  };
}

let c3 = f3();
c3.name;
c3.d; // Err: 联合类型，只能访问所有类型里公有的部分

```

### 类型守卫与类型区分
> 使用联合类型的值时，需要类型区分出明确的类型

- 用户定义类型守卫  
    - 多次使用类型断言 `as`

    ```ts
    
    let pet = getSmallPet();
    
    if ((pet as Fish).swim) {
        (pet as Fish).swim();
    } else if ((pet as Bird).fly) {
        (pet as Bird).fly();
    }
    
    ```

- 使用类型判定   
    - 使用类型谓词 `is`
    
    ```ts
    function isFish(pet:Fish|Bird):pet is Fish {
        return (pet as Fish).swim!==void 0;
    }
    
    ```
    - 使用 `in` 操作符
    
    ```ts
    function move(pet:Fish|Bird) {
        if ("swim" in pet) {
            pet.swim();
        } else {
            pet.fly();
        }
    }
    ```
- `typeof` 类型守卫

```ts
function padLeft(m: string | number) {
    if (typeof m === "number") {
        return m + 1;
    } else {
        return m.trim();
    }
}
```

- `instanceof` 类型守卫

```ts
function move2(pet: Fish | Bird) {
  if (pet instanceof Fish) {
    return pet.swim();
  } else {
    return pet.fly();
  }
}
```

### 可以为 `null` 的类型
> 默认情况下，`null` 和 `undefined` 可以赋值给任何类型，开启 "--strictNullChecks" 标记解决此错误


- 可选参数和可选属性
> 使用了 "--strictNullChecks" 标记，可选参数会被自动加上 "|undefined"

```ts
interface IA{
    // name:string|undefined
    name?:string;
    age:number
}
```

- 类型守卫和类型断言
> 由于使用联合类型，可能会包含 `null` 类型，你需要使用类型守卫去除它

- 显示判断

```ts
function f1(s:string|null){
    if (s===null) return;
    return s+"haha";
}
```

- 短路运算符

```ts
function f1(s:string|null){
    let char=s||"";
    return s+"haha";
}
```

- 添加语法后缀 `!`，去除 `null` 和 `undefined`
> 编辑器无法去除嵌套函数的 `null` 和 `undefined`，所以需要添加语法后缀 `!`，来去除 `null` 和 `undefined`
```ts
function f11(s: string | null) {
  s = s || "";
  function f12() {
    return s!.toString();
  }
  return f12();
}
```


### 类型别名
```ts
// 基础类型
type NewString = string;
// 函数
type TypeOfFn = (arg: string) => string;
// 泛型
type GenericObj<T> = {
  value: T;
};
type Tree<T> = {
  value: string;
  left: Tree<T>;
  right: Tree<T>;
} | null;

```

- 类型别名 vs 接口

    - 类型别名不可重复声明，而接口可以
    ```ts
    type A={
        name: string;
    };
    // Err: 标识 "A" 重复
    type A={
        age: string;
    };
    
    
    interface IA {
      name: string;
    }
    // ok
    interface IA {
      age: number;
    }
    
    let a: IA = {
      name: "",
      age: 1
    };
    
    ```

### 字符串字面量类型
```ts
type Easing = "ease-in" | "ease-out";
let status3: Easing = "ease-in";
status3="a"; // Err
```


- 字符串字面量类型还可以用于区分函数重载
```ts
function createElement(tagName: "img"): HTMLImageElement;
function createElement(tagName: "input"): HTMLInputElement;
// ... more overloads ...
function createElement(tagName: string): Element {
  // ... code goes here ...
  if (tagName === "img") {
    return new Image();
  }
  if (tagName === "input") {
    return document.createElement("input");
  }
  return document.createElement("div");
}

createElement("input");
createElement("img");
```


### 数字字面量类型

```ts
function sayCode(a: 1 | 2 | 3) {
  return a + 1;
}

sayCode(1);
sayCode(5); // Err
```

### 索引类型
> 使用索引类型，可以检查动态属性名

`keyof`，为 **索引类型查询操作符**  
`T[K]`，为 **索引访问操作符**
```ts
let o3 = {
  a: 1,
  b: "2",
  c: true
};

function f8<T, K extends keyof T>(obj: T, prop: K): T[K] {
  return obj[prop];
}

f8(o3, "b");
f8(o3, "a");
f8(o3, "d"); // Err: 不存在属性 "d"
```
- 索引类型和字符串签名
    - 如果你有一个带有字符串签名的类型 T ，那么 `keyof T` 的结果会是 `number|string`，因为 js 中，`object[1]` 和 `object["1"]` 都可以访问属性
    
    ```ts
    interface Dictionary {
        [key: string]: any;
    }
    
    let keys: keyof Dictionary; // keys:string | number
    ```
    - 如果是数字签名的话， 那么 `keyof T` 的结果会是 `number`

    ```ts
    interface Dictionary {
        [key: number]: any;
    }
    let keys: keyof Dictionary; // keys:number
    ```

### 映射类型

- 映射类型
```ts
type Readonly<U> = {
  readonly [T in keyof U]: U[T];
};
type Partial<U> = {
  [T in keyof U]?: U[T];
};

type ReadonlyPerson = Readonly<IPerson>;
type PartialPerson = Partial<IPerson>;
```

- 映射成员 (使用交叉类型)
```ts
// 这样使用 使用交叉类型
type PartialWithNewMember<T> = {
  [P in keyof T]?: T[P];
} & { newMember: boolean };

// 不要这样使用这会报错！
type PartialWithNewMember<T> = {
   [P in keyof T]?: T[P];
   newMember: boolean;
}
```

- 有条件类型  
`T extends U ? X : Y`
翻译： 如果 `T` 能赋值给 `U`，那么类型是 `X`，否则是 `Y`


## 使用工具类型

### Partial<T> 将 T 类型的属性变为可选属性

```ts
type Partial<T>={
    [P in keyof T]?:T[P]
}

```

### Required<T> 将 T 类型的属性变为必选属性

```ts
type Required<T>={
    // "-?" 代表去除可选 对应的还有 "+?" ，作用与 "-?" 相反，是把属性变为可选项
    [P in keyof T]-?:T[P]
}
```

### Readonly<T> 将 T 类型的属性变为只读属性

```ts
type Readonly<T>={
    readonly [P in keyof T]:T[P]
}
```

### Record<K,T> 将 K 类型的属性映射到 T 类型上

```ts
type Record<K extends keyof any,T>={
    [P in K]:T
}
```

### Exclude<T,U> 将 T 类型上的 U 类型的属性剔除 （求差集？）
```ts
type Exclude<T,U>=T extends U?never:T;
```

### Extract<T,U> 提取 T 类型上的 U 类型属性，作为一个新的类型 （求交集？）
```ts
type Extract<T,U>=T extends U?T:never;
```

### Pick<T,K> 从 T 类型中挑选 K 类型属性
```ts
type Pick<T,K extends keyof T>={
    [P in K]:T[P]
}
```

### Omit<T,K> 从 T 类型中剔除 K 类型的属性
```ts
type Omit<T,K extends keyof any>=Pick<T,Exclude<keyof T,K>>
```

### NonNullable<T> 从 T 类型中剔除 null 和 undefined 类型
```ts
type NonNullable<T>=T extends null|undefined? never: T;
```

### ReturnType<T> 返回函数类型 T 的返回值类型
```ts
type ReturnType<T extends (...args:any)=>any>=T extends (...args:any)=> infer R ? R : any;
```

### InstanceType<T> 返回构造函数类型 T 的实例类型
```ts
type InstanceType<T extends new (...args:any):any> = T extends new (...args:any):infer R ? R : any;
```

### ThisType<T> 作为上下文 this 类型的一个标记，需要开启 "--noImplictThis"
```ts
interface ThisType<T>{
    
}
```

## 模块和命名空间

### 模块 （外部模块）
> ts 沿用 es6 的模块概念

- 导出

StringValidator.ts
```ts
export interface StringValidator {
    isSafe:boolena;
}
```
- 导入

test.ts
```ts
import { StringValidator } from "./StringValidator";

class Validator implements StringValidator {
   isSafe:boolean = true; 
}

```
- 默认导出

StringValidator.ts
```ts
export default {
    version: "1.1.1"
}
```

- 导入其他 js 库 (要想描述非 TypeScript 编写的类库的类型，我们需要声明类库所暴露出的 API )

    - 外部模块
 
    utils.d.ts
    ```ts
    declare module "url" {
        export function parse() {
            // ...
        }
    }
    
    ```
    
    test.ts
    
    ```ts
    /// <reference path="utils.d.ts" />
    import { parse } from "url";
    
    ```
    
    - 外部模块简写
    > 简写模块里所有导出的类型将是 any
    
    utils.d.ts
    ```ts
    declare module "url";
    
    ```
    
    test.ts
    ```ts
    /// <reference path="utils.d.ts" />
    import { parse } from "url";
    // parse:any
    ```
    
    - 模块声明通配符
    
    global.d.ts
    ```ts
    declare module "*.json";
    ```


### 命名空间 (内部模块)

- 命名空间

```ts

namespace Validation {
    export interface StringValidator {
        isSafe:boolena;
    }

    export class Validator implements StringValidator {
       isSafe:boolean = true; 
    }
}

```

- 分割到多个文件

Validation.ts
```ts

namespace Validation {
    export interface StringValidator {
        isSafe:boolena;
    }
}

```
KlassValidation.ts
```ts
/// <reference path="Validation.ts" />
namespace Validation {
    export class Validator implements StringValidator {
       isSafe:boolean = true; 
    }
}

```

- 别名

```ts
namespace Validation {
    export interface StringValidator {
        isSafe:boolena;
    }
}


import SV = Validation.StringValidator;

```

- 使用其他的 js 库
> 为了描述不是用 TypeScript 编写的类库的类型，我们需要声明类库导出的API。 由于大部分程序库只提供少数的顶级对象，命名空间是用来表示它们的一个好办法。  
我们称其为声明是因为它不是外部程序的具体实现。 我们通常在 `.d.ts` 里写这些声明

D3.d.ts
```ts
declare namespace D3 {
  export interface Selectors {
    select: {
      (selector: string): Selection;
      (element: EventTarget): Selection;
    };
  }

  export interface Event {
    x: number;
    y: number;
  }

  export interface Base extends Selectors {
    event: Event;
  }
}

declare var d3: D3.Base;
```
