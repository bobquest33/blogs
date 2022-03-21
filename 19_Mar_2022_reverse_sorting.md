# Solving a Sorting Problem in Python

_19-Mar-2022_

## Background

As part of an interview question I got a sorting problem, where I was supposed to reverse sort a list of numbers using python without using the Python internal functions. I was unable to implement it in the given time and also realized how weak I am in basic Data Structures & Algorithms.
Later on then I checkout a book on Data Structures & Algorithm using Python and tried giving it a try again.

## The Problem

The problem while was a sorting problem, was not the simple sorting problem in the sense, there will be a unsorted list with nulls in it and has to be sorted in descending order with nulls positions not changing.

e.g.
```
For an input ip=[1,None,5,7,3] an op=[7,None,5,3,1] is expected
And for an input ip=[1,None,5,4,6,None,7,3] an op=[7,None,6,5,4,None,3,1] is expected 
```

While there are many options but during my interview the only solution using a max_val was coming. But the solution using inner for loop was not working, and since I was so used to standard python functions for sorting, this was a bigger shock, and also a realization that while python makes our lives so easy, forgetting basic data structures and algorithms was embarrassing, especially when I have been actively doing development in Python for so many years.

## The Aim

My aim in writing this blog and also trying to find the solution was two fold, one was to get a better understanding how sorting works. Another was to get back to groove by solving simpler problems and getting started in learning Data Structures & Algorithms. Next I read few blogs and books that I can find handy and thanks to StackOverflow I also got some examples of algorithms for sorting an integer list in descending order.

## My First Solution: Bubble Sort

One of the first solutions which I tried was to try bubble sort. For the definition part I refer Wikipedia

> Bubble sort, sometimes referred to as sinking sort, is a simple sorting algorithm that repeatedly steps through the list, compares adjacent elements and swaps them if they are in the wrong order. The pass through the list is repeated until the list is sorted.


```python
# Initialize Libraries
import copy
```


```python
def bubbleSort(theSeq):
    n = len(theSeq)
    # Perform n-1 bubble operations on the sequence
    for i in range( n-1, 0, -1 ) :
        # Bubble the largest item to the end.
        for j in range( i, 0 , -1) :
            bSwitch = False
            # Skip to next element if jth element is null
            if theSeq[j] is None:
                continue
                
            # Find the next element in descending order which doesn't have a -ve value
            sindex = j - 1
            if theSeq[sindex] is None:
                for k in range(sindex - 1, -1 , -1):
                    if theSeq[k] is not None:
                        # print(sindex,"|",k, "|", i, "|", j)
                        sindex = k
                        break

            if theSeq[j] > theSeq[sindex] : # swap the j and j-1 items or the lower index which has no null value.
                bSwitch = True
            if bSwitch:
                tmp = theSeq[j]
                theSeq[j] = theSeq[sindex]
                theSeq[sindex] = tmp
        print(theSeq)
    return theSeq

```


```python
# Bubble sort Example 1
ip=[1,None,5,7,3]
op=[7,None,5,3,1]
tp = copy.deepcopy(ip)
# The sorted list
tp = bubbleSort(tp)
match = tp == op
print("The input and out match for Reverse Bubble Sort:", match)
```

```
    [7, None, 1, 5, 3]
    [7, None, 5, 1, 3]
    [7, None, 5, 1, 3]
    [7, None, 5, 1, 3]
    The input and out match for Reverse Bubble Sort: False
```


```python
# Bubble sort Example 2
ip=[1,None,5,4,6,None,7,3]
op=[7,None,6,5,4,None,3,1]
tp = copy.deepcopy(ip)
# The sorted list
tp = bubbleSort(tp)
match = tp == op
print("The input and out match for Reverse Bubble Sort:", match)
```

```
    [7, None, 1, 5, 4, None, 6, 3]
    [7, None, 6, 1, 5, None, 4, 3]
    [7, None, 6, 5, 1, None, 4, 3]
    [7, None, 6, 5, 1, None, 4, 3]
    [7, None, 6, 5, 1, None, 4, 3]
    [7, None, 6, 5, 1, None, 4, 3]
    [7, None, 6, 5, 1, None, 4, 3]
    The input and out match for Reverse Bubble Sort: False
```

