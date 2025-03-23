# 在 Rust 中表征树形结构


树形结构在软件中相当常用。比如文件目录、右键菜单、多级收藏夹等。

在其它语言，如 C/C++ 中，表达树形结构可能仅需要指针即可，然而在 Rust 中需要稍费周折。

<!--more-->

![树示例](/images/tree_eg_content.png "Hugo 文章结构")

众所周知，Rust 中想要实现一个足够通用的双向链表并不是容易事，其根本原因是所有权机制。

在链表这样底层的数据结构中，所有权关系往往并没有那么明确。对于树来说也是同样。

下面我来尝试梳理一些解决方案。

## 应用场景

首先来假定一个简单的应用场景吧。

![Acrobat 书签](/images/adobe_acrobat_csapp_catalog.png "Acrobat 书签，CSAPP 目录")

假设我们希望实现 Acrobat 这样的多级书签功能，并且需要可以修改：包括增删改书签、调整顺序、调整级别。

这是一个广义的非二叉树结构。或者更准确点是个森林结构，因为顶级标签并无父节点，不过这无伤大雅。

总之抽象出来的需求就是：

1. 除根节点外，每个节点具有一个父节点
2. 每个节点可以具有多个子节点，也可以没有
3. 节点具有数据值（在这里是标签名）

## 1. 递归数据结构

### 定义

我们用 `Item` 代表每一个标签项。

一个优美而非常具有诱惑力的想法是朴素的递归数据结构：

```rust
struct Item {
    name: String,
    children: Vec<Item>,
}
```

> 顺便一提，如果希望树是异构的，比如需要在类型系统上区分内节点和叶节点，可以使用 `enum`

非常简单。而如果希望遍历整个树，代码也非常简洁：

```rust
impl Item {
    fn walk(&self) {
        println!("{}", self.name);
        for child in &self.children {
            child.walk();
        }
    }
}
```

这已经是几乎不能再短的代码量了。

### 缺点

然而这种简洁背后其实隐含的信息是：每个 `Item` 完整拥有其子节点的所有权。

或者说，任何一个子节点，都被看作是父节点的一部分。这意味一旦发生借用，将会影响到整个父节点。

比如说我需要根据节点的名称找到该节点，函数的签名可能类似于：

```rust
fn look_for(&self, name: &str) -> Option<&Item> {
    todo!()
}
```

这里返回的 `Item` 的引用可能需要暂存起来，之后再使用来做某些事。然而既然是引用，就需要服从 Rust 的所有权机制。

也就是说，这时候从根开始的整个树是不可变的。这显然极大地妨碍了可进行的操作。

更何况，有时候我们需要返回特定节点的可变引用，以待进行操作，如进行重命名等。

以书签为例，我们可能需要在发生鼠标事件时选中某个标签。也就是返回其引用。而直到很长一段时间之后，我可能忽然需要将它重命名，那么先前返回的就得是可变引用。

而在这段时间之内，就连不可变借用都无法发生。

<img src="/images/tree_error_0.png" style="zoom:80%;">

更糟糕的是，假如我需要调整某个标签的位置呢？如果是在同一层调整先后顺序，那就必须要获取父节点的可变引用，调整其 `children` 字段。

这几乎是不可完成的。所以需要换个思路。

### 变通方法

在换思路之前我们先来看看有没有一些（不优雅的）修补方法。

上述方法之所以不可用，是因为试图用引用来访问某个特定的节点。

既然如此，干脆不引用得了。最极端的方法：每次需要进行操作我都从根节点朝下遍历找到需要的节点。

在节点较少时，这种方案还算可以接受，毕竟代码比较容易写。

另一种稍微优化的方案是，对于每个节点，使用从根到其路径来代表它。

具体而言就是，用一个 `Vec<usize>` 记录路径，表示“根节点的第 x 个子节点的第 y 个子节点的...”。

如下述结构，其中 `life` 节点可以用 `[1,0]` 表示，代表它是根节点 `content` 的第 1 个子节点 `categories` 的第 0 个子节点。

```
content
├── about.md
└── categories
   ├── life
   └── tech
```

