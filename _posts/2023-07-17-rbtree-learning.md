---
layout: post
title: RBTree learning
date: 2023-07-17 18:57 +0900
categories: en
---

RB-tree is always an intriguing topic when it comes to tech interviews. There are numerous memes related to it. The measurement of a software engineer's skills is often a controversial subject, and the ability to write RB-tree-related code becomes one of its focal points. However, I'm not going to talk more about those arguments here. Instead, I'm here to record my learning process of RB-tree.
Today is Saturday and the upcoming Monday is a public holiday, which means I have three days in a row to learn it. I plan to spend at most 2 hours a day and write down every step. The main resource I'm using is a classic book for learning algorithms called "Introduction To Algorithms".

## What is RB-tree

A read-black tree is a binary search tree with one extra bit of storage per node: its color, which can be either RED or BLACK. This is a definition from the book. Now, it brings us one more question: What is a binary search tree? Fortunately, we can easily find a detailed explanation of this data structure in the same book, as it is discussed right before the RB-tree section. 
A binary search tree is a tree of course and its binary means every node have at most two child nodes. Here is the reason why there is a "search" in its name:

> Let x be a node in a binary search tree. If y is a node in the left subtree of x, then y.key<=x.key. If y is a node in the right subtree of x, then y.key>=x.key.

So here is what makes RB-tree different from a normal binary search tree:

> 1. Every node is either red or black.  
> 2. The root is black.
> 3. Every leaf(NIL) is black.
> 4. If a node is red, then both its children are black.
> 5. For each node, all simple paths from the node to descendant leaves contain the same number of black nodes. 

RB-tree is a well-balanced tree meaning the distance(height) from its root to any leaf node doesn't vary significantly.

## Rotations

When you add a node or delete a node from a well-balanced tree, you may change its balance. You may need to rotate its subtree or itself while making sure it's still a binary search tree after rotation.  
There are two rotations: LEFT-ROTATE and RIGHT-ROTATE.
![ROTATION](image.png)
According to the property of the binary search tree, y.key is must bigger or equal to x.key. No matter what rotation happens, the relative locations between x and y must be correct as a binary search tree.
All in all, rotation is a significant tool to make sure the tree is balanced.

Here is the rotation code:

```java
    public static void leftRotate(RBTree tree,TreeNode node){
        TreeNode right = node.rightChild;
        node.rightChild = right.leftChild;
        if(right.leftChild != tree.sentinel){
            right.leftChild.parentNode = node;
        }
        right.parentNode = node.parentNode;
        if(node.parentNode==tree.sentinel){
            tree.root = right;
        }else if(node == node.parentNode.leftChild){
            node.parentNode.leftChild = right;
        }else {
            node.parentNode.rightChild = right;
        }
        right.leftChild = node;
        node.parentNode = right;
    }

    public static void rightRotate(RBTree tree,TreeNode node){
        TreeNode left = node.leftChild;
        node.leftChild = left.rightChild;
        if(left.rightChild != tree.sentinel){
            left.rightChild.parentNode = node;
        }
        left.parentNode = node.parentNode;
        if(node.parentNode == tree.sentinel) {
            tree.root = left;
        }else if(node.parentNode.leftChild == node){
            node.parentNode.leftChild = left;
        }else{
            node.parentNode.rightChild = left;
        }
        left.rightChild = node;
        node.parentNode = left;
    }
```

## Insertion

Insertion also is a way to build a RB-tree, if we know how to insert a node into a RB-tree then we can build a whole new RB-tree from scratch.

Before that let's see how to build a normal binary search tree.

```java
    public static TreeNode buildBinarySearchTree(){
        TreeNode root=new TreeNode(generateNumber(),0);
        for(int i=0;i<10;i++){
            TreeNode node = new TreeNode(generateNumber(),0);
            insertNode(root,node);
        }
        return root;
    }

    public static void insertNode(TreeNode root,TreeNode node){
        TreeNode child = root;
        TreeNode parent = root;
        while(Objects.nonNull(child)){
            parent=child;
            if(node.key<parent.key){
                child=parent.leftChild;
            }else {
                child=parent.rightChild;
            }
        }
        if(parent.key>node.key){
            parent.leftChild=node;
        }else{
            parent.rightChild=node;
        }
        node.parentNode=parent;
    }    
```

