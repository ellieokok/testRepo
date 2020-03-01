**Translator: [Ziming](https://github.com/ML-ZimingMeng/LeetCode-Python3)，[ellieokok](https://github.com/ellieokok)**

 **Author: [labuladong](https://github.com/labuladong)**

# Detailed Explanation of Union-Find

 Today I will talk about Union-Find algorithm, which is often referred as  Disjoint-Set algorithm, mainly to solve the problem of "dynamic connectivity" in graph theory. The nouns look high-end, but they’re very easy to understand.I will explain it later. Moreover, the application of this algorithm is also very interesting.

 Speaking of this Union-Find, it should be considered as my "Enlightenment Algorithm", because this algorithm is introduced at the beginning of *Algorithms 4th edition*. I had to say this algorithm shocked me because of it is very brilliant! Later I discovered that leetcode also has related topics and is very interesting. In addition, the solution given by *Algorithms 4th edition* can be further optimized. With only a small modification, the time complexity can be reduced to O (1).


 First, I will explain the phrase "dynamic connectivity".

### Ⅰ. Problem Introduction

 Briefly, dynamic connectivity  can be abstracted to connect a graph with lines. For example the following figure have 10 nodes in total, and they are disconnected, respectively marked with 0 ~ 9:

 ![](../pictures/unionfind/1.jpg)

 Now our Union-Find algorithm mainly  implement these two APIs:

 ```java
 class UF {
     /* Connecting the p and q */
     public void union(int p, int q);
     /* Judge whether p and q are connected */
     public boolean connected(int p, int q);
     /* Returns the number of connected components in the graph */
     public int count();
 }
 ```

  The word "connectivity" here meas an equivalence relation, that  is to say here has the following three properties:

 1. reflexivity: node `p` and ` p` node are connected.

 2. Symmetry: If node `p` and` q` are connected, then `q` and `p` are also connected.

 3. Transitivity: If node `p` and` q` are connected, and `q` and` r` are connected, then `p` and` r` are also connected.

 For example, in the previous picture, any two **different points** from 0 to 9 are not connected, then calling `connected` will return false and the connected components is 10.

 Now, if you call `union (0, 1)`, the 0 and 1 will be connected and the connected components are reduced to 9.

 Then, if we call `union (1, 2)` again, Nodes 0,1,2 will be connected. Calling `connected (0, 2)` will also return true, and the connected components are reduced to 8. 

 ![](../pictures/unionfind/2.jpg)

 It is very practical to judge this equivalence relationship, such as, the Compiler can use it to judge different references to the same variable, and  counting the number of friends in social networks.

 Now, you probably understand what dynamic connectivity mean. The key to the Union-Find algorithm is the efficiency of the `union` and` connected` functions. Then what model can we use to represent the connected state of this graph? And which data structure is more appropriate to implement the codes?

 ### Ⅱ. Basic thinking

 Note that, just now I separated the "model" from specific "data structure", there's a reason. Because we use forests (several trees) to represent the dynamic connectivity of the graph, and use arrays to implement this forest.

 How to use forest to represent connectivity? We set each node of the tree have a pointer to its parent node, and if it is the root node, this pointer points to itself. For example, the graph of 10 nodes we used before didn't communicate with each other at the beginning, like this:

 ![](../pictures/unionfind/3.jpg)

 ```java
 class UF {
     // recording connected components
     private int count;
     // the parent of node x is parent [x]
     private int[] parent;

     /* constructor,  n is the total number of nodes in the graph */
     public UF(int n) {
         // disconnected at first
         this.count = n;
         // parent node pointer points to itself
         parent = new int[n];
         for (int i = 0; i < n; i++)
             parent[i] = i;
     }

     /* Other functions */
 }
 ```

 **If any two nodes are connected, then the root node of one(any) node is connected the root node of the other node:**

 ![](../pictures/unionfind/4.jpg)

 ```java
 public void union(int p, int q) {
     int rootP = find(p);
     int rootQ = find(q);
     if (rootP == rootQ)
         return;
     // Merge two trees into one
     parent[rootP] = rootQ;
     // parent[rootQ] = rootP 
     count--; // Combine two components into one
 }

 /* Returns the root node of a node x */
 private int find(int x) {
     // parent[x] == x
     while (parent[x] != x)
         x = parent[x];
     return x;
 }

 /* Returns the number of the current connected components */
 public int count() { 
     return count;
 }
 ```

 **In this way, if the nodes `p` and` q` are connected, they must have the same root node:**

 ![](../pictures/unionfind/5.jpg)

 ```java
 public boolean connected(int p, int q) {
     int rootP = find(p);
     int rootQ = find(q);
     return rootP == rootQ;
 }
 ```

 At this point, the Union-Find algorithm is basically completed. Isn't it amazing? We can use arrays like this to simulate a forest, it is so clever to solve this more complex problem!

 Then what is the complexity of this algorithm? We find that the complexity of the main API `connected` and` union` is caused by the `find` function, so they have the same complexity as` find`.

 The main function of `find` is to traverse from a certain node to the root of the tree, and its time complexity is the height of the tree. We may habitually think the height of the tree is `logN`, but this is not necessarily the case. The height of `logN` exists only in a balanced binary tree. For general trees, extreme imbalance may occur, which cause the“ tree ”to almost degenerate into a“ linked list ”. In the worst case, the tree height may become` N`.

 ![](../pictures/unionfind/6.jpg)

 So in the above solution, the time complexity of `find`,` union`, `connected` is O (N). This complexity is not ideal. What you want graph theory to solve is the problem of huge data scales such as social networks. The calls to `union` and` connected` are very frequent, and each call requires linear time is completely unbearable.

 **The point is, how to find ways to avoid tree imbalances?** You just need a small track.

 ### Ⅲ. Balance optimization

 The key to know which situation may lead to imbalance is the union process：

 ```java
 public void union(int p, int q) {
     int rootP = find(p);
     int rootQ = find(q);
     if (rootP == rootQ)
         return;
     // Merge two trees into one
     parent[rootP] = rootQ;
     // parent[rootQ] = rootP also works
     count--; 
 ```

 At the beginning, we simply connected the tree where `p` was located under the root node of the tree where` q` was located. Then, a "top-heavy" imbalance situation may occur here, such as the following situation:

 ![](../pictures/unionfind/7.jpg)

 Over time, the tree may grow imbalanced. **We actually hope that the smaller trees are connected to the larger ones, so that we can avoid top-heavy and the tree will be more balanced**. The solution is to use an additional `size` array to record the number of nodes in each tree. We call it as "weight":

 ```java
 class UF {
     private int count;
     private int[] parent;
     // Added an array record tree "weight"
     private int[] size;

     public UF(int n) {
         this.count = n;
         parent = new int[n];
         // At first, each tree had only one node 
         // the weight should be initialized as 1
         size = new int[n];
         for (int i = 0; i < n; i++) {
             parent[i] = i;
             size[i] = 1;
         }
     }
     /* Other functions */
 }
 ```

 For instance, `size [3] = 5` means that the tree with node` 3` as its root, has a total of `5` nodes. This way we can modify the `union` method:

 ```java
 public void union(int p, int q) {
     int rootP = find(p);
     int rootQ = find(q);
     if (rootP == rootQ)
         return;
     
     // put smaller tree under the bigger one, the result will be more balanced
     if (size[rootP] > size[rootQ]) {
         parent[rootQ] = rootP;
         size[rootP] += size[rootQ];
     } else {
         parent[rootP] = rootQ;
         size[rootQ] += size[rootP];
     }
     count--;
 }
}
 ```

 Like this, by comparing the weight of the tree, we can ensure that the growth of the tree is relatively balanced, and the height of the tree is about the order of `logN`, which greatly improves the execution efficiency.

 At this time, the time complexity of `find`,` union`, and `connected` has been reduced to O (logN), even the data size is hundreds of millions, the time required is very small.

 ### Ⅳ. Path compression

 This step of optimization is particularly simple and clever. Can we further compress the height of each tree to keep the height constant?

 ![](../pictures/unionfind/8.jpg)

 In this way, `Find` can find the root node of a node in O (1) time, corresponding, complexity of ` connected` and `union` are reduced to O (1).

 To do this, just add a line in `find` Code:

 ```java
 private int find(int x) {
     while (parent[x] != x) {
         // Path compression
         parent[x] = parent[parent[x]];
         x = parent[x];
     }
     return x;
 }
 ```

 This operation is a little tricky ,we can see the GIF to understand its role (for clarity, this tree is more extreme).

 ![](../pictures/unionfind/9.gif)

 Therefore, each time the `find` function is called to traverse to the root of the tree, the tree height is shortened by hand, and eventually all the tree heights will not exceed 3 (the height may reach 3 when` union`).

 PS: The reader may ask, after the find process in this GIF graph is completed, the tree height is exactly equal to 3, but if there is a higher tree, the height after compression will still greater than 3, what should we do? This GIF scenario was edited by me to make it easy for everyone to understand path compression, but in practice, path compression is performed every time it use `find`, so the tree could not have grown to such high level, and this worry is unnecessary.

 ### Ⅴ. Summary

 Let's take a look at the whole code:

 ```java
 class UF {
     // Number of connected components
     private int count;
     // Store a tree
     private int[] parent;
     // Record the "weight" of the tree
     private int[] size;

     public UF(int n) {
         this.count = n;
         parent = new int[n];
         size = new int[n];
         for (int i = 0; i < n; i++) {
             parent[i] = i;
             size[i] = 1;
         }
     }
     
     public void union(int p, int q) {
         int rootP = find(p);
         int rootQ = find(q);
         if (rootP == rootQ)
             return;
         
         // The small tree is more balanced under the big tree
         if (size[rootP] > size[rootQ]) {
             parent[rootQ] = rootP;
             size[rootP] += size[rootQ];
         } else {
             parent[rootP] = rootQ;
             size[rootQ] += size[rootP];
         }
         count--;
     }

     public boolean connected(int p, int q) {
         int rootP = find(p);
         int rootQ = find(q);
         return rootP == rootQ;
     }

     private int find(int x) {
         while (parent[x] != x) {
             // Path compression
             parent[x] = parent[parent[x]];
             x = parent[x];
         }
         return x;
     }

     public int count() {
         return count;
     }
}
```


 The complexity of the Union-Find algorithm can be analyzed as follows: the constructor initializes the data structure requires O (N) time and space complexity. However, the time complexity required for `union`,` connected` and `count` is O (1).



 [Previous: How to schedule candidates' seats](../高频面试系列/座位调度.md)

 [Next：Application of Union-Find algorithm](../算法思维系列/UnionFind算法应用.md)

 [Contents](../README.md#目录)