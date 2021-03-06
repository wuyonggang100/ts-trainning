# 一、keyof 关键字

> 对应任何类型`T`,`keyof T`的结果为该类型上所有公有属性key的联合;

## 1、保护类型的 keyof 

只有 readonly 和 public 属性值可以被 keyof 捕捉到

```ts
interface Eg1 {
  name: string,
  readonly age: number,
}
// T1的类型实则是name | age
type T1 = keyof Eg1

class Eg2 {
  private name: string;
  public readonly age: number;
  protected home: string;
}
// T2实则被约束为 age
// 而name和home不是公有属性，所以不能被keyof获取到
type T2 = keyof Eg2
```

## 2、T[K] 索引 

> `T[keyof T]`的方式，可以获取到`T`所有`key`的类型组成的联合类型； `T[keyof K]`的方式，获取到的是`T`中的`key`且同时存在于`K`时的类型组成的联合类型； 注意：如果`[]`中的key有不存在T中的，则是any；因为ts也不知道该key最终是什么类型，所以是any；且也会报错。

```ts
interface Eg1 {
  name: string,
  readonly age: number,
}
type V1 = Eg1['name'] // string
type V2 = Eg1['name' | 'age'] // string | number
type V2 = Eg1['name' | 'age2222'] // any
type V3 = Eg1[keyof Eg1] // string | number
```

# 二、extends 关键字

- 表示继承/拓展的含义 ： class

