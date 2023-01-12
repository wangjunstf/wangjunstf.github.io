---
title: 红黑树的基本概念和实现
date: 2021-09-07 11:50:28
updated:
categories: [数据结构]
tags: [c++,红黑树,二叉搜索树]
---

本文包含以下内容：

* **二叉树概念**
* **使用 C 语言 实现的二叉查找树的动态集合操作：**
  * **构造：使用二叉树的前序遍历构造二叉树。**
  * **插入，删除，查询**
  * **前驱和后继**
  * **最大值和最小值**
* **红黑树的概念**
<!-- more -->
* **使用 C++ 面向对象思想 实现的红黑树的动态集合操作：**
  * **构造：使用二叉树的前序遍历构造二叉树。**
  * **左旋和右旋**
  * **插入**
  * **颜色修复**
  * **删除**
由于本文较长，可按目录选择自己需要的章节。因为用到初始化列表。 g++ 版本确保支持 c++11，所有代码在以下环境中调试通过。

`系统 : ubuntu 18.04 64位`

`编程语言: c++`

`g++ 版本: g++ 8.4.0`


# 1、前言

二叉查找树是最核心的数据结构之一，是程序员必须了解的数据结构，基于二叉搜索树改进的的 AVL树(自平衡二叉树)和红黑树具有更好的平均性能，更加广泛地被用于从数据结构到数据库等系统，c++ 中的 map、set、multimap、multiset 的底层实现基于红黑树。可以说，理解了二叉查找树，红黑树后，将对数据结构有更深入的理解，也能加深对 c++ 的理解。本博客将从二叉查找树开始讲起，然后过度到红黑树。红黑树也是二叉查找树，只是后者具有更好的性能。具体的差异，听我娓娓道来。



一棵二叉树可以为空，当它不为空时，若它的左子树不为空，它的所有左子树节点都小于根节点，若它的右子树不为空，则它的所有右子树节点值都大于根节点值，它的左子树和右子树分别都是二叉排序树。**二叉搜索树可以快速地进行插入和删除操作，又具有快速查找的能力**，被广泛地用在文件系统或数据库系统，因为这些系统需要高效的检索能力。

<img src="https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E4%B8%8B%E8%BD%BD.png" alt="下载"  />

二叉搜索树可以存储一组有序的序列，如上图所示。通过二叉树的前序遍历，可以从小到大输出所有数据元素。二叉查找树能高效地完成许多动态集合操作，例如：**查找，获取最大值，获取最小值，获取元素的前驱和后继，插入，删除**等。二叉查找树的这些操作和它的高度成正比，对于含有 n 个节点的完全二叉树来说，这些操作的时间复杂度为 O($lg n$​)，最坏情况下，n 个节点的二叉搜索树的深度为 n 时行成了一个单链表，因为需要遍历所有节点，此时这些操作的时间复杂度为 O(n)。

<img src="https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-02%20%E4%B8%8B%E5%8D%8811.57.56.png" alt="截屏2021-09-02 下午11.57.56" style="zoom: 50%;" />



为了克服二叉搜索树的最坏情况，即为单链表的情况，下图为 AVL树(自平衡二叉查找树树)，它的任何节点的两个子树的高度差最大为1。二叉查找树的性能和二叉树的深度成正比，平衡二叉树的深度为 O(lg n)级别，它的查找时间复杂度为 O(lg n)。接下来讲解红黑树，它实现的功能和二叉搜索树一样，但它确有更好的平均性能，因为它近乎平衡二叉树，它的**查找，获取最大值，获取最小值，插入，删除**等操作都能在 O(lg n) 能完成。

<img src="https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-03%20%E4%B8%8A%E5%8D%8812.07.07.png" alt="截屏2021-09-03 上午12.07.07" style="zoom:50%;" />

# 2、二叉搜索树

以下代码，使用 c++ 实现。

二叉树节点的定义如下：

```c++
struct TreeNode {
     int val;
     TreeNode *left;
     TreeNode *right;
     TreeNode *parent;
     TreeNode() : val(0), left(nullptr), right(nullptr),parent(nullptr) {}
     // 下面是一些初始化函数，有些可能不常用
     TreeNode(int x) : val(x), left(nullptr), right(nullptr),parent(nullptr) {}
     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right),parent(nullptr) {}
     TreeNode(int x, TreeNode *left, TreeNode *right,TreeNode * parent) : val(x), left(left), right(right),parent(parent) {}
};

struct BinaryTree   // 二叉树
{
    TreeNode *root;
};
```

二叉树的遍历常见的有以下 3 种，统称为深度优先遍历。

前序遍历：根结点 ---> 左子树 ---> 右子树

中序遍历：左子树---> 根结点 ---> 右子树

后序遍历：左子树 ---> 右子树 ---> 根结点

除了上述 3 种方法外，还有层次遍历方法，称为广度优先遍历。

二叉查找树的 3 种遍历方法，可以使用简单递归思想来实现。

## 2.1 二叉树构造

注意：使用前序遍历可以唯一确定一棵二叉树，但前序和后序遍历不行，因为前序无法确定根节点，后续遍历无法确定左子树节点，但是前序和后续结合起来就可以唯一确定一棵二叉树。

根据前序遍历构造二叉树代码如下：

```c++
TreeNode *treeBuild(vector<int> &tab, int &index, TreeNode *parent)
{
    TreeNode *root = nullptr;
    if(index<tab.size() && tab[index]!=0){
        root = new TreeNode(tab[index]);
        root->parent = parent;
        root->left = treeBuild(tab, ++index,root);
        root->right = treeBuild(tab, ++index,root);
    }
    return root;
}
```

使用中序遍历可以从小到大打印所有元素。

```c++
// 二叉查找树的中序遍历 
void dfs(TreeNode* tree){
  if(tree!=nullptr){
    dfs(tree->left);
    cout<<tree->val<<" ";
    dfs(tree->right);
  }
}
```

通过改变输出节点值代码的位置，可以得到前序遍历和后序列遍历。

```c++
// 二叉查找树的前序遍历 
void dfs(TreeNode* tree){
  if(tree!=nullptr){
    cout<<tree->val<<" ";
    dfs(tree->left);
    dfs(tree->right);
  }
}
```