When we need to build a binary search tree we need to generate a root and insert the rest nodes one by one. Let's delve into how to build a RB-tree. As said before RB-tree is one kind of binary search tree but has its own properties. Here is the definition of a node in RB-tree, we also used this definition in the above code.

```java
    public static class TreeNode{
        public int key;
        public TreeNode leftChild;
        public TreeNode rightChild;
        public TreeNode parentNode;
        public int color;//1 red,2 black

        public TreeNode(int key,int color){
            this.key=key;
            this.color=color;
        }
    }
```

Because the root is black so we treat every new node as red.

```java
    public static RBTree buildRBTree(){
        RBTree rbTree = new RBTree();
        for(int i=0;i<10;i++){
            TreeNode node = new TreeNode(generateNumber(),1);
            node.leftChild = rbTree.sentinel;
            node.rightChild = rbTree.sentinel;
            if(rbTree.root == null){
                node.color = 2;
                node.parentNode = rbTree.sentinel;
                rbTree.root = node;
            }else{
                RBTreeInsert(rbTree,node);
            }
        }
        return rbTree;
    }

    public static void RBTreeInsert(RBTree rbTree,TreeNode node){
        TreeNode parent = rbTree.root;
        TreeNode child = rbTree.root;
        while(child!=rbTree.sentinel){
            parent = child;
            if(parent.key>node.key){
                child = parent.leftChild;
            }else{
                child = parent.rightChild;
            }
        }
        if(parent.key> node.key){
            parent.leftChild = node;
        }else{
            parent.rightChild = node;
        }
        node.parentNode = parent;
        RBInsertFixUp(rbTree,node);
    }
```

Compared with the normal binary search tree insertion, there few differences:

1. Every new node is red.
2. Using an object with root and sentinel to represent a tree.
3. After insertion, the tree is still a binary search tree but maybe not an RB-tree so need to fix that up by calling a method.

**RBInsertFixUp**

```java
    public static void RBInsertFixUp(RBTree rbTree,TreeNode node){
        while(node.parentNode.color == 1){
            if(node.parentNode == node.parentNode.parentNode.leftChild){
                TreeNode uncleNode = node.parentNode.parentNode.rightChild;
                if(uncleNode.color == 1){
                    node.parentNode.color = 2;
                    uncleNode.color = 2;
                    node.parentNode.parentNode.color = 1;
                    node = node.parentNode.parentNode;
                }else{
                    if(node == node.parentNode.rightChild){
                        node = node.parentNode;
                        leftRotate(rbTree,node);
                    }
                    node.parentNode.color = 2;
                    node.parentNode.parentNode.color = 1;
                    rightRotate(rbTree,node.parentNode.parentNode);
                }
            }else{
                TreeNode uncleNode = node.parentNode.parentNode.leftChild;
                if(uncleNode.color == 1){
                    node.parentNode.color = 2;
                    uncleNode.color = 2;
                    node.parentNode.parentNode.color = 1;
                    node = node.parentNode.parentNode;
                }else{
                    if(node == node.parentNode.leftChild){
                        node = node.parentNode;
                        rightRotate(rbTree,node);
                    }
                    node.parentNode.color = 2;
                    node.parentNode.parentNode.color = 1;
                    leftRotate(rbTree,node.parentNode.parentNode);
                }
            }
        }
        rbTree.root.color = 2;
    }
```

I'm not going to explain this stretch of code because I don't understand it thoroughly after 5 hours racking my brain. The code comes from the 13.3 section in "Introduction To Algorithm". Trying to understand it yourself if you want to.

From my perspective, the most important advantage of the RB-tree is that it's a well-balanced tree under any conditions. A well-balanced tree's height is very limited. 

Given an input list: 9,8,7,6,5,4,3,2,1 to build a normal binary search tree and an RB-tree. They would look like this:
**normal binary search tree**

```shell
                9
               /
              8
             /
            7
           /
          6
         /
        5
       /
      4
     /
    3
   /
  2
 /
1
```

