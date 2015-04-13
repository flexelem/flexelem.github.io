---
title: Binary Search Trees
author: buraktas
layout: post
permalink: /binary-search-trees/
dsq_thread_id:
  - 2730152008
categories:
  - Data Structures
tags:
  - binary-search-trees
  - data-structures
  - java
comments: true
---
A Binary Search Tree (BST) is a tree data structure that supports many dynamic operations includes ;

<div>
  <ul>
    <li>
      Search
    </li>

    <li>
      Minimum
    </li>

    <li>
      Maximum
    </li>

    <li>
      Insert
    </li>

    <li>
      Delete
    </li>

    <li>
      Predecessor
    </li>

    <li>
      Successor
    </li>
  </ul>
</div>

<!--more-->

Basic operations in a binary search tree take time proportional to the height of the tree. For a complete binary tree with *n* nodes, such operations run in \(&Theta;(lgn)\) worst-case time. If the tree is a linear chain of n nodes than the same operations will take \(&Theta;(n)\) worst-case time.

<h3> Binary Search Tree Property </h3>

There are a few properties to define a tree as a binary search tree;

<div>
  <ul>
    <li>
      Each node <b>x</b> has a left child, a right child , and a parent. If <b>x</b> doesn't have any child it should point <b>NULL</b>. Only root has no parent.
    </li>

    <li>
      Every node is represented by a key ( this can be any kind of comparable object - we will use Integers in this example- ). 
    </li>
    
    <li>
      For each node <b>x</b>, <b>x</b>'s left key is less than or equal to <b>x</b>, and its right key is greater than <b>x</b>.
    </li>
  </ul>
</div> 

Here is a representation a simple BST.

![]({{ site.url }}/public/images/2014/06/binary_tree_representation.png)
      
<h3> Height of BST </h3>

There are many possible trees for same set of keys. Thus, a height of a tree could be anywhere from ~log<sub>2</sub>n to ~n. Additionally, calculating height of a BST takes \(O(n)\) time because we have to visit every node.
      
![height_of_bst]({{ site.url }}/public/images/2014/06/height_of_BST.png)

<pre><code class="language-java">public int height(Node root) {
    if (root == null) {
        return 0;
    }

    // Height of every leaf is zero
    if (root.getLeftChild() == null && root.getRightChild() == null) {
        return 0;
    }

    int leftHeight = height(root.getLeftChild());
    int rightHeight = height(root.getRightChild());
    int max = Math.max(leftHeight, rightHeight) + 1;
    return max;
}</code>
</pre>

<h3> Tree Walk </h3>

<b>Inorder Tree Walk :</b>
The binary search tree property allows us to print keys in sorted order by a simple recursion algorithm which is called <b>inorder tree walk</b>. Walk order is;
      
<div>
  <ol>
    <li>
      left subtree
    </li>
    <li>
      root
    </li>
    <li>
      right subtree
    </li>
  </ol>
</div>
      
inorderTreeWalk procedure is follows;

<pre><code class="language-java">public void inorderTreeWalk(Node x) {
     if (x != null) {

         inorderTreeWalk(x.getLeftChild());
         System.out.println(x.getKey());
         inorderTreeWalk(x.getRightChild());
     }
}</code>
</pre>
      
In order tree walk take \(O(n)\) time because it traverses each node in the tree recursively.
      

<b>Preorder Tree Walk :</b><br>
Traverse order;

<div>
  <ol>
    <li>
      root
    </li>
    <li>
      left subtree
    </li>
    <li>
      right subtree
    </li>
  </ol>
</div>
      
Its running time is \(O(n)\) as well. Its procedure is as follows;
    
<pre><code class="language-java" >public void preorderTreeWalk(Node x) {
    if (x != null) {

		System.out.println(x.getKey());
        preorderTreeWalk(x.getLeftChild());
        preorderTreeWalk(x.getRightChild());
    }
}</code>
</pre>
      
<b>Postorder Tree Walk: </b><br>
Traverse order;
      
<div>
  <ol>
    <li>
      left subtree
    </li>
    <li>
      right subtree
    </li>
    <li>
      root
    </li>
  </ol>
</div>
      
Its running time is \(O(n)\) as well. Its procedure is as follows;

<pre><code class="language-java">public void postorderTreeWalk(Node x) {
    if (x != null) {

		System.out.println(x.getKey());
        postorderTreeWalk(x.getLeftChild());
        postorderTreeWalk(x.getRightChild());
    }
}</code>
</pre>
      
In addition, the iterative versions of tree walks could be find at its <a href="http://en.wikipedia.org/wiki/Tree_traversal" title="wiki">wiki page</a>
      
<h3> Query Operations </h3>
      
<b>Search : </b><br> 
To search for a key <b>k</b> in BST;
      
<div>
  <ul>
    <li>
      Start at the root.
    </li>
    <li>
      Traverse left <code>(if k < current key) / right (if k > current key)</code> child pointers as needed.
    </li>
    <li>
      Return node with key k or NULL ,as appropriate. 
    </li>
  </ul> 
</div> 

<pre><code class="language-java" >public Node search(Node x, int key) {
    while (x != null && key != x.getKey()) {

        if (key &lt; x.getKey()) {
            x = x.getLeftChild();
        } else {
            x = x.getRightChild();
        }
    }

    return x;
}</code>
</pre>
            
The running time of search procedure is \(O(height)\).
            
<b> Minimum and Maximum :</b><br /> To compute the minimum (maximum) of a BST ;
            
<div>
  <ul>
    <li>
      Start at root.
    </li>
    <li>
      Follow left child pointers (right pointers for maximum) until you can't anymore.
    </li>
    <li>
      return the last key found
    </li>
  </ul>
