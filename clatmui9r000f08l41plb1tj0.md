---
title: "Algorithms: Merge Sort"
datePublished: Wed Nov 23 2022 12:39:23 GMT+0000 (Coordinated Universal Time)
cuid: clatmui9r000f08l41plb1tj0
slug: algorithms-merge-sort
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1669206957958/7gcswYAb3.png
tags: blogswithcc

---

Disclaimer: It is advised to view this blog in a dark theme.

## Introduction

Hi, I am Deepanshu Rawat, I am currently pursuing a Bachelor's in Computer Science and Engineering from the University of Petroleum and Energy Studies. My major is DevOps.

Let's learn the Merge Sort algorithm in a much simpler way.😁

[via GIPHY](https://giphy.com/gifs/GalindoRealEstateGroup-lets-go-here-we-get-ready-1pXrkKeRNDUGS7u3ig)

# Technique

Standard merge sort algorithm, following a divide and conquer approach, division of sub-array by mid values till sub-array length reaches one, then merging parts by either out-place or in-place merging in either ascending or descending order.

Let me add a gif to explain the working of the algorithm

[via Wikipedia](https://commons.wikimedia.org/wiki/File:Merge-sort-example-300px.gif)

## Normal Approach 😒

In this approach, merging happens in a separate array of size, then it is passed on in the above function calls.

### In mergeSort() function:

*   Dividing the array into 2 subarrays(one is ***left*** and other is ***right*** ) till their length reaches one(because no further division can take place)
    
*   Then pass the left and right parts into the merge() function for sorting in the correct order.
    

### In merge() function:

*   We merge the two subarrays(***left*** and ***right***) into another array ***mix***.
    
*   This merge operation stores elements from ***left*** and ***right*** in ascending order, by comparing elements from individual subarrays.
    
*   Finally, since one subarray would have greater elements than the other one, so we add those elements separately. Then, we return the sorted subarray into the above function call.
    

This solution gives ***O(nlogn)*** time complexity and ***O(n)*** space complexity.

The following is the code written in java for the same algorithm☕.

```plaintext
class Solution {
    public int[] sortArray(int[] nums) {
        return mergeSort(nums);
    }
    public static int[] mergeSort(int[] arr) {
        if (arr.length == 1) {
            return arr;
        }

        int mid = arr.length / 2;
        // copying and sorting sub-array by division on the basis of mid-value
        int[] left = mergeSort(Arrays.copyOfRange(arr, 0, mid));
        int[] right = mergeSort(Arrays.copyOfRange(arr, mid, arr.length));

        // now merging the two subarrays into one sorted subarray
        return merge(left, right);
    }

    public static int[] merge(int[] first, int[] second) {
        int[] mix = new int[first.length + second.length];

        int i = 0;
        int j = 0;
        int k = 0;
        // adding elements in the mix array in ascending order
        while (i < first.length && j < second.length) {
            if (first[i] < second[j]) {
                mix[k] = first[i];
                i++;    
            } else {
                mix[k] = second[j];
                j++;
            }
            k++;
        }

        // it may be possible that one of the arrays is not complete
        // copy the remaining elements
        while (i < first.length) {
            mix[k] = first[i];
            i++;
            k++;
        }

        while (j < second.length) {
            mix[k] = second[j];
            j++;
            k++;
        }

        return mix;
    }
}
```

Merge Sort proves to be an efficient sorting algorithm, better than ***O(n^2)*** sorting algorithms(like selection sort, bubble sort or insertion sort). Merge Sort do wonders with large data sets, but for smaller data sets it's to use insertion sort.

There is one tradeoff in merge sort i.e. use of some **additional space**. We can implement an even better version of merge sort using the same technique of merge sort.

<iframe src="https://giphy.com/embed/w9xG5hsxZlqtevPlJQ" class="giphy-embed" width="480" height="251"></iframe>

[via GIPHY](https://giphy.com/gifs/reaction-mood-w9xG5hsxZlqtevPlJQ)

## Introducing In-Place Approach😮

In this approach, merging happens in place, changes are made in the original arrays themselves by modifying the reference variables.

### In mergeSort() function:

Dividing the array into 2 subarrays(by calling mergeSort() function recursively) till their length reaches one.

### In mergeInPlace() function:

*   We merge the two subarrays(one from s to mid and another from mid to e) into another array ***mix***.
    
*   This merge operation stores elements from the two sub-arrays in ascending order, by comparing elements from individual subarrays. Finally, since one subarray would have greater elements than the other one, so we add those elements separately.
    
*   Then, we place the sorted elements from ***mix*** into ***arr*** , finally changes in ***arr*** have been made in-place.
    

This solution gives ***O(nlogn)*** time complexity and ***O(1)*** space complexity.

The following is the code written in java for the same algorithm☕.

```plaintext
class Solution {
    public int[] sortArray(int[] nums) {
        mergeSort(nums, 0, nums.length);
        return nums;
    }
    public void mergeSort(int[] arr, int s, int e) {
        if (e - s == 1) {
            return;
        }

        int mid = (s + e) / 2;
        // dividing sub-arrays by mid values till, sub-array length reaches one
        mergeSort(arr, s, mid);
        mergeSort(arr, mid, e);

        // sorting the left portion(s to mid) and right portion(mid to e) into the same array arr
        mergeInPlace(arr, s, mid, e);
    }

    public static void mergeInPlace(int[] arr, int s, int m, int e) {
        int[] mix = new int[e - s];

        int i = s;
        int j = m;
        int k = 0;
        // adding elements in the mix array in ascending order
        while (i < m && j < e) {
            if (arr[i] < arr[j]) {
                mix[k] = arr[i];
                i++;
            } else {
                mix[k] = arr[j];
                j++;
            }
            k++;
        }

        // it may be possible that one of the arrays is not complete
        // copy the remaining elements
        while (i < m) {
            mix[k] = arr[i];
            i++;
            k++;
        }

        while (j < e) {
            mix[k] = arr[j];
            j++;
            k++;
        }

        // modifying the original arrays by replacing elements into their correct indices
        for (int l = 0; l < mix.length; l++) {
            arr[s + l] = mix[l];
        }
    }
}
```

Here's a practice problem in **leetcode**, where you can implement the merge sort, try experimenting with it.

[Practice Problem: Sort an Array](https://leetcode.com/problems/sort-an-array/)

So, this is the end of this blog (it's my first blog, so I'm kinda nervous about how this goes😅), if you guys liked the blog then you can give emotes to this blog(it would help me be motivated to write another blog).

Untill next time, Peace Out✌️.

<iframe src="https://giphy.com/embed/Ru9sjtZ09XOEg" class="giphy-embed" width="480" height="269"></iframe>

[via GIPHY](https://giphy.com/gifs/wwe-peace-out-seth-rollins-Ru9sjtZ09XOEg)