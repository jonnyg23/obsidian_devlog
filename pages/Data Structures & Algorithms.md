## Courses / Info
	- [[AlgoExpert]] - [Website](https://www.algoexpert.io)
### [[Python]]
	- #### Arrays
	- #### Binary Search Trees
	- #### Binary Trees
	- #### Dynamic Programming
	- #### Famous Algorithms
	- #### Graphs
	- #### Greedy Algorithms
	- #### Heaps
	- #### Linked Lists
	- #### Recursion
	- #### Searching
	- #### Sorting
	  collapsed:: true
		- **Bubble Sort**
		  collapsed:: true
			- **Goal**:
				- The goal is to sort a list of numbers from least to greatest.
			- **Conceptualize**:
				-
				  1. Start with the first pair of numbers and compare them.
				-
				  2. Whichever is larger we move to the rightmost position.
				-
				  3. Then move to the next two pair of numbers in the list & perform Steps 1 & 2.
				-
				  4. Iteratively do this until you reach of the end list. The largest number shall now be in the last position within the list.
				-
				  5. Return to Step 1 and restart the process. Continually perform these steps until all numbers are sorted in the list.
			- **Complexity**:
				- Worst: (Where N is the length of the input array)
					- **Time - O(N^2)**
					- **Space - O(1)**
				- Best:
					- Time - O(N)
					- Space - O(1)
			- **Implementation**:
				-
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
		-
	- #### Stacks
	- #### Strings
	- #### Tries