```c++
// 二叉查找树的后序遍历
void dfs(TreeNode* tree){
  if(tree!=nullptr){
    dfs(tree->left);
    dfs(tree->right);
    cout<<tree->val<<" ";
  }
}
```

## 2.2 查询二叉搜索树

我们有时需要查找一个存储在二叉搜索树的关键字，除了查找外，二叉查找树还支持获取最大值，获取最小值，获取元素的前驱和后继，插入，删除的操作。假设二叉查找树的高度为 h，那么它能在 O(h) 的时间内执行完每个操作。

### 查找

输入一个指向树根的的指针和一个关键字 k，如果这个节点存在，则返回指向关键字为 k 的节点的指针，否则返回 NULL。

以下是递归实现：

```c++
TreeNode *treeSearch(TreeNode *tree, int k){
  if(tree->val==k || tree->val==nullptr){
    return tree->val;
  }
  if(k<tree->val){  // k 小于根节点，说明 k 只可能在左子树上，递归搜索左子树
    return tree-search(tree->left,k);
  }else{
    return tree-search(tree->right,k);
  }
}
```

我们还可以使用循环，即迭代的思想来解决，对于大多数计算机，迭代版本的效率要高得多。通过这个例子，体会一下怎么把递归改为迭代。因为递归层次太多，可能会造成栈溢出。

```c++
TreeNode *treeSearch(TreeNode *tree, int k){
  while(tree!=nullptr && tree->val!=k){
    if(k < tree->val){
      tree = tree->left;
    }else{
      tree = tree->right;
    }
  }
  return tree;
}
```



### 最大关键字元素和最小关键字元素

通过从根开始沿着左孩子指针(左子树)，直到遇到一个 nullptr，我们总能够找到一个元素，这个元素就是这棵二叉查找树的最小元素。同理，从根开始沿着右孩子指针(左子树)，直到遇到一个 nullptr，我们总能够找到一个元素，这个元素就是这棵二叉查找树的最大元素。

```c++
// 获取二叉查找树最小值
TreeNode *treeMinimum(TreeNode *tree){
	while(tree->left!=nullptr){
  	tree = tree->left;
  }
  return tree;
}
```



同理获得最大值如下：

```c++
TreeNode *treeMaxmum(TreeNode *tree){
	while(tree->right!=nullptr){
  	tree = tree->right;
  }
  return tree;
}
```



### 前驱

在一棵高度为 h 的树上，treeSuccessor 的运行时间为 O(h)，因为该过程只是简单地沿树向上或沿树向下。求前驱过程treePredecessor是对称的，运行时间也是O(h)。

给定一棵二叉搜索树中的一个节点 tree，按中序遍历的次序查找它的前驱，如果所有的关键字都不相同，则一个节点的前驱是小于tree->val 的最大关键字节点。**使用前序遍历**

当二叉树的中序遍历为2 3 4 6 7 9 13 15 17 18 20 ，元素 `7`的前驱为6，后继为9

```c++
TreeNode *treePredecessor(TreeNode *tree){
  if(tree->left!=nullptr){
    return treeMaxmum(tree->left);
  }
  TreeNode *y = tree->parent;
  while(y!=nullptr && y->left!=tree){
    tree = y;
    y = y->parent;
  }
  return y;
}
```

这段代码也分两种情况：

* 左子树不为空，那么该节点的前驱就是左子树的最大节点。
* 左子树为空，此时 tree 沿树而上，直到遇到一个双亲有右孩子的节点。



### 后继

给定一棵二叉搜索树中的一个节点 tree，有时候需要按中序遍历的次序查找它的后继，如果所有的关键字都不相同，则一个节点的后继是大于tree->val 的最小关键字节点。

例如当

下例函数返回二叉查找树某节点的后继：**使用中序遍历**

```c++
TreeNode *treeSuccessor(TreeNode *tree){
  if(tree->right!=nullptr){
    return treeMinimum(tree->right);
  }
  TreeNode *y = tree->parent;
  while(y!=nullptr && tree==y->right){
    tree = y;
    y = y->parent;
  }
  return y;
}
```

上述代码分别处理两种情况：

* 右子树不为空，此时 tree 的后继就是右子树中最小节点，调用 treeMinimum(tree->right)即可获得
* 右子树为空，此时 tree 沿树而上，直到遇到一个双亲有左孩子的节点。

用下列二叉搜索树来测试代码

<img src="https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-03%20%E4%B8%8B%E5%8D%885.21.27.png" alt="截屏2021-09-03 下午5.21.27" style="zoom:50%;" />

### 测试代码

```c++
#include <iostream>
#include <vector>

using namespace std;
struct TreeNode
{
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode *parent;
    TreeNode() : val(0), left(nullptr), right(nullptr), parent(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr), parent(nullptr) {}
};

TreeNode *treeBuild(vector<int> &tab, int &index, TreeNode *parent)
{
    TreeNode *root = nullptr;
    if(index<tab.size() && tab[index]!=0){
        root = new TreeNode(tab[index]);
        root->parent = parent;
        root->left = treeBuild(tab, ++index,root);
        root->right = treeBuild(tab, ++index,root);
    }
    return root;
}

TreeNode *treeSearch(TreeNode *tree, int k)
{
    if (tree->val == k || tree == nullptr)
    {
        return tree;
    }
    if (k < tree->val)
    { // k 小于根节点，说明 k 只可能在左子树上，递归搜索左子树
        return treeSearch(tree->left, k);
    }
    else
    {
        return treeSearch(tree->right, k);
    }
}


void inOrder(TreeNode *tree)
{
    if (tree == nullptr){
        return;
    }
    inOrder(tree->left);
    cout << tree->val << " ";
    inOrder(tree->right);
}

// 获取二叉查找树最小值
TreeNode *treeMinimum(TreeNode *tree)
{
    while (tree->left != nullptr)
    {
        tree = tree->left;
    }
    return tree;
}

TreeNode *treeMaxmum(TreeNode *tree)
{
    while (tree->right != nullptr)
    {
        tree = tree->right;
    }
    return tree;
}

TreeNode *treeSuccessor(TreeNode *tree)
{
    if (tree->right != nullptr)
    {
        return treeMinimum(tree->right);
    }
    TreeNode *y = tree->parent;
    //cout << "y->val: " << y->val << endl;
    while (y != nullptr && tree == y->right)
    {
        tree = y;
        y = y->parent;
        //cout <<"y->val: "<<y->val<<endl;
    }
    return y;
}

TreeNode *treePredecessor(TreeNode *tree)
{
    if (tree->left != nullptr)
    {
        return treeMaxmum(tree->left);
    }
    TreeNode *y = tree->parent;
    while (y != nullptr && y->left != tree)
    {
        tree = y;
        y = y->parent;
    }
    return y;
}

int main(){
    vector<int>tab{15,6,3,2,0,0,4,0,0,7,0,13,9,0,0,0,18,17,0,0,20,0,0};
    int index=0;
  	BinaryTree *T = new BinaryTree;
    T->root = treeBuild(tab,index,nullptr);
    cout<<"中序遍历："<<endl;
    inOrder(root);
    cout<<endl;
    cout<<"maxmum: "<<treeMaxmum(T->root)->val<<endl;
    cout << "minimum: " << treeMinimum(T->root)->val << endl;
    int num = 15;
    TreeNode *p = treeSearch(T->root,num);
    cout << num << " 的前驱: " << treePredecessor(p)->val << endl;
    cout << num << " 的后继: " << treeSuccessor(p)->val<<endl;
    return 0;
}
```

