---
title: 'c++迭代实现二叉树遍历'
date: 2021-09-08 11:06:39
updated:
description: '分别用递归和迭代实现二叉树三种遍历方式：前序遍历，中序遍历，后序遍历。递归的缺点分析。'
categories: [数据结构]
tags: [c++,二叉树遍历,迭代,递归]
---

# 递归的缺点

二叉树的主要遍历方式有以下几种：

前序遍历：根- 左子树-右子树（**前序遍历可以唯一确定一棵二叉树**）

中序遍历：左子树-根-右子树（**二叉搜索树使用此遍历可以从小大到大输出所有元素。**）

后序遍历：左子树-右子树-根（**结合中序遍历可以唯一确定一棵二叉树**）

二叉树遍历最常见的，也是最简单的思路是使用递归算法。既然有了最简答的写法，为什么还有学习更复杂的迭代实现呢？主要是递归有以下缺点：

递归的本质是函数调用，对于大多数操作系统，每一次函数调用都是一个代价较高的过程，主要因为以下几个方面：

* 性能上考虑：**函数每一次调用都得往栈中压入函数返回地址，函数参数，以及为函数内的所有局部变量分配空间。每个进程的栈空间是有限的，递归层次太多，会导致往栈中存在太多数据，易导致栈溢出，进而程序崩溃。** 

* 效率上考虑：**当系统进行函数调用时，会向栈中压入函数返回地址，函数形参，为局部变量分配空间，当函数执行完毕时，需要进行弹栈(也称为清栈)操作使之与函数调用前一模一样，往栈中压入数据和弹出数据都需要时间，从某种程度上降低了程序的性能。**

说了递归的缺点，现在说下迭代的好处：**迭代没有函数调用所产生的额外开销，也没有压栈和弹栈操作，通常空间利用率更高，程序运行效率更好。**
`系统 : ubuntu 18.04 64位`

`编程语言: c++`

`g++ 版本: g++ 8.4.0`

<!-- more -->

节点定义：为了简化代码，省略了父节点。

```c++
struct TreeNode
{
    int key;
    TreeNode *left;
    TreeNode *right;
    TreeNode *parent;     
    TreeNode() : val(0), left(nullptr), right(nullptr){}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
```

为了封装二叉树的各种操作，我们创建一个二叉树类：

```c++
class BinaryTree {
public:
    TreeNode *root;  
    void insert(TreeNode *node);                           										// 插入元素
  	TreeNode* treeBuild(vector<int> &tab, int &index, TreeNode *parent)；     // 构造二叉树
    void preOrder(TreeNode *root);                          									// 前序遍历(递归)
  	void preIteration(TreeNode *root);																				// 前序遍历(迭代)
};
```

# 二叉树构造

为了测试遍历代码的正确性，往往需要构建一棵二叉树。二叉树的构建方法主要有两种：

## 前序遍历法

使用前序遍历可以唯一确定一棵二叉树，二叉树的前序遍历顺序：根-左子树-右子树，可以使用 vector 来存储二叉树的前序遍历，遍历序列中必须包含空子树标记。例如：现在给定一个前序遍历序列：`vector<int> pre{4,2,1,NULL,NULL,3,NULL,NULL,5,NULL,NULL};` 可以根据前序遍历的步骤来构建二叉树。

```c++
TreeNode*
BinaryTree::treeBuild(vector<int> &tab, int &index, TreeNode *parent)
{
    TreeNode *root = nullptr;
    if (index < tab.size() && tab[index] != 0)
    {
        root = new TreeNode(tab[index]);
        root->parent = parent;
        root->left = treeBuild(tab, ++index, root);
        root->right = treeBuild(tab, ++index, root);
    }
    return root;
}
```



## 随机插入法

此方法适用于二叉搜索树，一棵二叉搜索树的定义如下，若它的左子树不空，则左子树上所有结点的值均小于它的根结点的值； 若它的右子树不空，则右子树上所有结点的值均大于它的根结点的值。二叉搜索树的所有子树都为二叉搜索树。

