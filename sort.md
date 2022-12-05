# O(n²) 排序

## 冒泡排序

冒泡排序只会操作相邻的两个数据。每次冒泡操作都会对相邻的两个元素进行比较，看是否满足大小关系要求。如果不满足就让它俩互换。一次冒泡会让至少一个元素移动到它应该在的位置，重复 n 次，就完成了 n 个数据的排序工作。

例如：我们要对一组数据 3，5，4，1，2，6，从小到到大进行排序。

![冒泡.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35d95248496240a08ee76b463220e2cb~tplv-k3u1fbpfcp-watermark.image?)

```js
function bubbleSort(arr) {
    let flag = false
    // 提前退出冒泡循环的标志位
    for (let i = 0; i < arr.length; i++) {
        for (let j = 0; j < arr.length - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]]
                flag = true
            }
        }
        if (!flag) break
    }
    return arr
}
```

### 第一，冒泡排序是原地排序算法吗？

冒泡的过程只涉及相邻数据的交换操作，只需要常量级的临时空间，所以它的空间复杂度为O(1)，是一个原地排序算法。

### 第二，冒泡排序是稳定的排序算法吗？

在冒泡排序中，只有交换才可以改变两个元素的前后顺序。为了保证冒泡排序算法的稳定性，当有相邻的两个元素大小相等的时候，我们不做交换，相同大小的数据在排序前后不会改变顺序，所以冒泡排序是稳定的排序算法。

### 第三，冒泡排序的时间复杂度是多少？

最好情况下，要排序的数据已经是有序的了，我们只需要进行一次冒泡操作，就可以结束了，所以最好情况时间复杂度是 O(n)。而最坏的情况是，要排序的数据刚好是倒序排列的，我们需要进行 n 次冒泡操作，所以最坏情况时间复杂度为 O(n²)。

## 插入排序

首先，我们将数组中的数据分为两个区间，`已排序区间`和`未排序区间`。初始已排序区间只有一个元素，就是数组的第一个元素。插入算法的核心思想是取未排序区间中的元素，在已排序区间中找到合适的插入位置将其插入，并保证已排序区间数据一直有序。重复这个过程，直到未排序区间中元素为空，算法结束。

```js
function insertionSort(arr) {
    for (let i = 1; i < arr.length; i++) {
        let val = arr[i]
        let j = i - 1
        for (; j >= 0; j--) {
            if (arr[j] > val) {
                arr[j + 1] = arr[j]//数据移动
            } else {
                break//找到插入位置
            }
        }
        arr[j + 1] = val
    }
    return arr
}
```

### 第一，插入排序是原地排序算法吗？

从实现过程可以很明显地看出，插入排序算法的运行并不需要额外的存储空间，所以空间复杂度是 O(1)，也就是说，这是一个原地排序算法。

### 第二，插入排序是稳定的排序算法吗？

在插入排序中，对于值相同的元素，我们可以选择将后面出现的元素，插入到前面出现元素的后面，这样就可以保持原有的前后顺序不变，所以插入排序是稳定的排序算法。

### 第三，插入排序的时间复杂度是多少？

如果要排序的数据已经是有序的，我们并不需要搬移任何数据。如果我们从尾到头在有序数据组里面查找插入位置，每次只需要比较一个数据就能确定插入的位置。所以这种情况下，最好是时间复杂度为 O(n)。注意，这里是从尾到头遍历已经有序的数据。
如果数组是倒序的，每次插入都相当于在数组的第一个位置插入新的数据，所以需要移动大量的数据，所以最坏情况时间复杂度为 O(n²)。
还记得我们在数组中插入一个数据的平均时间复杂度是多少吗？没错，是 O(n)。所以，对于插入排序来说，每次插入操作都相当于在数组中插入一个数据，循环执行 n 次插入操作，所以平均时间复杂度为 O(n²)。

## 选择排序

选择排序算法的实现思路有点类似插入排序，也分已排序区间和未排序区间。但是选择排序每次会从未排序区间中找到最小的元素，将其放到已排序区间的末尾。

```js
function selectionSort(array) {
    for (let i = 0; i < array.length; i++) {
        let minIndex = i
        for (let j = i + 1; j < array.length; j++) {
            if (array[j] < array[minIndex]) {
                minIndex = j;
            }
        }
        [array[minIndex], array[i]] = [array[i], array[minIndex]];
    }
    return array
}
```

### 性能分析

首先，选择排序空间复杂度为 O(1)，是一种原地排序算法。选择排序的最好情况时间复杂度、最坏情况和平均情况时间复杂度都为 O(n²)。

那选择排序是稳定的排序算法吗？

答案是否定的，选择排序是一种不稳定的排序算法。选择排序每次都要找剩余未排序元素中的最小值，并和前面的元素交换位置，这样破坏了稳定性。