```shell
$ ./tree
中序遍历：
2 3 4 6 7 9 13 15 17 18 20 
maxmum: 20
minimum: 2
15的前驱: 13
15的后继: 17
```



### 总结

在一棵高度为 h 的二叉搜索树上，动态集合的操作：treeSearch,treeMinimum,treeMaxmum,treeSuccessor,treePredecessor 的时间复杂度为 O(h)



## 2.3 插入和删除

插入和删除会引起由二叉树表示的动态集合的变化。一定要修改数据结构来反应这个变化，该修改要保持二叉搜索树性质的成立。插入一个新节点带来的树修改要简单些，而删除的处理要复杂一些。



### 插入

将一个新值 v 插入到一棵二叉搜索树 T 中，需要调用 treeInsert，该过程以节点 z 作为输入。其中 `z.val=v，z.left=nullptr, z.right=nullptr,z.parent=nullptr`，这个过程要修改 T 和 z 的某些属性来把 z 插入到树中相应的位置。

```c++
void treeInsert(TreeNode *root, TreeNode *z)
{
    TreeNode *y = nullptr;
    TreeNode *x = root;
    while (x != nullptr)
    {
        y = x;
        if (z->val < x->val)
            x = x->left;
        else
            x = x->right;
    }
    z->parent = y;
    if (y == nullptr) // 如果树为空树
        root = z;
    else if (z->val < y->val)
        y->left = z;
    else
        y->right = z;
}
```

与其他搜索树上的原始操作一样，过程treeInsert在一棵高度为 h 的树上的运行时间为 O(h)



### 删除

从一棵二叉树 T 中删除一个节点 z 需要考虑以下 3 种情况，但只有一种最棘手。

1. 如果 z 没有孩子节点，那么只需简单地将它删除，并需改它的父节点，用`nullptr`作为孩子节点来替换 z

2. 如果 z 只有一个孩子，那么将这个孩子提升到树中 z 的位置，并修改 z 的父节点，用 z 的孩子来替换 z

如果 z 有两个孩子，那么找 z 的后继y(一定在 z 的右子树中，且没有左孩子)，并让 y 占据树中 z 的位置。z 的原来右子树的部分成为新的右子树，并且 z 的左子树成为 y 的新的左子树

**注意：如果一棵二叉搜索树的一个节点有两个孩子，那么它的后继没有左孩子，它的前驱没有右孩子。**

3. 如果 y 是 z 的右孩子，那么用 y 替换 z，并留下 y 的右孩子，y 没有左孩子

4. 如果 y 不是 z 的右孩子，y 位于 z 的右子树但并不是 z 的右孩子。在这种情况下，先用 y 的右孩子替换 y，然后再用 y 替换 z。



针对上述 4 种情况，下面依次用图片讲解：

从以下二叉树中，删除节点 z 

图(a) 和 图(b) 为 第 2 种情况：

<img src="https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-03%20%E4%B8%8B%E5%8D%884.31.57.png" alt="截屏2021-09-03 下午4.31.57" style="zoom:50%;" />



<img src="/Users/wangjun/Library/Application Support/typora-user-images/截屏2021-09-03 下午4.33.19.png" alt="截屏2021-09-03 下午4.33.19" style="zoom:50%;" />



图(c) 为第 3 种情况：

<img src="https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-03%20%E4%B8%8B%E5%8D%884.35.06.png" alt="截屏2021-09-03 下午4.35.06" style="zoom: 67%;" />

图(d) 为第 4 种情况：

<img src="https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-03%20%E4%B8%8B%E5%8D%884.36.07.png" alt="截屏2021-09-03 下午4.36.07" style="zoom: 67%;" />



### 子树替换

为了在二叉搜索树内移动子树，定义一个子过程 transplant，它是用另一棵子树替换一棵子树并成为其双亲的孩子节点。例如：当 transplant用一棵以 v 为根的子树来替换一棵以 u 为根的子树时，节点 u 的双亲就变为节点 v 的双亲，并且最后 v 成为 u的双亲的相应孩子。

```c++
// 用 v 子树替换 u 子树，不能改变 u 子树
void transplant(BinaryTree *T, TreeNode *u, TreeNode *v)
{
    // 用 v 子树来替换 u 子树，该函数允许 v 为空的情况
    if (u->parent == nullptr){
        // 处理 u 是 T 的树根的情况
        // 注：如果参数传递的是 TreeNode *root,root = v，并不能工作，因为此时root为局部变量
        T->root =v;
    } else if (u == u->parent->left)
    { 
      	// 如果 u 为右孩子
        u->parent->left = v;
    }
    else
    {
        u->parent->right = v;
    }
    if (v != nullptr)
    {
        v->parent = u->parent;
    }
}
```

```c++
void treeDelete(BinaryTree *T, TreeNode *z)
{
    TreeNode *y = nullptr;
    if(z->left==nullptr)
        transplant(T,z,z->right);
    else if(z->right==nullptr)
        transplant(T,z,z->left);
    else {
        y = treeMinimum(z->right);
        if(y->parent!=z){
            transplant(T,y,y->right);
            y->right = z->right;
            y->right->parent = y;
        }
        transplant(T,z, y);
        y->left = z->left;
        y->left->parent = y;
    }
}

```