给定一组序列，使用二叉搜索树的插入方法，可以构建一棵有序的二叉树。次方法不必知道二叉树的结构。

```c++
void 
BinaryTree::insert(TreeNode *node){
    TreeNode *x = root;
    TreeNode *y = nullptr;
    while (x != nullptr)
    {
        y = x;
        if (node->val < x->val)
        {
            x = x->left;
        }
        else
        {
            x = x->right;
        }
    }
    if (y == nullptr)
    {
        root = node;
    }
    else if (node->val < y->val)
    {
        y->left = node;
    }
    else
    {
        y->right = node;
    }
}
```

# 前序遍历

## 递归法

* 递归的本质是自己调用自己，每一次调用都是输出当前子树的根节点值。
* 函数第一次调用时，root 为根节点，首先输出根节点值。
* 然后再递归调用左子树，右子树。这样就依次输出了左节点值，右节点。和前序遍历的要求一致：根-左-右

递归本质是基于栈的回溯，即它在递归的时候，会在栈中记录所走的路径，回溯时输出节点值。可以看到递归法代码更加简洁，只需4行代码即可。

```c++
void BinaryTree::preOrder(TreeNode *root)
{
  	// 前序遍历，输出所有节点值
    if (root != nullptr)
    {
        cout << root->val << " ";
        preOrder(root->left);
        preOrder(root->right);
    }
}
```



## 迭代法

迭代的原理就是利用一个栈来记录路径，模拟函数调用的回溯。迭代法避免了函数调用产生的代价，具有更高的运行效率，但代码会更复杂一些。

* 循环开始前，将根节点入栈。
* 每一次循环都先输出当前子树根节点值，弹出当前子树根节点 cur，然后再 cur 的右子树，左子树入栈。因为栈本身的特性，后到先服务。
* 再下一轮循环中，首先处理左子树，然后处理右子树。这样就满足前序遍历：根-左子树-右子树的要求。

```c++
void 
BinaryTree::preIteration(TreeNode *root)
{
  	// 前序遍历，输出所有节点值
    stack<TreeNode *> s;
    s.push(root); 
    while(!s.empty()){
        TreeNode *p = s.top();
        s.pop();
        cout<<p->val<<" ";
        
        // 先将右节点入栈，后将左节点入栈，因为前序遍历，需要先访问左节点，再访问右节点
        if(p->right!=nullptr){
            s.push(p->right);
        }
        if(p->left!=nullptr){
            s.push(p->left);
        }
    }
}
```

# 中序遍历

## 递归法

只需改变输出根节点值的位置即可。

```c++
void 
BinaryTree::inOrder(TreeNode *root){
    // 中序遍历，输出所有节点值
    if (root != nullptr)
    {
        inOrder(root->left);
        cout << root->val << " ";
        inOrder(root->right);
    }
}
```



## 迭代法

中序遍历是；左-根-右。

* 使用 cur 作为当前节点，循环开始前 cur 的值为根节点。
* 因为要先输出左子树，在循环开始，使用一个子循环将 当前节点 cur 的左孩子节点，左孩子的左孩子入栈，直到 cur 为空为止。
* 此时栈顶就是输出的第一个节点。弹出栈顶到 cur 中，并输出，将 cur 设置为它的右孩子，如果其右孩子存在的话，对其右子树执行相同的操作，就是找到最左边节点；如果 其右孩子为空，则 cur 也为空，在循环开始处不会有节点在进栈，此时弹出的当前节点 cur 就是最先输出节点的父节点，将该节点输出，然后将 cur 设置为其右孩子，对其右孩子执行相同的操作。这样就实现了回溯的功能。

```c++
void 
BinaryTree::inIteration(TreeNode *root){
    // 中序遍历输出所有节点值
    stack<TreeNode *> s;
    TreeNode* cur = root;
    while(cur!=nullptr || !s.empty()){
        while(cur!=nullptr){
            s.push(cur);
            cur = cur->left;
        }
        cur = s.top();
        s.pop();
        cout<< cur->val<<" ";
        cur = cur->right;
    }
}
```

