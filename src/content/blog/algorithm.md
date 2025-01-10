---
title: '排序算法'
description: '基础的排序算法'
pubDate: 'Jan 10 2025'
tags: ['js','algorithm']
---

## 排序算法

### 冒泡排序

```js
function bubbleSort(arr) {
    for (let i = 0; i < arr.length; i++) {
        for (let j = 0; j < arr.length - i; j++) {
            if (arr[j] > arr[j + 1]) {
                temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
    return arr
}
```

### 快速排序

实现思路：

- 初始化两个指针：左指针 `i` 指向数组的起始位置，右指针 `j` 指向数组的末尾。
- 从右向左移动右指针 `j`，直到找到第一个小于等于基准值的元素。
- 将该元素移到左指针 `i` 的位置，并将左指针 `i` 向右移动一位。
- 从左向右移动左指针 `i`，直到找到第一个大于等于基准值的元素。
- 将该元素移到右指针 `j` 的位置，并将右指针 `j` 向左移动一位。
- 重复上述过程，直到 `i` 和 `j` 相遇。
- 参数（arr，left，right）

```js
function quickSort(arr,l,r){
  if(l < r){
    var i = l, j = r, x = arr[i];
    while(i<j){
      while(i<j && arr[j]>x)
        j--;
      
      if(i<j)
        //这里用i++，被换过来的必然比x小，赋值后直接让i自加，不用再比较，可以提高效率
        arr[i++] = arr[j];
      
      while(i<j && arr[i]<x)
        i++;
      
      if(i<j)
        //这里用j--，被换过来的必然比x大，赋值后直接让j自减，不用再比较，可以提高效率
        arr[j--] = arr[i];
    }
    arr[i] = x;
    
    quickSort(arr, l, i-1);
    quickSort(arr, i+1, r);
  }
}
```

### 选择排序

![alt 排序算法](/select.awebp)

按顺序选择，第二个循环选择一个比选择的要小的

```js
function selectSort(array){
    let min;
    for (let i = 0; i < array.length-1; i++) {
        min=i;
        for (let j = i+1; j < array.length; j++) {
            if(array[min]>array[j]) min=j
        }
        min!=i &&swap(array,i,min)
        
    }
    return array;
}
selectSort(array).forEach((item) => {
    console.log(item);
})
```

### 归并排序

![](/merge.awebp)

```js
function mergeSort(array){
    if(array.length<2){
        return array;
    }
    let m=(array.length>>1)
    let left=array.slice(0,m)
    let  right=array.slice(m)
    return merge(mergeSort(left),mergeSort(right))
}
function merge(left,right){
    const result=[];
    while(left.length&&right.length){
    var item=left[0]<=right[0]?left.shift():right.shift()
    result.push(item)
    }
    return result.concat(left.length?left:right)
}

mergeSort(array).forEach((item) => {
    console.log(item);
})
```

### 插入排序

#### 直接插入排序

它的基本思想是: 将待排序的元素按照大小顺序, 依次插入到一个已经排好序的数组之中, 直到所有的元素都插入进去.

![](/directInsertionSort.awebp)

```js
function directInsertionSort(array) {
   var length=array.length ,index,current;
    for(i=1;i<length;i++){
        index=i-1;
        current=array[i]
        while(index>=0&&array[index]>current){//找到插入的位置,
            array[index+1]=array[index]
            index--
        }
        if(index!=i){
            array[index+1]=current
        }
    }
    return array;
  }
```

#### 希尔排序

```js
//形参增加步数gap(实际上就相当于gap替换了原来的数字1)
function directInsertionSort(array, gap) {
  gap = (gap == undefined) ? 1 : gap;       //默认从下标为1的元素开始遍历
  var length = array.length, index, current;
  for (var i = gap; i < length; i++) {
    index = i - gap;    //待比较元素的下标
    current = array[i];    //当前元素
    while(index >= 0 && array[index] > current) { //前置条件之一:待比较元素比当前元素大
      array[index + gap] = array[index];    //将待比较元素后移gap位
      index -= gap;                           //游标前移gap位
    }
    if(index + gap != i){                   //避免同一个元素赋值给自身
      array[index + gap] = current;            //将当前元素插入预留空位
    }
  }
  return array;
}
function shellSort(array){
  var length = array.length, gap = length>>1, current, i, j;
  while(gap > 0){
    directInsertionSort(array, gap); //按指定步长进行直接插入排序
    gap = gap>>1;
  }
  return array;
}


```



