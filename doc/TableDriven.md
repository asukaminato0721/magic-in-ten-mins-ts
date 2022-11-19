# 十分钟魔法练习：表驱动编程

### By 「玩火」 改写 「Asuka Minato」

> 前置技能： 简单 TypeScript 基础

## Intro

表驱动编程被称为是普通程序员和高级程序员的分水岭，而它本身并没有那么难，甚至很多时候不知道的人也能常常重新发明它。而它本身在我看来是锻炼抽象思维的最佳途径，几乎所有复杂的系统都能利用表驱动法来进行进一步抽象优化，而这也非常考验程序员的水平。

## 数据表

学编程最开始总会遇到这样的经典习题：

> 输入成绩，返回等第， 90 以上 A ， 80 以上 B ， 70 以上 C ， 60 以上 D ，否则为 E

作为一道考察 `if` 语句的习题初学者总是会写出这样的代码：

```ts
const getLevel = (s: number) => {
    if (s >= 90) { return 'A'; }
    if (s >= 80) { return 'B' };
    if (s >= 70) { return 'C' };
    if (s >= 60) { return 'D' };
    return 'E';
}
```

等学了 `switch` 语句以后有些聪明的人会把它改成 `switch(s/10)` 的写法。

但是这两种写法都有个同样的问题：如果需要不断添加等第个数那最终 `getLevel` 函数就会变得很长很长，最终变得不可维护。

学会循环和数组后会有聪明人回头再看这个程序，会发现这个程序由反复的

<!-- verifier:skip -->
```ts
if (s >= _) return _;
```

构成，可以改成循环结构，把对应的数据塞进数组：

```ts
const score = [90, 80, 70, 60];
const level = ['A', 'B', 'C', 'D', 'E'];
const getLeve = (s: number) => {
    let pos = 0;
    for (; pos < score.length && s < score[pos]; pos++) { }
    return level[pos];
}
```

这样的好处是只需要在两个数组中添加一个值就能加一组等第而不需要碰 `getLevel` 的逻辑代码。

而且进一步讲， `score` 和 `level` 数组可以被存在外部文件中作为配置文件，与源代码分离，这样不用重新编译就能轻松添加一组等第。

这就是表驱动编程最初阶的形式，通过抽取相似的逻辑并把不同的数据放入表中来避免逻辑重复，提高可读性和可维护性。

再举个带修改的例子，写一个有特定商品的购物车：

<!-- verifier:prepend-to-following -->
<!-- #ShopList -->
```ts
class ShopList {
    items: Map<[name: string], { price: number, count: number }>;
    // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map#cloning_and_merging_maps
    constructor() {
        //@ts-ignore
        this.items = new Map([
            ["water", { price: 1, count: 0 }],
            ["cola", { price: 2, count: 0 }],
            ["choco", { price: 5, count: 0 }]
        ]);
    }

    buy(name: string) {
        //@ts-ignore
        this.items.get(name).count++;
        return this;
    }
    toString() {
        return [...this.items.entries()]
            .map(([name, { price, count }]) =>
                `${name}($${price}/per): ${count}`)
            .join("\n");
    }
}
```

> 本处我把原来的 Array 改写为了 Hashmap

<!-- #shop -->
```ts
const shop = new ShopList();
console.log(shop.buy("cola").buy("cola").buy("choco").toString());
```

<!-- #shop-output -->
```
water($1/per): 0
cola($2/per): 2
choco($5/per): 1
```

## 逻辑表

初学者在写习题的时候还会碰到另一种没啥规律的东西，比如：

> 用户输入 0 时购买 water ，输入 1 时购买 cola ，输入 2 时打印购买的情况，输入 3 退出系统。

看似没有可以抽取数据的相似逻辑。但是细想一下，真的没有公共逻辑吗？实际上公共的逻辑在于这些都是在同一个用户输入情况下触发的事件，区别就在于不同输入触发的逻辑不一样，那么其实可以就把逻辑制成表：

<!-- verifier:prepend-id-to-following:ShopList -->
```ts
const list = new ShopList();

const event1 = new Map([
    [0, () => list.buy("water")],
    [1, () => list.buy("cola")],
    [2, () => console.log(list)],
    [3, () => process.exit(1)]
]);

const runEvent = (e: 0 | 1 | 2 | 3) => {
    event1.get(e)();
}
```

> event1 是因为 event 在 lib dom 里有定义。

这样如果需要添加一个用户输入指令只需要在 `event1` 表中添加对应逻辑和索引，修改用户的指令对应的逻辑也变得非常方便，甚至可以把用户指令存在配置文件里提供自定义修改。这样用户输入和时间触发两个逻辑就不会串在一起，维护起来更加方便。

## 自动机

如果再加个逻辑表能修改的跳转状态就构成了自动机（Automaton）。这里举个例子，利用自动机实现了一个复杂的 UI ，在 `menu` 界面可以选择开始玩或者退出，在 `move` 界面可以选择移动或者打印位置或者返回 `menu` 界面：

<!-- verifier:prepend-to-following -->
```ts
// 界面绘制

class ComplexUI {
    state: number = 0;
    x: number = 0;
    y: number = 0;

    draw = [
        () => { /* draw menu */ },
        () => { /* draw play */ }
    ];
    private jumpTo(s: number) {
        this.state = s;
        (this.draw[this.state] ?? (() => console.log("invalid input")))();
    }
    menu = (c: string) => {
        const events = new Map([
            ['p', () => this.jumpTo(1)],
            ['q', () => process.exit(0)]
        ]);
        (events.get(c) ?? (() => console.log("invalid input")))();
    }

    move = (c: string) => {
        const events = new Map([
            ['w', () => this.y++],
            ['s', () => this.y--],
            ['d', () => this.x++],
            ['a', () => this.x--],
            ['e', () => console.log("{x=" + this.x + ";y=" + this.y + "}")],
            ['q', () => this.jumpTo(0)]
        ]);
        (events.get(c) ?? (() => console.log("invalid input")))();
    }

    jumpers = [
        this.menu,
        this.move
    ];

    runEvent = (c: string) => {
        this.jumpers[this.state](c);
    }
}
```
<!-- #table -->
```ts
const ui = new ComplexUI();
ui.runEvent('a'); // print: invalid key
ui.runEvent('p'); // jump to gameplay state & draw move
ui.runEvent('e'); // print: (0, 0)
ui.runEvent('w'); // coord changed to (0, 1) & draw move
ui.runEvent('e'); // print: (0, 1)
ui.runEvent('q'); // jump to menu state & draw menu
ui.runEvent('q'); // exit
```

<!-- #table-output -->
```
invalid input
{x=0;y=0}
{x=0;y=1}
```

实际上更标准的写法应该把状态设定成枚举，这里为了演示的简单期间并没有那样写。

同时推荐不用下标作为表的索引标签，并总是把所有相关状态打包起来放在同一个类里面而不是用不同数组的相同下标来访问，这样可以有更加紧凑的语义和更好的缓存命中率。