- 表示[约束](https://so.csdn.net/so/search?q=约束&spm=1001.2101.3001.7020)的含义

  ```ts
  // redux 里 dispatch 一个 action，必须包含 type属性：
  interface Dispatch<T extends { type: string }> {
    (action: T): T
  }
  ```

  

- 表示分配的含义

## 1.简单值的匹配

```ts
type Equal<X, Y> = X extends Y ? true : false;

type Num = Equal<1, 1>; // true
type Str = Equal<'a', 'a'>; // true
type Boo = Equal<true, false>; // false


type isNum<T> = T extends number ? number : string

type Num = isNum<1>   // number;
type Str = isNum<'1'> // string;
```

### 根据值类型，获取值类型名称的函数类型

```ts
type TypeName<T> =
    T extends string    ? "string" :
    T extends number    ? "number" :
    T extends boolean   ? "boolean" :
    T extends undefined ? "undefined" :
    T extends Function  ? "function" :
    "object";

type T0 = TypeName<string>;  // "string"
type T1 = TypeName<"a">;     // "string"
type T2 = TypeName<true>;    // "boolean"
type T3 = TypeName<() => void>;  // "function"
type T4 = TypeName<string[]>;    // "object"
```

### 判断联合类型

> 其中联合类型 A 的全部子类型，在联合类型 B 中存在，则条件满足。 若是咱们把返回值替换为其余的类型定义，就能根据判断，得出想要的类型。c

```TS
type A = 'x'| 4;
type B = 'x' | 'y' | 4;

type Y = A extends B ? true : false; // true
```



> 先进行 x  extends "x" 得到  "a" | "b"  , 再执行  y extends "x" 得到  'y' , 最后结果即 "a" | "b" | 'y'

```ts

type Other = "a" | "b";
type Merge<T> = T extends "x" ? Other : T; // T 等于除匹配类型的额外全部类型（官方叫候选类型）

type Values = Merge<"x" | "y">;// 获得 type Values = "a" | "b" | 'y';
```

> 由上面可以得到 Diff，

```ts
type Diff<T, U> = T extends U ? never : T; // 找出T的差集
type Filter<T, U> = T extends U ? never : T; // 找出交集
type Values = Filter<"x" | "y" | "z", "x">; // 获得 type Values = "y" | "z"
```



# 三、infer 关键字

> 在extends语句中，还支持`infer`关键字，能够推断一个类型变量，高效的对类型进行模式匹配。可是，这个类型变量只能在true的分支中使用，也就是下面的 R 只能在 ？左边使用；

```ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
```

### 1.1 取出数组中的类型

> 若是`T`是某个待推断类型的数组，则返回推断的类型 R，不然返回`T`

```ts
// 当?左边为 true 时就返回 数组中的类型, 否则返回 T
type ExtractArrayItemType<T> = T extends (infer R)[] ? R : T;

// 条件判断都为 false，返回 T
type T1 = ExtractArrayItemType<string>;         // string
type T2 = ExtractArrayItemType<() => number>;   // () => number
type T4 = ExtractArrayItemType<{ a: string }>;  // { a: string }

// 条件判断为 true，返回 U
type T3 = ExtractArrayItemType<Date[]>;     // Date
```

1.1 取出数组或元组中第一个的类型

```ts
// 元组第一项的类型，可用在 Hooks 风格的 React 组件中
type Head<T> = T extends [infer H, ...any[]] ? H : never;
```

### 1.2 获取函数返回值类型

```ts
// 解读: 若是泛型变量T是 () => infer R的`子集`，那么返回 经过infer获取到的函数返回值，不然返回boolean类型
// ()=>infer R 获取的是函数返回值
type Func<T> = T extends () => infer R ? R : boolean;

let func1: Func<number>; // boolean
let func2: Func<''>; // boolean
let func3: Func<() => Promise<number>>; // Promise<number>
```

推断函数返回值类型

> 若是`T`是一个函数，则返回函数的返回值，不然返回`any`

```ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

type func = () => number;
type variable = string;
type funcReturnType = ReturnType<func>; // funcReturnType 类型为 number
type varReturnType = ReturnType<variable>; // varReturnType 类型为 string
```



### 1.3 返回联合类型

> 同一个类型变量在推断的值有多种状况的时候会推断为联合类型

```ts
// 当a、b为不一样类型的时候，返回不一样类型的联合类型
type Obj<T> = T extends {a: infer VType, b: infer VType} ? VType : number;

let obj1: Obj<string>; // number
let obj2: Obj<true>; // number
let obj3: Obj<{a: number, b: number}>; // number
let obj4: Obj<{a: number, b: () => void}>; // number | () => void

// 同一个值类型可能会有多样
type ElementOf<T> = T extends (infer R)[] ? R : never;

// 元组转联合类型
type TTuple = [string, number];
type Union = ElementOf<TTuple>; // Union 类型为 string | number
// 方式2
type Res = TTuple[number];  // string | number
```



### 1.4 infer 的作用不止是推断返回值，还能够解包

> 获取  `Promise<xxx>`类型中的`xxx`类型

```ts
type Response = Promise<number[]>;
type Unpacked<T> = T extends Promise<infer R> ? R : T;

type resType = Unpacked<Response>; // resType 类型为number[]
```

> 返回 a, b 的联合类型

```ts
type Foo<T> = T extends { a: infer U; b: infer U } ? U : never;

type T10 = Foo<{ a: string; b: string }>; // T10类型为 string
type T11 = Foo<{ a: string; b: number }>; // T11类型为 string | number
```



#### 1.4.1 返回函数第一个参数类型

```ts
type ParamType<T> = T extends (...param: [infer P, ...infer R]) => any ? P : never;


type Fn = (name:number, age: number,address: string)=>any
type AA = ParamType<Fn> // number

interface User {
    name: string;
    age: number;
}
type Func = (user: User) => void
type Param = ParamType<Func>;   // Param = User
type BB = ParamType<string>;    // string

```



#### 1.4.2 提取构造函数中参数（实例）类型

> 一个构造函数可使用 `new` 来实例化，所以它的类型一般表示以下,将 infer R 移动到返回值位置就是获取返回值，移动到参数位置就是获取参数

```ts
type Constructor = new (...args: any[]) => any;
```

获取返回值类型，即实例类型

```ts
// 获取参数类型，得到数组
type ConstructorParameters<T extends new(...args:any[])=>any> = T extends new(...args: infer R) => any ? R: never

// 获取实例类型
type InstanceType<T extends new(...args:any[])=>any> = T extends new(...args:any[]) => infer R ? R:never

class TestClass {
    constructor( public name: string, public string: number ) {}
}
type Params = ConstructorParameters<typeof TestClass>;  // [string, numbder]
type Instance = InstanceType<typeof TestClass>;         // TestClass
```

## infer 案例

1. 如下实现一个 promisify， 接受一个值，若是它已是 `Promise` 了，就直接返回；若是不是，就把它包在一个 `Promise` 中返回；

   ```ts
   function promisify<T> (input: T): T extends Promise ? T : Promise<T> {
    if (input instanceOf Promise) {
       return input;
     }
     return Promise.resolve(input);
   }
   ```

2. 要把 `Promise` 中的值包在一个 { value: T } 的结构中返回，需要确定其返回值类型；

   ```ts
   function promisify2<T> (input: T) {
     if (input instanceof Promise) {
       return input.then(value => ({ value }));
     }
     return Promise.resolve({ value: input });
   }
   ```

   1. 首先，如果 value 是Promise<T> 就返回 T 类型， 否则就返回 value 类型

      ```ts
      type Unpromise<T> = T extends Promise<infer U> ? U : T;
      
      type Test1 = Unpromise<number>; // number
      type Test2 = Unpromise<Promise<string>>; // string
      ```

   2. 最后实现如下：

      ```ts
      type Unpromise<T> = T extends Promise<U> ? U : T; // 明确返回值类型
      type Container<T> = Promise<{ value: T }>; // 将返回值类型进行包装，最后得到的都是一个普通含有 value 属性的对象
      
      function promisify2<T>(input: T): Container<Unpromise<T>> {
        if (input instanceof Promise) {
          return input.then(value => ({ value }));
        }
        return Promise.resolve({ value: input });
      }
      ```

      

