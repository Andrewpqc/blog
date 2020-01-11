---
title: 二叉搜索树
comments: true
date: 2018-01-07 15:28:51
updated: 2018-01-07 15:28:51
tags: [algorithm, data structure,binary search tree]
categories: Algorithm
permalink:
---
`二叉搜索树`(binary search tree)支持许多动态集合操作，包括插入节点构建树，查找(最大值，最小值，前驱，后继，指定值)节点，遍历(前中后序遍历)树，删除节点等。在树上进行的操作所花费的时间与树的高度成正比，对于有n的节点的一个完全二叉树，这些操作的最坏运行时间为Θ(lg n).但是如果这棵树是一个n个节点连接而成的线性链，那么同样的操作的最坏运行时间就为Θ(n).为了尽量避免这种低效的二叉树，可以在构造树时采用随机构造的方式。随机构造的二叉树的期望高度O(lg n),因此在这样的一棵二叉树上的操作的平均运行时间就为Θ(lg n).但实际上，随机化的构造树的方式也不能完全杜绝最坏情况的出现，所以还有一些二叉搜索树的变种，如红黑树，B树等，他们可以保证基本操作具有好的最坏情况性能。
在本文中，不讨论红黑树和B树，我们只讨论二叉搜索树的各种操作。

# 什么是二叉搜索树?
一棵二叉搜索树就是以一个二叉树来组织的，树上的每一个节点就是一个对象，对象中除了key和卫星数据之外，还有用于维护二叉树结构的三个属性left,right,parent(在c语言中他们是三个指针),他们分别指向节点的左孩子，右孩子和父节点。其中key被称为节点的关键字，它是二叉搜索树中确定节点之间大小关系的关键属性，一般可以设置为整型类型。如果某一个节点的没有左孩子，右孩子或父节点，那么就将对应的属性指针置空即可。该树的根节点是整棵树中唯一一个parent属性为空的节点。这些置空的指针通常被用来确定树的边界。二叉搜索树之所以可以具有灵活高效的操作，是因为它具有如下的性质：
- 如果节点的左子树不空，则左子树上所有结点的值均小于等于它的根结点的值；
- 如果节点的右子树不空，则右子树上所有结点的值均大于等于它的根结点的值；
- 任意节点的左、右子树也分别为二叉搜索树

![二叉搜索树示意图](/images/bst.png)

# 二叉搜索树的基本操作
本文重点讨论的就是二叉搜索树上的操作，下面我们来逐一的讨论。首先声明:为了减少讨论的复杂度，`假设树上的所有节点的关键字均不相同`，下面我的代码也基于此，对于节点关键字有重复的情况，调整代码即可。下面我们先定义节点的数据结构：
``` c++
typedef struct node{
    int key;//节点数据,同时也是节点的关键字

    //下面的为二叉搜索树的维护数据
    struct node* parent=NULL;
    struct node* left=NULL;
    struct node* right=NULL;
}BST,*BST_p;
```
## 构建
我们想要实现在二叉搜索树上的操作，那么首先我们就需要构建出一棵二叉搜索树.我们可以从一个数组中构建二叉搜索树，逐个的从数组中读取数据，然后以插入的方式将数据插入到树中，即我们需要一个将数据插入到树中的操作。
### 插入
#### 递归插入
``` c++
/**
 * 递归构造搜索二叉树
 * root：指向树的根节点的指针
 * k:待插入的key
 * parent:父节点的指针
 **/ 
void bst_insert(BST_p& root,int k,BST_p parent=NULL){
    //如果当前二叉搜索树为空，则当前的待插入的值就为根节点中存放的值
    if(root==NULL){
        root=new BST;
        root->key=k;
        root->left=NULL;
        root->right=NULL;

        //根节点没有父节点
        root->parent=parent;
    }
    else{
        //关键字比root节点中存放的关键字大，那么该节点应该被存放在root节点的右子树
        if(k>root->key)
            bst_insert(root->right,k,root);
        //关键字比root节点中存放的关键字小，那么该节点应该被存放在root节点的左子树
        else
            bst_insert(root->left,k,root); 
    }
}
```