比如 5，8，5，2，9 这样一组数据，使用选择排序算法来排序的话，第一次找到最小元素2，与第一个 5 交换位置，那第一个 5 和中间的 5 顺序就变了，所以就不稳定了。正是因此，相对于冒泡排序和插入排序，选择排序就稍微逊色了。

## 重点：为什么插入排序要比冒泡排序更受欢迎呢？

我们前面分析冒泡排序和插入排序的时候讲到，冒泡排序与插入排序不管怎么优化，元素交换的次数与数据的有序程度有关。

但是，从代码实现上来看，冒泡排序的数据交换要比插入排序的数据移动要复杂，冒泡排序的赋值操作比插入排序的赋值操作更频繁。

```js
//冒泡
if (arr[j] > arr[j + 1]) {
    [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]]//数据交换
    flag = true
}
//插入排序
if (arr[j] > val) {
    arr[j + 1] = arr[j]//数据移动
}
```

所以，虽然冒泡排序和插入排序在时间复杂度上是一样的，都是 O(n²)，但我们首选插入排序。

## 小结

![n^2总结.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1aca4034a2343d9942d924c9d5b0f43~tplv-k3u1fbpfcp-watermark.image?)

# O(nlogn) 排序

## 归并排序

归并排序的核心思想还是蛮简单的。如果要排序一个数组，我们先把数组从中间分成前后两部分，然后对前后两部分分别排序，再将排好序的两部分合并在一起，这样整个数组就都有序了。


![归并.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc6804fd8c0742d79c93991d10d7115b~tplv-k3u1fbpfcp-watermark.image?)

```js
function mergeSort(array, left, right) {
    if (left < right) {
        const mid = Math.floor((left + right) / 2);
        mergeSort(array, left, mid)
        mergeSort(array, mid + 1, right)
        merge(array, left, mid, right);
    }
    return array;
}
function merge(array, left, mid, right) {
    let leftIndex = left, rightIndex = mid + 1;
    let temp = [], tempIndex = 0;
    while (leftIndex <= mid && rightIndex <= right) {
        if (array[leftIndex] <= array[rightIndex]) {
            temp[tempIndex++] = array[leftIndex++]
        } else {
            temp[tempIndex++] = array[rightIndex++]
        }
    }
    while (leftIndex <= mid) {
        temp[tempIndex++] = array[leftIndex++]
    }
    while (rightIndex <= right) {
        temp[tempIndex++] = array[rightIndex++]
    }
    tempIndex = 0;
    for (let i = left; i <= right; i++) {
        array[i] = temp[tempIndex++];
    }
}
```

### 第一，归并排序是稳定的排序算法吗？

从合并的过程中我们可以看出，归并排序是一个稳定的排序算法。

### 第二，归并排序的时间复杂度是多少？

从代码可以看出，归并排序的执行效率与要排序的原始数组的有序程度无关，所以其时间复杂度是非常稳定的，不管是最好情况、最坏情况，还是平均情况，时间复杂度都是 O(nlogn)。

### 第三，归并排序的空间复杂度是多少？

归并排序的时间复杂度任何情况下都是 O( nlogn )，看起来非常优秀。（待会儿你会发现快速排序最坏情况下时间复杂度是 O( n^2 ) ）。但是，归并排序并没有像快排那样，应用广泛，这是为什么呢？因为归并排序不是原地排序算法。

这是因为归并排序的合并函数，在合并两个有序数组为一个有序数组时，需要借助额外的存储空间。这一点你应该很容易理解。

递归代码的空间复杂度并不能像时间复杂度那样累加。所以空间复杂度是O(n)。

## 快速排序

快排的思想是这样的：如果要排序数组中下标从 p 到 r 之间的一组数据，我们选择 p 到 r之间的任意一个数据作为 pivot（分区点）。

我们遍历 p 到 r 之间的数据，将小于 pivot 的放到左边，将大于 pivot 的放到右边，将pivot 放到中间。经过这一步骤之后，数组 p 到 r 之间的数据就被分成了三个部分，前面 p到 q-1 之间都是小于 pivot 的，中间是 pivot，后面的 q+1 到 r 之间是大于 pivot 的。


![快排.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/acaa176dda894606a56a3c30721dfe59~tplv-k3u1fbpfcp-watermark.image?)

```js
function quickSort(array, start, end) {
    if (end - start < 1) {
        return;
    }
    const target = array[start];
    let l = start;
    let r = end;
    while (l < r) {
        while (l < r && array[r] > target) {
            r--;
        }
        array[l] = array[r];
        while (l < r && array[l] < target) {
            l++;
        }
        array[r] = array[l];
    }
    array[l] = target;
    quickSort(array, start, l - 1);
    quickSort(array, l + 1, end);
    return array;
}
```

### 性能分析

