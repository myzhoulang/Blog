---
title: TypeScript基础类型.md
date: 2019-09-17 14:26:25
tags: [TypeScript, 前端]
author: 周浪
thumbnail: https://myzhoulang.github.io/blog/2019/09/17/TypeScript%E5%9F%BA%E7%A1%80%E7%B1%BB%E5%9E%8B-md/ts.jpg
---

## 基础类型

- number(数字)
  `number`包含整数和小数。可以有`二级制`，`八进制`，`十进制`,`十六进制`表示法。

```typescript
let num: number = 6; // 十进制
let num: number = 0xf00d; // 十六进制
let num: number = 0b1010; // 二进制
let num: number = 0o744; // 八进制
```

- string(字符串)
  `string`类型使用`"`，`'`，`` (`) ``表示。

```typescript
let str: string = "string";
let str: string = "string";
let str: string = `string${foo}`;
```

- boolean(布尔)
  `boolean`的值只有 `true` 和 `false`。

```typescript
let checked: boolean = true;
let checked: boolean = false;
let checked: boolean = Boolean(true);
let checked: boolean = Boolean(false);
```

- array(数组)
  `array`类型在`ts`中有两种表示方法。类型定义一种使用`数据类型[]`,另一种使用`Array<数据类型>`。
  如果需要在一个数组中有多种不同类型的可以结合`联合类型`定义。

```typescript
let arr: number[] = [1, 2, 3, 4];
let arr: Array<number> = [1, 2, 3, 4];
```

- tuple(元组)
  `tuple`类型是`ts`中新增的。表示一个**已知元素数量**和**类型**的`数组`。并且值需要和类型的顺序保存一致。
  ⚡️：`元组`和`数组`之间的区别在于: 元组更严格，元素的`数量`，`类型`, `顺序`,都需要保持一致。

```typescript
let t: [string, number] = ["abc", 1];

// ❌ 案例
let t: [string, number] = [1, "abc"]; // 值和类型的定义顺序不对
let t: [string, number] = ["1bc", 1, 2]; // 元素得数量不匹配
```

- enum(枚举)
  `enum`类型也是`ts`中新增的。使用枚举类型可以为一组数值赋予友好的名字。
  表示一个可以通过`key`取到`value`,也可以通过`value`取到`key`的对象。

```typescript
// 定义
const Colors = {
  RED,
  BLUE,
  YELLOW
};

// 使用
const c: Colors = Colors.RED; // 0
const c: Colors = Colors.BLUE; // 1
const c: string = Colors[0]; // RED
```

- any
  `any`类型表示变量可以是任意类型。`ts`不会去对这个变量做类型检测。

```typescript
let foo: any = 4;
foo = "string";
foo = true;

let arr: any[] = [1, "a", true];
```

- void
  `void`表示没有任何类型。它的值只能是 `undefined`或`null`。

```typescript
function fn(): void {
  console.log(1);
}

let foo: void = null;
let foo: void = undefined;
```

- null 和 undefined
  `null`和`undefined`是其他所有类型的子类型。
  ⚡️：`null`值和`undefined`值可以赋给其他任何类型。当设置`strictNullChecks`的时候，只能赋给它们自己和`void`。

```typescript
let u: undefined = undefined;
let n: null = null;
```

- never
  `never`表示永远不存在的值的类型，常用于`抛出错误`，`死循环`, `返回error()`的函数，

```typescript
function err(): never {
  throw new Error();
}

function fail(): never {
  return error("error");
}

function fn(): never {
  while (true) {}
}
```

- object(对象)
  `object`表示非原始类型。

```typescript
let obj: object = {
  number: 12
};

obj.number = 13; // error 不能修改, 因为 number 属性没有在 object 中声明

let obj: { x: number; y: string } = { x: 1, y: "abc" };
obj.x = 2; // ok
```

## 联合类型

对一些值可以是多种类型的变量，需要使用联合类型。

```typescript
// 基本类型变量
let foo: number | string | boolean;
foo = 123;
foo = "abc";
foo = false;

// 函数
function fn(value: string | number): void {
  // ...
}

fn(1);
fn("abc");

// 泛型和数组
let arr: (number | string)[] = [1, "string"];
let arr: Array<number | string | boolean>;
arr = [1, 1, boolean, "abc"];
```

## 类型别名

类型别名是给一组或一个类型起一个名称。他只是引用了别的类型，本身不会创建一个类型。

```typescript
type TypeName = number | string | boolean;
let foo: TypeName = 123;
let foo: TypeName = "abc";
let foo: TypeName = false;
```

## 类型断言

当一个变量可以是多个类型的时候。你已经明确知道这个变量的值是某一种类型，使用这个类型上的某个方法的时候，可能会引发`ts`报错。
这时候需要使用`类型断言`告诉 tsc，我已明确知道了这个类型。请忽略类型检查。

```typescript
let str1: string | number = "123";
// let len: number = str1.length;           // error
let len: number = (str1 as string).length; // 方法1
let len: number = (<string>str1).length; // 方法2  不能再jsx 中使用
```
