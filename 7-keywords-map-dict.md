7-键值列表-图-字典
================
[键值列表](#71)    
[图](#72-%E5%9B%BEmaps)    
[字典](#73-%E5%AD%97%E5%85%B8dicts)    

到目前还没有讲到任何关联性数据结构，即那种可以将一个或几个值关联到一个key上。
不同语言有不同的叫法，如字典，哈希，关联数组，图，等等。

Elixir中有两种主要的关联性结构：键值列表（keyword list）和图（map）。

## 7.1-键值列表
在很多函数式语言中，常用二元元组的列表来表示关联性数据结构。在Elixir中也是这样。
当我们有了一个元组（不一定仅有两个元素的元组）的列表，并且每个元组的第一个元素是个 **原子**，
那就称之为键值列表：
```elixir
iex> list = [{:a, 1}, {:b, 2}]
[a: 1, b: 2]
iex> list == [a: 1, b: 2]
true
iex> list[:a]
1
```  

>当原子key和关联的值之间没有逗号分隔时，可以把原子的冒号拿到字母的后面。这时，原子与后面的数值之间要有一个空格。

如你所见，Elixir使用比较特殊的语法来定义这样的列表，但实际上它们会映射到一个元组列表。
因为它们是简单的列表而已，所有针对列表的操作，键值列表也可以用。

比如，可以用```++```运算符为列表添加元素：
```elixir
iex> list ++ [c: 3]
[a: 1, b: 2, c: 3]
iex> [a: 0] ++ list
[a: 0, a: 1, b: 2]
```
上面例子中重复出现了```:a```这个key，这是允许的。
以这个key取值时，取回来的是第一个找到的（因为有序）：
```elixir
iex> new_list = [a: 0] ++ list
[a: 0, a: 1, b: 2]
iex> new_list[:a]
0
```

键值列表十分重要，它有两大特点：
- 有序
- key可以重复（！仔细看上面两个示例）

例如，[Ecto库](https://github.com/elixir-lang/ecto)使用这两个特点
写出了精巧的DSL（用来写数据库query）：
```elixir
query = from w in Weather,
          where: w.prcp > 0,
          where: w.temp < 20,
        select: w
```

这些特性使得键值列表成了Elixir中为函数提供额外选项的默认手段。
在第5章我们讨论了```if/2```宏，提到了下方的语法：
```elixir
iex> if false, do: :this, else: :that
:that
```

do: <block>和else: <block> 就是键值列表！事实上代码等同于：
```elixir
iex> if(false, [do: :this, else: :that])
:that
```

当键值列表是函数最后一个参数时，方括号就成了可选的。

为了操作关键字列表，Elixir提供了
[键值（keyword）模块](http://elixir-lang.org/docs/stable/elixir/Keyword.html)。
记住，键值列表就是简单的列表，和列表一样提供了线性的性能。
列表越长，获取长度或找到一个键值的速度越慢。
因此，关键字列表在Elixir中一般就作为函数调用的可选项。
如果你要存储大量数据，并且保证一个键只对应最多一个值，那就使用图。

对键值列表做模式匹配：
```elixir
iex> [a: a] = [a: 1]
[a: 1]
iex> a
1
iex> [a: a] = [a: 1, b: 2]
** (MatchError) no match of right hand side value: [a: 1, b: 2]
iex> [b: b, a: a] = [a: 1, b: 2]
** (MatchError) no match of right hand side value: [a: 1, b: 2]
```
尽管如此，对列表使用模式匹配很少用到。因为不但要元素个数相等，顺序还要匹配。

## 7.2-图（maps）
无论何时想用键-值结构，图都应该是你的第一选择。Elixir中，用```%{}```定义图：
```elixir
iex> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> map[:a]
1
iex> map[2]
:b
```

和键值列表对比，图有两主要区别：
- 图允许任何类型值作为键
- 图的键没有顺序

如果你向图添加一个已有的键，将会覆盖之前的键-值对：
```elixir
iex> %{1 => 1, 1 => 2}
%{1 => 2}
```

如果图中的键都是原子，那么你也可以用键值列表中的一些语法：
```elixir
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
```

对比键值列表，图的模式匹配很是有用：
```elixir
iex> %{} = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}
iex> %{:a => a} = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}
iex> a
1
iex> %{:c => c} = %{:a => 1, 2 => :b}
** (MatchError) no match of right hand side value: %{2 => :b, :a => 1}
```
如上所示，图A与另一个图B做匹配。
图B中只要包含有图A的键，那么两个图就能匹配上。若图A是个空的，那么任意图B都能匹配上。
但是如果图B里不包含图A的键，那就匹配失败了。

图还有个有趣的功能：它提供了特殊的语法来修改和访问原子键：
```elixir
iex> map = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}
iex> map.a
1
iex> %{map | :a => 2}
%{:a => 2, 2 => :b}
iex> %{map | :c => 3}
** (ArgumentError) argument error
```

使用上面两种语法要求的前提是所给的键是切实存在的。最后一条语句错误的原因就是键```:c```不存在。

未来几章中我们还将讨论结构体（structs）。结构体提供了编译时的保证，它是Elixir多态的基础。
结构体是基于图的，上面例子提到的修改键值的前提就变得十分重要。

[图模块](http://elixir-lang.org/docs/stable/elixir/Map.html)提供了许多关于图的操作。
它提供了与键值列表许多相似的API，因为这两个数据结构都实现了字典的行为。

>图是最近连同[EEP 43](http://www.erlang.org/eeps/eep-0043.html)被引入Erlang虚拟机的。
Erlang 17提供了EEP的部分实现，只支持_一小部分_图功能。
这意味着图仅在存储不多的键时，图的性能还行。
为了解决这个问题，Elixir还提供了
[HashDict模块](http://elixir-lang.org/docs/stable/elixir/HashDict.html)。
该模块提供了一个字典来支持大量的键，并且性能不错。

## 7.3-字典（Dicts）
Elixir中，键值列表和图都被称作字典。
换句话说，一个字典就像一个接口（在Elixir中称之为行为behaviour）。
键值列表和图模块实现了该接口。

这个接口定义于[Dict模块](http://elixir-lang.org/docs/stable/elixir/Dict.html)，
该模块还提供了底层实现的一个API：
```elixir
iex> keyword = []
[]
iex> map = %{}
%{}
iex> Dict.put(keyword, :a, 1)
[a: 1]
iex> Dict.put(map, :a, 1)
%{a: 1}
```

字典模块允许开发者实现自己的字典形式，提供一些特殊的功能。
字典模块还提供了所有字典类型都可以使用的函数。
如，```Dicr.equal?/2```可以比较两个字典类型（可以是不同的实现）。

你会疑惑些程序时用keyword，Map还是Dict模块呢？答案是：看情况。   

如果你的代码期望接受一个关键字作为参数，那么使用关键字列表模块。
如果你想操作一个图，那就使用图模块。
如果你想你的API对所有字典类型的实现都有用，
那就使用字典模块（确保以不同的实现作为参数测试一下）。
