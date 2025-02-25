---
title: Trie的使用场景和优缺点
date: 2025-01-23 14:45:00
tags: DataStructure
toc: true
---
## Trie基本介绍和示例代码

其实leetcode也有这个题

![leetcode208.png](https://telegraph-image-cnr.pages.dev/api/rfile/leetcode208.png)

https://en.wikipedia.org/wiki/Trie
![trie.png](https://telegraph-image-cnr.pages.dev/api/rfile/trie.png)

Trie 树的这个应用可以扩展到更加广泛的一个应用上，就是自动输入补全，比如输入法自动补全功能、IDE 代码编辑器自动补全功能、浏览器网址输入的自动补全功能等等。

## trie的优缺点和使用场景
### 缺点
Trie 树是非常耗内存的，用的是一种空间换时间的思路”。这是什么原因呢

Trie 树的优势并不在于，用它来做动态集合数据的查找，因为，这个工作完全可以用更加合适的散列表或者红黑树来替代。Trie 树最有优势的是查找前缀匹配的字符串，比如搜索引擎中的关键词提示功能这个场景，就比较适合用它来解决，也是 Trie 树比较经典的应用场景

在 Trie 树中，查找某个字符串的时间复杂度是多少？

如果要在一组字符串中，频繁地查询某些字符串，用 Trie 树会非常高效。构建 Trie 树的过程，需要扫描所有的字符串，时间复杂度是 O(n)（n 表示所有字符串的长度和）。但是一旦构建成功之后，后续的查询操作会非常高效

每次查询时，如果要查询的字符串长度是 k，那我们只需要比对大约 k 个节点，就能完成查询操作。跟原本那组字符串的长度和个数没有任何关系。所以说，构建好 Trie 树后，在其中查找字符串的时间复杂度是 O(k)，k 表示要查找的字符串的长度

## 当你有一些数据，如何判断是否要使用trie来实现功能
Trie 树应用场合对数据要求比较苛刻，比如字符串的字符集不能太大，前缀重合比较多等。
如果现在给你一个很大的字符串集合，比如包含 1 万条记录，如何通过编程量化分析这组字符串集合是否比较适合用 Trie 树解决呢？也就是如何统计字符串的字符集大小，以及前缀重合的程度呢？