**RB-Tree**

```shell
                 6(black)
            /                  \
          4(red)              8(red)
        /       \           /       \
    2(black)  5(black)   7(black)  9(black)   
   /    \
1(red)  3(red)
```

As you can see the RB-tree is more balanced which means it has more efficiency in the worst-case.

## DELETION

Deleting a node also violates the properties of the RB-tree. Let's recall what should be done when deleting a node from a normal binary search tree. When a node is removed from a tree normally we need to find another node to supplement its original position. That process is called a transplant. Here is the transplant code we may need:

```java
    public static void transplant(BSTree bsTree,TreeNode node,TreeNode transplantTo){
        if(transplantTo.parentNode==null){
            bsTree.root = node;
            return;
        }
        if(transplantTo == transplantTo.parentNode.leftChild){
            transplantTo.parentNode.leftChild = node;
        }else{
            transplantTo.parentNode.rightChild = node;
        }
        if(node!=null){
            node.parentNode = transplantTo.parentNode;
        }
    }

    public static class BSTree{
        public TreeNode root;

        public BSTree(TreeNode treeNode){
            this.root = treeNode;
        }
    }
```

The **node** is what we want to tranplant from; it's the node we want to replace the removed node.

Here is the code that uses the above transplant code to delete a node from a normal binary search tree:

```java
    public static void deleteBSTreeNode(BSTree bsTree,TreeNode node){
        TreeNode leftTree = node.leftChild;
        TreeNode rightTree = node.rightChild;
        if(leftTree==null){
            transplant(bsTree,rightTree,node);
        }else if(rightTree==null){
            transplant(bsTree,leftTree,node);
        }else{
            TreeNode mini = miniMum(rightTree);//find the minimum value as successor. Please keep in mind the nimi doesn't hava a left node otherwise it's not the nimimum.
            if(mini != rightTree){
                transplant(bsTree,mini.rightChild,mini);
                mini.rightChild = node.rightChild;
                mini.rightChild.parentNode = mini;
            }
            transplant(bsTree,mini,node);
            mini.leftChild = node.leftChild;
            mini.leftChild.parentNode = mini;
        }
    }

    public static TreeNode miniMum(TreeNode node){
        TreeNode currentNode = node;
        TreeNode leftNode = currentNode.leftChild;
        while(leftNode!=null){
            currentNode = leftNode;
            leftNode = currentNode.leftChild;
        }
        return currentNode;
    }    
```

As shown in the code above, you can tell that there are two cases overall:

1. The node is deleted directly and its location is replaced by its child.
2. The node is replaced by a node whose direct parent is not the deleted node, so after the deleted node has been replaced the node that used to replace deleted node also needs to be replaced.

Let's begin talking about how to delete a node from an RB-tree. When the successor takes the removed node's position, it inherits the color of the removed node. From the point of removed node view, nothing has changed since the color remains. However, the removed node is not the only change here, the successor's position is also changed, and it's the most complicated one when deleting.