#### 非递归插入
``` c++
/**
 * 非递归插入操作
 **/ 
void bst_insert_nonRecur(BST_p& root,int k){
    BST_p pre=NULL;//记录上一个节点
    BST_p t=root;//不能直接使用root操作，这里的root为主函数中的全局root的引用，对他的更改会影响全局的root

    //按照规则找到key应该插入的位置，并用pre记录该位置的上一个节点
    while(t != NULL){
        pre = t;
        if(k < t->key){
            t = t->left;
        }
        else{
            t = t->right;
        }
    }

    BST_p node = new BST;
    node->key = k;
    node->left = NULL;
    node->right = NULL;
    node->parent = pre;

    //将node连接到树上
    if(pre == NULL)//若为空树，node即为根节点
        root = node;
    else{//树非空，则判断node应该被连接到pre的左子树还是右子树
        if(k < pre->key){
            pre->left = node;
        }
        else{
            pre->right = node;
        }
    }
}

```

### 构建二叉查找树
``` c++
void init_bst(BST_p& root,int * a ,size_t size){
     for(int i=0;i<size;i++){
        // bst_insert(root,a[i]);
        bst_insert_nonRecur(root,a[i]);
    }
}
```
有了上面的这几个函数，我们就可以像下面这样来构建一个二叉搜索树：
``` c++
#include <iostream>
using namespace std;
int main (void){
    BST_p root=NULL;
    int a[4]={2,5,3,6};
    init_bst(root,a,4);

    //some operating
 
    return 0;
}
```
## 遍历
二叉搜索树的性质允许我们通过一个简单的递归算法来按序输出二叉搜索树中的所有关键字，这种算法称之为**中序遍历**(inorder walk tree),这样命名的原因是输出的子树根的关键字位于其左子树的递归调用语句和右子树的递归调用语句之间。类似的**先序遍历**，**后续遍历**也是根据关键字输出语句与其左右子树递归调用语句之间的先后关系确定的。
下面是三种遍历的代码：
``` c++
/**
 * 中序遍历二叉搜索树
 **/ 
void inorder_tree_walk(BST_p root){
    if(root!=NULL){
        inorder_tree_walk(root->left);
        cout<<root->key<<" ";
        inorder_tree_walk(root->right);
    }
}

/**
 * 前序遍历二叉树
 **/ 
void preorder_tree_walk(BST_p root){
    if(root!=NULL){
        cout<<root->key<<" ";
        preorder_tree_walk(root->left);
        preorder_tree_walk(root->right);
    }
}

/**
 * 后序遍历二叉树
 **/
void postorder_tree_walk(BST_p root){
    if(root!=NULL){
        postorder_tree_walk(root->left);
        postorder_tree_walk(root->right);
        cout<<root->key<<" ";        
    }
}
```
上述代码中的中序遍历可以按照从小到大的顺序依次输出树中所有节点的关键字。

## 查找
在二叉查找树中可以快速的查找元素，下面我们就来看看在二叉查找树中各种查找操作都是如何进行的。
### 查找最大值与最小值
找具有最值关键字的节点在二叉查找树中非常的直接。比如要找具有最小关键字的节点，我们可以从根节点一直向左查找，如果一个节点具有左孩子，那么它的左孩子的关键字一定比它本身的关键字小，这时就可以迭代的去查看其左孩子，直到某一节点的左孩子为空，这样就是找到了树中最左边的节点，也就具有最小关键字的节点。上述寻找最小值的代码如下:
``` c++
/**
 * 找最小值，一直向左找，直到找到最左边的一个节点
 **/ 
BST_p minimum(BST_p& root){
    if(root==NULL){
        cout<<"该树为空，没有最小值!"<<endl;
        exit(EXIT_SUCCESS);
    }
    else{
        BST_p t=root;//不能用root直接操作
        //找到最左边的那个节点
        while(t->left!=NULL)
            t=t->left;
        return t;
    }
```
同理，找最大值的代码如下:
``` c++
/**
 * 找最大值,一直向右找，直到找到最右边的一个节点
 **/ 
BST_p maximum(BST_p& root){
    if(root==NULL){
        cout<<"该树为空，没有最大值"<<endl;
        exit(EXIT_FAILURE);
    }
    else{
        BST_p t=root;//不能用root直接操作
        //找到最右边的那个节点
        while(t->right!=NULL)
            t=t->right;
        return t;
    }         
}
```

### 查找前驱与后继
给定一棵二叉搜索树中的节点，有时需要按照**中序遍历**的次序(从小到大)查找它的**后继**(successor)或**前驱**(predecessor)。找前驱和后继在逻辑上是对称的，我们这里以找一个节点的后继节点作为例子讲解。按照定义我们可以知道，找节点a的后继节点就是找树中关键字比a的关键字大的所有节点中的最小关键字节点(虽然很绕，但就是这个道理:)。如何才能找到它呢？我们需要分两种情况讨论:
- 给定的节点a具有右孩子
在这种情况下，a的后继节点必定存在于a的右子树中，并且为右子树中的关键字最小节点。

