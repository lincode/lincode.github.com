---
layout: post
title: 二叉树
description: “LeetCode Swift” 二叉树部分的总结，主要涉及二叉树的遍历。
category: blog
---

## 树

树结构有广泛的使用场景。这是因为我们的现实生活中存在大量的层次关系。例如，家谱，部门组织结构等。

树实际相当于对序列（线性结构，例如：数组，栈，队列，链表）建立了一个索引。这个索引可以把原来O(n)的查找操作变为O(logn)。可以简单地理解为在线性结构上构造了一个二分查找算法。可以说，树是通过其结构来表达了一种划分查找方法，这一方法相比于遍历搜索的复杂度O(n)，一般情况下复杂度仅有O(logn)。

在拥有比线性结构更好的效率的同时，树结构又拥有动态改变存储空间的便利。如果，仅仅考虑查找效率，散列表比树更优秀，其查找时间复杂度仅为O(1)。之所以不能用散列来取代树，是因为散列需要预先开辟大量空间，并不是所有场景下都可以这么做；而如果空间不够，则会出现散列冲突（索引结构被破坏）。对散列表扩容需要对全表数据重新排列，在大数据量的情况下，这几乎不可能。树结构则可以很容易地动态添加和删除。

## 二叉树

二叉树的每个节点最多含有两个子树。普通树都可以通过转化为二叉树。对普通树使用儿子兄弟表示法，再旋转树即可将普通树转化为二叉树。

### 二叉树的存储