As you can see my code has a major bug in swapping numbers across null cells. At this point I was seriously hoping to get it right rather than working on the performance aspect. 

## Selection Sort to the Rescue

On further reading, I learnt about selection sort, while initially I thought it was not so much performant, but as an algorithm it was much simpler and in line with my original solution which I thought in the interview. For the definition again I refer Wikipedia.

> The algorithm divides the input list into two parts: a sorted sub-list of items which is built up from left to right at the front (left) of the list and a sub-list of the remaining unsorted items that occupy the rest of the list. Initially, the sorted sub-list is empty and the unsorted sub-list is the entire input list. The algorithm proceeds by finding the smallest (or largest, depending on sorting order) element in the unsorted sub-list, exchanging (swapping) it with the leftmost unsorted element (putting it in sorted order), and moving the sub-list boundaries one element to the right.

But the issue here for me was also as given in Wikipedia:
> It has an O(n2) time complexity, which makes it inefficient on large lists, and generally performs worse than the similar insertion sort. 

But as I felt, Selection sort is noted for its simplicity and has performance advantages over more complicated algorithms in certain situations, particularly where auxiliary memory is limited. Hence with some changes I implemented as below.


```python
# Sorts a sequence in descending order using the selection sort algorithm.
def selectionSort( theSeq ):

    # Scan the list from start to end
    for i in range(len(theSeq)):
        # Set the index of the max value in the iteration as i
        max_index = i
        
        # Parse all the elements from i till end of the list and find the max value
        for j in range(i+1, len(theSeq)):
            # If current index is Null skip to next iteration
            if theSeq[j] is None or theSeq[max_index] is None:
                continue
            # If current index is greater than the value at max_index, set it that index to max_index                
            if theSeq[j] > theSeq[max_index]:
                max_index = j
        print(theSeq)
        # Swap the values at max_index & i
        theSeq[i],theSeq[max_index] = theSeq[max_index],theSeq[i]
    return theSeq

```


```python
# Selection sort Example 1
ip=[1,None,5,7,3]
op=[7,None,5,3,1]
tp = copy.deepcopy(ip)
# The sorted list
tp = selectionSort(tp)
match = tp == op
print("The input and out match for Reverse Selection Sort:", match)
```

```
    [1, None, 5, 7, 3]
    [7, None, 5, 1, 3]
    [7, None, 5, 1, 3]
    [7, None, 5, 1, 3]
    [7, None, 5, 3, 1]
    The input and out match for Reverse Selection Sort: True
```


```python
# Selection sort Example 2
ip=[1,None,5,4,6,None,7,3]
op=[7,None,6,5,4,None,3,1]
tp = copy.deepcopy(ip)
# The sorted list
tp = selectionSort(tp)
match = tp == op
print("The input and out match for Reverse Selection Sort:", match)
```

```
    [1, None, 5, 4, 6, None, 7, 3]
    [7, None, 5, 4, 6, None, 1, 3]
    [7, None, 5, 4, 6, None, 1, 3]
    [7, None, 6, 4, 5, None, 1, 3]
    [7, None, 6, 5, 4, None, 1, 3]
    [7, None, 6, 5, 4, None, 1, 3]
    [7, None, 6, 5, 4, None, 1, 3]
    [7, None, 6, 5, 4, None, 3, 1]
    The input and out match for Reverse Selection Sort: True
```

As I have noted earlier, the use of max_index and searching through a list is not a very optimal solution and time consuming, but at least I was able to understand this solution and implement successfully.

## Finally

From this quick exercise I not only learnt how knowing various sorting methods are important but also depending on the situation to judge which sorting logic may work for the given idea. I am sure I have to learn a lot but this a starting journey after which I am sure I will learn various sorting and searching algorithms which will ultimately benefits me.