<img src="https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-03%20%E4%B8%8B%E5%8D%885.21.271.png" alt="截屏2021-09-03 下午5.21.27" style="zoom:50%;" />

下面用上图所示二叉搜索树来测试二叉树的插入和删除。

```c++
#include <iostream>
#include <vector>

using namespace std;
struct TreeNode
{
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode *parent;
    TreeNode() : val(0), left(nullptr), right(nullptr), parent(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr), parent(nullptr) {}
};

struct BinaryTree   // 二叉树
{
    TreeNode *root;
};

TreeNode *treeBuild(vector<int> &tab, int &index, TreeNode *parent)
{
    TreeNode *root = nullptr;
    if(index<tab.size() && tab[index]!=0){
        root = new TreeNode(tab[index]);
        root->parent = parent;
        root->left = treeBuild(tab, ++index,root);
        root->right = treeBuild(tab, ++index,root);
    }
    return root;
}


TreeNode *treeSearch(TreeNode *tree, int k)
{
    if (tree->val == k || tree == nullptr)
    {
        return tree;
    }
    if (k < tree->val)
    { // k 小于根节点，说明 k 只可能在左子树上，递归搜索左子树
        return treeSearch(tree->left, k);
    }
    else
    {
        return treeSearch(tree->right, k);
    }
}


void dfs(TreeNode *tree)
{
    if (tree == nullptr){
        return;
    }
    dfs(tree->left);
    cout << tree->val << " ";
    dfs(tree->right);
}

// 获取二叉查找树最小值
TreeNode *treeMinimum(TreeNode *tree)
{
    while (tree->left != nullptr)
    {
        tree = tree->left;
    }
    return tree;
}

TreeNode *treeMaxmum(TreeNode *tree)
{
    while (tree->right != nullptr)
    {
        tree = tree->right;
    }
    return tree;
}

TreeNode *treeSuccessor(TreeNode *tree)
{
    if (tree->right != nullptr)
    {
        return treeMinimum(tree->right);
    }
    TreeNode *y = tree->parent;
    //cout << "y->val: " << y->val << endl;
    while (y != nullptr && tree == y->right)
    {
        tree = y;
        y = y->parent;
        //cout <<"y->val: "<<y->val<<endl;
    }
    return y;
}

TreeNode *treePredecessor(TreeNode *tree)
{
    if (tree->left != nullptr)
    {
        return treeMaxmum(tree->left);
    }
    TreeNode *y = tree->parent;
    while (y != nullptr && y->left != tree)
    {
        tree = y;
        y = y->parent;
    }
    return y;
}

void treeInsert(TreeNode *root, TreeNode *z)
{
    TreeNode *y = nullptr;
    TreeNode *x = root;
    while (x != nullptr)
    {
        y = x;
        if (z->val < x->val)
            x = x->left;
        else
            x = x->right;
    }
    z->parent = y;
    if (y == nullptr) // 如果树为空树
        root = z;
    else if (z->val < y->val)
        y->left = z;
    else
        y->right = z;
}

void transplant(BinaryTree *T, TreeNode *u, TreeNode *v)
{
    // 用 v 子树来替换 u 子树，该函数允许 v 为空的情况
    if (u->parent == nullptr){
        // 处理 u 是 T 的树根的情况
        //root = v;
        T->root =v;
    } else if (u == u->parent->left)
    { // 如果 u 为右孩子
        u->parent->left = v;
    }
    else
    {
        u->parent->right = v;
    }
    if (v != nullptr)
    {
        v->parent = u->parent;
    }
}

void treeDelete(BinaryTree *T, TreeNode *z)
{
    TreeNode *y = nullptr;
    if(z->left==nullptr)
        transplant(T,z,z->right);
    else if(z->right==nullptr)
        transplant(T,z,z->left);
    else {
        y = treeMinimum(z->right);
        if(y->parent!=z){
            transplant(T,y,y->right);
            y->right = z->right;
            y->right->parent = y;
        }
        transplant(T,z, y);
        y->left = z->left;
        y->left->parent = y;
    }
    z->left=nullptr;
    z->right=nullptr;
    z->parent=nullptr;
}




int main(){
    vector<int>tab{15,6,3,2,0,0,4,0,0,7,0,13,9,0,0,0,18,17,0,0,20,0,0};
    int index=0;
    BinaryTree *T = new BinaryTree;
    T->root = treeBuild(tab,index,nullptr);
    cout<<"原二叉搜索树(中序遍历)：";
    inOrder(T->root);
    cout<<endl;
  

    TreeNode *pdel1 = treeSearch(T->root, 6);
    TreeNode *pdel2 = treeSearch(T->root, 18);
    TreeNode *pdel3 = treeSearch(T->root, 15);

    cout << "删除节点6 ";
    treeDelete(T,pdel1);
    cout << "中序遍历：";
    dfs(T->root);
    cout << endl;

    cout << "删除节点18 " ;
    treeDelete(T, pdel2);
    cout << "中序遍历：";
    dfs(T->root);
    cout << endl;

    cout << "删除节点15 ";
    treeDelete(T, pdel3);
    cout << "中序遍历：";
    dfs(T->root);
    cout << endl;

    cout << "插入节点15 ";
    treeInsert(T->root, pdel3);
    cout << "中序遍历：";
    dfs(T->root);
    cout << endl;

    cout << "插入节点18 ";
    treeInsert(T->root, pdel2);
    cout << "中序遍历：";
    dfs(T->root);
    cout << endl;

    cout << "插入节点6 ";
    treeInsert(T->root, pdel1);
    cout << "中序遍历：";
    dfs(T->root);
    cout << endl;

    return 0;
}
```

```shell
$ ./tree
原二叉搜索树(中序遍历)：2 3 4 6 7 9 13 15 17 18 20 
删除节点6 中序遍历：2 3 4 7 9 13 15 17 18 20 
删除节点18 中序遍历：2 3 4 7 9 13 15 17 20 
删除节点15 中序遍历：2 3 4 7 9 13 17 20 
插入节点15 中序遍历：2 3 4 7 9 13 15 17 20 
插入节点18 中序遍历：2 3 4 7 9 13 15 17 18 20 
插入节点6 中序遍历：2 3 4 6 7 9 13 15 17 18 20 
```