如果是[完全二叉树](https://zh.wikipedia.org/wiki/%E4%BA%8C%E5%8F%89%E6%A0%91#.E4.BA.8C.E5.8F.89.E6.A0.91.E7.9A.84.E7.B1.BB.E5.9E.8B)，或者接近完全的二叉树，使用数组存储树结构是一个不错的选择。在不完全的情况下，使用数组表示树结构会出现表示无节点的很多空白空间。这是对存储空间的浪费。数组表示法，相当于把树补全为完全二叉树，无数据的地方补空位标记。然后广度遍历树所得到的数组。

但这样使用数组存储树，就失去了动态改变存储空间的便利。所以，一般情况下，特别是在内存中使用数时，我们还是使用链式结构表示树结构。

链式结构在程序中表现为指针。使用 Swift 代码，可以实现为如下的类：

    public class TreeNode {

      public var val: Int
      public var left: TreeNode?
      public var right: TreeNode?

      init(_ val: Int) {
        self.val = val
        self.left = nil
        self.right = nil
      }
    }

[297. Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) 是一道序列化，和反序列化二叉树的题目。这道题，以及 LeetCode 上关于树的序列化，皆是使用数组表示法。

## 二叉树的遍历

遍历二叉树是对二叉树最重要的操作。举个例子，如果我们把一个数学表达式用一颗二叉树表示。对这颗二叉树分别进行先序，中序和后序遍历就可以得到这个数学表达式的前缀表达式，中缀表达式，和后缀表达式。

二叉树的遍历可以分为两种方式：深度优先，和广度优先。

### 深度优先

深度优先，可以分为先序遍历，中序遍历和后序遍历三种。

#### 先序

先序遍历过程可以描述为：

- 访问根节点
- 先序遍历其左子树
- 先序遍历其右子树

##### 递归实现

先序遍历遍历过程的描述其实是一个递归过程。可以直观地写成如下代码：

    func preorderTraversalRecursive(root: TreeNode?) {
      if let root = root {
        print(root.val) // visit root node
        let left = preorderTraversalRecursive(root.left) // recursively visit left subtree
        let right = preorderTraversalRecursive(root.right) // recursively visit right subtree
      }
    }

##### 迭代实现

如果不使用递归，可以使用迭代。利用 Swift 的 GeneratorType 将迭代过程提出来成为一个 Generator，这样就达到了代码复用的目的，在不同的地方可以方便地进行树的遍历。每次对 Generator 调用 `next()` 将返回一个待访问的节点。这里需要借助一个辅助的数据结构：栈(Stack) 来实现迭代过程。

    public class BinaryTreePreorderTravesalGenerator: GeneratorType {

      var stack = Stack<TreeNode>()

      init(root: TreeNode) {
        self.stack.push(root)
      }

      public typealias Element = Int

      private func hasNext() -> Bool {
        return !self.stack.empty()
      }

      public func next() -> Element? {

        if hasNext() {
          let node = self.stack.pop()

          if let right = node.right {
            stack.push(right)
          }

          if let left = node.left {
            stack.push(left)
          }

          return node.val
        }
        
        return nil
      }
    }

如果希望像下面代码中那样使用 `for in` 形式方便地遍历二叉树，或者将树和 map, filter, reduce 等函数配合使用。

    for node in tree {
        print(node)
    }

    tree.map { $0 }

你只需要使用 extension 为树扩展 SequenceType 协议：

    extension TreeNode: SequenceType {

      public func generate() -> BinaryTreePreorderTravesalGenerator {
        return BinaryTreePreorderTravesalGenerator(root: self)
      }

    }

此后，就能方便地先序遍历二叉树。

##### Morris

递归，和使用栈的迭代实现的空间复杂度都是O(n) (递归本身就是系统使用栈空间实现的)。Morris 遍历可以做到O(1)的空间复杂度，即不使用栈，队列这些辅助数据结构就可以实现遍历。

Morris 遍历的步骤：

1. 如果当前节点的左孩子为空，则输出当前节点，并将其右孩子作为当前节点。
2. 如果当前节点的左孩子不为空，在当前节点的左子树中找到当前节点在中序遍历下的前驱节点。
    2.1. 如果前驱节点的右孩子为空，将它的右孩子设置为当前节点（标记当前节点）。输出当前节点。当前节点更新为当前节点的左孩子。
    2.2. 如果前驱节点的右孩子为当前节点，将它的右孩子重新设为空（恢复树的形状）。当前节点更新为当前节点的右孩子。
3. 重复以上1，2直到当前节点为空。

其代码实现如下：

    func preorderTraversalMorris(root: TreeNode?) -> [Int] {
      guard let root = root else {
        return []
      }

      var result: [Int] = []

      var current: TreeNode? = root
      while current != nil {

        guard let left = current!.left else {
          // 1.如果当前节点的左孩子为空，则输出当前节点，并将其右孩子作为当前节点。
          result.append(current!.val)
          current = current!.right
          continue
        }

        // 2. 如果当前节点的左孩子不为空，在当前节点的左子树中找到当前节点在中序遍历下的前驱节点。
        var preNode = left
        while preNode.right != nil && preNode.right !== current {
          preNode = preNode.right!
        }

        if let _ = preNode.right {
          //2.2. 如果前驱节点的右孩子为当前节点，将它的右孩子重新设为空（恢复树的形状）。当前节点更新为当前节点的右孩子。
          preNode.right = nil
          current = current!.right
        } else {
          //2.1. 如果前驱节点的右孩子为空，将它的右孩子设置为当前节点（标记当前节点）。输出当前节点。当前节点更新为当前节点的左孩子。
          preNode.right = current
          result.append(current!.val)
          current = current!.left
        }

      }

      return result
    }


完整的代码可以参考：[144. Binary Tree Preorder Traversal](https://github.com/lincode/LeetCode-Swift/blob/master/LeetCode-Swift/LeetCode-Swift/Solution/Medium/BinaryTreePreorderTraversal_M144.swift)。

#### 中序

中序遍历过程：

- 中序遍历其左子树
- 访问根节点
- 中序遍历其右子树

中序遍历的递归实现，形式上和先序遍历类似，只是调整了访问根节点的位置。其迭代遍历仍然需要使用栈作为辅助数据结构。

完整的代码可以参考：[94. Binary Tree Inorder Traversal](https://github.com/lincode/LeetCode-Swift/blob/master/LeetCode-Swift/LeetCode-Swift/Solution/Medium/BinaryTreeInorderTraversal_M94.swift)。

#### 后序

后序遍历过程：

- 后序遍历其左子树
- 后序遍历其右子树
- 访问根节点

后序遍历的递归实现，形式上和先序遍历类似，只是调整了访问根节点的位置。其迭代遍历需要使用两个栈作为辅助数据结构，来存储后续遍历左右子树的结果。

完整的代码可以参考：[144. Binary Tree Postorder Traversal](https://github.com/lincode/LeetCode-Swift/blob/master/LeetCode-Swift/LeetCode-Swift/Solution/Hard/BinaryTreePostorderTraversal_H145.swift)。

### 广度优先遍历

广度优先遍历又称为层次遍历，即优先访问离根节点最近的节点。广度优先遍历需要借助队列作为辅助的数据结构。这里给出广度优先遍历的 Generator 代码。每次调用 `next()` 都会以数组形式返回一个层次的节点值。


    public class BinaryTreeLevelOrderGenerator: GeneratorType {

        typealias QueueType = Queue<TreeNode>
      private var queue: QueueType

      public typealias Element = [Int]

      init(root: TreeNode) {
        queue = QueueType()
        queue.push(root)
      }

      private func hasNext() -> Bool {
        return !self.queue.empty()
      }

      public func next() -> Element? {
        var result: Element = []
        var queueNext = QueueType()

        if !hasNext() {
          return nil
        }

        for node in queue {

          if let right = node.right {
            queueNext.push(right)
          }

          if let left = node.left {
            queueNext.push(left)
          }

          result.insert(node.val, atIndex: 0)
        }
        queue = queueNext
        return result
      }

    }

完整的代码可以参考：[102. Binary Tree Level Order Traversak](https://github.com/lincode/LeetCode-Swift/blob/master/LeetCode-Swift/LeetCode-Swift/Solution/Easy/BinaryTreeLevelOrderTraversal_E102.swift)

## 参考

- [LeetCode OJ](http://www.leetcode.com/), online judge platform for preparing technical coding interviews。
- [LeetCode Swift](https://github.com/lincode/LeetCode-Swift)，LeetCode 的 Swift 答案集。