```java
    public static void deleteRBTreeNode(RBTree rbTree,TreeNode node){
        TreeNode leftNode = node.leftChild;
        TreeNode rightNode = node.rightChild;
        int originalColor = node.color;
        TreeNode supplementNode = node;
        if(leftNode == rbTree.sentinel){
            RBTransplant(rbTree,rightNode,node);
            supplementNode = rightNode;
        }else if(rightNode == rbTree.sentinel){
            RBTransplant(rbTree,leftNode,node);
            supplementNode = leftNode;
        }else {
            TreeNode successor = miniMumRB(rbTree,rightNode);
            originalColor = successor.color;
            supplementNode = successor.rightChild;
            if(successor != rightNode){
                RBTransplant(rbTree,successor.rightChild,successor);
                successor.rightChild = node.rightChild;
                successor.rightChild.parentNode = successor;
            }else{
                supplementNode.parentNode = successor; // in case successor.rightChild is sentinel
            }
            RBTransplant(rbTree,successor,node);
            successor.leftChild = node.leftChild;
            successor.leftChild.parentNode = successor;
            successor.color = node.color;
        }
        if(originalColor == 2){
            RBDeleteFixUp(rbTree,supplementNode);
        }
    }

    public static void RBDeleteFixUp(RBTree rbTree, TreeNode node){
        while(node != rbTree.root && node.color ==2){
            if(node == node.parentNode.leftChild){
                TreeNode sibling = node.parentNode.rightChild;
                if(sibling.color == 1){
                    sibling.color = 2;
                    node.parentNode.color = 1;
                    leftRotate(rbTree,node.parentNode);
                    sibling = node.parentNode.rightChild;
                }
                if(sibling.leftChild.color==2 && sibling.rightChild.color==2){
                    sibling.color = 1;
                    node = node.parentNode;
                }else{
                    if(sibling.rightChild.color==2){
                        sibling.leftChild.color = 2;
                        sibling.color = 1;
                        rightRotate(rbTree,sibling);
                        sibling = node.parentNode.rightChild;
                    }
                    sibling.color = node.parentNode.color;
                    node.parentNode.color = 2;
                    sibling.rightChild.color = 2;
                    leftRotate(rbTree,node.parentNode);
                    node = rbTree.root;
                }
            }else{
                TreeNode sibling = node.parentNode.leftChild;
                if(sibling.color == 1){
                    sibling.color = 2;
                    node.parentNode.color = 1;
                    rightRotate(rbTree,node.parentNode);
                    sibling = node.parentNode.leftChild;
                }
                if(sibling.leftChild.color ==2 && sibling.rightChild.color==2){
                    sibling.color = 1;
                    node = node.parentNode;
                }else{
                    if(sibling.leftChild.color ==2){
                        sibling.color = 1;
                        sibling.rightChild.color = 2;
                        leftRotate(rbTree,sibling);
                        sibling = node.parentNode.leftChild;
                    }
                    sibling.color = node.parentNode.color;
                    node.parentNode.color = 2;
                    sibling.leftChild.color = 2;
                    rightRotate(rbTree,node.parentNode);
                    node = rbTree.root;
                }
            }
        }
        node.color = 2;
    }    
```

I also don't understand this code so I'm not going to explain it here.

## The End

RB-tree is indeed a complicated structure. I can't understand it even after more than 10 hours of study. My head is a mess right now, can't think anything efficiently. Maybe I'll review this code to rethink it again. Maybe....

## Review(2023/7/22)

Here comes the review, after another round of thinking. I think I may explain the RBTree insertion and deletion.

### Review about Insertion

The ultimate goal of RBTree is to build a well-balanced tree. When a node is added that goal might be disrupted: one branch becomes longer. If that branch is the left one, we may need a right rotation to correct it and vice versa。 That's the reason why we need to discuss two situations separately. Let's use if the new node is added to its parent's left branch as an example to explain.

Let's be clear: adding a node doesn't mean the balance is sabotaged. If we have three nodes already in the tree:

> R means red and B means black.

```shell
  2B
 / \
1R  3R
```

Then we add a new node 0.

```shell
     2B
    / \
   1B  3B
  /
 0R 
```

As you can tell, the color may change but the structure maintains. There is no rotation after insertion. In this sutiation the corresponding code is:

```java
if(uncleNode.color == 1){
    node.parentNode.color = 2;
    uncleNode.color = 2;
    node.parentNode.parentNode.color = 1;
    node = node.parentNode.parentNode;
    .....
rbTree.root.color = 2;
```

Why do we need to check the color of uncle node?

Before we call the fix-up method there are three things we already knew:

1. The new node is red.
2. The new node's parent is red.
3. The grandparent node is black.

If the uncle node is red, then the parent node and uncle node are the same color and that means before the insertion from the new node's grandparent node to every leaf, the number of nodes are same. If we see this from the perspective of the grandparent node, the new node is creating a new layer in the tree and it's the first node of that layer, like the example we give above.

When the uncle node is black, means the colors between the parent node and the uncle node are different and this is the case where we need a rotation. It's easy to understand from the perspective of the grandparent node: the left subtree and the right subtree have the same black node but the left tree has two consecutive red nodes. The left subtree has more nodes than the right subtree and we need a rotation. Here, the left subtree has more red nodes. If the two consecutive red nodes are in a row on the left path of the grandparent node, then we can just do a right rotation to move one node to its right subtree.