- 给定的节点a没有右孩子
在这种情况下，a的后继节点在哪里呢？为了找到a的后继节点，我们就要从a节点开始追根溯源，向上找它的祖先，直到找到第一个这样的祖先s:s的左孩子s'也是ａ的祖先，那么节点s就是这种情况下a的后继节点。

下面是上述两种找后继节点的示意图。声明:`这两张图片借自别人的博客`(读书人的东西怎么能叫偷呢:)!
![找后继1](/images/bst_successor1.gif)
![找后继2](/images/bst_successor2.gif)

下面是找后继节点的代码:
``` c++
BST_p successor(BST_p node){
    if(node->right!=NULL)
    //有右孩子，直接返回指向右子树的最小关键字节点的指针
        return minimum(node->right);
    else{
        //没有右孩子
        BST_p p=node->parent;
        while(p!=NULL&&node!=p->left){
            node=node->parent;
            p=p->parent;
        }
        return p;
    }
}
```
找前驱节点的代码:
``` c++
BST_p perdecessor(BST_p node){
    if(node->left!=NULL)
        return maximum(node->left);
    else{
        BST_p p=node->parent;
        while(p!=NULL&&node!=p->right){
            node=node->parent;
            p=p->parent;
        }
        return p;
    }
}
```
### 查找具有指定关键字的节点
给定一个节点和一个关键字的值，找到树中关键字的值与给定值相等的节点。这个可以用递归和非递归两种方式实现。
#### 递归
``` c++
/**
 * 查找包含关键字k的节点，并返回指向该节点的指针
 * 这里假设树中的key不重复
 **/ 
BST_p bst_search(BST_p& root,int k){
    if(root==NULL || k==root->key)
        return root;
    else{
        if(k<root->key)
            return bst_search(root->left,k);
        else
            return bst_search(root->right,k);
    }
}
```
#### 非递归
``` c++
/**
 * 非递归查找
 **/ 
BST_p bst_search_nonRecur(BST_p& root,int k){
    if(root==NULL)//为空时，直接返回NULL
        return root;
    //不为空
    BST_p t=root;
    while(k!=t->key){//依次遍历，直到k与t指向的key相等
        if(k<t->key)//如果k小于当前节点的key，那么就需要到当前节点的左子树中去找
            t=t->left;
        else//如果k大于当前节点的key，那么就需要到当前节点的右子树中去找
            t=t->right;
    }
    return t;
}
```
## 删除
给定一个树中的节点a，要求删除该节点并且保持二叉搜索树的结构。删除这一操作根据给定节点a的情况的不同，可分成一下三种情况讨论:
- 节点a没有孩子节点
对于节点a没有孩子节点的情况，我们只需要做以下操作：通过比较a节点的关键字与其父结点的关键字大小从而确定a节点是其父结点的左孩子还是右孩子，然后将a的父节点的对应指针置空，并且释放a所指向节点所占用的内存。
![示意图](/images/delete0.gif)
- 节点a只有一个孩子节点
对这种情况，首先要确定a是其父结点的左孩子还是右孩子，然后根据此信息将a的父节点的对应指针指向a的那个唯一的孩子节点，并且将a的孩子节点的父指针指向a的父节点，最后释放a所指向的节点占用的内存。
![示意图](/images/delete1.gif)
- 节点a有两个孩子节点
首先找到节点a的后继节点b，然后根据b是否是a的右孩子分两种情况讨论：
1.b是a的右孩子
如果b是a的右孩子，将b提升到a节点的位置，并且将原来a节点的左子树，连接到b节点的左支上(b节点的左支初始时为空)，最后释放a所指向的节点的内存。
![示意图](/images/delete21.gif)
2.b不是a的右孩子
首先将b节点的右孩子提升到b的位置，即将b的右孩子连接到b的父节点的左支上(想一想为什么一定是左支上呢?因为b是a的后继节点，而且现在的讨论又是在b不是a的右孩子的基础上)。然后将原来a节点的左子树连接到b节点的左支上(该支初始时为空)，最后将b提升到a的位置，并且释放a节点占用的内存。
![示意图](/images/delete22.gif)