二叉查找树的插入和删除时间复杂度都为 O(h)，h 为树的高度。



## 2.4 随机构建二叉搜索树

上述内容已经说明，二叉搜索树上的每个基本操作都能在O(h)时间内完成，其中 h 是这棵树的高度。随着元素的插入和删除，二叉搜索树的高度是变化的。例如，当 n 个关键子按严格递增的次序被插入，则这棵树的高度为 n-1 的一条链子，这是二叉搜索树性能最低的情况。**但是，和快速排序一样，我们可以证明平均情形更接近最好情况，而不是最坏情况。**

**一棵有 n 个不同关键字的随机构建二叉搜索树的期望高度为 O(lg n)**



# 3、红黑树

通过上面的讲解，我们知道二叉搜索树支持任何一种基本动态集合操作。例如查找(search)，前驱(predecessor)，后继(successor)，最小值(minimum)，最大值(maxmum)，插入(insert)，删除(delete)等。其时间复杂度都是 O(h)，h 为树的高度

。红黑树是许多“平衡”搜索树的一种，可以保证在最坏情况下基本动态集合的时间复杂度为O(ln n)。



## 3.1 红黑树的性质

红黑树每个节点包含 5 个属性: color, key, left, right 和 p。分别表示颜色，节点值，左孩子，右孩子和父亲节点。如果一个节点没有子节点和父节点，则该节点称为外部节点，该节点只作标记，不存储实际值。我们把存储实际值的节点称为内部节点。

一棵红黑树是满足下面**红黑性质**的二叉搜索树：

1. **每个节点或是红色，或是黑色的。**

2. **根节点是黑色的**

3. **每个叶节点(NIL)是黑色的。**

4. **如果一个节点是红色的，则它的两个子节点都是黑色的。**

5. **对每个节点，从该节点到其所有后代叶节点的简单路径上，均包含相同数目的黑色节点。**

为了便于处理红黑树代码中的边界条件，使用一个哨兵来代表 NIL。哨兵和内部节点一样，都具有 5 个属性，不同的是它的 color 为黑色，其他属性为随机值。入下图所示：

![截屏2021-09-03 下午9.45.06](https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-03%20%E4%B8%8B%E5%8D%889.45.06.png)



上面 3 幅都表示同一棵红黑树，图(b) 表示所有NIL节点用同一个节点表示，更节省空间。图(c) 省略了 NIL 节点。

**从某个节点 x 出发(不含该节点)到达一个叶节点的任意一条简单路径上的黑色节点个数称为该节点的黑高(black-height)，记作bh(x)。**

记住一个结论：**一棵有 n 个内部节点的红黑树树的高度至多为 2lg(n+1)**，更多请参考《算法导论》原书第 3 版 308页。

设 h 为树的高度，根据性质 4 ，从根到叶节点(不包括根节点)的任何一条简单路径上都至少有一半的节点为黑色。因此，根的黑高至少是 h/2。于是有 n >= 2<sup>k/2</sup>-1，把 1 移到不等式的左边，再对两边取对数得到 lg(n+1) >=h/2，或者h<=2lg(n+1)。由此可知**(search)，前驱(predecessor)，后继(successor)，最小值(minimum)，最大值(maxmum)，插入(insert)，删除(delete)等动态集合操作的时间复杂度为 O(lgn)**



因为涉及到代码，二叉搜索树用的是 c 语言代码，为了使代码结构更加清晰，下面红黑树使用 c++ 面向对象思想来编写：

下面为红黑树类 及 红黑树对象的结构定义：

```c++
#ifndef _RBTree_H_
#define _RBTree_H_

#include <vector>
#include <iostream>
#include <utility>
using namespace std;

enum E_COLOR
{
    BLACK,
    RED
};

struct TreeNode
{
    int key;
    E_COLOR color;
    TreeNode *left;
    TreeNode *right;
    TreeNode *p;

    TreeNode() : key(0), left(nullptr), right(nullptr), p(nullptr) {}
    TreeNode(int key) : key(key), left(nullptr), right(nullptr), p(nullptr) {}
    TreeNode(E_COLOR c) : key(0),color(c),left(nullptr), right(nullptr) {}
    TreeNode(int x, E_COLOR c) : key(x), left(nullptr), right(nullptr), p(nullptr), color(c) {}
    TreeNode(int x, E_COLOR c, TreeNode *left, TreeNode *right, TreeNode *p) : key(x), color(c),left(left), right(right), p(p) {}
};

struct RBTREE
{
    TreeNode *root; // 红黑树头节点
    TreeNode *NIL;
};

class RBTree{
public:
    RBTREE *rbtree;                  // 红黑树对象
    RBTree():rbtree(nullptr){};      // 默认构造函数

    // 使用中序遍历构造红黑树，vector的元素为 pair 类型（键值对），key 为元素值，value 为 红黑值
    RBTree(vector<pair<int,E_COLOR>> &inorder); 

    void preOrder(TreeNode *root);                  // 前序遍历二叉树
    void inOrder(TreeNode *root);                   // 中序遍历二叉树
    TreeNode* search(TreeNode *tree, int k);
    TreeNode* minimum(TreeNode *tree);
    TreeNode* maximum(TreeNode *tree);
    TreeNode* successor(TreeNode *tree);
    TreeNode* predecessor(TreeNode *tree);
    void leftRotate(RBTREE *T, TreeNode *x);        // 左旋
    void rightRotate(RBTREE *T, TreeNode *x);       // 右旋

    ~RBTree(){};
private: 
    TreeNode *rbBuild(vector<pair<int, E_COLOR>> &inorder, int &index, TreeNode *p);
};
```



## 3.2 旋转

在插入和删除节点之后，可能会破坏红黑性质，为了维护这些性质，必须改变某些节点的颜色和指针结构。

指针结构的修改是通过`旋转`来完成的，这是一种能保持二叉搜索树性质的搜索树局部操作。分为左旋和右旋。

如下图所示：从右到走称为左旋，其中 x 为其右孩子不是 T.NIL 节点树内任意节点。

