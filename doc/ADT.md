# 十分钟魔法练习：代数数据类型

### By 「玩火」 改写 「Asuka Minato」

> 前置技能：TypeScript 基础

## 积类型（Product type）

积类型是指同时包括多个值的类型，比如 TypeScript 中的 class 就会包括多个字段：

```ts
class Student {
    name: string;
    id: number;
}
```

而上面这段代码中 Student 的类型中既有 String 类型的值也有 int 类型的值，可以表示为 String 和 int 的「积」，即 `String * int` 。

## 和类型（Sum type）

和类型是指可以是某一些类型之一的类型，在 TypeScript 中可以用继承来表示：

```ts
interface SchoolPerson {}
class Student implements SchoolPerson {
    name: string;
    id: number;
}
class Teacher implements SchoolPerson {
    name: string;
    office: string;
}
```

SchoolPerson 可能是 Student 也可能是 Teacher ，可以表示为 Student 和 Teacher 的「和」，即 `String * int + String * String` 。而使用时只需要用 `instanceof` 就能知道当前的 StudentPerson 具体是 Student 还是 Teacher 。

## 代数数据类型（ADT, Algebraic Data Type）

由和类型与积类型组合构造出的类型就是代数数据类型，其中代数指的就是和与积的操作。

利用和类型的枚举特性与积类型的组合特性，我们可以构造出 TypeScript 中本来很基础的基础类型，比如枚举布尔的两个量来构造布尔类型：

```ts
interface Bool {}
class True implements Bool {}
class False implements Bool {}
```

然后用 `t instanceof True` 就可以用来判定 t 作为 Bool 的值是不是 True 。

比如利用 S 的数量表示的自然数：

```ts
interface Nat {}
class Z implements Nat {}
class S implements Nat {
    value: Nat;
    
    S(v: Nat) { this.value = v; }
}
```

这里提一下自然数的皮亚诺构造，一个自然数要么是 0 （也就是上面的 Z ) 要么是比它小一的自然数 +1 （也就是上面的 S ) ，例如 3 可以用 `new S(new S(new S(new Z))` 来表示。

再比如链表：

<!-- verifier:prepend-to-following -->
```ts
interface List<T> {}
class Nil<T> implements List<T> {}
class Cons<T> implements List<T> {
    value: T;
    next: List<T> ;
    
    constructor(v: T, n: List<T>) {
        this.value = v;
        this.next = n;
    }
}
```

`[1, 3, 4]` 就表示为 `new Cons(1, new Cons(3, new Cons(4, new Nil)))`


<!-- #adt -->
```ts
console.log((new Cons(1, new Cons(3, new Cons(4, new Nil())))))
```

<!-- #adt-output -->
```json
{"value":1,"next":{"value":3,"next":{"value":4,"next":{}}}}
```

更奇妙的是代数数据类型对应着数据类型可能的实例数量。

很显然积类型的实例数量来自各个字段可能情况的组合也就是各字段实例数量相乘，而和类型的实例数量就是各种可能类型的实例数量之和。

比如 Bool 的类型是 `1+1 `而其实例只有 True 和 False ，而 Nat 的类型是 `1+1+1+...` 其中每一个 1 都代表一个自然数，至于 List 的类型就是`1+x(1+x(...))` 也就是 `1+x^2+x^3...` 其中 x 就是 List 所存对象的实例数量。

## 实际运用

ADT 最适合构造树状的结构，比如解析 JSON 出的结果需要一个聚合数据结构。

```ts
interface JsonValue {}
class JsonBool implements JsonValue {
    value: boolean;
}
class JsonInt implements JsonValue {
    value: number;
}
class JsonString implements JsonValue {
    value: string;
}
class JsonArray implements JsonValue {
    value: JsonValue[] ;
}
class JsonMap implements JsonValue {
    value: Map<String, JsonValue> ;
}
```

> 注 1：上面的写法是基于变量非空假设的，也就是代码中不会出现 null ，所有变量也不为 null 。
