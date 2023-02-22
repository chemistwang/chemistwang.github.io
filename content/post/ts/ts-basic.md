---
# layout: post
title: "TS 基本玩法"
subtitle: ""
date: 2022-11-04
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "TS"
tags:
  - ts
---




``` ts
type MyType<T> = T extends any ? T : never;
// type res = MyType<string | number | boolean>

// 1. 从 T 中剔除 U
type Res1 = Exclude<string | number, number>
// 2. 从 T 中剔除 null undefined
type Res2 = NonNullable<string | number | null | undefined>
// 3. 获取函数返回值类型
type Res3 = ReturnType<() => number>
// 4. 获取一个类的构造函数组成的元组类型
type Res4 = ConstructorParameters<typeof Person>
class Person {
    constructor(name: string, age: number) { }
}

// 5. 获得函数参数类型组成的元组类型
type Res5 = Parameters<typeof showFn>
function showFn(name: string, age: number) { }


// infer 关键字
type ElementOf<T> = T extends Array<infer E> ? E : T

// eg: 
type IName = string[];
type ID = number[];
type Unpacked<T> = T extends IName ? string : T extends ID ? number : T;

type nameType1 = Unpacked<boolean>
type nameType2 = ElementOf<IName>
type nameType3 = ElementOf<ID>
type nameType4 = ElementOf<boolean>

// infer 关键字可以推断出联合类型
type Foo<T> = T extends { a: infer U; b: infer U } ? U : never
type T11 = Foo<{ a: string, b: number }>


// Readonly | Partial 类型
interface Res6 {
    name: string
    age: number
}

// 通过 + - 指定添加或者删除
type ReadOnlyObj<T> = {
    -readonly [P in keyof T]-?: T[P]
}

type res1 = Readonly<Res6>
type res2 = Partial<Res6>

// Record 类型 将一个类型的所有值都映射到另一个类型上并创造一个新的类型
type RecordType1 = 'person' | 'animal'
type RecordType2 = {
    name: string
    age: number
}
type RecordType3 = Record<RecordType1, RecordType2>

let recordType3: RecordType3 = {
    person: {
        name: 'mars',
        age: 22
    },
    animal: {
        name: 'bear',
        age: 2
    }
}

type A = ['a', 'b', 'c']
type C = A['length'] // 3
type B = A[number] // "a" | "b" | "c"

type TupleToObject<T extends ReadonlyArray<string>> = {
    [K in T[number]]: K 
  }





// Pick 类型 将原有类型中的部分内容映射到新类型中
type PickType1 = {
    name: string
    age: number
}
type PickType3 = Pick<PickType1, 'name'>
const pickType3: PickType3 = {
    name: 'something'
}

// Required 类型
type RequiredType1 = {
    name: string
    age: number
}
type RequiredType3 = Required<RequiredType1>
const requiredType3: RequiredType3 = {
    name: 'mars',
    age: 18
}


// Omit 类型
type OmitType1 = {
    name: string
    age: number
    score: number
    gender: boolean
}

type OmitType3 = Omit<OmitType1, 'name'>

// OmitThisParameter 类型
function f0(this: object, x: number) { }
function f1(x: number) { }

type T0 = OmitThisParameter<typeof f0>
type T1 = OmitThisParameter<typeof f1>
type T2 = OmitThisParameter<string | number>


// namespace 命名空间
namespace A {
    export const a = 100
}
console.log(A.a)

namespace B {
    export const b = 200
    export namespace C {
        export const c = 300
    }
}
console.log(B.b)
console.log(B.C.c)

import c = B.C.c
console.log(c)


// 三斜线
// 如果一个命名空间在一个单独的 typescript 文件中，则嘴应该使用三斜杠引用


/// <reference types="node" />

```