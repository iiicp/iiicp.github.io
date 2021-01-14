---
layout: post
title: 关于树类型数据结构总结
tags: datastruct
excerpt: tree data struct
---

### 前言  

这几天总结下一些关于树的数据结构，里面包含了树状数组类型。这些数据结构分别是堆、线段树、并查集、trie(前缀树)、二分搜索树、AVL树和红黑树。其中堆、线段树和并查集都属于树状数组，简而言之就是用数组来实现树的逻辑结构，堆和线段树的思想是采用满二叉树的方式来存储，并查集是采用parent数组的方式，因为在并查集中只关心当前结点的父亲结点是谁。这些树数据结构都属于比较高阶的数据结构了，以BST为代表的树，是充分利用在已排序的情况下采用二分查找是最优的这个性质，在插入和删除操作的情况下也时刻保持有序性，为了解决BST有可能退化成链表的可能，科学家提出了平衡的概念，提出了AVL和红黑树。其它的树是专注于特定的领域，提供特定操作，这些操作解决特定的问题，因而是最高效的，一般而言操作是O(logn)级别，采用普通的数据结构去解决，往往是O(n)级别，不够高效。所有的源代码可以参考[我的仓库](https://github.com/iiicp/datastruct-algorithm)


### 堆(heap) 

堆提供的抽象是在一堆数据中动态维护最大值或者最小值。最大值称之为大顶堆，最小值称之为小顶堆。这里面有两点信息很关键，第一是只维护最大值或者最小值，即不会维护第二大、第三大等信息，这也是高效原因。第二是动态维护，是指取走一个最大值或者最小值之后，能够自动调整，让最大或者最小填充到堆顶。  

堆是一颗完全二叉树，所以堆通常用数组来完美实现。堆支持的操作有，建堆、添加元素和抽取元素，这三个操作都需要保持堆的性质。 

除了heapify、add、extractElement面向用户的三个操作以外。核心的操作是 shiftDown和shiftUp .  

``` 
#ifndef TREE_HEAP_H
#define TREE_HEAP_H
#include <vector>
using namespace std;
template <typename E>

/**
 *  最大堆实现，采用数组前面空一个元素的方案
 *  左右孩子与根的关系为
 *  i -> 2 * i
 *  i -> 2 * i + 1
 *  i/2 -> parent
 */
class Heap{
private:
  vector<E> data;
  /// 数组末尾索引
  int size;
  int capacity;
public:

  Heap(int cap) {
    data = vector<E>(cap + 1, 0);
    size = 0;
    this->capacity = cap;
  }

  Heap(vector<E> &inData) {
    size = inData.size();
    capacity = size;

    data = vector<E>(inData.size() + 1, 0);
    for (int i = 0; i < inData.size(); ++i)
      data[i + 1] = inData[i];

    /// heapify
    for (int i = size/2; i >= 1; --i) {
      shiftDown(i);
    }
  }

  void add(E element) {
    assert(size < capacity);
    data[size + 1] = element;
    size++;
    shiftUp(size);
  }

  E extractElement() {
    assert(size > 0);
    E ret = data[1];

    swap(data[1], data[size]);
    size--;

    shiftDown(1);

    return ret;
  }

private:
  void shiftDown(int k) {
    while (2 * k <= size) {
      int f = 2 * k;
      if (f+1 <= size && data[f] < data[f+1])
        f = f + 1;
      /// 如果当前孩子小于左右孩子的最大值就进行交换
      if (data[k] >= data[f])
        break;
      swap(data[k], data[f]);
      k = f;
    }
  }

  void shiftUp(int k) {
    while (k > 1 && data[k/2] < data[k]) {
      swap(data[k/2], data[k]);
      k = k/2;
    }
  }
};

#endif // TREE_HEAP_H 
``` 

### 并查集(Union-Find)   

并查集设计出来是用来查询和合并连通区域的。它能够高效的判断两个区间是否连通，也能够高效的合并两个区域，操作复杂度均为O(logn). 

``` 
#ifndef TREE_UNIONFIND_H
#define TREE_UNIONFIND_H

#include <vector>
using namespace std;
class UF {
private:
  vector<int> parent;
public:
  UF(int size) {
    assert(size > 0);
    parent = vector<int>(size, 0);
    for (int i = 0; i < size; ++i)
      parent[i] = i; /// 默认当前id的parent就是其自己
  }

  /// 判断区域p和区域q是否连通
  bool isConnected(int p, int q) {
    return find(p) == find(q);
  }

  /// 将区域p和区域q进行连通
  void unionElement(int p, int q) {
    int p1 = find(p);
    int p2 = find(q);
    if (p1 == p2)
      return;
    parent[p1] = p2;
  }

  void print() {
    for (int i = 0; i < parent.size(); ++i)
      std::cout << parent[i] << " ";
    std::cout << std::endl;
  }

private:
  /// 查找p所对应的区域的根
  int find(int p) {
    while (parent[p] != p)
      p = parent[p];
    return p;
  }
};

#endif // TREE_UNIONFIND_H
``` 

### 线段树(区间树) 

线段树也叫做区间树，设计出来是来维护一个区间的动态信息，比如这个区间的和、最小值、最大值等统计信息，这些信息随着区间的更新操作是实时在变化的。线段树能这么高效的原因是利用了平衡二叉树原理，当进行区间查找和更新时候的都是O(logn)的时间复杂度。线段树的每个结点都维护了一个区间的信息，这个区间的范围是按照二分的范围来限定。比如根节点的范围为[l, r], 首先求出mid = l + (r-l)/2， 那么左孩子的范围就是[l, mid]，右孩子的范围为[mid+1, r]，没个节点所表示的范围都是父亲的一半。在具体实现上，线段树采用满二叉树的方式去存储，也是为了实现简单。假设共有n个节点，n恰好为2^k次方，则总共约有2n个节点.(最后一层节点约为前面所有层节点之和)，若n为2^k+1，则还需要一层节点，由于前面共有2n个节点了，最后一层也需2n个，那么总共需要4n个结点。即使用满二叉树的方式来存储，最大需要4n的存储空间。 

线段树最重要的操作是更新和查询。 

``` 
#ifndef TREE_SEGMENTTREE2_H
#define TREE_SEGMENTTREE2_H

#include <vector>
#include <climits>

using namespace std;
template <typename E>
class SegmentTree2 {
private:
  vector<E> data;
  vector<E> tree; /// 树状数组
  E (*merge) (E, E);
public:
  SegmentTree2(vector<E> &inData, E(*mergeOps)(E a, E b)) {
    assert(inData.size() > 0);
    assert(mergeOps);

    data = vector<E>(inData);
    /// 采用满二叉树的方式来存储，最多的结点数为4n，使用INT_MIN表示null结点
    tree = vector<E>(data.size() * 4, INT_MIN);

    this->merge = mergeOps;

    /// buildSegmentTree
    buildSegmentTree(0, 0, data.size() - 1);
  }

  E query(int l, int r) {
    return query(0, 0, data.size() - 1, l, r);
  }

  void update(int index, E val) {
    assert(index>=0 && index < data.size());

    /// 先更新数据区
    data[index] = val;

    /// 更新线段树
    update(0, 0, data.size() - 1, index, val);
  }

  void printTree() {
    for (int i = 0; i < tree.size(); ++i)
      std::cout << tree[i] << " ";
    std::cout << std::endl;
  }

private:
  void buildSegmentTree(int treeIndex, int l, int r) {
    if (l == r) {
      tree[treeIndex] = data[l]; // or data[r]
      return;
    }

    int leftIndex = leftChild(treeIndex);
    int rightIndex = rightChild(treeIndex);

    int mid = l + (r - l) / 2;
    buildSegmentTree(leftIndex, l, mid);
    buildSegmentTree(rightIndex, mid + 1, r);

    /// 后序遍历的过程，当前结点的值
    tree[treeIndex] = merge(tree[leftIndex], tree[rightIndex]);
  }

  E query(int treeIndex, int tl, int tr, int l, int r) {
    if (tl == l && tr == r)
      return tree[treeIndex];

    int leftIndex = leftChild(treeIndex);
    int rightIndex = rightChild(treeIndex);

    int mid = tl + (tr - tl) / 2;

    if (l >= mid + 1) {
      return query(rightIndex, mid+1, tr, l, r);
    }else if (r <= mid) {
      return query(leftIndex, tl, mid, l, r);
    }

    int a = query(leftIndex, tl, mid, l, mid);
    int b = query(rightIndex, mid+1, tr, mid+1, r);
    return merge(a, b);
  }

  void update(int treeIndex, int tl, int tr, int index, E val) {
    if (tl == tr) {
      tree[treeIndex] = val;
      return;
    }

    int leftIndex = leftChild(treeIndex);
    int rightIndex = rightChild(treeIndex);

    int mid = tl + (tr - tl) / 2;

    if (index <= mid) {
      update(leftIndex, tl, mid, index, val);
    }else {
      update(rightIndex, mid+1, tr, index, val);
    }

    tree[treeIndex] = merge(tree[leftIndex], tree[rightIndex]);
  }

  int leftChild(int index) {
    return index * 2 + 1;
  }

  int rightChild(int index) {
    return index * 2 + 2;
  }
};

#endif // TREE_SEGMENTTREE2_H 
``` 

### trie(单词树，前缀树)   

trie被设计出来就是用来做单次查找的，每次查找的效率只与单次的长度有关，与存储的单次总量没有关系，即复杂度为O(h). 
trie是一颗多叉树，每一层维护了该层所有方向的映射。所以trie最大的不足就在于存储空间。  

trie最重要的操作是添加单次和查询单次，也支持查询前缀。 

```  
#include <map>
#include <string>
using namespace std;
class Trie {
private:
  class Node {
  public:
    bool isWord;
    map<char, Node*> next;

    Node(bool isWord) {
      this->isWord = isWord;
    }

    Node() {
      isWord = false;
    }
  };
  Node *root;
  int count;
public:
  Trie() {
    root = new Node();
    count = 0;
  }

  int size() {
    return count;
  }

  void add(string word) {
    Node *cur = root;
    for (int i = 0; i < word.size(); ++i) {
      char c = word[i];
      if (cur->next.find(c) == cur->next.end()) {
        cur->next[c] = new Node();
      }
      cur = cur->next[c];
    }
    /// 防止重复添加 count
    if (cur->isWord == false) {
      cur->isWord = true;
      count++;
    }
  }

  bool contains(string word) {
    Node *cur = root;
    for (int i = 0; i < word.size(); ++i) {
      char c = word[i];
      if (cur->next.find(c) == cur->next.end())
        return false;
      cur = cur->next[c];
    }
    return cur->isWord;
  }

  bool isPrefix(string prefix) {
    Node *cur = root;
    for (int i = 0; i < prefix.size(); ++i) {
      char c = prefix[i];
      if (cur->next.find(c) == cur->next.end())
        return false;
      cur = cur->next[c];
    }
    return true;
  }
};

``` 

### 二分搜索树(BST)   

二分搜索树特性是对于每一个结点，其左孩子结点都小于它，其右孩子结点都大于它。二分搜索树是借鉴了有序数组中进行二分查找效率最高的这个性质。在插入和删除操作时也要保持中序遍历有序性。 

二分搜索树支持查询、插入和删除，其相对复杂的操作是删除，需要考虑左孩子为空、右孩子为空，和左右孩子均不为空的场景。 

``` 
namespace BST {
  template <typename Key, typename Value>
  class BinarySearchTree {
  private:
      class Node {
      public:
        Key key;
        Value value;
        Node * left;
        Node * right;

        Node(Key key, Value value) {
          this->key = key;
          this->value = value;
          this->left = nullptr;
          this->right = nullptr;
        }

        Node(Node *node) {
          this->key = node->key;
          this->value = node->value;
          this->left = node->left;
          this->right = node->right;
        }
      };

      Node *root;
      int count;

    public:
      BinarySearchTree() {
        root = nullptr;
        count = 0;
      }

      ~BinarySearchTree() {
        destroy(root);
      }

      /// 外层的接口不用管返回值
      void insert(Key key, Value value) {
        root = insert(root, key, value);
      }

      void erase(Key key) {
        root = erase(root, key);
      }

      bool contain(Key key) {
        return contain(root, key);
      }

      Value search(Key key) {
        return search(root, key);
      }

      void preOrderTravel() {
        preOrderTravel(root);
        std::cout << std::endl;
      }

      void inOrderTravel() {
        inOrderTravel(root);
        std::cout << std::endl;
      }

      void postOrderTravel() {
        postOrderTravel(root);
        std::cout << std::endl;
      }

      void levelOrderTravel() {
        levelOrderTravel(root);
        std::cout << std::endl;
      }

      Node *minimum() {
        return minimum(root);
      }

      Node *maximum() {
        return maximum(root);
      }

      void deleteMinNode() {
        root = deleteMinNode(root);
      }

      void deleteMaxNode() {
        root = deleteMaxNode(root);
      }

      int size() {
        return count;
      }

      bool isEmpty() {
        return count == 0;
      }

    private:
      Node *insert(Node *node, Key key, Value value) {
        if (node == nullptr) {
          count++;
          return new Node(key, value);
        }

        if (key < node->key) {
          node->left = insert(node->left, key, value);
        }else if (key > node->key) {
          node->right = insert(node->right, key, value);
        }else { // update
          node->value = value;
        }

        return node;
      }

      Node *minimum(Node *node) {
        if (node->left == nullptr)
          return node;
        return minimum(node->left);
      }

      Node *maximum(Node *node) {
        if (node->right == nullptr)
          return node;
        return maximum(node->right);
      }

      Node *deleteMinNode(Node *node) {
        if (node->left == nullptr) {
          Node *rightNode = node->right;
          delete node;
          count--;
          return rightNode;
        }
        return deleteMaxNode(node->left);
      }

      Node *deleteMaxNode(Node *node) {
        if (node->right == nullptr) {
          Node *leftNode = node->left;
          delete node;
          count--;
          return leftNode;
        }
        return deleteMaxNode(node->right);
      }

      Node *erase(Node *node, Key key) {
        if (node == nullptr)
          return node;

        /// 分类不要分错....
        if (key < node->key) {
          node->left = erase(node->left, key);
          return node;
        }else if (key > node->key) {
          node->right = erase(node->right, key);
          return node;
        }else {
          /// 删除节点
          if (node->left == nullptr) {
            Node *rightNode = node->right;
            delete node;
            count--;
            return rightNode;
          } else if (node->right == nullptr) {
            Node *leftNode = node->left;
            delete node;
            count--;
            return leftNode;
          } else {
            Node *successor = new Node(minimum(node->right));
            /// 移除右边的最小值
            successor->right = erase(node->right, successor->key);
            successor->left = node->left;
            delete node;
            return successor;
          }
        }
      }

      void destroy(Node *node) {
        /// 后序遍历删除
        if (node == nullptr)
          return;
        destroy(node->left);
        destroy(node->right);
        delete node;
        count--;
      }

      void preOrderTravel(Node *node) {
        if (node == nullptr)
          return;
        std::cout << node->value << ", ";
        preOrderTravel(node->left);
        preOrderTravel(node->right);
      }

      void inOrderTravel(Node *node) {
        if (node == nullptr)
          return;
        inOrderTravel(node->left);
        std::cout << node->value << ", ";
        inOrderTravel(node->right);
      }

      void postOrderTravel(Node *node) {
        if (node == nullptr)
          return;
        postOrderTravel(node->left);
        postOrderTravel(node->right);
        std::cout << node->value << ", ";
      }

      void levelOrderTravel(Node *node) {
        if (node == nullptr)
          return;

        queue<Node *> q;
        q.push(node);

        while (!q.empty()) {
          Node *node = q.front();
          q.pop();

          std::cout << node->value << ", ";

          if (node->left)
            q.push(node->left);

          if (node->right)
            q.push(node->right);
        }
      }

      bool contain(Node *node, Key key) {
        if (node == nullptr)
          return false;

        if (node->key == key)
          return true;
        else if (key < node->key)
          return contain(node->left, key);
        else
          return contain(node->right, key);
      }

      Value search(Node *node, Key key) {
        if (node == nullptr)
          return 0;
        if (key == node->key)
          return node->value;
        else if (key < node->key)
          return search(node->left, key);
        else
          return search(node->right, key);
      }
  };
}
``` 

### AVL树 

AVL树是在BST的基础上引入了平衡因子。即任意一个节点其左右子树的高度差绝对值小于等于1就是平衡的。平衡的好处是降低了树的高度，使得插入、删除、查找的效率都是O(logn). 

在BST的基础上，只需要在每个结点上维护一个height属性. 默认新生成的结点高度为1. 在插入和删除节点的时候，需要更新高度然后判断平衡因子是否大于1或者小于-1. AVL树引入了4种调整平衡的操作，分别是左旋(让其右孩子为根)、右旋(让其左孩子为根)、左旋右旋结合、右旋和左旋结合. 具体的判断和示意图见代码。   

要注意，AVL树仍然是一颗BST，所以其左旋、右旋操作也是要维护这个性质的。(左旋和右旋结合了BST的性质才发生的)

``` 
/**
     *        y                           x
     *      x   T4                    z      y
     *    z  T3         ==>         T1 T2  T3 T4
     *  T1 T2
     */
    Node *rightRotate(Node *y) {
        Node *x = y->left;
        y->left = x->right;
        x->right = y;
        y->height = 1 + std::max(getNodeHeight(y->left), getNodeHeight(y->right));
        x->height = 1 + std::max(getNodeHeight(x->left), getNodeHeight(x->right));
        return x;
    }

    /**
     *       y
     *    T1    x                           x
     *       T2    z        ==>          y      z
     *          T3  T4                T1  T2  T3 T4
     */
    Node *leftRotate(Node *y) {
       Node *x = y->right;
       y->right = x->left;
       x->left = y;
       y->height = 1 + std::max(getNodeHeight(y->left), getNodeHeight(y->right));
       x->height = 1 + std::max(getNodeHeight(x->left), getNodeHeight(x->right));
       return x;
    }

 	Node *insert(Node *node, Key key, Value value) {
      if (node == nullptr) {
        count++;
        return new Node(key, value);
      }
      if (key < node->key) {
        node->left = insert(node->left, key, value);
      }else if (key > node->key) {
        node->right = insert(node->right, key, value);
      }else {
        node->value = value;
      }

      /// 更新结点高度
      node->height = 1 + std::max(getNodeHeight(node->left), getNodeHeight(node->right));
      int balanceFactor = getBalanceFactor(node);

      /// 1. ll
      if (balanceFactor > 1 && getBalanceFactor(node->left) >= 0) {
        return rightRotate(node);
      }
      /// 2. rr
      if (balanceFactor < -1 && getBalanceFactor(node->right) <= 0) {
        return leftRotate(node);
      }
      /// 3. lr
      /**
       *           y                     y
       *        x    T4               z     T4                z
       *     T1   z         ==>     x   T3        =>      x       y
       *       T2  T3             T1 T2                 T1 T2   T3 T4
       */
      if (balanceFactor > 1 && getBalanceFactor(node->left) < 0) {
        node->left = leftRotate(node->left);
        return rightRotate(node);
      }

      /// 4. rl
      /**
       *          y                   y                           z
       *       T1   x       =>     T1   z           =>        y       x
       *          z   T4              T2  x                 T1 T2   T3 T4
       *        T2 T3                   T3  T4
       */
      if (balanceFactor < -1 && getBalanceFactor(node->right) > 0) {
        node->right = rightRotate(node->right);
        return leftRotate(node);
      }

      return node;
    }

    Node *erase(Node *node, Key key) {
      if (node == nullptr)
        return node;

      Node *retNode;
      if (key < node->key) {
        node->left = erase(node->left, key);
        retNode = node;
      }else if (key > node->key) {
        node->right = erase(node->right, key);
        retNode = node;
      }else {
        if (node->left == nullptr) {
          Node *rightNode = node->right;
          delete node;
          count--;
          retNode = rightNode;
        } else if (node->right == nullptr) {
          Node *leftNode = node->left;
          delete node;
          count--;
          retNode = leftNode;
        } else {
          Node *successor = new Node(minimum(node->right));
          successor->right = erase(node->right, successor->key);
          successor->left = node->left;
          delete node;
          retNode = successor;
        }
      }
      if (retNode == nullptr)
        return retNode;

      /// 更新结点高度
      retNode->height = 1 + std::max(getNodeHeight(retNode->left), getNodeHeight(retNode->right));
      int balanceFactor = getBalanceFactor(retNode);

      /// 1. ll
      if (balanceFactor > 1 && getBalanceFactor(retNode->left) >= 0) {
        return rightRotate(retNode);
      }
      /// 2. rr
      if (balanceFactor < -1 && getBalanceFactor(retNode->right) <= 0) {
        return leftRotate(retNode);
      }
      /// 3. lr
      /**
       *           y                     y
       *        x    T4               z     T4                z
       *     T1   z         ==>     x   T3        =>      x       y
       *       T2  T3             T1 T2                 T1 T2   T3 T4
       */
      if (balanceFactor > 1 && getBalanceFactor(retNode->left) < 0) {
        retNode->left = leftRotate(retNode->left);
        return rightRotate(retNode);
      }

      /// 4. rl
      /**
       *          y                   y                           z
       *       T1   x       =>     T1   z           =>        y       x
       *          z   T4              T2  x                 T1 T2   T3 T4
       *        T2 T3                   T3  T4
       */
      if (balanceFactor < -1 && getBalanceFactor(retNode->right) > 0) {
        retNode->right = rightRotate(retNode->right);
        return leftRotate(retNode);
      }
      return retNode;
    }
``` 

### 红黑树  

红黑树是个经典的平衡二叉树，但是其平衡指的是黑平衡，其最大高度可能为2(logn).  

红黑树有5个性质，满足其5个性质的就称为红黑树。

1. 所有节点为黑即红  
2. 根节点是黑色结点
3. 所有处于叶子的空节点是黑色结点
4. 红色结点的孩子是黑色结点 
5. 所有从根节点出发到叶子结点的路径中，黑色结点的数目一样. (黑平衡)

红黑树和2-3树等价，理解了2-3树，那么红黑树也就能理解为啥有两种颜色。黑色结点对应2-3树的2结点，表示不能融合。
红色结点表示2-3结点的3结点，表示要与父亲结点融合成3结点。所以红色其实是维护了要融合的这个信息。每次加入元素的时候仍然是按照BST的性质去查找。那么如何去维护红黑树的性质了，这其中也涉及到旋转操作。下面我列出红黑树插入节点的相关代码.  

插入节点需要考虑是加入到2孩子结点还是3孩子结点，不管是加入2孩子还是3孩子都可以统一成下面的插入流程。 

默认新生成的结点颜色为红色，表示新生成的结点是可以发生融合的，因为新生成的结点必然是要与2孩子结点或者3孩子结点融合。

下面的实现中红色结点不能插入到父结点的右边，是因为红色结点表示的是2-3结点左边的结点，所以红色结点具备左倾性质。 

红黑树插入和删除的操作比AVL树更少，多以在插入和删除操作为主的情况下，红黑树性能更优。

``` 
 /**
     *    Node        左旋转              X
     *    /   \       ===>             /   \
     *   T1    X                     Node   T3
     *        /  \                   /  \
     *       T2  T3                T1   T2
     */
    Node *leftRotate(Node *node) {
        Node *x = node->right;
        node->right = x->left;
        x->left = node;

        x->color = node->color;   /// 左旋保持跟结点颜色不变
        node->color = Color::red;
        return x;
    }

    /**
     *      Node                X
     *      /  \       右旋转   /  \
     *     X   T3      =>    T1   Node
     *    / \                     /  \
     *   T1 T2                   T2  T3
     */
    Node *rightRotate(Node *node) {
        Node *x = node->left;
        node->left = x->right;
        x->right = node;

        x->color = node->color;   /// 旋转时根节点颜色传递
        node->color = Color::red;
        return x;
    }

    /**
     *      黑      flipColor       红
     *    /   \     ==>           /   \
     *   红    红                黑    黑
     */
    void flipColors(Node *node) {
        node->color = Color::red;
        node->left->color = Color::black;
        node->right->color = Color::black;
    }

    Node *insert(Node *node, Key key, Value value) {
      if (node == nullptr) {
        count++;
        return new Node(key, value);
      }
      if (key < node->key) {
        node->left = insert(node->left, key, value);
      }else if (key > node->key) {
        node->right = insert(node->right, key, value);
      }else {
        node->value = value;
      }

      /// 红黑树调整平衡
      /**
       *     black     black        black
       *     /          /           /          black             red
       *   red    ->  red    ->    red   ->    /   \     ->     /    \
       *                 \         /         red   red        black  black
       *                 red      red
       */

      if (isRed(node->right) && !isRed(node->left))
        node = leftRotate(node); /// 因为node->right更大，所以要作为根，即进行左旋操作

      if (isRed(node->left) && isRed(node->left->left))
        node = rightRotate(node);

      if (isRed(node->left) && isRed(node->right))
        flipColors(node);

      return node;
    }  
``` 

### 总结 

对每一种树提出来解决的问题要了解，对于其提供的操作要了解。