![截屏2021-09-03 下午10.18.13](https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-03%20%E4%B8%8B%E5%8D%8810.18.13.png)

左旋和右旋是互为对称的操作。

下面演示左旋，假设 `x->right!=T.nil 且根节点的父节点为 T.nil`

```c++
void 
RBTree::leftRotate(RBTREE *T, TreeNode *x){
  	// T 为红黑树对象 
    // 假设 x.right != NIL 且根节点的父节点为 NIL
    TreeNode *y = x->right;  
    x->right = y->left;           // 将 y 的左子树作为 x 的右子树
    if(y->left!=T->NIL){          
        y->left->p = x;
    }
    y->p = x->p;                  // 设置 y 的父节点
    if(x->p==T->NIL)              // 如果 x 为根节点，则将 y 作为根节点
        T->root = y;
    else if (x==x->p->left)       // 否则的话，判断 x 为左节点还是右节点
        x->p->left = y;
    else 
        x->p->right = y;
    y->left = x;                  // 将 x 作为 y 的 左孩子
    x->p = y;                     // 这样就完成了 左旋转，二叉树搜索树的性质不变
}
```



右旋：

```c++
void 
RBTree::rightRotate(RBTREE *T, TreeNode *x){
    // 假设 x 的左节点不等于 NIL
    TreeNode *y =  x->left;    // y->left 可以作为空节点使用
    x->left = y->right;        // 用 x 的右孩子替代 y 的左孩子
    if(y->right!=T->NIL){
        y->right->p = x;
    }

    if(x->p==T->NIL){
        T->root = y;
    }else if(x->p->left==x){
        x->p->left = y;
    }else{
        x->p->right = y;
    }
    y->right = x;
    x->p = y;

}
```



## 3.3 插入

插入一个节点后，将该节点的左孩子，和右孩子设置为 NIL，颜色设置为红色。因为该节点可能会破坏红黑特性，还需要一个函数来对红黑树进行调整，以满足该二叉搜索树的红黑特性。

首先看插入函数：插入函数和二叉排序树的插入差不多，唯一的区别是所有叶子节点的左右孩子指针指向 NIL，根节点的父指针指向 NIL，最后 4 行代码给新添加的节点上色

```c++
void 
RBTree::insert(RBTREE *T, TreeNode *z)
{
    TreeNode *y = T->NIL;
    TreeNode *x = T->root;
    while (x != T->NIL)
    {
        y = x; // 记录父节点
        if (z->key < x->key)
        {
            x = x->left;
        }
        else
        {
            x = x->right;
        }
    }
    z->p = y;
    if (y == T->NIL)
    {
        T->root = z;
    }
    else if (z->key < y->key)
    {
        y->left = z;
    }
    else
    {
        y->right = z;
    }
    z->color = RED;
    z->left = T->NIL;
    z->right = T->NIL;
    insertFixup(T,z);
}
```

`void  RBTree::insertFixup(RBTREE *T, TreeNode *z)`

新插入的节点可能会破坏某些红黑性质，根据出现的情况需要进行不同的操作。在写代码之前，我们需要知道在调用 insertFixup 时哪些红黑性质可能会被破坏。性质1 和性质3 成立，因为新插入的红节点的两个子节点都是哨兵 NIL，性质 5 也成立，因为即使为空树，因为根节点的父节点为 NIL，NIL 为黑色节点。

综上所述：可能被破坏的只有性质 2 和性质 4，即根节点需要为黑色，以及一个红节点不能有红孩子。这两个性质被破坏是因为 z 被着为红色，

如果 z 是根节点，则破坏了性质2；

如果 z 的父节点是红节点，则破坏了性质 4；

循环的结束条件为 z 的父节点为黑色，如果 z 的父节点一直为红，则一直循环。在循环体内，我们根据不同的情况作相应的调整：**记住只要在循环体内，z 和 z 的父节点都为红色**

根据 z 的父节点作为左子树还是右子树分为两大类

（1）z 的父节点作为左子树

### 情况1 

![截屏2021-09-04 下午9.31.48](https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-04%20%E4%B8%8B%E5%8D%889.31.48.png)



在情况1下，**z为红色，z 的父节点为红色，z 的叔节点为红色时**，此时为第 1 种情况，如上图所示。需要要z 的父节点和叔节点调整为黑色，z 的祖父节点调整为红色，并让 z 想上移动两层。如下代码：

```c++
z->p->color = BLACK;
y->color = BLACK;
z->p->color=RED;
z = z->p->p;
```

z 向上爬两层。得到下图：

### 情况2 

![截屏2021-09-04 下午9.37.48](https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-04%20%E4%B8%8B%E5%8D%889.37.48.png)



在情况 2 下， **z 为红色 ，z 的父节点为红色，z 的叔节点为黑色，z 此时为右节点，**将 z 上爬一层，然后将 z 左旋转，这样就可以让 z 满足性质4了。左旋之后得到下图：

操作代码如下：

```c++
z = z->p;
leftRotate(T,z);
```

### 情况3

![截屏2021-09-04 下午9.44.06](https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-04%20%E4%B8%8B%E5%8D%889.44.06.png)



在情况3下，**z 为红色，z的父节点为红色，z的叔节点为黑色，z 此时为左节点**

将 z 的父节点改为黑色，将 z 的祖辈节点 改为红色，本次修改破坏了性质5，因为此时z 所在子树的黑节点多了一个，而 z 的叔节点所在子树黑节点少了一个，此时将 z 的祖父节点右旋，就能让 z 所在子树的黑节点变少一个，因为计算黑高并不包括根节点。



![截屏2021-09-04 下午9.55.46](https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-04%20%E4%B8%8B%E5%8D%889.55.46.png)

此时 z 的父节点已经为黑色，故推出循环。且整棵树符合红黑树的所有性质。



**注意：情况2和情况3的唯一区别就是，z 作为右子树还是作为左子树，如果位于右子树，就需要多一步旋转的操作，让 z 位于左子树上。最终还是来到情况3。 **

z 的父节点作为右子树

此时和（1）是对称的，只需将(1) 中的相应的左和右进行交换即可：

完整代码如下：

