# 538 Puzzle - Can you escape a maze with no walls?

The original puzzle is posted [here](https://fivethirtyeight.com/features/can-you-escape-a-maze-without-walls/) by [Oliver Roeder](http://twitter.com/ollie) from the desk of Tom Hanrahan.

First of all, let's just solve the generic problem in order to solve this specific one. 

Second, let's reconsider the problem as an *escape* problem. You're at the smiley face, and you want to get out of the maze to the border. The behavior of S, U, and X squares is the same, but L squares send you right and R squares send you left. **We want to find an escape path from the goal to any boundary in the fewest number of hops**.

The key to both of the solutions below is that we'll split each grid square into what it really is: four nodes (top left bottom right), each of which has a different ability to connect to neighboring nodes. 

## Network Visualization Approach
Why don't we just *build* this network and color the nodes by the type of node. If there is a connected path between a goal node and a boundary node, we'll just be able to see it. 

We'll use some code called [webweb](https://webwebpage.github.io) that makes network visualizations easy from Python, recently rewritten by my grad student, [Hunter Wapman](http://twitter.com/hneutr). 

The code plots the network as a grid, as described above, and it freeze the position of the nodes. **You can move the nodes around with your mouse if you like** and trace the escape path. I get a path length of 18. You can also uncheck the box that freezes node movement, and the system will "relax" allowing you to chart the number of hops from the Goal to the Boundary. Image here, followed by the code and notes to reproduce all of this.

![grid view](538/grid.png)

![relaxed view](538/relaxed.png "Logo Title Text 1")



## Network Traversal Approach (no viz)
Number the positions of the grid like a matrix, with (1,1) being the upper left entry, and (n,n) being the bottom right entry. Now consider a network whose directed edges describe movement as dictated by the rules of the maze... but rather than considering each square in the gride as a single vertex, let each square in the grid be represented by four vertices, enumerated {1,2,3,4} corresponding to {up,left,down,right} directions. 

This means that we're going to take the $n^2$ positions in the grid and from them create $4n^2$ vertices in a network of one-hop moves. We'll also add another set of vertices representing the possible entry points to the game board from the outside, another $4n$ vertices. In total, our graph will have $N = 4n^2 + 4n$ vertices, or, if you've been taught to factor everything, $N = 4n(n+1)$ vertices. 

For convenience, we'll enumerate the vertices using 3 coordinates $(i,j,k)$ where $i$ and $j$ represent the position in the matrix (with row/column 0 and row/column $n$ being the boundaries) and $k \in \{1,2,3,4\}$ representing the four directions. (When this is coded up, we'll write 1-to-4 as 0-to-3 because python is zero-indexed).

Once we have the "forward" network, whose directed adjacency matrix is given by $A$---where $A_{uv}$ means that you can move from $u \to v$ in one step---our questions are generic network traversal questions. The transpose $B = A^{T}$ represents the movements _backward_ along the graph, and the entries of $B^t$ represent the number of paths there are from one node to another, backwards, in exactly $t$ steps. This means that we can consider the set of entries of $B^t$ that represent backward paths from **our goal** (a set of 4 nodes!) to **any boundary** (a set of 4n nodes!). Increment $t$ until one of those entries of $B^t$ is nonzero, and that's your answer to the puzzle. 

Note that the puzzle doesn't actually ask for the path, just the minimum number of steps, so we're good to use this method. 

Also note that _if_ the goal is unreachable, there will be no value of $t$ that allows us to move all the way through the graph. A cheap upper bound on $t$ should be $N-1$, since the longest traversal of the network would touch all nodes before getting to the goal, and I call this a cheap bound because $N$ includes the boundary nodes, which any maximum-length traversal would include only one of.