*red nodes in a row, 1B is the grandparent node.*

```java
       2B
      / \
     1B  3B
    /
   0R 
  /
-1R
```

What if the new node is its parent right node?

```java
     2B
    / \
   1B  3B
  /
 0R 
  \
   0.5R
```

In this case, let's just rotate it as the red nodes are in a row.

```java
     2B
    / \
   0B  3B
    \
     1R
    /
   0.5R  
```

The new node follows its parent node becomes the right node of its grandparent node, which means the problem remains.

Let's rewind to what we need to do here:

1. move one red node to the right subtree.

So if the new node is its parent node's right tree, the rotation doesn't meet this goal. We need a left rotation between the new node and its parent node.

```java
if(node == node.parentNode.rightChild){
    node = node.parentNode;
    leftRotate(rbTree,node);
}
```

Let's do the right rotation on the grandparent node now. There is a color change between the parent node and the grandparent node, which means the grandparent node with red color now is moved to its original right subtree and the new node is still on the left subtree.

```shell
```java
       2B
      / \
     0B  3B
    / \
 -1R   1R
```

Now from the original grandparent node's view, all its subtrees are well balanced.

### Review about Deletion

Now, let's delve into what happens when deleting a node from a RBTree. The deleted node has a supplement and gets its color, so nothing changes. However, the supplement node has a supplement node and that can be a problem only when the supplement node used to replace deleted node is black.

**Let's call the node that has the original position of the supplement node as A.**

If A is its parent node's left node, then from the standpoint of the parent node its left subtree misses one black node. We need to do a left rotation to correct that.

If A's sibling node is red, then what do we have here?

1. The parent node must be black.
2. A's color is undetermined.

Let' go back to before deletition, the supplement node is black but its sibling is black and it was a well-balacked tree, so the sibling tree must have more nodes.

> From the standpoint of parent node, all subtrees' black nodes are same but one of its direct children is red.

After deletion, the left subtree loses one more node. Here, the subtree rooted parent node is definitly not balanced and we need a left rotation to correct is first.

```java
if(sibling.color == 1){
    sibling.color = 2;
    node.parentNode.color = 1;
    leftRotate(rbTree,node.parentNode);
    sibling = node.parentNode.rightChild;
}
```

If the sibling node is black and all the sibling node's children nodes are black, the A node must be black. Because before the deletion A node is at the same layer with sibling node's children, and it also means the number of nodes are same. In this case we can just reduce one black node on the right subtree.

```java
if(sibling.leftChild.color ==2 && sibling.rightChild.color==2){
    sibling.color = 1;
    node = node.parentNode;
}
```

Here, a final left rotation is inevitable. Before the rotation, let's think from the perspective of the sibling node if its left child node is red, then the left child has more nodes than its right node. After the rotation, the left child node is also in the left subtree and the original right subtree becomes an uncle node, which also means any leaf from the left child node tree to its grandparent node has a longer path. Before the rotation, the left subtree is already longer and after rotation, it becomes even longer than the right subtree. So before the final rotation, if the left node is red, we need to do a right rotation on the sibling node to move one node to its right subtree.

```java
if(sibling.rightChild.color==2){
    sibling.leftChild.color = 2;
    sibling.color = 1;
    rightRotate(rbTree,sibling);
    sibling = node.parentNode.rightChild;
}
```

Untill now, here is the info we have:

1. The sibling node is black.
2. The sibling node's right child node is red.

The final rotation here will add one black node, which is the sibling node, in the left subtree and reduce one black node, which is the sibling node, in the sibling subtree. The difference between these two subtrees means the RBTree isn't balanced, so we add a new black node in the sibling subtree before rotation.

```java
sibling.color = node.parentNode.color;
node.parentNode.color = 2;
sibling.rightChild.color = 2;// add a new black node
```

**There is no formal mathematical proof process here, only my speculations and conjectures so they may not be correct.**
