# Data Structures & Algorithms

## Courses / Info
- [[AlgoExpert]] - [Website](https://www.algoexpert.io)

## [[Python]]
### Arrays
### Binary Search Trees
### Binary Trees
### Dynamic Programming
### Famous Algorithms
### Graphs

#### Depth-first Search

- **Goal**:
	- Traverse the tree navigating from left to right & store all of the nodes' names in the input array, and return it.

- **Conceptualize**:
	- Each variable in the graph is called a node & those lower in the tree are referred to as children nodes.
	  1. The algorithm traverses through the leftmost side of the tree down through all of the children nodes until it hits the bottom where there are no more children nodes. It will iteratively add these node names to an array which will later be returned.
	  2. It will then step back and traverse other leaf/children nodes until all the left side of the tree has been iterated through.
	  3. It will then repeat this process through all of the initial children nodes from the top of the graph/tree.

- **Complexity**:
	- Worst: (Where v is the number of vertices of the input graph and e is the number of edges of the input graph)
		- Time - O(v + e)
		- Space - O(v)

**Implementation**

```python
class Node:
  def __init__(self, name):
	  self.children = []
	  self.name = name

  def addChild(self, name):
	  self.children.append(Node(name))
	  return self

  def depthFirstSearch(self, array):
	  array.append(self.name)
	for child in self.children:
		child.depthFirstSearch(array)
	return array
```

### Greedy Algorithms
### Heaps
### Linked Lists
### Recursion
### Searching
### Sorting

#### Bubble Sort

- **Goal**:
	- The goal is to sort a list of numbers from least to greatest.

- **Conceptualize**:
	  1. Start with the first pair of numbers and compare them.
	  2. Whichever is larger we move to the rightmost position.
	  3. Then move to the next two pair of numbers in the list & perform Steps 1 & 2.
	  4. Iteratively do this until you reach of the end list. The largest number shall now be in the last position within the list.
	  5. Return to Step 1 and restart the process. Continually perform these steps until all numbers are sorted in the list.

- **Complexity**:
	- Worst: (Where N is the length of the input array)
		- **Time - O(N^2)**
		- **Space - O(1)**
	- Best:
		- Time - O(N)
		- Space - O(1)

**Implementation**

```python
def bubbleSort(array):
for n in array:
	for i in range(len(array) - 1):
		left = array[i]
		right = array[i + 1]
		if (right > left) or (right == left):
			continue
		else:
			array[i] = right
			array[i + 1] = left
return array
```

### Stacks
### Strings
### Tries