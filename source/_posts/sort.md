---
title: JavaScript 排序算法
date: 2016-10-13 17:12:08
tags: algorithm
---

排序算法作为老生常谈的话题，其实现方式多种多样，这里给出冒泡排序、选择排序、插入排序、合并排序、快速排序等 5 种排序的一种实现，以便将来参考和在此基础上改进。

<!-- more -->

### 测试用例：

``` 
var a = [85, 24, 63, 45, 17, 31, 96, 50];
```

``` 
var b = [3,5,3,7,65,4,45,43,33];
```

``` 
var c = [3, 5, 3, 7, 33, 4];
```

**(1) 冒泡排序**

　　依次比较相邻两数，如果前者大于后者则交换位置，一趟下来最大数就在数组末尾。对除末尾元素外的数组元素重复该过程，直至全部数字升序排列。

```
//交换数组相邻两元素
function swap(arr,i){
    var temp = arr[i]; 
    arr[i] = arr[i+1];
    arr[i+1] = temp;
}

//冒泡排序,会改变原数组
function bubbleSort(arr){
    var len = arr.length - 1,
        i = 0,
        j = 0;

    for (;i < len;i++){
        for (j = 0;j < len - i; j++){
            if (arr[j] > arr[j+1]){
                swap(arr,j);
            }
        }
    }
    return arr;
} 
```

**(2) 选择排序**

　　找出数组中最大的数字与最末尾数字位置交换。对除末尾元素外的数组元素重复该过程，直至全部数字升序排列。

```
//将数组最大元素放置在最末尾
function selectMax(arr,len){
    var length = arr.length || 0,
        len = len || length,
        i = 0,
        max = arr[0],
        index = 0,
        temp = 0;
    
    for (;i < len;i ++){
        if (arr[i] > max){
            max = arr[i];
            index = i;
        }
    }

    temp = arr[len-1];
    arr[len-1] = arr[index];
    arr[index] = temp;		
}

//选择排序，会改变原数组
function selectSort(arr){
    var len = arr.length,
        i = 0;

    for (;i < len - 1;i++){
        selectMax(arr,len-i);
    }
    return arr;
}

```

**(3) 插入排序**

　　它将数组分成“已排序”和“未排序”两部分，一开始的时候，“已排序”的部分只有一个元素，然后将它后面一个元素从“未排序”部分插入“已排序”部分，从而“已排序”部分增加一个元素，“未排序”部分减少一个元素。以此类推，完成全部排序。

```
//向升序数组插入数字
function insertNum(arr,num){
    var length = arr.length,
        i = 0,
        j = 0;
    
    for (;i < length;i++){
        if (num <= arr[i]){
            for (j = length;j > i;j--){
                arr[j] = arr[j-1]
            }
            arr[i] = num;
            break;
        }
    }
    if (length === arr.length){
        arr[length] = num;
    }
}

//插入排序，不会改变原数组
function insertSort(arr){
    var arr1 = [arr[0]],
        len = arr.length - 1,
        i = 0;
    
    for (;i < len;i++){
        insertNum(arr1,arr[i+1]);
    }
    return arr1;
}
```
**(4) 合并排序**

　　将数组拆开，分成 n 个只有一个元素的数组，然后不断地两两合并，直到全部排序完成。

```
//合并两个有序数组
function mergeArr(arr1,arr2){
    var length = arr2.length,
        i = 0;

    for (;i < length;i++){
        insertNum(arr1,arr2[i]);
    }
    return arr1;
}

//合并排序,不会影响原数组
function mergeSort(arr){
    var length = arr.length,
        i = 0,
        j = 0;
        arr1 = [];
    
    for (;i < length;i++){
        arr1[i] = [arr[i]];
    }

    for (;j < length - 1;j++){
        arr1[j+1] = mergeArr(arr1[j],arr1[j+1]);
    }
    return arr1[length - 1];
}
```
**(5) 快速排序**

　　先确定一个“支点”（pivot），然后将所有小于“支点”的值都放在该点的左侧，大于等于“支点”的值都放在该点的右侧，然后对左右两侧不断重复这个过程，直到所有排序完成。

```
//连接数组，不影响原数组
function joinArr(arr1,num,arr2){
    var len1 = arr1.length,
        len2 = arr2.length,
        i = 0,
        j = 0,
        arr = [];
    
    for (;i < len1;i++){
        arr[i] = arr1[i];
    }
    arr[i] = num;
    for (;j < len2;j++){
        arr[i+1+j] = arr2[j];
    }
    return arr;
}

//快速排序
function quickSort(arr){
    var len = arr.length,
        pivotIdx = Math.floor((len - 0.5) / 2),
        pivot = arr[pivotIdx],
        i = 0,
        leftArr = [],
        rightArr = [];

    if (len < 2){
        return arr;
    }

    for (;i < len;i++){
        if (i === pivotIdx){
            continue;
        } else {
            if (arr[i] < pivot){
                leftArr[leftArr.length] = arr[i];
            } else {
                rightArr[rightArr.length] = arr[i];
            }
        }
    }
    return joinArr(quickSort(leftArr),pivot,
                    quickSort(rightArr));
}
```