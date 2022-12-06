# 十分钟魔法练习：简单类型 λ 演算

### By 「玩火」 改写 「Asuka Minato」

> 前置技能：TypeScript 基础，ADT，λ 演算

## 简单类型 λ 演算

简单类型 λ 演算（Simply-Typed Lambda Calculus）是在无类型 λ 演算（Untyped Lambda Calculus）的基础上加了个非常简单的类型系统。

这个类型系统包含两种类型结构，一种是内建的基础类型 `T` ，一种是函数类型 `A → B` ，其中函数类型由源类型 `A` 和目标类型 `B` 组成：

```
Type = BaseType + FunctionType
FunctionType = Type * Type
```

注意函数类型的符号是右结合的，也就是说 `A → A → A` 等价于 `A → (A → A)` 。

用 TypeScript 代码可以表示为：

```ts
// 构造函数
interface Type { }
// BaseType
class TVal implements Type {
    name: string;
    public toString(): string {
        return this.name;
    }
}
// FunctionType
class TArr implements Type {
    src: Type;
    tar: Type;
    public toString() {
        return "(" + this.src + " → " + this.tar + ")";
    }
}
```

## 年轻人的第一个 TypeChecker

然后需把类型嵌入到 λ 演算的语法树中：

```ts
interface Expr { }
// 构造函数， toString 已省去
class Val implements Expr {
    x: string;
    type: Type;
}
class Fun implements Expr {
    x: Val;
    e: Expr;
}
class App implements Expr {
    f: Expr;
    x: Expr;
}
```

注意只有函数定义的变量需要标记类型，表达式的类型是可以被简单推导出的。同时还需要一个环境来保存定义变量的类型（其实是一个不可变链表）：

```ts
interface Env {
    lookup(s: string): Type;
}
class NilEnv implements Env {
    public lookup(s: string): Type {
        throw new TypeError();
    }
}
class ConsEnv implements Env {
    v: Val;
    next: Env;
    public lookup = (s: string) => (s === (this.v.x)) ? this.v.type : this.next.lookup(s);
}
```

而对于这样简单的模型，类型检查只需要判断 `F X` 中的 `F` 需要是函数类型，并且 `(λ x. F) E` 中 `x` 的类型和 `E` 的类型一致。

而类型推导也很简单：变量的类型就是它被标记的类型；函数定义的类型就是以它变量的标记类型为源，它函数体的类型为目标的函数类型；而函数应用的类型就是函数的目标类型，在能通过类型检查的情况下。

以上用 TypeScript 代码描述就是：

```ts
// 构造函数， toString 已省去
interface Expr {
    Type checkType(Env env) throws BadTypeException;
}
class Val implements Expr {
    public Type checkType(Env env) throws BadTypeException {
        if (type != null) return type;
        return env.lookup(x);
    }
}
class Fun implements Expr {
    public Type checkType(Env env) throws BadTypeException {
    	return new TArr(x.type, e.checkType(new ConsEnv(x, env)));
    }
}
class App implements Expr {
    public Type checkType(Env env) throws BadTypeException {
        Type tf = f.checkType(env);
        if (tf instanceof TArr &&
                ((TArr) tf).src.equals(x.checkType(env)))
            return ((TArr) tf).tar;
        else throw new BadTypeException();
    }
}
```

下面的测试代码对

 ````
(λ (x: int). (λ (y: int → bool). (y x)))
 ````

进行了类型检查，会打印输出 `(int → ((int → bool) → bool))` ：

```ts
interface STLambda {
    static void main(String[] args) throws BadTypeException {
        console.log(
            new Fun(new Val("x", new TVal("int")),
            new Fun(new Val("y", new TArr(new TVal("int"), new TVal("bool"))),
                new App(new Val("y"), new Val("x")))).checkType(new NilEnv()));
    }
}
```

而如果对

```
(λ (x: bool). (λ (y: int → bool). (y x)))
```

进行类型检查就会抛出 `BadTypeException` 。