# 后序遍历

## 递归版本

```c++
void 
BinaryTree::postOrder(TreeNode *root){
    // 后续遍历输出所有节点值
    if(root!=nullptr){
        postOrder(root->left);
        postOrder(root->right);
        cout<<root->val<<" ";
    }
}
```



## 迭代版本

后续遍历的迭代版本会更加复杂一些。

* 后序遍历: 左-右-根。输出左孩子简单，使用一个子循环就可以找到最左边孩子，重点是从右节点到根节点之后，需要判断它是否从右孩子回溯而来，如果是，则表示它的右孩子已经处理，可以处理根节点，如果不是回溯而来，则需要先处理它的右子树。
* 使用一个遍历 pre 来记录上一次访问的节点，这样就可以容易的判断当前节点是不是由右孩子回溯而来。
* 第一次循环：当前节点是根节点，pre 为空节点。使用一个子循环将当前节点 cur 的左孩子节点，左孩子的左孩子入栈，知道 cur 为空。
* 弹出栈顶元素到 cur，此时判断 cur 是否为回溯而来，只需判断 cur 的右孩子是否是 pre，不是则将 cur 再次入栈，并将 cur 的右孩子入栈，执行相同的操作。如果 cur 的右孩子为 pre，则表示 cur 是由 pre 回溯而来，当前 cur 作为当前子树的根节点，将 pre 设置为 cur，将 cur 设置为 空。在下一次循环中，就来到了上一层节点，实现了回溯的功能。

```c++
void
BinaryTree::postIteration(TreeNode *root){
    stack<TreeNode *> s;
    TreeNode *cur = root;
    TreeNode *pre = nullptr;
    while(cur || !s.empty()){
        while(cur){
            s.push(cur);
            cur = cur->left;
        }
        if(!s.empty()){
            cur = s.top();
            s.pop();
            if(cur->right==nullptr || pre == cur->right){
                cout<<cur->val<<" ";
                pre = cur;
                cur = nullptr;
            }else{
                s.push(cur);
                cur = cur->right;
            }
        }
    }
}
```

# 测试代码

```c++
int main(){
    vector<int> pre{4,2,1,NULL,NULL,3,NULL,NULL,5,NULL,NULL};   // 中序遍历，未用到

    vector<int>tab {1,3,2,6,4,5};
    BinaryTree b;
    for(auto x:tab){
        TreeNode *node = new TreeNode(x);
        b.insert(node);
    }

    cout<<"前序遍历(递归): ";
    b.preOrder(b.root);
    cout<<endl;

    cout<<"前序遍历(迭代): ";
    b.preIteration(b.root);
    cout<<endl;

    cout<<"中序遍历(递归): ";
    b.inOrder(b.root);
    cout<<endl;

    cout<<"中序遍历(迭代): ";
    b.inIteration(b.root);
    cout<<endl;

    cout<<"后序遍历(递归): ";
    b.postOrder(b.root);
    cout<<endl;

    cout<<"后续遍历(迭代): ";
    b.postIteration(b.root);
    cout<<endl;

    return 0;
}
```

```shell
前序遍历(递归): 1 3 2 6 4 5 
前序遍历(迭代): 1 3 2 6 4 5 
中序遍历(递归): 1 2 3 4 5 6 
中序遍历(迭代): 1 2 3 4 5 6 
后序遍历(递归): 2 5 4 6 3 1 
后续遍历(迭代): 2 5 4 6 3 1 
```

# 编程练习

学习完二叉树遍历方法，可以做题巩固一下，建议把递归和迭代都实现一遍。

* [二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/) 
* [二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/) 
* [二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/) 

# 参考

http://icejoywoo.github.io/2020/03/31/binary-tree-traversal.html

https://zhuanlan.zhihu.com/p/80578741

