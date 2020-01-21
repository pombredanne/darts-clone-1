# Darts-clone: A clone of Darts (Double-ARray Trie System)

Darts-clone is a clone of Darts (Double-ARray Trie System)
which is a C++ header library for a static double-array trie structure.

The features of Darts-clone are as follows:

* Half-size elements
  * Darts-clone uses 32-bit elements and Darts uses 64-bit elements.
   This feature simply halves the size of dictionaries.
* Directed Acyclic Word Graph (DAWG)
  * Darts uses a basic trie to implement a dictionary.
   On the other hand, Darts-clone uses a Directed Acyclic Word Graph (DAWG)
   which is derived from a basic trie by merging its common subtrees.
   Darts-clone thus requires less elements than Darts if a given keyset
   contains many duplicate values.

Due to these features, Darts-clone can achieve better space efficiency
without degrading the search performance.

* Documentation
  * [Introduction](https://github.com/s-yata/darts-clone/blob/master/doc/en/Introduction.md)
  * [Interface](https://github.com/s-yata/darts-clone/blob/master/doc/en/Interface.md)

----

# Darts-clone: Darts（Double-ARay Trie System）のクローン

Darts-clone はダブル配列の C++ ヘッダライブラリである Darts のクローンであり，
以下のような特徴を持っています．

* 要素のサイズが半分
  * Darts が 64-bit の要素を使うのに対し， Darts-clone は 32-bit の要素を使います．
   そのため，辞書のサイズは単純に半減します．
* Directed Acyclic Word Graph (DAWG)
  * Darts がトライを木構造で表現しているのに対し，
   Darts-clone は Directed Acyclic Word Graph (DAWG) というグラフ構造を使います．
   DAWG は木構造の共通部分を併合することで得られるデータ構造なので，
   キーに関連付ける値に重複がたくさん含まれていれば，
   Darts-clone の方が少ない要素で辞書を構成できます．

これらの特徴により， Darts-clone は検索機能や速度を劣化させることなく，
よりコンパクトな辞書を実現できます．

* ドキュメント
  * [説明と使い方](https://github.com/s-yata/darts-clone/blob/master/doc/ja/Introduction.md)
  * [インタフェース](https://github.com/s-yata/darts-clone/blob/master/doc/ja/Interface.md)
  * [使用方法](https://github.com/s-yata/darts-clone/blob/master/doc/ja/Applications.md)
  * [性能評価](https://github.com/s-yata/darts-clone/blob/master/doc/ja/Evaluation.md)
  * [変更履歴](https://github.com/s-yata/darts-clone/blob/master/doc/ja/ChangeLog.md)



## darts-clone内存占用减半的双数组trie树实现

trie树是一种常用的词典存储逻辑结构。双数组trie树是trie树的一种性能较好的具体实现方法。
由日本学者发明，知名NLP工具CRF++的词典即是由此实现。
CRF++所用的darts是实现双数组trie树的一份代码（文档中文翻译），整个词典就是数组元素由两个int组成（即“双”数组名字的来历）。
**darts-clone有与darts相同的接口，但词典数组由一个int搞定，空间减小一半的同时速度还快些。**

关于trie树与双数组trie树的原理非一两句话能介绍透彻，不过好在网上的资料都比较多，不难找到。 本文想介绍一下darts-clone用了啥黑科技达到这种效果的。

darts代码里的两个int， 分别叫做base和check。base用于计算当前节点的儿子节点，即base+儿子节点对应编码=儿子节点下标。check 用于检查儿子节点的正确性，因为不同的base+不同的编码可能得到相同的下标，check一般存的是其父亲节点的base。 如果是叶子节点，base存储的即是词条对应的value。

所以看到base和check的量纲都是下标，用4个字节的int似乎也无可厚非。下面就来看看darts-clone用来哪些方法用一个int替代两个int的，并且速度还有所提升。

#### **用异或代替加法**

用base找儿子节点的下标时，使用异或。不需要进位，感觉会比加法快一些，不过不太了解现在的CPU。

#### **用一个bit表示是否是叶子节点**

是叶子节点，剩下31bit用来表示value。虽然比darts少了一个bit，不过大多数情况下是可以接受的。

#### **非叶子节点，用一个bit表示是否自身有叶子节点**

免去了根据base读一次内存的时间。

#### **非叶子节点，用一个byte代替check的功能**

通常trie树用来存字符串，一个节点对应的值是一个char，所以这个假设也可以接受。darts一个节点的值可以是int。

#### **非叶子节点，用剩下的22个bit存储了一部分的30bit数**

这一点也比较巧妙。22个bit中有一个指示位。为0的时候，剩下21bit表示一个21位的数；为1的时候，剩下21bit表示一个低8位为0，高21位可以非零的29位的数。
