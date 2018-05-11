---
layout: post
title:      "Sorting Out Sorting Algorithms"
date:       2018-05-11 13:45:26 +0000
permalink:  sorting_out_sorting_algorithms
---

![](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c5/Merge_sort_animation2.gif/220px-Merge_sort_animation2.gif)

# Introduction

This paper is geared to understanding what goes into several standard searching algorithms. I will look at several factors affecting their performance, such as random and non-random numbers, sorted and unsorted data sets, and finally, varying amounts of data. In doing so, hopefully a better understanding of the limitations and specifications of these algorithms is gained, as well as insight into where they may be most appropriately implemented.

# The Problem

When should we use a given sorting function? Are there better methods for specific types of data sets? For example, perhaps one route would be more effective for random data, but less so for data that is already close to being sorted. It is also a possibility that while one method may be incredibly efficient for smaller data sets, it becomes far less efficient when the size of the data set is large. We need concrete answers to these questions if we want to write smart, concise and efficient programs.

# Solution Plan

In order to address these features of algorithm analysis, we will use a selection of sorting functions that include: Insertion Sort, Quicksort, Quick Median of Three Sort, Shell Sort, Merge Sort, Selection Sort and Sink Sort. In addition, there will be three sized lists in which data will be processed, the first of which containing only 10 items, followed by 100 and 1000. So that we may gain better insight into the nuances of these alternate sorting methods, different types of data sets of each given size will be used. Initially, each method will be tested with purely random numbers, followed by: numbers in order, numbers in reverse order, 10% random numbers, and 10% ordered numbers. By offering a wide amount of testing conditions, better and more conclusive details can be drawn from the algorithmic analysis.

# Algorithm Descriptions

Before we step into the analysis of how each of these sorting algorithms perform, let’s take a moment to discuss how each one of them works. The ultimate goal will be the same, to sort a collection of data, but the means of accomplishing this varies greatly in the approaches. Below is a short description of each strategy accompanied by a visual representation, thanks to Wikipedia.


# Insertion

First is the Insertion sort, which is perhaps the most naive or simple approach. Insertion sorting is the process of stepping through the data set one at a time, and comparing each value to it’s adjacent value, swapping if necessary. It continues this process through the entire data set, when it reaches the end, the collection has been properly sorted. 