```c++
void 
RBTree::insertFixup(RBTREE *T, TreeNode *z){
    while(z->p->color == RED){
        // 第一大类 z 的父节点作为左孩子
        if(z->p == z->p->p->left){        
            TreeNode *y = z->p->p->right;   // z 的叔节点
            if(y->color==RED){
                // case1: 叔叔节点是红色
                z->p->color = BLACK;
                y->color = BLACK;
                z->p->p->color=RED;
                z = z->p->p;
                continue;
            }

            // case2: 叔叔是黑色 且当且节点是右孩子
            if(z==z->p->right){
                
                z = z->p;
                leftRotate(T,z);
            }

            // case3: 叔叔是黑色，且当亲节点是左孩子
            z->p->color = BLACK;
            z->p->p->color = RED;
            rightRotate(T,z->p->p);
        }else { // 第二大类 z 的父节点作为右孩子
            TreeNode *y = z->p->p->left;

            // case1: 叔叔节点为红色
            if (y->color == RED)
            {
                z->p->color = BLACK;
                y->color = BLACK;
                z->p->p->color = RED;
                z = z->p->p;
                continue;
            }
            
            // case2: 叔叔节点为黑色 且当前节点是左孩子
            if(z==z->p->left){
                z = z->p;
                rightRotate(T,z);
            }

            // case3: 叔叔节点为黑色 且当前节点是右孩子
            z->p->color = BLACK;
            z->p->p->color = RED;
            leftRotate(T, z->p->p);
        }
    }
    T->root->color = BLACK;     // 将根节点设置为黑色。
}
```



### 时间复杂度分析

含有 n 个节点的红黑树的高度为O(lg n)，因此**红黑树的插入部分`insert`时间复杂度为 O(lg n)**时间，在颜色修正部分，仅当情况 1 发生，然后指针沿树上升 2 层，while 循环才会重复执行，所以 while 循环被执行的总次数为 O(lg n)，因此**`insertFixup`的时间复杂度为 O(lg n)**。注意：该程序所做的旋转操作不会超过两次，因为只要执行了情况2 或 情况3，while 循环就结束了。



## 3.4 删除

红黑树的删除会比插入更复杂一些，因为删除一个节点可能会破坏某些红黑性质。红黑树的删除代码和二叉搜索树的删除代码差不多，主要的区别稍后给出：

```c++
void 
RBTree::remove(RBTREE *T, TreeNode *del){
    TreeNode *orig = del;               // del 为需要移除的节点  orig 记录 del 的节点，或即将取代 del 的节点信息
    E_COLOR orig_color = del->color;    // orig_color del 的颜色 或 取代 del 的节点颜色
    TreeNode *replace = nullptr;        // replace 为删除 del 之后，取代它的点 或者 orig 节点仅有的的右孩子
    if(del->left==T->NIL){
        replace = del->right;
        transplant(T,del,del->right);
    }else if(del->right==T->NIL){
        replace = del->left;
        transplant(T,del,del->left);
    }else {
        orig = minimum(del->right);    // 获取 del 右子树最小节点，该节点一定无左子树，删除 del 节点，用 orig 节点取代
        orig_color = orig->color;
        replace = orig->right;         // 记录 orig  节点仅有的的右孩子
        if (orig->p != del)            // orig 并不是 del 的直接孩子节点，否则直接取代
        {
            transplant(T, orig, orig->right);      // 用 orig 的右子树替代 orig
            orig->right = del->right;              // del 的右子树赋值给 origin 的右子树
            orig->right->p = orig;
        }
        transplant(T, del, orig);                  // 用 origin 替换 del
        orig->left = del->left;                    // 将 del 的左子树替代 ori 的左子树
        orig->left->p = orig;
        orig->color = del->color;                  // 将已经删除节点的颜色 del 赋值给 orig
    }

    if (orig_color == BLACK)         
    {
        removeFixup(T,replace);
    }
}
```

主要区别：

* 始终维持 orig 从树中删除的节点或者移到树内的节点。当 del 只有一个子节点时是前者，当  del 有两个子节点时为后者。
* 因为最终要让 orig 的节点颜色等于 del。orig_color 记录 orig 发生改变前的颜色。如果 orig_color 是黑色，则删除或移动 orig 会引起红黑性质的破坏。
* replace 则保存 orig 的唯一子节点，或 NIL(orig 没有子节点)

**如果orig_color 为红色，红黑性质仍然保存，原因如下：**

1. 树中黑高没有变化

2. 不会存在两个相邻的红节点。如果 orig_color 为红色，则 orig 的子节点 replace 都为黑色， replace 用于替换 原 orig 位置。
3. 如果 orig_color 为红色，其不可能是黑节点。



**相反如果orig_color是黑色，则会导致以下问题 **

1. 如果 orig 为原来的根节点，而 orig 的一个红色的孩子成为了新的根节点，违反了性质2。
2. 如果 orig 和 orig 的父节点都是红色，则违反了性质4。
3. 在树中移动 orig 将导致先前包含 y 的任何简单路径上黑节点个数少1，在这种假设下性质 5 成立。



`orig_color 为黑色，会导致所有含有 orig 节点的简单路径少了一个黑色节点。orig 的替代节点为 replace，如果 replace 为红色，则把replace 设置为黑色即可。如果 replace 为黑色，则需要为含有orig节点的简单路径增加一个黑色节点，而且不能破坏红黑性质。   `

当被删除元素为黑色或后继为黑色时，需要调用`removeFixup`来修复红黑树的颜色。

修复颜色，总共有 4 种情况，额外的 4 种是对称的。

`removeFixup(T,replace);`该函数调用中，replace 在 removeFixup 用 x 来表示，x 的兄弟节点用 w 表示。

现在讨论的是 x 作为左节点的情况：

**orig_color为黑色，x 也为黑色，需要将 x 看作一个双重黑色看待，需要一步步往上找到一个红色节点，并将其置为红色。如下图：**

<img src="https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-07%20%E4%B8%8A%E5%8D%8812.11.38.png" alt="截屏2021-09-07 上午12.11.38" style="zoom:50%;" />

**如果orig 为黑， x 为红色，则它作为 红黑色看待，只需将其置为红色即可**

![截屏2021-09-07 上午12.20.58](https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-07%20%E4%B8%8A%E5%8D%8812.20.58.png)



### 情况1