在大部分情况下，树形结构的深度都不会太夸张，那么这个方案也是可以接受的。

不过需要注意的是，路径随时可能失效。比如我有一天忽然删除了 `about.md` 这个节点，那么 `categories` 就变成了 `content` 的第 0 个子节点了。

## 2. 内部可变性与共享所有权

### 定义

朴素方法的核心矛盾在于，无法长久保存一个节点的可变引用。

此时我们就得想办法绕过 Rust 的借用检查机制了。

比如使用 [`Rc`](https://doc.rust-lang.org/std/rc/struct.Rc.html) 共享所有权，以及使用 [`RefCell`](https://doc.rust-lang.org/std/cell/struct.RefCell.html) 获得内部可变性。

```rust
struct Item {
    name: String,
    children: Vec<Rc<RefCell<Item>>>,
}
```

用 `Rc` 是必不可少的，因为 `RefCell` 并非完全避开借用检查，只是将编译时运行检查推迟到运行时而已。这也意味着仍然无法同时持有可变引用和不可变引用。

举个例子：

```rust
let root: RefCell<Item> = get_example();
let item_ref = &root.borrow().children[1];
root.borrow_mut().children.pop();
```

第 2 行借用了 1 号子节点，然而第 3 行却又操作了子节点数组，这个过程无可避免地用到了可变借用，于是发生了运行时错误：`thread 'refcell::lab::test' panicked at 'already borrowed: BorrowMutError'`。

为了保证节点的生命周期，我们需要用 `Rc` 来共享所有权。

除去对子节点的引用外，我们还需要保留对父节点的引用，以方便某些操作（如调整某节点在父节点列表中的顺序）。

然而，这里却不可以再采取 `Rc` 的方法了，原因是形成了循环引用。

```rust
#[derive(Debug)]
struct Item {
    name: String,
    parent: Option<Rc<RefCell<Item>>>,
    children: Vec<Rc<RefCell<Item>>>,
}
```

由于根节点可以没有父节点，所以采用 `Option` 包装了 `parent` 字段。

以上结构在 Rust 中是合法的。然而它其实因为循环引用而发生了内存泄露，如需证明，可以运行以下代码。

```rust
let l1 = Rc::new(RefCell::new(Item {
    name: "l1".to_string(),
    parent: None,
    children: vec![],
}));
let l2 = Rc::new(RefCell::new(Item {
    name: "l2".to_string(),
    parent: Some(l1.clone()),
    children: vec![],
}));
l1.borrow_mut().children.push(l2);
println!("{:?}", l1);
```

这会输出无穷无尽的内容，直到编译器报错 `thread 'refcell::lab::test' has overflowed its stack` 并退出。

稍微分析以下就知道，子节点 `l2` 引用了父节点 `l1`，而 `l1` 又引用了 `l2`，这样的循环引用导致循环计数永不归零，发生内存泄漏。

为了避免这个问题，需要使用弱引用 [`Weak`](https://doc.rust-lang.org/std/rc/struct.Weak.html)。

> 其实不是用弱引用也是可以的，只需要手动实现 `Drop` trait，在析构时手动解除引用即可——`Rc` 循环引用的问题在于它不是自动析构的
>
> 另外提一点，递归结构的默认析构可能会导致递归析构，因而栈溢出。为了避免这种情况也可以手动实现 `Drop`

```rust
// 为了方便起个短的别名
type Parent = Weak<RefCell<Item>>;
type ItemRef = Rc<RefCell<Item>>;

struct Item {
    name: String,
    parent: Parent,
    children: Vec<ItemRef>,
}
```

值得一提的是，这里 `Parent` 类型可以不用包装为 `Option`，因为 `Weak` 本身也是可以指向空的。

还是以遍历和获取特定节点为例：

```rust
fn walk(&self) {
    println!("{}", self.name);
    for child in &self.children {
        child.borrow().walk();
    }
}
fn look_for(item: &ItemRef, name: &str) -> Option<ItemRef> {
    if item.borrow().name == name {
        return Some(item.clone());
    } else {
        for child in &item.borrow().children {
            let ret = Item::look_for(child, name);
            if ret.is_some() {
                return ret;
            }
        }
        return None;
    }
}
```

`walk` 的实现大差不大，由于智能指针的 `Deref` trait，只需要加上一个 `.borrow()` 即可。而 `look_for` 的 API 稍微改了一下，没有以 `&self` 为第一参数，而是 `&ItemRef`。这是为了方便 `clone` 出 `Rc` 返回给调用者。

如果希望以 `&self` 为参数，作为类型的方法的话，可能需要另外的一个 `struct` 封装，这里不细讲了。

这个方案还有一个好处在于它可以轻松转换为线程安全的版本：只需将 `Rc` 改为 `Arc`，`RefCell` 改为 `RwLock` 或 `Mutex` 即可。当然内部实现也需要略微变化，不过概念上基本保持一致。

而 `Arc<Mutex<T>>` 的类型本来也就是多线程编程中常用的。

### 缺点

这个方案已经足够满足我们的需求了。在持有父节点引用的情况下，无论是调整节点的顺序或者层级都是可行的。而 `Rc` 又允许我们长久地保存节点，对于重命名等场景也足够好用。

硬要说缺点的话一个是类型的包装比较复杂，如果直接向外界提供接口会暴露很多实现细节。而如果希望封装这些细节则少不了一些样板代码。

另一个是无法方便地集成 `serde` 进行序列化和反序列化。这是因为 `Rc` 没有实现 `Serialize` 和 `Deserialize`。而且由于涉及到引用的问题，父节点和子节点之间具有依赖关系，必须先后创建。

还有一个就是 `RefCell` 和 `Rc` 的运行时开销问题。`RefCell` 会进行运行时的借用检查，而 `Rc` 需要维护引用计数，二者都具有一定的开销，总体来说是不如朴素方案的。

思考这些，说到底还是所有权和引用的问题。

`RefCell` 说是避开了编译器检查，但实际上还是免不了运行时检查。

而 `Rc` 的共享所有权也仍然没有改变节点之间的依赖关系：父节点仍然是拥有子节点的，而子节点也弱引用着父节点。只不过在这种方案下，由于 `Rc` 的存在，可以创造出临时或长期的其它变量拥有某个节点的所有权。

而下一个方案则是用巧妙的方法更改了所有权的结构，并且避开了引用。

## 3. Arena

> 这个方法我最初是在[这篇文章](https://developerlife.com/2022/02/24/rust-non-binary-tree/)中看到的。

Arena 意为竞技场、大舞台等。关于其在计算机科学中的定义可以参考 [wiki](https://en.wikipedia.org/wiki/Region-based_memory_management)。

简单而言，它就是一块较大的内存空间，我们将所需的数据存放在这块空间中，而非需要时再向操作系统申请。

首先，我们先定义 `Arena` 和 `Item` 结构体。

```rust
struct Arena(HashMap<usize, Item>);
struct Item {
    name: String,
    parent: Option<usize>,
    children: Vec<usize>,
}
```

`Arean` 就是包装一下 `HashMap`，它将 `usize` 映射到某个 `Item` 上面。这里可以将 `usize` 理解为 ID。也就是说，我们给每个 `Item` 一个独一无二的编号，类似于数据库中的主键。

这种编号一经确定不再更改，那么在之后的任何时刻，都可以用一个廉价的 `usize` 来指代一个 `Item`，直到需要使用时再从 `Arena` 中查询即可。

常做算法题或者参与过竞赛的同学应该在图论、树等题目中见过类似的表示。这些题目中往往用节点编号直接表示节点。而且由于题目常常不涉及到删除，可以直接用数组存储，不像这里需要考虑复杂的增删改查所以使用了哈希表。

理解这个之后再看 `Item`，显然，现在 `parent` 也通过一个 `usize` 来表示，当然因为可以没有父节点所以包装为 `Option`。而 `children` 子节点列表也可以用 `Vec<usize>` 表示。

在这种写法下，遍历和查找可以如此表示：

```rust
// 这里是作为 `Arena` 的方法
// 实际实现不一定采取这样的接口
// 比如可以实现为 `Item` 的方法，并以 `Arena` 为参数
fn walk(&self, curr: usize) {
    let node = &self.0[&curr];
    println!("{}", node.name);
    for child in &node.children {
        self.walk(*child);
    }
}
fn look_for(&self, curr: usize, name: &str) -> Option<usize> {
    let item = &self.0[&curr];
    if item.name == name {
        return Some(curr);
    } else {
        for child in &item.children {
            let ret = self.look_for(*child, name);
            if ret.is_some() {
                return ret;
            }
        }
        return None;
    }
}
```

思考一下 `Arena` 这种方法的本质。它实际上让所有权结构由原来的树形结构变为了依托于 `HashMap` 的平铺结构，然后不通过引用或者 `RefCell` 等进行索引，而是使用廉价的 ID 作为代表，这也是某种意义上的引用。

由于这里的所有类型都是比较平常的，因此它可以轻松集成 `serde`。而如果需要多线程，则使用并发原语保护 `Arena` 即可。

不过这里其实有一些微妙之处：`children` 中的 `usize` 可以重复吗？一般而言不会想让它重复，所以我们可以用 `HashSet` 或 `BTreeSet` 来代替 `Vec`。

但这样的修改又会带来一些微妙之处：在类似于书签这样的应用场景下，我们希望子节点是有序的。`HashSet` 显然无序，而 `BTreeSet` 虽然有序，却是根据键值决定的顺序，不修改键值也就是 ID 就无法调整顺序。因此这里使用了第三方库 [`indexmap`](https://docs.rs/indexmap/1.9.1/indexmap/) 中的 [`IndexSet`](https://docs.rs/indexmap/1.9.1/indexmap/set/struct.IndexSet.html) 来同时保证唯一性和有序性。

## 总结

仔细思考，在朴素的方案中，由于递归数据结构，它天然就保证了节点的有序性，而且从类型系统上就保证了它是严格的树形结构。

至于使用 `Rc` 和 `RefCell` 的方式，其实是完全可以用其表示链表结构——有点像一棵退化的树，也可以表示更复杂的图结构——然而这种情况下极易发生循环引用而引发内存泄露。

`Arena` 的方案其实也无法保证一定是树形结构，不过反过来说，它也可以轻易扩展成更复杂的图结构，而且也不会有内存问题。

从使用的难易程度来看，递归数据结构和 `Arena` 都比较简单灵活，而 `Rc`+`RefCell` 的方式则需要多加注意。

而性能层面我没有具体计算和测试过，不过 `Arena` 中有诸多 `Hash` 和索引操作，所以可能是有不少额外开销的。`Rc`+`RefCell` 的方式也许略好，而如果是简单遍历，那么朴素方法应当是最好的。

其实除去这些之外，还有一个终极方案是 `unsafe`。事实上我个人觉得这种底层数据结构才是 `unsafe` 大展拳脚的地方。更上层的领域中所有权和借用等概念则是很好的帮手。因为 `unsafe` 有许多需要注意的点，我本人也没有太深入接触过，因此这里就不详细写了。

面对 `unsafe` 的一个基本态度是：先假定 safe 可以解决问题，待到实在不可行时才考虑 `unsafe`。

感谢阅读。

> 2022/10/9 更新：
>
> 前段时间看了 [Too Many List](https://rust-unofficial.github.io/too-many-lists/) 这本书，颇为相见恨晚。作者深入浅出地解析了 Rust 中链表的各种写法，最终实现了一个生产级别的 Unsafe 双向链表。
>
> 无论是作为 Unsafe Rust 的入门书还是解决树形结构问题都是很好的参考书籍，推荐阅读。当然，也有些体会到 Unsafe 代码写起来战战兢兢的感觉，还是 Safe Rust 舒适 :D
>
> 另外，我在 [crates.io](https://crates.io) 上发现了 [slab](https://crates.io/crates/slab) 这个库，感觉可以一定程度上作为 Arena 方案来使用。

> 版权声明：本文采用 [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/) 进行许可，转载请注明出处。
>
> 本文链接：[{{< permalink >}}]({{< permalink >}})


---

> 作者: <no value>  
> URL: http://idlercloud.xyz/posts/tech/tree_in_rust/  