因为分区的过程涉及交换操作，如果数组中有两个相同的元素，经过分区操作之后，两个相同元素的相对顺序就会改变。所以，快速排序并不是一个稳定的排序算法。

如果每次分区操作，都能正好把数组分成大小接近相等的两个小区间，那么快排的时间复杂度是 O(nlogn)。

但如果那每次分区得到的两个区间都是极其不均等的，二叉树的深度便会变成 n ，快排的时间复杂度就从O(nlogn) 退化成了 O(n²)。

### 如何避免快排性能退化？

1. 三数取中法
我们从区间的首、尾、中间，分别取出一个数，然后对比大小，取这 3 个数的中间值作为
分区点。这样每间隔某个固定的长度，取数据出来比较，将中间值作为分区点的分区算法，肯定要比单纯取某一个数据更好。但是，如果要排序的数组比较大，那“三数取中”可能就不够了，可能要“五数取中”或者“十数取中”。
2. 随机法
随机法就是每次从要排序的区间中，随机选择一个元素作为分区点。这种方法并不能保证每次分区点都选的比较好，但是从概率的角度来看，也不大可能会出现每次分区点都选的很差的情况，所以平均情况下，这样选的分区点是比较好的。时间复杂度退化为最糟糕的 O(n²)的情况，出现的可能性不大。

![归并快排却别.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88512c7c143e4e8381b0c257f793e480~tplv-k3u1fbpfcp-watermark.image?)
## 归并与快排的区别？

可以发现，归并排序的处理过程是由下到上的，先处理子问题，然后再合并。

而快排正好相反，它的处理过程是由上到下的，先分区，然后再处理子问题。

归并排序虽然是稳定的、时间复杂度为 O(nlogn) 的排序算法，但是它是非原地排序算法。

我们前面讲过，归并之所以是非原地排序算法，主要原因是合并函数无法在原地执行。

## 堆排序

```js
function heapSort(array) {
  creatHeap(array);
  // 交换第一个和最后一个元素，然后重新调整大顶堆
  for (let i = array.length - 1; i >= 1; i--) {
    // 0~i 为大顶堆
    [array[i], array[0]] = [array[0], array[i]];
    adjust(array, 0, i - 1);
  }
  return array;
}
// 从第一个非叶子节点开始，构建大顶堆
function creatHeap(array) {
  for (let i = Math.floor(array.length / 2) - 1; i >= 0; i--) {
    adjust(array, i, array.length - 1);
  }
}
// 孩子节点有比他大的就下沉
function adjust(array, target, lastIndex) {
  for (let i = 2 * target + 1; i <= lastIndex; i = 2 * target + 1) {
    // 找到孩子节点中最大的
    if (i + 1 <= lastIndex && array[i + 1] > array[i]) {
      i = i + 1;
    }
    // 下沉
    if (array[i] > array[target]) {
      [array[i], array[target]] = [array[target], array[i]];
      target = i;
    } else {
      break;
    }
  }
}
```

### 性能分析

堆排序是原地排序算法。

堆排序包括 建堆 和 排序 两个操作，建堆过程的时间复杂度是 O(n)，排序过程的时间复杂度是 O(nlogn)，所以，堆排序整体的时间复杂度是 O(nlogn) （最好、最坏、平均）。

堆排序不是稳定的排序算法，因为在排序的过程，存在将堆的最后一个节点跟堆顶节点互换的操作，所以就有可能改变值相同数据的原始相对顺序。

## 为什么选择快排而不是堆排序？

### 堆排序算法的数据交换次数要多于快速排序

整个排序过程就是由两个基本的操作组成的，比较和交换（或移动）。

快速排序数据交换的次数不会比逆序度多。

但是堆排序的第一步是建堆，建堆的过程会打乱数据原有的相对先后顺序，导致原数据的有序度降低。

# 线性排序

之所以能做到线性的时间复杂度，主要原因是，这三个算法是非基于比较的排序算法，都不涉及元素之间的比较操作。但是对要排序的数据要求很苛刻，所以我们学习重点是适用场景。

## 桶排序

桶排序，核心思想是将要排序的数据分到几个有序的桶里，每个桶里的数据再单独进行排序。桶内排完序之后，再把每个桶里的数据按照顺序依次取出，组成的序列就是有序的了。

### 性能

如果要排序的数据有 n 个，我们把它们均匀地划分到 m 个桶内，每个桶里就有 k=n/m 个元素。每个桶内部使用快速排序，时间复杂度为 O(k * logk)。m 个桶排序的时间复杂度就是 O(m * k * logk)，因为 k=n/m，所以整个桶排序的时间复杂度就是 O(n*log(n/m))。

当桶的个数 m 接近数据个数 n 时，log(n/m) 就是一个非常小的常量，这个时候桶排序的
时间复杂度接近 O(n)。

