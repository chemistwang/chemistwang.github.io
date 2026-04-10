---
# layout: post
title: "TS Challenges"
subtitle: ""
date: 2022-11-01
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "TS"
tags:
  - ts
---

对 `ts` 的掌握一定要 `练`，如果在开发中没有条件的话，这有个宝藏的 `Github项目`🌟 [type-challenges](https://github.com/type-challenges/type-challenges)

## Warm-up

### 1. Hello World

```ts
// expected to be string
type HelloWorld = any;

// answer
type HelloWorld = string;
```

了解基础类型

## Easy

### 1. Pick

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = MyPick<Todo, "title" | "completed">;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};

// answer
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

实现一个 `Pick` 类型，用途是将原有类型的部分内容映射到新类型中

知识点：

- `extends`
- `keyof`
- `in`

### 2. Readonly

```ts
// answer
type MyReadonly<T> = {
  readonly [K in keyof T]: T[K];
};
```

实现一个 `Readonly` 类型，保证属性均是只读

### 3. Tuple to Object

```ts
const tuple = ["tesla", "model 3", "model X", "model Y"] as const;
type result = TupleToObject<typeof tuple>; // expected { tesla: 'tesla', 'model 3': 'model 3', 'model X': 'model X', 'model Y': 'model Y'}

// answer
type TupleToObject<T extends ReadonlyArray<string | number>> = {
  [K in T[number]]: K;
};
```

知识点：

- `as const`
- `数组和元组区别`
- `ReadonlyArray`
- `T[number]`

### 4. First of Array

```ts
type arr1 = ["a", "b", "c"];
type arr2 = [3, 2, 1];

type head1 = First<arr1>; // expected to be 'a'
type head2 = First<arr2>; // expected to be 3

// answer
type First<T extends any[]> = T[number] extends never ? never : T[0];
```

知识点：

- `never`
- `?`

### 5. Length of Tuple

```ts
type tesla = ["tesla", "model 3", "model X", "model Y"];
type spaceX = [
  "FALCON 9",
  "FALCON HEAVY",
  "DRAGON",
  "STARSHIP",
  "HUMAN SPACEFLIGHT"
];

type teslaLength = Length<tesla>; // expected 4
type spaceXLength = Length<spaceX>; // expected 5

// answer
type Length<T extends readonly any[]> = T["length"];
```

知识点：

- `length`

### 6. Exclude

```ts
type Result = MyExclude<"a" | "b" | "c", "a">; // 'b' | 'c'

// answer
type MyExclude<T, U> = T extends U ? never : T;
```

知识点：

- `Exclude`

### 7. Awaited

```ts

```

- infer

### 8. If

```ts
type A = If<true, "a", "b">; // expected to be 'a'
type B = If<false, "a", "b">; // expected to be 'b'

// answer
type If<C extends boolean, T, F> = C extends true ? T : F;
```

### 9. Concat

```ts
type Result = Concat<[1], [2]>;

// answer
type Concat<T extends unknown[], U extends unknown[]> = [...T, ...U];
```

### 10. Includes

### 11. Push

```ts
type Result = Push<[1, 2], "3">; // [1, 2, '3']

// answer
type Push<T extends unknown[], U> = [...T, U];
```

### 12. Unshift

```ts
type Result = Unshift<[1, 2], 0>; // [0, 1, 2,]

// answer
type Unshift<T extends unknown[], U> = [U, ...T];
```

### 13. Parameters

```ts
const foo = (arg1: string, arg2: number): void => {};
type FunctionParamsType = MyParameters<typeof foo>; // [arg1: string, arg2: number]

// answer
```

## Meduim

### 1. Get Return Type

```ts
const fn = (v: boolean) => {
  if (v) return 1;
  else return 2;
};

type a = MyReturnType<typeof fn>; // should be "1 | 2"

// answer
```

### 2. Omit

```ts

```

- as

### 3. Readonly 2

### 4. Deep Readonly

### 5. Tuple to Union

```ts
type Arr = ["1", "2", "3"];
type Test = TupleToUnion<Arr>; // expected to be '1' | '2' | '3'

// answer
type TupleToUnion<T> = T extends unknown[] ? T[number] : never;
```

### 6. Chainable Options

### 7. Last of Array

### 8. Pop

### 9. Promise.all

### 10. Type Lookup

### 11. Trim Left

### 12. Trim

### 13. Capitalize

### 14. Replace

### 15. ReplaceAll

### 16. Append Argument

### 17. Permutation

### 18. Length of String

### 19. Flatten

### 20. Append to object

### 21. Absolute

### 22. String to Union

### 23. Merge

### 24. KebabCase

### 25. Diff

### 26. AnyOf

### 27. IsNever

### 28. IsUnion

### 29. ReplaceKeys

### 30. Remove Index Signature

### 31. Percentage Parser

## Hard

## Extreme
