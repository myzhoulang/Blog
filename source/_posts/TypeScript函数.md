---
title: TypeScript函数
date: 2019-11-13 10:27:15
tags: [TypsScript, 前端]
author: 周浪
thumbnail: https://myzhoulang.github.io/blog/2019/09/17/TypeScript%E5%9F%BA%E7%A1%80%E7%B1%BB%E5%9E%8B-md/ts.jpg
---

## 函数类型和函数定义

`TypeScript`中的函数的定义和`JavaScript`中定义形式是一样的，无非就是在定义的时候对参数，返回值指定了类型。

- 函数类型声明

函数类型的格式是 `(x: <类型>, y: <类型>) => <类型>`

```typescript
// 函数声明
function add(x: number, y: number): number {
  return x * y;
}

// 函数表达式
// 这个 add 函数的类型是 (x: number, y: number) => number
let add: (x: number, y: number) => number = function(
  x: number,
  y: number
): number {
  return x * y;
};
```

## 函数参数

函数调用是的传入的参数必须和定义时指定的参数个数和类型都需要保持一致。返回值也是一样。

- 可选参数
  在参数名后面添加`?`代表这个参数是可选的。

  ```typescript
  // 参数lastName是可选的
  function join(A: string, B?: string): string {
    if (lastName) {
      return `${A} ${B}`;
    }

    return A;
  }

  join("A"); // A
  join("A", "B"); // A B
  join("A", "B", "C"); // 出错了 传入的参数大于定义的参数
  ```

- 默认参数

  可以在函数定义的时候，可以为一些参数设置一些默认值。如果用户没有传递这个参数或则传入 undefined。都会使用这个默认参数。

  ```typescript
  function join(A: string, B: string = "default"): string {
    if (lastName) {
      return `${A} ${B}`;
    }

    return A;
  }
  join("A"); // A default
  join("A", "B"); // A B
  join("A", "B", "C"); // 出错了 传入的参数大于定义的参数
  ```

- 剩余参数
  当传入的参数个数不定的时候，可以使用剩余参数处理。他会将这些剩余参数放在数组中。

  ```typescript
  function join(A: string, ...rest: Array<string>): string {
    return `${A} ${rest.join(" ")}`;
  }
  ```

**可选参数和剩余参数都必须要放在参数的最后面**

## 函数重载

函数重载是`TypeScript`中的，是使用相同函数名称和不同参数数量和类型创建多个方法的一种能力，函数重载由 `重载签名`和`重载实现`组成。

```typescript
interface Result {
  code: number;
  message: string;
}

// 重载签名
function test(x: Result): Result;
function test(x?: number, message?: string): Result;

// 重载实现
function test(x: Result | number = 200, message: string = "ok"): Result {
  if (typeof x === "object") {
    return x;
  }

  return {
    code: x,
    message
  };
}

console.log(test({ code: 400, message: "bad Request" })); // { code: 400, message: 'bad Request' }
console.log(test(401, "未登录")); // { code: 401, message: '未登录' }
console.log(test()); // { code: 200, message: 'ok' }
```

## 泛型函数

```typescript
// 声明一个泛型函数
// 格式 function 函数名 <T>(参数名: T): T
function print<T>(arg: T): T {
  return arg;
}

// 使用泛型函数
print<string>("Hello");
```
