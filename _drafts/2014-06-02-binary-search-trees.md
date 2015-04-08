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
---
A Binary Search Tree (BST) is a tree data structure that supports many dynamic operations includes ;

<div class="bullet list">
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

Basic operations in a binary search tree take time proportional to the height of the tree. For a complete binary tree with *n* nodes, such operations run in \(&Theta;(lgn)\) worst-case time. If the tree is a linear chain of n nodes than the same operations will take \(&Theta;(n)\) worst-case time.

### Binary Search Tree Property 

There are a few properties to define a tree as a binary search tree;

<div class="bullet list">
  <ul>
    <li>
      Each node <b>x</b> has a left child, a right child , and a parent. If <b>x</b> doesn&#8217;t have any child it should point <b>NULL</b>. Only root has no parent.
    </li>
    <li>
      Every node is represented by a key ( this can be any kind of comparable object &#8211; we will use Integers in this example- ). <li>
        For each node <b>x</b>, <b>x</b>&#8216;s left key is less than or equal to <b>x</b>, and its right key is greater than <b>x</b>.
      </li></ul> </div> 
      <p>
        Here is a representation a simple BST.
      </p>
      
      <p>
        <a href="http://www.buraktas.com/wp-content/uploads/2014/06/binary_tree_representation.png"><img src="http://www.buraktas.com/wp-content/uploads/2014/06/binary_tree_representation.png" alt="binary_tree_representation" width="362" height="230" class="aligncenter size-full wp-image-244" /></a>
      </p>
      
      <h3>
        Height of BST
      </h3>
      
      <p>
        There are many possible trees for same set of keys. Thus, a height of a tree could be anywhere from ~log<sub>2</sub>n to ~n. Additionally, calculating height of a BST takes \(O(n)\) time because we have to visit every node.
      </p>
      
      <p>
        <a href="http://www.buraktas.com/wp-content/uploads/2014/06/height_of_BST.png"><img src="http://www.buraktas.com/wp-content/uploads/2014/06/height_of_BST.png" alt="height_of_BST" width="465" height="408" class="aligncenter size-full wp-image-280" /></a>
      </p>
      
      <pre class="lang:java decode:true " >public int height(Node root) {
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
}</pre>
      
      <h3>
        Tree Walk
      </h3>
      
      <p>
        <b>Inorder Tree Walk :</b><br /> The binary search tree property allows us to print keys in sorted order by a simple recursion algorithm which is called <b>inorder tree walk</b>. Walk order is;
      </p>
      
      <div class="bullet list">
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
      
      <p>
        inorderTreeWalk procedure is follows;
      </p>
      
      <pre class="lang:java decode:true " >public void inorderTreeWalk(Node x) {

     if (x != null) {

         inorderTreeWalk(x.getLeftChild());
         System.out.println(x.getKey());
         inorderTreeWalk(x.getRightChild());
     }
}</pre>
      
      <p>
        In order tree walk take \(O(n)\) time because it traverses each node in the tree recursively.
      </p>
      
      <p>
        <b>Preorder Tree Walk :</b><br /> Traverse order;
      </p>
      
      <div class="bullet list">
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
      
      <p>
        Its running time is \(O(n)\) as well. Its procedure is as follows;
      </p>
      
      <pre class="lang:java decode:true " >public void preorderTreeWalk(Node x) {

    if (x != null) {

		System.out.println(x.getKey());
        preorderTreeWalk(x.getLeftChild());
        preorderTreeWalk(x.getRightChild());
    }
}</pre>
      
      <p>
        <b>Postorder Tree Walk:</b><br /> Traverse order;
      </p>
      
      <div class="bullet list">
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
      
      <p>
        Its running time is \(O(n)\) as well. Its procedure is as follows;
      </p>
      
      <pre class="lang:java decode:true " >public void postorderTreeWalk(Node x) {

    if (x != null) {

		System.out.println(x.getKey());
        postorderTreeWalk(x.getLeftChild());
        postorderTreeWalk(x.getRightChild());
    }
}</pre>
      
      <p>
        In addition, the iterative versions of tree walks could be find at its <a href="http://en.wikipedia.org/wiki/Tree_traversal" title="wiki">wiki page</a>
      </p>
      
      <h3>
        Query Operations
      </h3>
      
      <p>
        <b>Search : </b><br /> To search for a key <b>k</b> in BST;
      </p>
      
      <div class="bullet list">
        <ul>
          <li>
            Start at the root.
          </li>
          <li>
            Traverse left (if k < current key) / right (if k > current key) child pointers as needed.
          </li>
          <li>
            Return node with key k or NULL ,as appropriate. </ul> </div> <pre class="lang:java decode:true " >public Node search(Node x, int key) {

    while (x != null && key != x.getKey()) {

        if (key &lt; x.getKey()) {
            x = x.getLeftChild();
        } else {
            x = x.getRightChild();
        }
    }

    return x;
}</pre>
            
            <p>
              The running time of search procedure is \(O(height)\).
            </p>
            
            <p>
              <b> Minimum and Maximum :</b><br /> To compute the minimum (maximum) of a BST ;
            </p>
            
            <div class="bullet list">
              <ul>
                <li>
                  Start at root.
                </li>
                <li>
                  Follow left child pointers (right pointers for maximum) until you can&#8217;t anymore.
                </li>
                <li>
                  return the last key found
                </li>
              </ul>
            </div>
            
            <pre class="lang:java decode:true " >public Node treeMinimum(Node x) {

    while (x.getLeftChild() != null) {

        x = x.getLeftChild();
    }

    return x;
}</pre>
            
            <pre class="lang:java decode:true " >public Node treeMaximum(Node x) {

    while (x.getRightChild() != null) {

        x = x.getRightChild();
    }

    return x;
}</pre>
            
            <p>
              Both the procedures run in \(O(height)\) time.
            </p>
            
            <p>
              <b>Successor : </b><br /> The successor of a node <b>x</b> is the node with smallest key greater than <b>x</b>&#8216;s key. To compute the successor of a node <b>x</b> ;
            </p>
            
            <div class="bullet list">
              <ul>
                <li>
                  Easy Case : If <b>x</b>&#8216;s right subtree is nonempty, return minimum key in right subtree.
                </li>
                <li>
                  Otherwise : Find the first ancestor <b>y</b> which is also a left child of it&#8217;s parent <b>z</b>. </ul> </div> <p>
                    <a href="http://www.buraktas.com/wp-content/uploads/2014/06/bst_successor-.png"><img src="http://www.buraktas.com/wp-content/uploads/2014/06/bst_successor-.png" alt="bst_successor" width="628" height="291" class="aligncenter size-full wp-image-272" /></a>
                  </p>
                  
                  <p>
                    The procedure is as follow;
                  </p>
                  
                  <pre class="lang:java decode:true " >public Node treeSuccessor(Node x) {

    if (x.getRightChild() != null) {

        return treeMinimum(x.getRightChild());
    }

    Node parent = x.getParent();

    while (parent != null && parent.getRightChild() == x) {

        x = parent;
        parent = x.getParent();
    }

    return parent;
}</pre>
                  
                  <p>
                    <b>Predecessor : </b><br /> The predecessor of a node <b>x</b> is the node with greatest key less than <b>x</b>&#8216;s key. To compute the predecessor of a node <b>x</b> ;
                  </p>
                  
                  <div class="bullet list">
                    <ul>
                      <li>
                        Easy Case : If <b>x</b>&#8216;s left subtree is nonempty, return maximum key in left subtree.
                      </li>
                      <li>
                        Otherwise : Follow parent pointers until you get to a key less than k.
                      </li>
                    </ul>
                  </div>
                  
                  <p>
                    <a href="http://www.buraktas.com/wp-content/uploads/2014/06/bst_predecessor1.png"><img src="http://www.buraktas.com/wp-content/uploads/2014/06/bst_predecessor1.png" alt="bst_predecessor" width="543" height="291" class="aligncenter size-full wp-image-450" /></a>
                  </p>
                  
                  <p>
                    The procedure is as follow;
                  </p>
                  
                  <pre class="lang:java decode:true " >public Node treePredecessor(Node x) {

    if (x.getLeftChild() != null) {

        return treeMaximum(x.getLeftChild());
    }

    Node parent = x.getParent();

    while (parent != null && parent.getKey() &gt; x.getKey()) {

        parent = parent.getParent();
    }

    return parent;
}</pre>
                  
                  <p>
                    Both the procedures run in \(O(height)\) time.
                  </p>
                  
                  <h3>
                    Insertion and Deletion
                  </h3>
                  
                  <p>
                    While insertion and deletion operations the binary search tree property should be preserved.
                  </p>
                  
                  <p>
                    <b> Insertion : </b><br /> To insert a key <b>k</b>;
                  </p>
                  
                  <div class="bullet list">
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
                  
                  <p>
                    An illustration of inserting node with key 13;
                  </p>
                  
                  <p>
                    <a href="http://www.buraktas.com/wp-content/uploads/2014/06/insert_key_in_BST.png"><img src="http://www.buraktas.com/wp-content/uploads/2014/06/insert_key_in_BST.png" alt="insert_key_in_BST" width="616" height="355" class="aligncenter size-full wp-image-278" /></a>
                  </p>
                  
                  <p>
                    insert procedure is as follows;
                  </p>
                  
                  <pre class="lang:java decode:true " >public void insert(Node x) {

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
}</pre>
                  
                  <p>
                    The running time of insertion is \(O(height)\).
                  </p>
                  
                  <p>
                    <b> Deletion : </b><br /> To delete a key <b>k</b> from a search tree;
                  </p>
                  
                  <div class="bullet list">
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
                        Difficult Case (k's two children) : Compute <b>k</b>’s predecessor <b>l</b>, swap them, and deleting <b>l</b> will do the trick. </ul> </div> <p>
                          An illustration of deleting a key for each case;
                        </p>
                        
                        <p>
                          <a href="http://www.buraktas.com/wp-content/uploads/2014/06/delete_bst.png"><img src="http://www.buraktas.com/wp-content/uploads/2014/06/delete_bst.png" alt="delete_bst" width="960" height="268" class="aligncenter size-full wp-image-279" /></a>
                        </p>
                        
                        <p>
                          The delete procedure will use a subroutine which is called <em>transplant</em> to handle with the cases when <b>x</b> has no children, or only one child. Transplant and delete procedures are as follows;
                        </p>
                        
                        <pre class="lang:java decode:true " >public void delete(int key) {
 
    Node delNode = search(root, key);
 
    if (delNode.getLeftChild() == null) {
 
        transplant(delNode, delNode.getRightChild());
    } else if (delNode.getRightChild() == null) {
 
        transplant(delNode, delNode.getLeftChild());
    } else {
 
        Node y = treePredecessor(delNode);
        swapKeys(delNode, y);
 
		Node z = y.getLeftChild();
        if (y.getParent().getLeftChild() == y) {
 
            y.getParent().setLeftChild(z);
        } else {
 
            y.getParent().setRightChild(z);
        }
    }
}</pre>
                        
                        <pre class="lang:java decode:true " >private void transplant(Node u, Node v) {

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
}</pre>
                        
                        <p>
                          The running time of delete procedure is \(O(height)\).
                        </p>
                        
                        <p>
                          Here you can find the complete implementation of a BST by java.<br /> <a href="https://gist.github.com/flexelem/63c6897c075e4b8c4139">BST</a>
                        </p>