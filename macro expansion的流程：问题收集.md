我学习macro expansion的主要参考资料是https://veykril.github.io/tlborm/introduction.html和https://github.com/rust-lang/reference。以下的内容都是我结合rustc编译器相关的源码摘录原文有误导性的段落。

#### 一类TokenTree

先构造以下两段代码。

```
macro_rules! what_is {
    (self) => {"the keyword self"};
    ($i:ident) => {concat!("the identifier `", stringify!(i), "`")};
}

macro_rules! call_with_ident {
    (c:ident(c:ident(i:ident)) => {c!(c!(i)};
}

fn main() {
    println!("{}", what_is!(self));
    println!("{}", call_with_ident!(what_is(self)));
}
```

```
macro_rules! what_is {
    ($i:ident) => {concat!("the identifier `", stringify!($i), "`")};
    (self) => {"the keyword `self`"};
}

macro_rules! call_with_ident {
    ($c:ident($i:ident)) => {$c!($i)};
}

fn main() {
    println!("{}", what_is!(self));
    println!("{}", call_with_ident!(what_is(self)));
}
```

这两段代码的运行结果完全不同，再看看macro expand的源码有没有关于这种情况的说明。枚举类型[rustc_expand::mbe::TokenTree](https://doc.rust-lang.org/nightly/nightly-rustc/rustc_expand/mbe/enum.TokenTree.html "TokenTree外部文档")的注释中有这么一段。

> ```
> `$i`, `$i:ident`, and `$(...)` are "first-class" token trees. Useful for parsing macros.
> ```

第一种$i是个笔误，不存在不带fragment的元变量，我自己尝试的等价写法是直接写token，比如 `what_is`中的self。第二种和第三种符合MBE的句法。

#### 句法检查、类型检查

声明宏的expand得到的是TokenStream，调用parse_ast_fragment，https://doc.rust-lang.org/nightly/nightly-rustc/rustc_expand/expand/struct.MacroExpander.html#method.parse_ast_fragment，构造这段TokenStream的Ast，这段过程在expand_invoc。https://doc.rust-lang.org/nightly/nightly-rustc/rustc_expand/expand/struct.MacroExpander.html#method.expand_invoc。parse_ast_fragment在构造AST的过程中检查有没有语法错误，如下代码的第一处错误是因为这时编译器还不知道元变量代表的token是什么。第二处错误 `expected unit struct, unit variant or constant, found local variable `self `，
的修改建议就有点莫名其妙了，左侧和右侧当然都不能出现self。即使考虑到它们三个的用法，也不应该出现在左侧而是右侧。
实际上我倾向于这里应该报“需要一个标识符，但是得到self”，

先看看原文给出的代码，这里的错误实际上相当隐蔽，因为只要疏忽宏处理的其中一个流程，就很容易陷入莫名其妙的状态。这段代码会报两个错误，都与编译器的执行流程有关。声明宏的句法翻译代码在compile_declarative_macro，。第一处错误是因为这时编译器还不知道元变量代表的token是什么。

```
macro_rules! make_mutable {
    ($i:ident) => {let mut $i = $i;};
}

struct Dummy(i32);

impl Dummy {
    fn double(self) -> Dummy {
        make_mutable!(self);
        self.0 *= 2;
        self
    }
}
```

LegacyBang

编译器区分了不同种类的句法翻译，Bang型是过程宏中的类函数宏，LegacyBang型就是声明宏。今天的主角是后者。

编译器对不同种类的宏的执行流程

这段代码报的两个错误，第一个与编译器如何处理声明宏有关，第二个是类型检查期间的错误。声明宏的处理步骤不只有我即将谈论的这些，我只在这里论述与错误相关的步骤。

首先是第一处错误，

发生在[AstFragment的语法分析阶段](https://doc.rust-lang.org/nightly/nightly-rustc/src/rustc_expand/expand.rs.html#858-924)，模式匹配的右侧构造了一个let表达式，但不符合[IdentifierPattern](https://doc.rust-lang.org/reference/patterns.html#identifier-patterns)句法定义。

tips：如果是元变量捕获类型对不上，在[对TokenTree语法分析](https://doc.rust-lang.org/nightly/nightly-rustc/src/rustc_expand/mbe/quoted.rs.html#136-228)的时候就能发现了，报的是expected identifier, found `self`

```
macro_rules! make_mutable {
    ($i:expr) => {let mut $i = 5;};
}

struct Dummy(i32);

impl Dummy {
    fn double(self) -> Dummy {
        make_mutable!(self);
        self.0 *= 2;
        self
    }
}
```

第二处错误 `expected unit struct, unit variant or constant, found local variable `self `是类型检查期间产生的，修改建议就有点莫名其妙了，左侧和右侧当然都不能出现self。即使考虑到它们三个的用法，也不应该出现在左侧而是右侧。

实际上我倾向于这里应该报“需要一个标识符，但是得到self”，类型检查阶段像self这种关键字具体是什么类型都是清楚的，能用self关键字的场合就那几个。

#### Path

严格地说，每个关键字确实同时又是标识符，编译器在词法分类上这么看待标识符和关键字，但是在语法分析阶段标识符和关键字又相互区分开了，因为两者能出现的语法位置不同所以语义不一样。既是关键字又是标识符的看法只停留在词法分析的阶段，忽略了rust宏的实现策略。我们知道语法分析阶段是为了构建AST，而宏的处理流程中的一步就是从tokenstream构造AstFragment，最终合并这些AstFragment得到一个完整的AST。但这个案例并不是发生在语法分析阶段，此例中self符合[路径表达式](https://doc.rust-lang.org/nightly/nightly-rustc/src/rustc_driver/lib.rs.html#195-435)的形式文法，案例的错误是语义的问题，self的语义是module，语法分析阶段是无法知道模块语义的。模块语义的解析发生在遍历AST的过程中，准确的报错点在[smart_resolve_path_segment](https://doc.rust-lang.org/nightly/nightly-rustc/src/rustc_resolve/late.rs.html#1924-2094)，错误消息由[smart_resolve_report_errors](https://doc.rust-lang.org/nightly/nightly-rustc/src/rustc_resolve/late/diagnostics.rs.html#132-609)输出。smart_resolve_path_segment负责了所有的路径语法的语义求解，但是let语句需要一个具体的值，self关键字在这里是没有值语义的。

```
macro_rules! make_self_mutable {
    ($i:ident) => {let mut $i = self;};
}

struct Dummy(i32);

impl Dummy {
    fn double(self) -> Dummy {
        make_self_mutable!(mut_self);
        mut_self.0 *= 2;
        mut_self
    }
}
```

#### 右手侧
