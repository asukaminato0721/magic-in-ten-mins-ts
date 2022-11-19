# 十分钟魔法练习：斯科特编码

### By 「玩火」 改写 「Asuka Minato」

> 前置技能：构造演算， ADT ，μ

斯科特编码（Scott Encoding）可以在 λ 演算上编码 ADT 。其核心思想就是利用解构函数来处理和类型不同的分支，比如对于如下类型：

```
Either<A, B> = Left<A> + Right<B>
```

在构造演算中拥有类型：

```
Either = λ A: *. λ B: *. (C: *) → (A → C) → (B → C) → C
```

它接受两个解构函数，分别用来处理 Left 分支和 Right 分支然后返回其中一个分支的处理结果。可以按照这个类型签名构造出以下两个类型构造器：

```
Left  = λ A: *. λ B: *. λ val: A. (λ C: *. λ l: A → C. λ r: B → C. l val)
Right = λ A: *. λ B: *. λ val: B. (λ C: *. λ l: A → C. λ r: B → C. r val)
```

乍一看挺复杂的，不过两个构造器具有非常相似的结构，区别仅仅是 `val` 的类型和最内侧调用的函数。实际上构造一个 `Left` 的值时先填入对应 `Either` 的类型参数然后再填入储存的值就可以得到一个符合 `Either` 类型签名的实例，解构时填入不同分支的解构函数就一定会得到 `Left` 分支解构函数处理的结果。

再举个 `List` 的例子：

```
List = λ T: *. (μ L: *. (R: *) → R → (T → L → R) → R)

Nil  = λ T: *. (λ R: *. λ nil: R. λ cons: T → List T → R. nil)
Cons = λ T: *. λ val: T. λ next: List T. 
    (λ R: *. λ nil: R. λ cons: T → List T → T. cons val next)

map = λ A: *. λ B: *. λ f: A → B. μ m: List A → List B.
    λ list: List A. 
    list (List B)
    (Nil B)
    (λ x: A. λ xs: List A. Cons B (f x) (m xs))
```

也就是说，积类型 `A * B * ... * Z` 会被翻译为

```
(A: *) → (B: *) → ... → (Z: *) →
    (Res: *) → (A → B → ... → Z → Res) → Res
```

和类型 `A + B + ... + Z` 会被翻译为

```
(A: *) → (B: *) → ... → (Z: *) →
    (Res: *) → (A → Res) → (B → Res) → ... → (Z → Res) → Res
```

并且两者可以互相嵌套从而构成复杂的类型。

如果给和类型的每个分支取个名字，并且允许在解构调用的时候按照名字索引，随意改变分支顺序，在解糖阶段把解构函数调整成正确的顺序那么就可以得到很多函数式语言里面的模式匹配（Pattern match）。然后就可以像这样表示 `List` ：

```
type List: * → * 
| Nil: (T: *) → List T
| Cons: (T: *) → T → List T → List T
```

解糖的时候利用类型签名可以重建构造函数。像这样使用 `List` ：

```
map = λ A: *. λ B: *. λ f: A → B. μ m: List A → List B. 
    λ list: List A. 
    match list (List B)
    | Cons _ x xs → Cons B (f x) (m xs)
    | Nil  _      → Nil B
```