</div>

<pre><code class="language-java" >public Node treeMinimum(Node x) {
    while (x.getLeftChild() != null) {
        x = x.getLeftChild();
    }

    return x;
}</code>
</pre>

<pre><code class="language-java" >public Node treeMaximum(Node x) {
    while (x.getRightChild() != null) {
        x = x.getRightChild();
    }

    return x;
}</code>
</pre>
            
Both the procedures run in \(O(height)\) time.

<b>Successor : </b><br /> 
The successor of a node <b>x</b> is the node with smallest key greater than <b>x</b>'s key. To compute the successor of a node <b>x</b> ;            
<div>
  <ul>
    <li>
      Easy Case : If <b>x</b>'s right subtree is nonempty, return minimum key in right subtree.
    </li>
    <li>
      Otherwise : Find the first ancestor <b>y</b> which is also a left child of it's parent <b>z</b>. 
    </li>
  </ul> 
</div> 

![bst_successor]({{ site.url }}/public/images/2014/06/bst_successor-.png)

The procedure is as follow;

<pre><code class="language-java">public Node treeSuccessor(Node x) {
    if (x.getRightChild() != null) {
        return treeMinimum(x.getRightChild());
    }

    Node parent = x.getParent();

    while (parent != null && parent.getRightChild() == x) {
        x = parent;
        parent = x.getParent();
    }

    return parent;
}</code>
</pre>
                  

<b>Predecessor : </b><br> 
The predecessor of a node <b>x</b> is the node with greatest key less than <b>x</b>'s key. To compute the predecessor of a node <b>x</b> ;

                  
<div>
  <ul>
    <li>
      Easy Case : If <b>x</b>'s left subtree is nonempty, return maximum key in left subtree.
    </li>
    <li>
      Otherwise : Follow parent pointers until you get to a key less than k.
    </li>
  </ul>
</div>
                  
![predecessor]({{ site.url }}/public/images/2014/06/bst_predecessor1.png)

The procedure is as follow;

<pre><code class="language-java">public Node treePredecessor(Node x) {
    if (x.getLeftChild() != null) {
        return treeMaximum(x.getLeftChild());
    }

    Node parent = x.getParent();

    while (parent != null && parent.getKey() &gt; x.getKey()) {
        parent = parent.getParent();
    }

    return parent;
}</code>
</pre>

Both the procedures run in \(O(height)\) time.

<h3> Insertion and Deletion </h3>
                  
While insertion and deletion operations the binary search tree property should be preserved.
                  
<b> Insertion : </b><br> 
To insert a key <b>k</b>;
                  
<div>
  <ul>
    <li>
      Search a proper position for <b>x</b>.
    </li>
    <li>
      While searching maintain a trailing pointer <b>y</b> which is parent of <b>x</b>.
    </li>
    <li>
      Insert <b>x</b> as left / right child of its parent <b>y</b>.
    </li>
  </ul>
</div>


![insert_key]({{ site.url }}/public/images/2014/06/insert_key_in_BST.png)

insert procedure is as follows;

<pre><code class="language-java">public void insert(Node x) {
    Node y = null;
    Node z = root;

    while (z != null) {
        y = z;

        if (x.getKey() &lt;= z.getKey()) {
            z = z.getLeftChild();
        } else {
            z = z.getRightChild();
        }
    }

	// set y as x's parent
    x.setParent(y);

	// if root is already null then set x as root
    if (root == null) {
        root = x;
    } else if (x.getKey() &lt;= y.getKey()) { // set x as left child of its parent y
        y.setLeftChild(x);
    } else { // set x as right child of its parent y
        y.setRightChild(x);
    }
}</code>
</pre>

The running time of insertion is \(O(height)\).
                  
<b> Deletion : </b><br> 
To delete a key <b>k</b> from a search tree;
                  
<div>
  <ul>
    <li>
      Search for <b>k</b>.
    </li>
    <li>
      Easy Case (k's no children) : Delete <b>k</b>'s node from tree.
    </li>
    <li>
      Medium Case (k's one child) : Unique child has to know its parents position.
    </li>
    <li>
      Difficult Case (k's two children) : Compute <b>k</b>’s predecessor <b>l</b>, swap them, and deleting <b>l</b> will do the trick.
    </li> 
  </ul> 
</div> 

An illustration of deleting a key for each case;
                        
![delete_key]({{ site.url }}/public/images/2014/06/delete_bst.png)

The delete procedure will use a subroutine which is called <em>transplant</em> to handle with the cases when <b>x</b> has no children, or only one child. Transplant and delete procedures are as follows;

<pre><code class="language-java">public void delete(int key) {
    Node delNode = search(root, key);
 
    if (delNode.getLeftChild() == null) {
        transplant(delNode, delNode.getRightChild());
    } else if (delNode.getRightChild() == null) {
        transplant(delNode, delNode.getLeftChild());
    } else {
 
        Node y = treePredecessor(delNode);
        swapKeys(delNode, y);
 
        Node z = y.getLeftChild();
        transplant(y, z);
    }
}</code>
</pre>

<pre><code class="language-java">private void transplant(Node u, Node v) {
    if (u.getParent() == null) { // we delete the root
        root = v;
     } else if (u.getParent().getLeftChild() == u) {
        u.getParent().setLeftChild(v);
    } else {
        u.getParent().setRightChild(v);
    }

    if (v != null) {
        v.setParent(u.getParent());
    }
}</code>
</pre>
                        
The running time of delete procedure is \(O(height)\).

Here you can find the complete implementation of a BST by java.<br /> <a href="https://gist.github.com/flexelem/63c6897c075e4b8c4139">BST</a>