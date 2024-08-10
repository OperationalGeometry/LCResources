# Binary Search Deep Dive

Binary search is the best way of finding a target element in a sorted collection. Here, I will decompose the algorithm into its' most fundamental components, generalize its' functionality, and then apply it to various prototypical leetcode problems.

Lesson Chunks:
- Algorithm Introduction and Decomposition
- Application #1
- Algorithm Generalization
- Application #2

## Binary Search Intro


```python
def binarySearch(sorted, target):
    # returns the index of a specified target element in a sorted array in O(log(n)) time. if sorted does not contain the target, return -1.

    L, R = 0, len(sorted) - 1
    while L <= R:
        mid = (L + R) // 2

        if sorted[mid] > target:
            R = mid - 1
        elif sorted[mid] < target:
            L = mid + 1
        else:
            return mid
    return -1

sorted = [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
targetIdx = binarySearch(sorted, 34)
targetIdx
```




    9



## High Level Overview
The key idea here is that when we guess an index of where the target element might be in the array, even if we get it wrong, we still get information regarding where the target element is relative to the guess. Let's say we always guess the middle of the array first. If the target is less than the middle element, we know that the target must be in the bottom half of the array. If the target is greater, it has to be in the upper half. Conversely, consider an unsorted collection. If we get the guess wrong, we don't get information about where to search next. This is why binary search only works on *sorted* arrays.

- We initalize two pointers, a left and a right at the beginning and end elements of the array respectively. This bounds our search space. In order to achieve O(log(n)) runtime, each iteration of our guess needs to divide our search space by a constant factor.

- We start our while loop, and iterate only if the left pointer is less than or equal to the right pointer. This is because if the pointers ever cross each other, that means we've searched the entire array without finding the target, so we exit the loop and return -1. 

- Loop Invariant:
    1) Get the middle value by floor dividing the sum of the L and R indices, ensuring we don't get a float value. The mid index partitions the search space into a lower and upper portion.
    2) If the element at mid is greater than the target, that means the target element is in the lower portion. Therefore, change the R pointer to mid - 1. 
    3) Else, if the element at mid is less than the target, that means the target element is in the upper portion. Therefore, change the L pointer to mid + 1.
    4) Otherwise, return mid


A good way to check your understanding of this concept is to do this problem:
https://leetcode.com/problems/search-a-2d-matrix/

## Application #1

(From LC #74) \
You are given an m x n integer matrix matrix with the following two properties:
- Each row is sorted in non-decreasing order.
- The first integer of each row is greater than the last integer of the previous row.

Given an integer target, return true if target is in matrix or false otherwise. You must write a solution in O(log(m * n)) time complexity.

### Decomposition

Let's break down the problem.
 We are searching for a target element in a matrix (a list of lists). All the rows in the matrix are sorted, and the columns in the matrix are sorted. Our desired runtime complexity is O(log(m) + log(n)). If we bruteforced the search with two for loops, we would get O(m * n). This isn't good enough. The runtime complexity implies that we need to do two binary searches.

The optimal way approach this would be to binary search the first column to find out which row potentially contains the target, and then do binary search on the row we identified to find (or not find) the target.

Searching the given row is the prototypical example of what binary search is, with no real variation, so we can implement that from the get go:


```python

def searchRow(matrix: list[list[int]], target: int, row: int) -> bool:
    L, R = 0, len(matrix[row]) - 1

    while L <= R:
        mid = (L + R) // 2

        if matrix[row][mid] < target:
            L = mid + 1
        elif matrix[row][mid] > target:
            R = mid - 1
        else:
            return True
        
    return False
```

Searching the first column is a bit trickier. We can't use our template because if the target is greater than the current guess, we can't exclude the current guess. This means we have to find the least column index where the target is greater than current guess. So we need to keep track of the least column index where the first element in the row given by that index is less than the target. Here's what it looks like:


```python
def searchFirstColumn(matrix: list[list[int]], target: int) -> int:
    L, R = 0, len(matrix) - 1
    row = 0

    while L <= R:
        mid = (L + R) // 2

        if matrix[mid][0] > target:
            R = mid - 1
        else:
            row = mid
            L = mid + 1
    
    return row
```

Putting it all together:


```python
def searchMatrix(matrix: list[list[int]], target: int) -> bool:
    row = searchFirstColumn(matrix, target)
    return searchRow(matrix, target, row)


matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]]
target1 = 3
target2 = 13

print(searchMatrix(matrix, target1))
print(searchMatrix(matrix, target2))


```

    True
    False
    