x 的兄弟节点 w 为红色，此时 w 的两个子节点都为黑色(性质4)，如下图所示：

![截屏2021-09-07 上午12.03.01](https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-07%20%E4%B8%8A%E5%8D%8812.03.01.png)

操作：将 w 置为黑色，x 的父节点置为红色，这样 w 所在子树的黑色节点就和左子树的黑色节点个数相等。但 x 的父节点所在的子树少了一个黑色节点，将 x 向上移动一层，并将它作为双重黑色看待。通过情况1，就转换为情况2，3，4了。



**以下情况都表示 x的兄弟节点 w 为黑色：**

### 情况2

w 的两个孩子节点都为黑色，如下图所示：

![截屏2021-09-07 上午12.24.02](https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-07%20%E4%B8%8A%E5%8D%8812.24.02.png)

需要从 x 和 w 上去掉一层黑色，使得 x 只有一层黑色， w 为红色。为了补偿从 x 和 w 上去掉的一重黑色，在原来是红色或黑色的 x 的父节点上新增一重黑色。然后让 x = x->p，来循环 x，直到不满足情况 2 的条件，就进入情况3 和 4



### 情况 3

x 的兄弟节点是黑色的，w 的左孩子是红色的，w 的右孩子是黑色的。如下图所示：

![截屏2021-09-07 上午12.30.54](https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-07%20%E4%B8%8A%E5%8D%8812.30.54.png)



交换 w 和其左孩子的颜色，然后对 w 进行右旋转，从而不违反任何红黑性质。现在 x 的新兄弟节点 w 是黑色节点，并且 w 的右孩子是红色的，这样我们就由情况 3 进入情况 4。



### 情况 4

x 的兄弟节点 w 是黑色的，且 w 的右孩子是红色的。如下图所示：

![截屏2021-09-07 上午12.36.24](https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-09-07%20%E4%B8%8A%E5%8D%8812.36.24.png)



将 w 的颜色 置为  x 的父节点的颜色，将 x 的父节点置为黑色，将 w 的右孩子节点置为黑色，对 x 的父节点进行左旋转。这样就去掉了 x 的额外一重黑色。

代码如下：

```c++
void 
RBTree::removeFixup(RBTREE *T, TreeNode *x){
    while(x!=T->NIL && T->root && x->color==BLACK){
        if(x == x->p->left){
            TreeNode *w = x->p->right;
            if(w->color==RED){
                // 情况1 x 的兄弟节点为红色

                // 交换 w 和 w 的父节点的颜色
                w->color = BLACK;
                x->p->color = RED;

                // 左旋，为保持红黑平衡
                leftRotate(T,x->p);
                w = w->p->right;
            }

            if(w->left->color==BLACK && w->right->color==BLACK){
                // 情况2 w 的左右孩子都为黑色
                w->color = RED;
                x =x->p;
            }else{
                if(w->right->color == BLACK){
                // 情况 3 w 的右孩子为黑色，则 w 的左孩子为红色

                // 交换 w 和 其左孩子的颜色
                w->left->color = BLACK;
                w->color = RED;

                // 右旋，保证 w 子树的每条简单路径的黑色节点数相同
                rightRotate(T,w);
                w = x->p->right;

                // 经过了情况3 必然来到 情况 4，
                w->color = x->p->color;
                x->p->color = BLACK;
                w->right->color = BLACK;
                leftRotate(T,x->p);
                x = T->root;
                }

            }
        }else{
            TreeNode *w = x->p->left;
            if(w->color == RED){
                // 情况1 x 的兄弟节点为红色

                // 改变 w 和 w 的节点
                w->color = BLACK;
                x->p->color = RED;
                // 左旋，为保持红黑平衡
                rightRotate(T, x->p);
                w = x->p->left;
            }

            if(w->left->color==BLACK && w->right->color == BLACK){
                // 情况2 w 的左右孩子都为黑
                w->color = RED;
                x = x->p;
            }else{
                if(w->left->color==BLACK){
                    // 情况3 w 左孩子为红，右孩子为黑
                    w->right->color = BLACK;
                    w->color = RED;
                    leftRotate(T,w);
                    w = x->p->right;
                }

                w->color = x->p->color;
                x->p->color = BLACK;
                w->left->color = BLACK;
                rightRotate(T,x->p);
                x==T->root;
            }
        }
    }
    x->color = BLACK;
}
```



完整代码实现：https://github.com/wangjunstf/Data-Structure/tree/main/Red%E2%80%93Black-Tree



下图作为测试图：

![IMG_4C19A2C5EF65-1](https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/IMG_4C19A2C5EF65-1.jpeg)

[测试代码](https://github.com/wangjunstf/Data-Structure/blob/main/Red%E2%80%93Black-Tree/RedBlack.cpp)

```
$ ./rbtree 
中序遍历:3 7 10 12 14 15 16 17 19 20 21 23 26 28 30 35 38 39 41 47 
最小值: 3 最大值: 47

根节点的前驱23
根节点的前驱28

插入:40后结果：
中序遍历:3 7 10 12 14 15 16 17 19 20 21 23 26 28 30 35 38 39 40 41 47 

删除21
中序遍历:3 7 10 12 14 15 16 17 19 20 23 26 28 30 35 38 39 40 41 47 

注：以下测试可能不符合红黑性质但符合二叉搜索树性质
将节点 14 左旋后的结果:
中序遍历:3 7 10 12 14 15 16 17 19 20 23 26 28 30 35 38 39 40 41 47 

将节点 41右旋后的结果:
中序遍历:3 7 10 12 14 15 16 17 19 20 23 26 28 30 35 38 39 40 41 47 

注：以下测试可能不符合红黑性质及二叉搜索树性质
用 10 取代了 26 之后: 
中序遍历:3 7 10 12 
```



### 时间复杂度分析

一棵含有 n 个节点的红黑树的高度至多为 O(lg n) ，不进行颜色修复时，删除元素的时间复杂度为O(lg n)，进行颜色修复的过程 removeFixup ，情况 1，3，4各执行常数次的颜色改变和3次旋转便终止了，情况 2 是唯一会循环多次的情况，至多沿树上升 O(lg n) 次。所以过程 removeFixup需要花费时间是 O(lg n)。因此 删除元素总的时间复杂度为 O(lg n)。








