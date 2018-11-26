# Fantasy Land 规范

[![Build Status](https://travis-ci.org/fantasyland/fantasy-land.svg?branch=master)](https://travis-ci.org/fantasyland/fantasy-land) [![Join the chat at https://gitter.im/fantasyland/fantasy-land](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/fantasyland/fantasy-land?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

(又称 "代数 JavaScript 规范")

<img src="logo.png" width="200" height="200" />

这个项目规范了常见代数结构的互操作性：

* [Setoid](#setoid)
* [Ord](#ord)
* [Semigroupoid](#semigroupoid)
* [Category](#category)
* [Semigroup](#semigroup)
* [Monoid](#monoid)
* [Group](#group)
* [Filterable](#filterable)
* [Functor](#functor)
* [Contravariant](#contravariant)
* [Apply](#apply)
* [Applicative](#applicative)
* [Alt](#alt)
* [Plus](#plus)
* [Alternative](#alternative)
* [Foldable](#foldable)
* [Traversable](#traversable)
* [Chain](#chain)
* [ChainRec](#chainrec)
* [Monad](#monad)
* [Extend](#extend)
* [Comonad](#comonad)
* [Bifunctor](#bifunctor)
* [Profunctor](#profunctor)

<img src="figures/dependencies.png" width="888" height="234" />

## 概述

一个代数是一个值、操作符和所遵循的规则的集合。

每一个Fantasy Land代数是一个单独的规范。一个代数可能依赖于其他必须实现的代数。

## 术语

1. “值”是任意JavaScript值，包含任意以下定义的结构。
2. "等价性" 定义给定值相等的关系。
    这个定义应该保证两个值能在一个考虑抽象概念的程序中安全地进行交换。 例如：
    - 两个列表是等价的如果它们在所有下标处的值都是相等的.
    - 两个普通的JavaScript对象, 翻译成字典, 是等价的，如果对应所有键的值都是相等的.
    - 两个promises是等价的当他们生成相等的值。
    - 两个函数是等价的如果对应相同的输入他们生成相同的输出。

## 类型特征

用于本文档的类型标记描述如下:<sup
id="sanctuary-types-return">[1](#sanctuary-types)</sup>

* `::` _”是成员".
    - `e :: t` 可以被读作: "表达式 `e` 是类型 `t` 的成员".
    - `true :: Boolean` - "`true` 是类型 `Boolean` 的成员".
    - `42 :: Integer, Number` - "`42` 是类型 `Integer和Number`的成员".
* _新类型可以通过类型构造函数创建._
    - 类型构造函数可以有零个或多个类型参数.
    - `Array` 是一个接受一个类型参数的构造函数.
    - `Array String` 类型表示字符数组. 以下都属于`Array String`类型: `[]`, `['foo', 'bar', 'baz']`.
    - `Array (Array String)` 表示字符数组的数组.
      以下都属于 `Array (Array String)`类型: `[]`, `[ [], []
      ]`, `[ [], ['foo'], ['bar', 'baz'] ]`.
* _小写字母代表类型变量._
    - 类型变量可以是任何类型，除非通过类型约束进行限制（参考下面关于胖箭头的描述）
* `->` (箭头) _函数类型构造函数._
    - `->` 是一个带俩个参数的 _中缀_ 类型构造函数，其中左参数是输入类型，右参数是输出类型。
    - `->`'的输入类型可以是类型的组合，用于创建带0个或多个参数的函数类型. 语法:
      `(<输入类型>) -> <输出类型>`, 其中 `<输入类型>` 包含0个或多个逗号加空格 (`, `) 分隔的类型表达,
      另外在单参数函数中圆括号可以被忽略。
    - `String -> Array String` 函数类型接受一个`String`类型的参数，返回一个`Array String`类型的值.
    - `String -> Array String -> Array String` 函数类型接受一个`String`类型的参数，返回一个函数，
      后者接受一个`Array String`类型的参数，返回一个`Array String`类型的值.
    - `(String, Array String) -> Array String` 函数类型以一个`String`和一个`Array String`作为输入参数，
      返回一个`Array String`类型的值.
    - `() -> Number` 函数类型不带参数，返回一个`Number`类型的值.
* `~>` (波浪箭头) _方法构造函数._
    - 当一个函数是一个对象的属性, 它被称之为方法. 所有的方法都有一个隐式的参数类型，就是包含它们作为属性的对象的类型.
    - `a ~> a -> a` 是`a`类型对象的方法，它接受一个`a`类型作为输入参数，返回一个`a`类型的值。
* `=>` (胖箭头) _表示加在类型变量上的约束._
    - 在 `a ~> a -> a` (参考上面波浪箭头)中, `a`可以是任何类型.
      `Semigroup a => a ~> a -> a` 添加一个约束使类型`a`必须满足`Semigroup`类型类.
      满足一个类型类意味着实现它指定的所有函数/方法.

示例:

```
traverse :: Applicative f, Traversable t => t a ~> (TypeRep f, a -> f b) -> f (t b)
'------'    '--------------------------'    '-'    '-------------------'    '-----'
 '           '                               '      '                        '
 '           ' - 类型约束                     '      ' - 参数类型               ' - 返回值类型
 '                                           '
 '- 方法名                                    ' - 方法target类型
```

- - -
1. <a name="sanctuary-types"></a>参考Sanctuary的文档中[Types](https://sanctuary.js.org/#types)
   部分获取更多信息. [↩](#sanctuary-types-return)

## 前缀方法名

一个数据类型要与Fantasy Land兼容，它的值必须包含某些属性。这些属性带有`fantasy-land/`前缀.
 例如:

```js
//  MyType#fantasy-land/map :: MyType a ~> (a -> b) -> MyType b
MyType.prototype['fantasy-land/map'] = ...
```

在该文档中中未加前缀的名字仅用于降低噪音.

为了方便起见你可以使用`fantasy-land`包:

```js
var fl = require('fantasy-land')

// ...

MyType.prototype[fl.map] = ...

// ...

var foo = bar[fl.map](x => x + 1)
```

##  类型表达

某些行为是从一个类型的成员的角度定义的.另外一些行为则不要求是成员。
因此某些代数要求一个类型提供值层级的表达（用某些属性）。
例如Identity类型,  可以提供`Id`作为它的类型表达:
`Id :: TypeRep Identity`.

如果一个类型提供一个类型表达, 它的每一个成员都必须有一个`constructor`属性，
后者是这个类型表达的引用.

## 代数

### Setoid

1. `a.equals(a) === true` (反身性)
2. `a.equals(b) === b.equals(a)` (对称性)
3. 若 `a.equals(b)` 且 `b.equals(c)`, 则 `a.equals(c)` (传递性)

#### `equals` 方法

```hs
equals :: Setoid a => a ~> a -> Boolean
```

拥有Setoid的值必须提供一个`equals` 方法.
`equals` 方法带一个参数:

    a.equals(b)

1. `b`必须拥有相同的Setoid

    1. 如果`b`的Setoid不同, `equals`的行为是不确定的(建议返回`false`).

2. `equals`必须返回一个boolean(`true`或`false`).

### Ord

实现Ord规范的值必须同时实现[Setoid](#setoid)规范.

1. `a.lte(b)` or `b.lte(a)` (全关系)
2. 若 `a.lte(b)` 且 `b.lte(a)`, 则 `a.equals(b)` (反对称性)
3. 若 `a.lte(b)` 且 `b.lte(c)`, 则 `a.lte(c)` (传递性)

#### `lte` 方法

```hs
lte :: Ord a => a ~> a -> Boolean
```

拥有Ord的值必须提供一个`lte`方法。`lte`方法带一个参数:

     a.lte(b)

1. `b`必须拥有相同的Ord

    1. 如果`b`的Ord不同, `lte`的行为是不确定的(建议返回`false`).

2. `lte`必须返回一个boolean(`true`或`false`).

### Semigroupoid

1. `a.compose(b).compose(c) === a.compose(b.compose(c))` (结合律)

#### `compose` 方法

```hs
compose :: Semigroupoid c => c i j ~> c j k -> c i k
```

 拥有Semigroupoid的值必须提供一个`compose`方法. `compose`方法带一个参数:

    a.compose(b)

1. `b`必须拥有相同的Semigroupoid

    1. 如果`b`的semigroupoid不同, `compose`的行为是不确定的.

2. `compose`必须返回一个带相同Semigroupoid的值.

### Category

 实现Category规范的值必须同时实现[Semigroupoid](#semigroupoid)规范.

1. `a.compose(C.id())` 等价于 `a` (右幺元)
2. `C.id().compose(a)` 等价于 `a` (左幺元)

#### `id` 方法

```hs
id :: Category c => () -> c a a
```

拥有Category的值必须提供一个其[类型表达](#type-representatives)的`id`函数:

    C.id()

给定一个`c`值, 你就可以通过`constructor`属性访问其类型表达:

    c.constructor.id()

1. `id必须返回一个同Category的值

### Semigroup

1. `a.concat(b).concat(c)`  等价于 `a.concat(b.concat(c))` (结合律)

#### `concat` 方法

```hs
concat :: Semigroup a => a ~> a -> a
```

拥有Semigroup的值必须提供一个`concat`方法. `concat`方法带一个参数:

    s.concat(b)

1. `b` 必须有相同的Semigroup

    1. 如果`b`的semigroup不同, `concat`的行为是不确定的.

2. `concat`必须返回一个同Semigroup的值.

### Monoid

实现Monoid规范的值必须同时实现[Semigroup](#semigroup)规范.

1. `m.concat(M.empty())`  等价于 `m` (有幺元)
2. `M.empty().concat(m)`  左幺元 `m` (左幺元)

#### `empty` 方法

```hs
empty :: Monoid m => () -> m
```

 拥有Monoid的值必须提供一个其[类表达型](#type-representatives)的`empty`函数:

    M.empty()

给定一个`m`值, 你就可以通过`constructor`属性访问其类型表达:

    m.constructor.empty()

1. `empty`必须返回一个同Monoid的值

### Group

实现Group规范的值必须同时实现[Monoid](#monoid)实现.

1. `g.concat(g.invert())` 等价于 `g.constructor.empty()` (右逆元)
2. `g.invert().concat(g)`  等价于 `g.constructor.empty()` ( 左逆元)

#### `invert` 方法

```hs
invert :: Group g => g ~> () -> g
```

拥有Group的值必须提供一个`invert`方法. `invert`方法不带任何参数:

    g.invert()

1. `invert`必须返回一个同Group的值.

### Filterable

1. `v.filter(x => p(x) && q(x))` 等价于 `v.filter(p).filter(q)` (分配律)
2. `v.filter(x => true)` 等价于 `v` (幺元)
3. `v.filter(x => false)` 等价于 `w.filter(x => false)`
   如果`v`和`w`拥有相同的Filterable (湮灭)

#### `filter` 方法

```hs
filter :: Filterable f => f a ~> (a -> Boolean) -> f a
```

拥有Filterable的值必须提供一个`filter`方法. `filter`方法带一个参数:

    v.filter(p)

1. `p` 必须是一个函数.

    1. 如果`p`不是一个函数, `filter`的行为是不确定的.
    2. `p`必须返回`true`或`false`. 如果它返回其他值，`filter`的行为是不确定的.

2. `filter`必须返回一个同Filterable的值.

### Functor

1. `u.map(a => a)` 等价于 `u` (幺元)
2. `u.map(x => f(g(x)))`  等价于 `u.map(g).map(f)` (复合)

#### `map` 方法

```hs
map :: Functor f => f a ~> (a -> b) -> f b
```

 拥有Functor的值必须提供一个`map`方法.`map`方法带一个参数:

    u.map(f)

1. `f`必须是一个函数,

    1. 如果`f`不是一个函数, `map`的行为是不确定的.
    2. `f`可以返回任何任何值.
    3. 不用检查`f`的返回值的任何部分.

2. `map`必须返回一个同Functor的值

### Contravariant

1. `u.contramap(a => a)` 等价于 `u` (幺元)
2. `u.contramap(x => f(g(x)))` 等价于 `u.contramap(f).contramap(g)`
(复合)

#### `contramap` 方法

```hs
contramap :: Contravariant f => f a ~> (b -> a) -> f b
```

拥有Contravariant的值必须提供一个`contramap`方法. `contramap`方法带一个参数:

    u.contramap(f)

1. `f`必须是一个函数,

    1. 如果`f`不是一个函数, `contramap`的行为是不确定的.
    2. `f`可以返回任何任何值.
    3. 不用检查`f`的返回值的任何部分.

2. `contramap`必须返回一个同Contravariant的值

### Apply

实现Apply规范的值必须同时实现[Functor](#functor)规范.

1. `v.ap(u.ap(a.map(f => g => x => f(g(x)))))` 等价于 `v.ap(u).ap(a)` (复合)

#### `ap` 方法

```hs
ap :: Apply f => f a ~> f (a -> b) -> f b
```

拥有Apply的值必须提供一个`ap`方法. `ap`方法带一个函数:

    a.ap(b)

1. `b`必须是一个函数的Apply

    1. 如果`b`不表示一个函数, `ap`的行为是不确定的.
    2. `b`必须拥有和`a`一样的Apply.

2. `a`必须是一个值的Apply

3. `ap`必须将Apply `b`的函数应用到Apply `a`的值

   1. 不用检查函数的返回值的任何部分.

4. `ap`必须返回和`a`、 `b`一样的`Apply`

### Applicative

实现Applicative规范必须同时实现[Apply](#apply)规范.

1. `v.ap(A.of(x => x))` 等价于 `v` (幺元)
2. `A.of(x).ap(A.of(f))` 等价于 `A.of(f(x))` (同态)
3. `A.of(y).ap(u)`  等价于 `u.ap(A.of(f => f(y)))` (互换)

#### `of` 方法

```hs
of :: Applicative f => a -> f a
```

拥有Applicative必须提供一个其[类型表达](#type-representatives)的`of`函数.
`of`函数带一个参数:

    F.of(a)

给定一个`f`值, 你就可以通过`constructor`属性访问其类型表达:

    f.constructor.of(a)

1. `of`必须提供一个同Applicative的值

    1. 不用检查`a`的任何部分

### Alt

实现Alt规范的值必须同时实现[Functor](#functor)规范

1. `a.alt(b).alt(c)` 等价于 `a.alt(b.alt(c))` (结合律)
2. `a.alt(b).map(f)` 等价于 `a.map(f).alt(b.map(f))` (分配律)

#### `alt` 方法

```hs
alt :: Alt f => f a ~> f a -> f a
```

拥有Alt的值必须提供一个`alt`方法. `alt`方法带一个参数:

    a.alt(b)

1. `b`必须是一个同Alt的值

    1. 如果`b`的Alt不同, `alt`的行为是不确定的.
    2. `a`和`b`可以包含同类型的任何值.
    3. 不用检查`a`和`b`所包含的值的任何部分.

2. `alt`必须返回一个同Alt的值.

### Plus

实现Plus规范的值必须同时实现[Alt](#alt)规范.

1. `x.alt(A.zero())` 等价于 `x` (右幺元)
2. `A.zero().alt(x)` 等价于 `x` (左幺元)
3. `A.zero().map(f)` 等价于 `A.zero()` (湮灭)

#### `zero` 方法

```hs
zero :: Plus f => () -> f a
```

拥有Plus的值必须提供一个其[类型表达](#type-representatives)的`zero`函数:

    A.zero()

给定一个`x`值, 你就可以通过`constructor`属性访问其类型表达:

    x.constructor.zero()

1. `zero`必须返回一个同Plus的值

### Alternative

实现Alternative规范的值必须同时实现[Applicative](#applicative)和[Plus](#plus)规范.

1. `x.ap(f.alt(g))` 等价于 `x.ap(f).alt(x.ap(g))` (分配律)
2. `x.ap(A.zero())` 等价于 `A.zero()` (湮灭)

### Foldable

1. `u.reduce` 等价于 `u.reduce((acc, x) => acc.concat([x]), []).reduce`

#### `reduce` 方法

```hs
reduce :: Foldable f => f a ~> ((b, a) -> b, b) -> b
```

拥有Foldable的值必须提供一个`reduce`方法. `reduce`方法带两个参数:

    u.reduce(f, x)

1. `f`必须是一个二元函数

    1. 如果`f`不是一个函数, `reduce`的行为是不确定的.
    2. `f`的第一个参数的类型必须和`x`的类型一致.
    3. `f`必须返回一个与`x`类型相同的值.
    4. 不用检查`f`的返回值的任何部分.

1. `x`是减法的初始累加器值

    1. 不用检查`x`的任何部分.

### Traversable

实现Traversable规范的值必须同时实现[Functor](#functor)和[Foldable](#foldable)规范.

1. 对任意`t`，`t(u.traverse(F, x => x))` 等价于 `u.traverse(G, t)`，
  使得`t(a).map(f)`等价于`t(a.map(f))` (自然性)

2. 对任意Applicative `F`，`u.traverse(F, F.of)` 等价于 `F.of(u)` (幺元)

3. 对下面定义的`Compose`和Applicatives `F` 和 `G`，
  `u.traverse(Compose, x => new Compose(x))` 等价于
   `new Compose(u.traverse(F, x => x).map(x => x.traverse(G, x => x)))`
   (复合)

```js
var Compose = function(c) {
  this.c = c;
};

Compose.of = function(x) {
  return new Compose(F.of(G.of(x)));
};

Compose.prototype.ap = function(f) {
  return new Compose(this.c.ap(f.c.map(u => y => y.ap(u))));
};

Compose.prototype.map = function(f) {
  return new Compose(this.c.map(y => y.map(f)));
};
```

#### `traverse` 方法

```hs
traverse :: Applicative f, Traversable t => t a ~> (TypeRep f, a -> f b) -> f (t b)
```

拥有Traversable的方法必须提供一个`traverse`方法. `traverse`方法带两个参数:

    u.traverse(A, f)

1. `A`必须是[类型表达=](#type-representatives)的一个Applicative.

2. `f`必须是一个带单个返回值的函数

    1. 如果`f`不是一个函数, `traverse`的行为是不确定的.
    2. `f`必须返回一个与`A`所表示的类型相同的值.

3. `traverse`必须返回一个与`A`所表示的类型相同的值.

### Chain

实现Chain规范的值必须同时实现[Apply](#apply)规范.

1. `m.chain(f).chain(g)` 等价于 `m.chain(x => f(x).chain(g))` (结合律)

#### `chain` 方法

```hs
chain :: Chain m => m a ~> (a -> m b) -> m b
```

拥有Chain的值必须提供一个`chain`方法. `chain`方法带一个参数:

    m.chain(f)

1. `f`必须是一个带单个返回值的函数

    1. 如果`f`不是一个函数, `chain`的行为是不确定的.
    2. `f`必须返回一个同Chain的值

2. `chain`必须返回一个同Chain的值

### ChainRec

实现ChainRec规范的值必须同时实现[Chain](#chain)规范.

1. `M.chainRec((next, done, v) => p(v) ? d(v).map(done) : n(v).map(next), i)`
    等价于
   `(function step(v) { return p(v) ? d(v) : n(v).chain(step); }(i))` (等价关系)
2. `M.chainRec(f, i)`的栈占用必须最多只能是`f`本身的栈占用的固定倍数.

#### `chainRec` 方法

```hs
chainRec :: ChainRec m => ((a -> c, b -> c, a) -> m c, a) -> m b
```

拥有ChainRec的值必须同时提供一个其[类型表达](#type-representatives)的`chainRec`函数.
`chainRec`函数带两个参数:

    M.chainRec(f, i)

给定一个`m`值, 你就可以通过`constructor`属性访问其类型表达::

    m.constructor.chainRec(f, i)

1. `f`必须是一个带单个返回值的函数
    1. 如果`f`不是一个函数, `chainRec`的行为是不确定的.
    2. `f`带三个参数： `next`、 `done` 和 `value`
        1. `next`是一个函数，它带一个和`i`同类型的参数，可以返回任何值
        2. `done`是一个函数，它带一个参数，返回一个和`next`返回值类型相同的值
        3. `value`是和`i`类型相同的一些值
    3. `f`必须返回一个同ChainRec的值，后者包含一个从`done`或`next`返回的值
2. `chainRec`必须返回一个同ChainRec的值，后者包含一个和`done`参数类型相同的值

### Monad

实现Monad规范的值必须同时实现[Applicative](#applicative)和[Chain](#chain)规范.

1. `M.of(a).chain(f)` 等价于 `f(a)` (左幺元)
2. `m.chain(M.of)` 等价于 `m` (右幺元)

### Extend

实现Extend规范的值必须同时实现[Functor](#functor)规范.

1. `w.extend(g).extend(f)` 等价于 `w.extend(_w => f(_w.extend(g)))`

#### `extend` 方法

```hs
extend :: Extend w => w a ~> (w a -> b) -> w b
```

一个Extend必须提供一个`extend`方法. `extend`方法带一个参数:

     w.extend(f)

1. `f`必须是一个带单个返回值的函数

    1. 如果`f`不是一个函数, `extend`的行为是不确定的.
    2. `f`必须返回一个`v`类型的值, 对于某些变量`v`包含在`w`中.
    3. 不用检查`f`的返回值的任何部分.

2. `extend`必须返回一个同Extend的值.

### Comonad

实现Comonad规范的值必须同时实现[Extend](#extend)规范.

1. `w.extend(_w => _w.extract())` 等价于 `w` (左幺元)
2. `w.extend(f).extract()` 等价于 `f(w)` (右幺元)

#### `extract` 方法

```hs
extract :: Comonad w => w a ~> () -> a
```

拥有Comonad的值必须提供一个其自身的`extract`方法.
`extract`方法不带参数:

    w.extract()

1. `extract`必须返回一个`v`类型的值, 对某些变量`v`包含在`w`中.
    1. `v`的类型必须和`f`在`extend`中返回的类型一致.

### Bifunctor

实现Bifunctor的值必须同时实现[Functor](#functor)规范.

1. `p.bimap(a => a, b => b)` 等价于 `p` (幺元)
2. `p.bimap(a => f(g(a)), b => h(i(b))` 等价于 `p.bimap(g, i).bimap(f, h)` (复合)

#### `bimap` 方法

```hs
bimap :: Bifunctor f => f a c ~> (a -> b, c -> d) -> f b d
```

拥有Bifunctor的值必须提供一个`bimap`方法. `bimap`方法带两个参数:

    c.bimap(f, g)

1. `f`必须是一个带单个返回值的函数

    1. 如果`f`不是一个函数, `bimap`的行为是不确定的.
    2. `f`可以返回任何类型.
    3. 不用检查`f`的返回值的任何部分.

2. `g`必须是一个带单个返回值的函数

    1. 如果`g`不是一个函数, `bimap`的行为是不确定的.
    2. `g`可以返回任何类型.
    3. 不用检查`g`的返回值的任何部分.

3. `bimap`必须返回一个同Bifunctor的值.

### Profunctor

实现Profunctor规范的值必须同时实现[Functor](#functor)规范.

1. `p.promap(a => a, b => b)` 等价于 `p` (幺元)
2. `p.promap(a => f(g(a)), b => h(i(b)))` 等价于 `p.promap(f, i).promap(g, h)` (复合)

#### `promap` 方法

```hs
promap :: Profunctor p => p b c ~> (a -> b, c -> d) -> p a d
```

拥有Profunctor的值必须提供一个`promap`方法.

`profunctor`方法带两个参数:

    c.promap(f, g)

1. `f`必须是一个带单个返回值的函数

    1. 如果`f`不是一个函数, `promap`的行为是不确定的.
    2. `f`可以返回任何类型.
    3. 不用检查`f`的返回值的任何部分.

2. `g` must be a function which returns a value

    1. 如果`g`不是一个函数, `promap`的行为是不确定的.
    2. `g`可以返回任何类型.
    3. 不用检查`g`的返回值的任何部分.

3. `promap`不行返回一个同Profunctor的值

## Derivations

当创建满足多个代数逻辑的数据类型时, 作者必须选择实现某些方法，然后派生剩下的方法. 派生:

  - [`equals`][] 可能派生自 [`lte`][]:

    ```js
    function(other) { return this.lte(other) && other.lte(this); }
    ```

  - [`map`][] 可能派生自 [`ap`][] and [`of`][]:

    ```js
    function(f) { return this.ap(this.of(f)); }
    ```

  - [`map`][] 可能派生自 [`chain`][] and [`of`][]:

    ```js
    function(f) { return this.chain(a => this.of(f(a))); }
    ```

  - [`map`][] 可能派生自 [`bimap`]:

    ```js
    function(f) { return this.bimap(a => a, f); }
    ```

  - [`map`][] 可能派生自 [`promap`]:

    ```js
    function(f) { return this.promap(a => a, f); }
    ```

  - [`ap`][] 可能派生自 [`chain`][]:

    ```js
    function(m) { return m.chain(f => this.map(f)); }
    ```

  - [`reduce`][] 可能派生自 follows:

    ```js
    function(f, acc) {
      function Const(value) {
        this.value = value;
      }
      Const.of = function(_) {
        return new Const(acc);
      };
      Const.prototype.map = function(_) {
        return this;
      };
      Const.prototype.ap = function(b) {
        return new Const(f(b.value, this.value));
      };
      return this.traverse(x => new Const(x), Const.of).value;
    }
    ```

  - [`map`][] 可能派生成自如下代码:

    ```js
    function(f) {
      function Id(value) {
        this.value = value;
      }
      Id.of = function(x) {
        return new Id(x);
      };
      Id.prototype.map = function(f) {
        return new Id(f(this.value));
      };
      Id.prototype.ap = function(b) {
        return new Id(this.value(b.value));
      };
      return this.traverse(x => Id.of(f(x)), Id.of).value;
    }
    ```

  - [`filter`][] 可能派生自 [`of`][], [`chain`][], and [`zero`][]:

    ```js
    function(pred) {
      var F = this.constructor;
      return this.chain(x => pred(x) ? F.of(x) : F.zero());
    }
    ```

  - [`filter`][] 可能派生自 [`concat`][], [`of`][], [`zero`][], and
    [`reduce`][]:

    ```js
    function(pred) {
      var F = this.constructor;
      return this.reduce((f, x) => pred(x) ? f.concat(F.of(x)) : f, F.zero());
    }
    ```

如果一个数据类型提供一个可以被派生的方法，它的行为必须和派生物一致.

## 注意

1. 如果存在多种方式实现方法和定律, 实现代码里应选择其中一种方式，同时为其他方式提供包裹函数.
2. 不鼓励重载已规范的方法. 因为这样可能导致怪异的行为.
3. 推荐为未定义行为抛一个异常.
4. [sanctuary-identity](https://github.com/sanctuary-js/sanctuary-identity)
  提供了一个`Identity`容器，它实现了上面的很多方法.


[`ap`]: #ap-method
[`bimap`]: #bimap-method
[`chain`]: #chain-method
[`concat`]: #concat-method
[`empty`]: #empty-method
[`equals`]: #equals-method
[`extend`]: #extend-method
[`extract`]: #extract-method
[`filter`]: #filter-method
[`lte`]: #lte-method
[`map`]: #map-method
[`of`]: #of-method
[`promap`]: #promap-method
[`reduce`]: #reduce-method
[`sequence`]: #sequence-method
[`zero`]: #zero-method

## 其他选择

[Static Land Specification](https://github.com/rpominov/static-land)
与Fantasy Land思想大体一致，但是基于静态方法而不是实例方法.
