不赘述B树与B+树的概念了，了解B+树相当于B树的优势，就懂一切。   
## b+树相比于b树的查询优势：

1. b+树的中间节点不保存数据，所以磁盘页能容纳更多节点元素，更“矮胖”；
2. b+树查询必须查找到叶子节点，b树只要匹配到即可不用管元素位置，因此b+树查找更稳定（并不慢）；
3. 对于范围查找来说，b+树只需遍历叶子节点链表即可，b树却需要重复地中序遍历。

前两条很好理解，第三条解释下。需要查找[3,11]，这个区间，图1是B+树的查找过程，图2是B树的查找过程。B树的查找需要从根9出发，遍历左子树，在左1节点上继续遍历子树，左子树遍历结束再遍历右子树，再遍历左子树，右子树。重复的中序遍历。而B+树只需要找到范围的两个边界，然后遍历边界之间的叶子节点。
**中序遍历（LDR）**：二叉树遍历的一种，也叫做中根遍历、中序周游。在二叉树中，中序遍历首先遍历左子树，然后访问根结点，最后遍历右子树。

[![](https://github.com/flysnow911/Blogs/blob/master/imgs/B%2B%E6%A0%91%E6%8C%89%E5%8C%BA%E9%97%B4%E6%9F%A5%E6%89%BE.png)](https://github.com/flysnow911/Blogs/blob/master/imgs/B%2B%E6%A0%91%E6%8C%89%E5%8C%BA%E9%97%B4%E6%9F%A5%E6%89%BE.png)

[![](https://github.com/flysnow911/Blogs/blob/master/imgs/B%E6%A0%91%E6%8C%89%E5%8C%BA%E9%97%B4%E6%9F%A5%E6%89%BE.png)](https://github.com/flysnow911/Blogs/blob/master/imgs/B%E6%A0%91%E6%8C%89%E5%8C%BA%E9%97%B4%E6%9F%A5%E6%89%BE.png)