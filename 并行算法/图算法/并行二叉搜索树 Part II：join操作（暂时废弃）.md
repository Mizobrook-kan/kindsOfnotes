###### 前言

join操作包含再平衡、增广、持久化三个部分，这篇文章深入代码中具体看看每个二叉搜索树变体的join操作执行逻辑。

###### AVL树的join操作

~~AVL树、加权平衡树~~

~~再平衡是一类rank函数，与BST变体的高度相关的一个函数。AVL树的再平衡$\operatorname{rank}(T)=h(T)$。~~

AVL的$\operatorname{rank}(T)=h(T)$的

AVL的join操作根据左树和右树

备注：看看每个BST变体是怎么增广的，其他的我就不关心了

```
static node* node_join(node* t1, node* t2, node* k) {
    // t1的平衡因子大，添加在右侧仍然满足AVL不变量
    if (Node::is_left_heavy(t1, t2)) return right_join(t1, t2, k);
    // t2的平衡因子大，添加在左侧
    if (Node::is_left_heavy(t2, t1)) return left_join(t1, t2, k);
    return balanced_join(t1,t2,k);
}
```

看看其中一段代码

```
static node* left_join(node* t1, node* t2, node* k) {
    if (!Node::is_left_heavy(t2, t1)) return balanced_join(t1, t2, k);
    node* t = GC::copy_if_needed(t2);
    t->lc = left_join(t1, t->lc, k);

    // rebalance if needed
    if (Node::is_left_heavy(t->lc, t->rc)) {
      if (Node::is_single_rotation(t->lc, 1)) t = rotate_right(t);
      else t = double_rotate_right(t);
    } else Node::update(t);
    return t;
  }
```

**加权平衡树的join操作**

加权平衡树的再平衡$\operatorname{rank}(T)=\log _{2}(w(T))-1$。

与AVL树一致，实际的入口都是balance_utils文件下的node_join方法。

```
static inline bool is_left_heavy(node* t1, node* t2) {
      return ratio * weight(t1) > weight(t2);
    }
```

###### 红黑树的join操作

红黑树的再平衡是红树时$\operatorname{rank}(T)=2 \hat{h}(T)-1$、黑树时$\operatorname{rank}(T)=2(\hat{h}(T)-1)$。

```
static node* node_join(node* t1, node* t2, node* k) {
      //right join
	  if (height(t1) > height(t2)) {
		node* t = right_join(t1, t2, k);
  
	    if (t->color == BLACK && color(t->rc) == RED && color(t->rc->rc) == RED) {
			t->rc->rc->color = BLACK;
			t->rc->rc->height++;
			t = t_utils::rotate_left(t);
        } else update(t);
	  
		if (t->color == RED && color(t->rc) == RED) {
		  t->color = BLACK; t->height++;}
		return t;
      }
	  
	  //left join
      if (height(t2) > height(t1)) {
		node* t = left_join(t1, t2, k);

		if (t->color == BLACK && color(t->lc) == RED && color(t->lc->lc) == RED) {
			t->lc->lc->color = BLACK;
			t->lc->lc->height++;
			t = t_utils::rotate_right(t);
        } else update(t);

		if (t->color == RED && color(t->lc) == RED) {
		  t->color = BLACK; t->height++;}
		return t;
      }
	  
	  //balanced join
      if (color(t1) == BLACK && color(t2) == BLACK)
		return balanced_join(t1,t2,k,RED);
      return balanced_join(t1,t2,k,BLACK);
    }
```

rank

###### 树堆的join操作

树堆的再平衡$\operatorname{rank}(T)=\log _{2}(w(T))-1$。

```
static node* node_join(node* t1, node* t2, node* k) {
      if (priority(k) >= priority(t1) && priority(k) >= priority(t2)) {
	k->lc = t1; k->rc = t2;
	Node::update(k);
	return k;
      } else if (priority(t1) > priority(t2)) {
	node* t = GC::copy_if_needed(t1);
	t->rc = node_join(t->rc, t2, k);
	Node::update(t);
	return t;
      } else {
	node* t = GC::copy_if_needed(t2);
	t->lc = node_join(t1, t->lc, k);
	Node::update(t);
	return t;
      }
    }
```

###### ~~可合并的（joinable）~~

~~当键的全序关系和树的中序一致时，称这棵树是一个二叉搜索树。AVL树、红黑树、加权平衡树、树堆（treap）都可以通过增广BST的方式实现。AVL树的增广操作是$\operatorname{rank}(T)=h(T)$，红黑树的增广操作是红树时$\operatorname{rank}(T)=2 \hat{h}(T)-1$、黑树时$\operatorname{rank}(T)=2(\hat{h}(T)-1)$，加权平衡树和树堆通过$\operatorname{rank}(T)=\log _{2}(w(T))-1$增广BST。~~

