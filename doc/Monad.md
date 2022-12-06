# 十分钟魔法练习：单子

### By 「玩火」 改写 「Asuka Minato」

> 前置技能：TypeScript 基础，HKT

reference: https://medium.com/flock-community/monads-simplified-with-generators-in-typescript-part-1-33486bf9d887

## 单子

单子 (Monad) 是指一种有一个类型参数的数据结构，拥有 `pure` （也叫 `unit` 或者 `return` ）和 `flatMap` （也叫 `bind` 或者 `>>=` ）两种操作：

其中 `pure/of` 要求返回一个包含参数类型内容的数据结构， `flatMap` 要求把 `ma` 中的值经过 `f` 以后再串起来。

举个最经典的例子，在 JavaScript/TypeScript 中，Promise 就是 Monad.

```ts
type Monad<A> = Promise<A>;

function of<A>(a: A): Monad<A> {
  return Promise.resolve(a);
}

function flatMap<A, B>(monad: Monad<A>, cb: (a: A) => Monad<B>): Monad<B> {
  return monad.then(cb);
}
```

## List Monad

```ts
type Monad<A> = Array<A>;

function pure<A>(a: A): Monad<A> {
  return [a];
}

function flatMap<A, B>(monad: Monad<A>, cb: (a: A) => Monad<B>): Monad<B> {
  return monad.flatMap(cb);
}
```

简单来说 `pure(v)` 将得到 `[v]` ，而 `[1, 2, 3].flatMap(v => [v + 1, v + 2])` 将得到 `[2, 3, 3, 4, 4, 5]` 。这都是 TypeScript 里面非常常见的操作了，并没有什么新意。

而使用生成器，可以非常轻松地写出 flatMap

<!-- #flatMap -->
```ts
function* flatMap(f, xs){
    for(const x of xs){
        yield* f(x);
    }
}
const result = Array.from(flatMap(v => [v + 1, v + 2], [1, 2, 3]))
console.log(result);
```

<!-- #flatMap-output -->
```ts
[2, 3, 3, 4, 4, 5]
```

## Maybe Monad

TypeScript 不是一个空安全的语言，也就是说任何对象类型的变量都有可能为 `null/undefined` 。对于一串可能出现空值的逻辑来说，判空常常是件麻烦事：

TS 3.7 / ES2020 后引入了 `?.` 和 `??` 运算符，可以简化一些判空的逻辑：

```ts
const addI = (ma: number | undefined, mb: number | undefined): number | undefined => {
    if (ma === undefined || mb === undefined) {
        return undefined;
    }
    return ma + mb;
}
```

其中 `Maybe` 是个包装类型：

> reference https://codewithstyle.info/advanced-functional-programming-in-typescript-maybe-monad/

<!-- verifier:prepend-to-following -->
```ts
class Maybe<T> {
    private constructor(private value: T | null) { }

    static some<T>(value: T) {
        if (!value) {
            throw Error("Provided value must not be empty");
        }
        return new Maybe(value);
    }

    static none<T>() {
        return new Maybe<T>(null);
    }

    static fromValue<T>(value: T) {
        return value ? Maybe.some(value) : Maybe.none<T>();
    }

    getOrElse(defaultValue: T) {
        return this.value ?? defaultValue;
    }
    // 以及 flatMap 的实现。
    flatMap<R>(f: (wrapped: T) => Maybe<R>): Maybe<R> {
        if (this.value === null) {
            return Maybe.none<R>();
        } else {
            return f(this.value);
        }
    }
    toString(){
        return `Maybe{ value: ${this.value}}`
    }
}
```


上面 `addI` 的代码就可以改成：

<!-- verifier:prepend-to-following -->
```ts
const addM = (ma: Maybe<number>, mb: Maybe<number>) => {
                                            // do
    return ma.flatMap(a =>                  //   a <- ma
        mb.flatMap(b =>                     //   b <- mb
            Maybe.fromValue(a + b)          // pure (a + b)
        ))
}
```

<!-- #test-addM -->
```ts
console.log(addM(Maybe.fromValue(1) , Maybe.fromValue(2)).toString())
```

<!-- #test-addM-output -->
```
Maybe{ value: 3}
```

这样看上去就比上面的连续 `if-return` 优雅很多。在一些有语法糖的语言 (`Haskell`) 里面 Monad 的逻辑甚至可以像上面右边的注释一样简单明了。

> 我知道会有人说，啊，我有更简单的写法：
>
> ```ts
> const addE = (ma: Maybe<number>, mb: Maybe<number>) => {
>     try {
>        const ans = ma.getOrElse(undefined) + mb.getOrElse(undefined);
>        if (isNaN(ans)) {
>            throw Error("bad operator")
>        } else {
>            return Maybe.fromValue(ans);
>        }
>    } catch (e) {
>        return Maybe.none<number>();
>  }
>}
> ```
>
> 确实，这样写也挺简洁直观的， `Maybe Monad` 在有异常的 TypeScript 里面确实不是一个很好的例子，不过 `Maybe Monad` 确实是在其他没有异常的函数式语言里面最为常见的 Monad 用法之一。而之后我也会介绍一些异常也无能为力的 Monad 用法。