``` c++
/**
 * 二叉查找树删除node指向的节点
 **/ 
void bst_delete(BST_p node){
    //node没有孩子节点
    if(node->left==NULL&&node->right==NULL){
        BST_p p=node->parent;
        if(node->key<p->key)//这里说明node节点是其父节点的左节点
            p->left=NULL;//置空左指针
        else
            p->right=NULL;//置空右指针
        //释放node指向的内存块的内存
        delete node;
    }

    //node只有一个孩子节点
    else if(node->left==NULL || node->right==NULL){
        BST_p p=node->parent;
        if(node->key>p->key){//node是p的右孩子
            if(node->right==NULL){
                p->right=node->left;
                node->left->parent=p;
            }else{
                p->right=node->right;
                node->right->parent=p;
            }
        }else{
            if(node->right==NULL){
                p->left=node->left;
                node->left->parent=p;
            }else{
                p->left=node->right;
                node->right->parent=p;
            }
        }
        delete node;
    }

    //node有两个孩子节点
    else if(node->left!=NULL&& node->right!=NULL){
        BST_p left=node->left;//左子树的根节点
        BST_p right=node->right;//右子树的根节点
        BST_p parent=node->parent;//父节点

        //node节点的后继节点,由于存在右子树，所以这里直接使用了求右子树的最小节点的函数
        BST_p successor=minimum(right);

        //后继节点就是node的右孩子，这里假设树中不存在key值相同的节点
        if(successor->key==right->key){
            //将node的左支连接到后继节点的左孩子上(初始时该节点为空)
            successor->left=left;
            left->parent=successor;
            //将后继节点提升到node的位置
            //这里首先需要判断node是其父节点的左孩子还是右孩子
            if(node->key>parent->key){
                //node是其父节点的右节点
                parent->right=successor;
                successor->parent=parent;
            }else{
                 //node是其父节点的左节点
                parent->left=successor;
                successor->parent=parent;
            }
        }else{
            //后继节点successor不是node的右孩子
            //首先将successor的右节点提升到successor的位置，然后将node的左支连接到
            //successor的左支上(初始为空支)，然后将successor提升到node的位置

            BST_p succ_parent=successor->parent;//后继节点的父节点
            
            //这里后继节点一定位于其父节点的左支，所以直接将后继节点的右节点连接到
            //后继节点的父节点的左支上
            succ_parent->left=successor->right;
            successor->right->parent=succ_parent;

            //将node的左子树连接到后继节点的左支上(该支初始时为空)
            successor->left=left;
            left->parent=successor;

            //将successor提升到node的位置，同样的需要判断node位于其父结点的左还是右
            if(node->key>parent->key){
                //node是其父节点的右节点
                parent->right=successor;
                successor->parent=parent;
            }else{
                 //node是其父节点的左节点
                parent->left=successor;
                successor->parent=parent;
            }
        }
        delete node;
    }
}
```


# 随机构建
在文章开头介绍了，如果一棵二叉树是一个n个节点连接而成的线性链，那么该二叉树的效率就极其的低下，它就与链表无异了，无法发挥出搜索二叉树的威力。就拿我们上面的`init_bst`函数为例吧，我们是以用户提供的数组来初始化我们的二叉搜索树的，假如用户提供了这样的一个数组{1,2,3,4,5},或者是这样的{9,8,6,4,3,2}，大家想想我们给构造出来的是一什么样的二叉树呢？第一个数组构造出来的二叉搜索树的每个节点只有右孩子没有左孩子，而第二个构造出来的二叉搜索树的每个节点则只有左孩子没有右孩子。这就是极其低效的二叉树。怎样去避免这样的问题呢？

显然，我们不应该对用户的输入做任何限制，那么我们就只能改变自己来适应用户了:)。我们可以在用户提供的数组中通过不重复无遗漏的随机取元素的方法来构造一个随机序列，然后用该序列来构造一个随机二叉搜索树。代码如下:
``` c++
void swap(int &a, int &b)  {  
	int tmp = a;  
	a = b;  
	b = tmp;  
}

void random_init_bst(BST_p& root,int * a ,size_t size){
    srand(time(NULL));
    for(int i=size-1;i>=0;i--){
        int j = rand() % (i+1);	
        bst_insert_nonRecur(root,a[j]);//插入
        swap(arr[j], arr[i]);
    }
}
```
上述随机构建二叉搜索树的方法，在初始化构建树的时候，确实可以降低因用户给定的数组有序而创建低效二叉搜索树的情况，但是它无法做出保证。并且我们在对二叉搜索树这一动态集合进行插入删除等动态操作时，搜索二叉树的形态仍然可能走向低效。前面的动态操作会降低后面操作的效率。怎么办呢?这就要看其他的一些二叉搜索树的变体了！

# 参考
Introduction to Algorithm (Third Edition)
[多动态图详细讲解二叉搜索树聪聪的个人网站](https://lufficc.com/blog/binary-search-tree)
[二叉查找树（BST） | 神奕的博客](https://songlee24.github.io/2015/01/13/binary-search-tree/)