![](https://upload.wikimedia.org/wikipedia/commons/4/42/Insertion_sort.gif)
# Quicksort

Quicksort is a classic sorting algorithm, and one of the best performing. It is considered a divide and conquer strategy, because, as opposed to good ol’ Insertion sorting, Quicksort splits the data into partitions. It does this by first selecting an initial ‘pivot’ point, and pushing all data into two partitions, one ‘above’ and one ‘below’ the pivot point. It then recursively performs this operation until the the partitions are atomic or non-existant, at which point our data set is in order.

![](https://upload.wikimedia.org/wikipedia/commons/6/6a/Sorting_quicksort_anim.gif)

# Quick Median of 3

This is a variation of Quicksort. As such, its basic approach is the same. Choose a pivot point, split the data into partitions, recursively repeat these operations. The key variation is in the manner in which the pivot point is selected. In an ordinary Quicksort, often the first element is selected as the pivot, which for sorted arrays, results in the worst case performance. Common alternatives are to choose the last element or the middle element. Quick median of 3 opts for the median of these three values. By selecting this as our pivot, we can generally ensure better performance for sorted arrays.

# Shell

Shell sort is an in-place comparison sort. It operates by first sorting a pair of data elements that are rather separated within the data set, and then progressively reducing the gap between the paired elements. At its core, Shell sorting performs the same operations as Insertion sorting, with the key deviation being the gap in the compared elements. How you specify this gap greatly affects the algorithm’s performance.

![](https://upload.wikimedia.org/wikipedia/commons/d/d8/Sorting_shellsort_anim.gif)

# Merge

Merge sorting is another divide and conquer strategy similar to the Quicksorts. As one might imagine, our data set is first split into atomic units, and then progressively merged until ordered. This is often achieved recursively, by first combining the individual elements into groups of 2, then 4, then 8, so on and so forth. By comparing the starting values of each chunk of data, we can ensure the merging is done properly. 

![](https://upload.wikimedia.org/wikipedia/commons/c/cc/Merge-sort-example-300px.gif)

# Selection

Selection sorting is perhaps the simplest of our algorithms. This basically reduces to, put the data in order one value at a time. First a ‘sorted’ collection of data without contents is created, and then the smallest value is found from the ‘unsorted’ collections and placed in the begging of the ‘sorted’ collection. This continues until the sorted collection contains all the data and the unsorted is empty. The advantage here is that once an element is placed in the sorted array, we can be sure that that is where it belongs, unlike some alternatives in which an element is positioned multiple times. This still doesn’t make up for its significantly slower performance, especially with large values of N.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b0/Selection_sort_animation.gif/250px-Selection_sort_animation.gif)

# Sink

Sink sort, or often referred to as bubble sort, is another simple approach. It begins at the begging of the collections, and steps through, comparing adjacent elements. If need be, it swaps the elements, and proceeds through the data set. It gets its name, because (depending on your perspective) the larger data elements appear to sink down or bubble up the data set. When we reach the end of the collection, we can be sure we have correctly sorted the data. This approach is naive, and only works well with data that is almost already in order. 

![](https://upload.wikimedia.org/wikipedia/commons/c/c8/Bubble-sort-example-300px.gif)

# Tasks Completed

To accomplish these tasks, I constructed a simple console program in C# that contains methods necessary for implementing all of the algorithms we are analyzing, those being: Insertion Sort, Quick Sort, Quick Median of Three Sort, Shell Sort, Merge Sort, Selection Sort and Sink Sort. In addition, I created named lists for each individual method as well as a base list that will be populated with our various data types, those being: random numbers, numbers in order, numbers in reverse order, 10% random numbers, and 10% ordered numbers. Then, by modifying the general list, such that all combinations of size and ordering are realized, each named list is populated with a copy of the base list. This ensures that our analysis is consistent in that each algorithm is working with an identical data set during each run of the profiler. With all of that set up, all that’s left is to crunch the numbers, and obtain the average inclusive time spent in each method.


# Results and Conclusions

In analyzing the results, there are a couple of general observations about sorting with varying types of data sets, as well as specific insights we gained into the efficiency and relative efficiency of the multiple sorting algorithms we tested. First, we will look at some of the general results, and then look at the conclusions we can draw for each individual method. 
**Note**: All computing times are recorded in microseconds.


# General:

As was most likely expected, it takes longer to process large data sets for the majority of sorting algorithms, although the increased time is not consistent, meaning that some algorithms do not lose nearly as much efficiency when operating with large data sets, while other become far less efficient. In addition, sets of data that are already ordered take less time to process, although here as well, there are exceptions.


# Insertion Sort: O (n^2)

Insertion Sort is one of the more simplistic algorithms, and as a result, works incredibly quickly with small data sets, with the largest time for N = 10 being .03 milliseconds for the 10% random set. This is what we could expect, and within the data set of N = 10, Insertion sort is the fastest, while Shell Sort is very close. When you increase N to 100, it becomes less efficient compared to the other algorithms. We can see that it does not do well when the data set is in reverse order, which makes sense, because the way the algorithm is written would require to adjust every element in the list multiple times in order to sort it. Scaling N up to 1000 just continues to confirm these results, and proves that Insertion Sort operates far better with data sets that are in or close to being in order, and decreases in efficiency as N grows and the list becomes less ordered.


# Quicksort Original: O(n^2)

Quick sort is the most consistently performing algorithm. It takes the same amount of time, nearly identically actually, to sort all of the types of data types within the same N value. When N moves from 10 to 100, there is very little change in the processing speed, in fact, it is only when N = 1000 that we can spot a notable jump in performance time, but even that is less than a 100% increase. While the original quick sort is not the fastest algorithm for smaller N values, it is certainly the most consistently fast at N = 1000. In addition, its seeming indifference to the randomness or order of the data it is sorting would make it my preferred pick for sorting when we have little or no idea as to how the data is arranged. It also always outperforms the Median 3 Quicksort method.


# Quick Median of 3 Sort: O(n^2)

Surprising to me was the fact that this algorithm is always slower than its original counterpart. It only performs competitively for large values of N, and takes roughly twice as long as the original quick sort for small values of N. Also, it is far more difficult for it to handle data in which some of the numbers have already been ordered. In fact, when N = 1000, it takes 356 microseconds to process the data as opposed to less than 2 microseconds for any other data type of the same N value. As a result, I can see no clear scenario in which to implement this algorithm. It does not outperform anything consistently enough to validate its use.


# Shell Sort: O ((nlog(n))^2)

Shell sort appears to be the quickest algorithm for all sizes of N. It is outperformed by Original Quicksort when N = 100, and struggles with partially random data sets when N is small, but outside of these exceptions it is consistently fast. In fact, the largest observed time for any data set or N value is 1.7 microseconds, and for small ordered data sets, the time is almost completely negligible. I would use this algorithm for smaller data sets, provided they are at least somewhat in order, and unless I was operating with very large values of N, in which case quick sort is faster, there are no consistently better alternatives. The theoretical analysis here makes sense, because while time does increase with N, it is at a rate far slower than the other algorithms.


# Merge Sort: O (nlog(n))

Merge is interesting in that it is one of the worst performing algorithms for smaller N values, and one of the fastest for larger N values. The only exception is that lists in reverse order are far more time consuming than any of the alternative data sets, although this is mostly observable when N is large. In fact, for smaller N values, Merge Sort is actually more efficient and processing data in reverse order than correct ordering. Here again, the theoretical analysis is supported, because merge sorting operates best when N is very large.


# Selection Sort: O (n^2)

This algorithm is far worse at processing data that is 10% random than any other data type, and this is true for all N values. It is not particularly fast compared to the other algorithms, and becomes increasingly less so as N increases. The only clear advantage this algorithm appears to have is that it processes both random and sorted data at about the same speed.


# Sink Sort: O (n^2)

Sinking is very clearly the most effective algorithm if the data set is already sorted. As a matter of fact, there is almost no change in the processing times for varying sizes of N if the data is in order, or close to it. That being said, this algorithm performs very poorly with any other data type. It isn’t the least efficient at low values of N, and while it is startlingly fast at large values of N for ordered data sets, it also posts the highest processing time of all algorithms at nearly 2200 microseconds for data in reverse order when N = 1000. The only time I would use this algorithm is if I want to check and make sure a list is sorted, but if the data is know to be random or in reverse order, almost any other algorithm would provide a better alternative.


# Summary

In conclusion, there is quite a bit one can say about the various algorithms we have analyzed. I am very impressed with the overall performance of the Quick Sort method. I would use this most generally, as it has the best performance independent of variables. If I know for a fact that the amount of items we are processing is very small, however, then Insertion Sort becomes the algorithm of choice. Similarly, if dealing with very large data sets that should be in order or at least close to it, Sink Sort is very effective. I’m not sure of when Quick Median of 3 sort would be used effectively, as it is consistently outperformed by its original counterpart. Shell sort is faster than the Quick sort, however, until we start approaching large data sets, and then it is still rather efficient. Merge Sort is another that algorithm that is seemingly only useful for specific situations, namely large data sets as long as they are not in reverse order. As a result of this analysis, I have the means and data to support the choice of one algorithm over another in a variety of cases and in more general scenarios. There isn’t one algorithm that is inherently superior to all of the others, and therefore careful consideration must always be paid when selecting a sorting method, but with empirical analysis, this decision can be made with confidence.


# Data & Graphs

![](https://i.imgur.com/h96WFni.png)



![](https://i.imgur.com/cYgLn6A.png)



![](https://i.imgur.com/PYhyFdD.png)