~~再平衡也是根据增广值来计算。~~

~~**定义1（强可合并树）** 一个平衡树方案S是强可连接的，如果可以对S中的每个子树给出一个rank，在S的两个子树T1、T2上存在一个join(T1,e,T2)，满足以下规则：~~

~~1.空规则：树为空时rank是0~~

~~2.单调规则：对于$C=join(A,e,B)$，$max(rank(A),rank(B) \leq rank(C))$~~

~~3.子模块规则：假设$C=node(A,e,B)$、$C'=join(A',e,B')$。如果$0 \leq rank(A')-rank(A) \leq 0$ 、$0 \leq rank(B')-rank(B) \leq x$ ，那么$0 \leq rank(C')-rank(C) \leq x$ （增长）。如果$0 \leq rank(A)-rank(A')$ 、$0 \leq rank(B)-rank(B')$ ，那么$0 \leq rank(C)-rank(C')$ （下降）~~

~~4.复杂度规则：join($A,e,B$)的时间复杂度是$O(|rank(A)-rank(B)|)$ 。~~

~~5.平衡规则：对于一个节点A，$max(rank(lc(A)),rank(rc(A)))+c_1 \leq rank(A) \leq min(rank(lc(A)),rank(rc(A)))+c_u$，$c_1<=1$、$c_u>=1$且都是常数。~~

~~**定义2（弱可合并树）** 一个平衡树方案S是弱可连接的，如果它满足空规则、单调规则以及子模块规则，且满足如下规则：~~

~~1.宽松平衡规则：存在常数$c_l$、$c_r$、$0<p_l<=1$且$0<p_u<=1$，对节点A以及该节点的任一子节点B满足：~~

~~$rank(B)<=rank(A)-c_l$成立的概率至少为$p_l$~~

~~$rank(B)>=rank(A)-c_u$成立的概率至少为$p_u$~~

~~2.弱复杂度规则：高概率满足join(A,e,B)的时间复杂度为$O(rank(A)+rank(B))$~~

~~**定义3** 可合并树中，节点T位于i层，如果$i<=rank(T)<i+1$~~

~~**定义4** 可合并树中，节点v是rank/rank(i)的根，如果v在i层且v的父节点不在i层。~~

~~**定义5** 在BST中，节点集合V是无祖先的，当且仅当对于$V$中的任意两个节点$v_1$、$v_2$，$v_1$不是$v_2$的祖先。~~

##### ~~path-copying算法~~

~~join操作不是in-place的，在每次更新时都生成一个新树。DBMS的数据库回滚功能需要历史版本。~~

~~数据库的并发查询和更新需要保证一致性，这种情况保留数据库的历史版本对于非阻塞的更新至关重要，这种技术叫多版本并发控制（MVCC）。~~

~~持久化技术是path-copying算法，多个树共享树节点。这样的节点共享需要垃圾收集器实现，~~

~~引用计数的垃圾收集器实现节点共享，~~

~~P-Tree是如何实现持久化的~~

~~path-copying算法的过程~~

~~路径拷贝算法总是在更新树的时候复制树节点~~

~~函数式join，当必要的时候，算法使用旋转让树恢复平衡。~~

~~最终，算法将它们join回来，使用所有复制的节点作为pivot。~~

~~因为~~

~~备注：我把实际的算法代码放过来。~~

**~~重平衡~~**

##### **~~HTAP数据库系统~~**

~~MVCC中每次写事务都创建一个新的版本而不影响旧的版本，所以使用者每次读取旧版本总是得到正确的结果。在SI中，每个事务都仅仅看到了。许多DBMS都支持SI，包括面向磁盘的和内存。~~

~~现代的DBMS开发了一些方法来加速重读的workload，但是更新的代价却相对昂贵。为了在更新和查询方面获得高性能，我们使用P-Tree获得多版本控制和in-memeory数据库存储。路径复制是一种写入时复制（copy-on-write），实现SI。树使用分治方法并行化数据库表的大量操作，包括filter、map、multi-insert、multi-delete、reduce、range query。~~

~~P-Tree支持嵌套的索引，这种嵌套索引改善了OLAP查询性能，仍然允许高效的更新。~~

~~使用纯函数式操作意味着DBMS必须序列化更新到全局视图或组合它们到batch中。~~