#### 然而条件太苛刻

首先，要排序的数据需要很容易就能划分成 m 个桶，并且，桶与桶之间有着天然的大小顺序。这样每个桶内的数据都排序完之后，桶与桶之间的数据不需要再进行排序。

其次，数据在各个桶之间的分布是比较均匀的。如果数据经过桶的划分之后，有些桶里的数据非常多，有些非常少，很不平均，那桶内数据排序的时间复杂度就不是常量级了。在极端情况下，如果数据都被划分到一个桶里，那就退化为 O(nlogn) 的排序算法了。

### 适用场景

桶排序比较适合用在外部排序中。所谓的外部排序就是数据存储在外部磁盘中，数据量比较大，内存有限，无法将数据全部加载到内存中。

例：我们有 10GB 的订单数据，我们希望按订单金额（假设金额都是正整数）进行排序，但是我们的内存有限，只有几百 MB，没办法一次性把 10GB 的数据都加载到内存中。

如果订单金额在 1 到 10 万之间均匀分布，那订单会被均匀划分到 100 个文件中，每个小文件中存储大约 100MB 的订单数据，我们就可以将这 100 个小文件依次放到内存中，用快排来排序。等所有文件都排好序之后，我们只需要按照文件编号，从小到大依次读取每个小文件中的订单数据，并将其写入到一个文件中，那这个文件中存储的就是按照金额从小到大排序的订单数据了。

针对这些划分之后还是比较大的文件，我们可以继续划分。如果划分之后还是太多，无法一次性读入内存，那就继续再划分，直到所有的文件都能读入内存为止。

# `V8` 如何实现 `sort` ？

## 概括( `ES9` 以前)

`sort` 并没有被要求 `稳定性`。

排序采用的算法跟数组的长度有关，当数组长度小于等于 10 时，采用插入排序，大于 10 的时候，采用快速排序。

### 为什么选择插入排序？

虽然`插入排序`理论上说是O(n²)的算法，`快速排序`是一个O(nlogn)级别的算法。但是别忘了，这只是理论上的估算，在实际情况中大O表示法会忽略低阶、系数、常数系数的， 当 n 足够小的时候，快速排序`nlogn`的优势会越来越小，倘若插入排序O(n^2)前面的系数足够小，那么就会超过快排。

因此，对于很小的数据量， `插入排序` 是一个非常不错的选择。

###  `V8` 基准选择

当数组长度大于 10 但是小于 1000 的时候，采用三数取中法，实现代码为：

```js
// 基准的下标
// >> 1 相当于除以 2 (忽略余数)
third_index = from + ((to - from) >> 1);
```

当数组长度大于 1000 的时候，每隔 200 ~ 215 个元素取一个值，然后将这些值进行排序，取中间值的下标，实现的代码为：

```js
function GetThirdIndex(a, from, to) {
    var t_array = new Array();

    // & 位运算符
    var increment = 200 + ((to - from) & 15);

    var j = 0;
    from += 1;
    to -= 1;

    for (var i = from; i < to; i += increment) {
        t_array[j] = [i, a[i]];
        j++;
    }
    // 对随机挑选的这些值进行排序
    t_array.sort(function(a, b) {
        return comparefn(a[1], b[1]);
    });
    // 取中间值的下标
    var third_index = t_array[t_array.length >> 1][0];
    return third_index;
}
```

## `ES9` 以后的 `V8`

规范中要求了 `sort` 方法要具有稳定性，所以在 V8 引擎 **7.0 版本之后** 就舍弃了快速排序。

所以如今 V8 使用一种混合排序算法 TimSort 。

在数据量小的子数组中使用 **插入排序**，然后再使用**归并排序**将有序的子数组进行合并排序，时间复杂度为 `O(nlogn)` 。

 Timsort 会遍历所有数据，找出数据中所有有序的分区（run），然后按照一定的规则将这些分区（run）归并为一个。

具体过程为：

- 扫描数组，一个 run 可以认为是已经排序的小数组，也包括以逆向排序的，因为这些数组可以简单地翻转（reverse）就成为一个run
- 计算最小合并序列长度 run，小于的 run 会通过 **插入排序** 补足为长度大于最小长度的 run，并将run压栈
- 每次有新的 run 被压入栈时保证栈内任意3个连续的 run 从下至上满足 `run0>run1+run2 && run1>run2` ，不满足的话进行调整直至满足（为了提升性能,避免归并长度相差很大的片段 )
- 归并相邻 run ，直至完成

对于已经排序好的数组，会以 O(n) 的时间内完成排序（这就是最好情况的时间复杂度），因为这样的数组将只产生单个 run ，不需要合并操作。

最坏的情况是 O(n log n) 。

这样的算法性能参数，以及 `Timsort`的稳定性 是我们最终选择 `Timsort` 而非快排的几个原因。