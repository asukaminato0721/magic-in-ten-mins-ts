# 十分钟魔法练习：单位半群

### By 「玩火」 改写 「Asuka Minato」

> 前置技能：TypeScript 基础

## 半群（Semigroup）

半群是一种代数结构，在集合 `A` 上包含一个将两个 `A` 的元素映射到 `A` 上的运算即 `<> : (A, A) -> A​` ，同时该运算满足**结合律**即 `(a <> b) <> c === a <> (b <> c)` ，那么代数结构 `{<>, A}` 就是一个半群。

比如在自然数集上的加法或者乘法可以构成一个半群，再比如字符串集上字符串的连接构成一个半群。

```ts
interface Semigroup<T> {
    op(other: T): T;
}
```

## 单位半群（Monoid）

单位半群是一种带单位元的半群，对于集合 `A` 上的半群 `{<>, A}` ， `A` 中的元素 `a` 使 `A` 中的所有元素 `x` 满足 `x <> a` 和 `a <> x` 都等于 `x`，则 `a` 就是 `{<>, A}` 上的单位元。

举个例子， `{+, 自然数集}` 的单位元就是 0 ， `{*, 自然数集}` 的单位元就是 1 ， `{+, 字符串集}` 的单位元就是空串 `""` 。

用 TypeScript 代码可以表示为：


<!-- #MonoidAcc -->
```ts
interface MonoidAcc<T> {
    value: T;
    concat: (v: T) => MonoidAcc<T>
}
```

## 应用：Optional

参考了这篇文章 https://functionalprogramming.medium.com/monoids-in-typescript-59a9c1510993

<!-- #sumacc -->
<!-- verifier:prepend-id-to-following:MonoidAcc -->
```ts
class SumAcc implements MonoidAcc<number> {
    constructor(public value: number) {
        this.value = value;
    }
    concat = (v: number) => this.value !== 0 ? this : new SumAcc(v);
}
```

这样对于 concat 来说我们将获得一串 SumAcc 中第一个不为 0 的值，对于需要进行一连串尝试操作可以这样写：

<!-- verifier:prepend-id-to-following:sumacc -->
<!-- verifier:prepend-id-to-following:MonoidAcc -->
<!-- #sumaccTest -->
```ts
const sum: MonoidAcc<number> = new SumAcc(0)
    .concat(1)
    .concat(0)
    .concat(3);
console.log(sum.value);
```

<!-- #sumaccTest-output -->
```
1
```

## 应用：Ordering

可以构造出：

<!-- verifier:prepend-id-to-following:MonoidAcc -->
<!-- #type -->
```ts
enum Ordering {
    LT = -1,
    EQ = 0,
    GT = 1
}

class OrderingAcc implements MonoidAcc<Ordering> {
    constructor(public value: Ordering) {
        this.value = value;
    }
    concat = (v: Ordering) => this.value === Ordering.EQ ? new OrderingAcc(v) : this;
}

const compare = (a: any, b: any): Ordering => {
    if (a < b) {
        return Ordering.LT;
    }
    else if (a > b) {
        return Ordering.GT;
    }
    else {
        return Ordering.EQ;
    }
}
```

同样如果有一串带有优先级的比较操作就可以用 appends 串起来，比如：

<!-- verifier:prepend-id-to-following:type -->
<!-- #Student -->
```ts
class Student {
    constructor(public name: string, public gender: string, public from: string) {
        this.name = name;
        this.gender = gender;
        this.from = from;
    }
    compare = (other: Student): Ordering => new OrderingAcc(Ordering.EQ)
            .concat(compare(this.name, other.name))
            .concat(compare(this.gender, other.gender))
            .concat(compare(this.from, other.from))
            .value;
}
```

<!-- verifier:prepend-id-to-following:Student -->
<!-- #testStudent -->
```ts
(() => {
    const student_1 = new Student("Alice", "Female", "Utopia");
    const student_2 = new Student("Dorothy", "Female", "Utopia");
    const student_3 = new Student("Alice", "Female", "Vulcan");
    console.log(student_1.compare(student_2));
    console.log(student_1.compare(student_3));
    console.log(student_2.compare(student_3));
})()
```

<!-- #testStudent-output -->
```
-1
-1
1
```


这样的写法比一连串 `if-else` 优雅太多。

## 扩展

加方法可以支持更多方便的操作：

<!-- #extend -->
```ts
const when = (c: boolean, then: Function) => c ? then : () => { };
const cond = (c: boolean, then: Function, els: Function) => c ? then : els;
class Todo {
    appends = (v: Function[]) => () => v.forEach(x => x());
}
```

然后就可以像下面这样使用上面的定义：

<!-- verifier:prepend-id-to-following:extend -->
<!-- #Todo -->
```ts
new Todo().appends([
    () => console.log(1),
    () => console.log(2),
    () => console.log(3),
    when(true, () => console.log(4)),
    when(false, () => console.log(5)),
    cond(true, () => console.log(6), () => console.log(7)),
    cond(false, () => console.log(8), () => console.log(9)),
])()
```


<!-- #Todo-output -->
```
1
2
3
4
6
9
```

> 注：上面的 Optional 并不是 lazy 的，实际运用中加上非空短路能提高效率。