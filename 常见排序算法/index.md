## 常见排序算法 
### 基本操作

#### 交换

``` ts

function swap(a, b) {
  [a, b] = [b, a]
}
```

#### 比较

``` ts
function compare_gt(a, b) {
  return a > b;
}
```

``` ts
function compare_lt(a, b) {
  return a < b;
}
```
### 算法

#### 冒泡排序 (Bubble Sort)

| 情况     | 时间复杂度  | 描述                                                                 |
|----------|-------------|----------------------------------------------------------------------|
| 最坏情况 | O(n^2)      | 数组是反序的，每个元素都需要移动到它正确的位置。                     |
| 最好情况 | O(n)        | 数组已经有序，优化版本在第一趟遍历后检测到没有需要交换的元素，提前退出。|
| 平均情况 | O(n^2)      | 考虑所有可能的输入排列，总是需要进行 $O(n^2)$ 次比较和交换操作。   |

```typescript
function bubbleSort(arr: number[]): number[] {
  let n = arr.length;
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < n - i - 1; j++) {
      if (compare_gt(arr[j], arr[j + 1])) {
        swap(arr[j], arr[j + 1])
      }
    }
  }
  return arr;
}
```

{{< iframe src="https://www.lumin.tech/tools/sort/?type=bubbleSort" width="100%" >}}

#### 选择排序 (Selection Sort)

```typescript
function selectionSort(arr: number[]): number[] {
  let n = arr.length;
  for (let i = 0; i < n; i++) {
    let minIdx = i;
    for (let j = i + 1; j < n; j++) {
      if (compare_lt(arr[j], arr[minIdx])) {
        minIdx = j;
      }
    }

    swap(arr[minIdx], arr[i])
    
  }
  return arr;
}
```

{{< iframe src="https://www.lumin.tech/tools/sort/?type=selectionSort" width="100%" >}}

#### 插入排序 (Insertion Sort)

插入排序（Insertion Sort）的时间复杂度取决于输入数据的`初始顺序`。

时间复杂度：

| 情况     | 时间复杂度  | 描述                                                                 |
|----------|-------------|----------------------------------------------------------------------|
| 最坏情况 | O(n^2)      | 数组是反序的，每个元素都需要移动到它正确的位置。                     |
| 最好情况 | O(n)        | 数组已经有序，每次插入只需进行一次比较操作，不需要移动。               |
| 平均情况 | O(n^2)      | 考虑所有可能的输入排列，平均需要进行 $O(n^2)$ 次比较和移动操作。     |


```typescript
function insertionSort(arr: number[]): number[] {
  let n = arr.length;
  for (let i = 1; i < n; i++) {
    let key = arr[i];
    let j = i - 1;
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j = j - 1;
    }
    arr[j + 1] = key;
  }
  return arr;
}
```

{{< iframe src="https://www.lumin.tech/tools/sort/?type=insertionSort" width="100%" >}}

#### 希尔排序 (Shell Sort)

```typescript
function shellSort(arr: number[]): number[] {
  let n = arr.length;
  for (let gap = Math.floor(n / 2); gap > 0; gap = Math.floor(gap / 2)) {
    for (let i = gap; i < n; i++) {
      let temp = arr[i];
      let j: number;
      for (j = i; j >= gap && arr[j - gap] > temp; j -= gap) {
        arr[j] = arr[j - gap];
      }
      arr[j] = temp;
    }
  }
  return arr;
}
```

#### 归并排序 (Merge Sort)

```typescript
function mergeSort(arr: number[]): number[] {
  if (arr.length <= 1) {
    return arr;
  }

  const mid = Math.floor(arr.length / 2);
  const left = arr.slice(0, mid);
  const right = arr.slice(mid);

  return merge(mergeSort(left), mergeSort(right));
}

function merge(left: number[], right: number[]): number[] {
  let result: number[] = [];
  let leftIndex = 0;
  let rightIndex = 0;

  while (leftIndex < left.length && rightIndex < right.length) {
    if (left[leftIndex] < right[rightIndex]) {
      result.push(left[leftIndex]);
      leftIndex++;
    } else {
      result.push(right[rightIndex]);
      rightIndex++;
    }
  }

  return result.concat(left.slice(leftIndex)).concat(right.slice(rightIndex));
}
```

{{< iframe src="https://www.lumin.tech/tools/sort/?type=mergeSort" width="100%" >}}

#### 快速排序 (Quick Sort)

算法步骤：

1. **选择基准**（Pivot）：从数组中选择一个元素作为基准。
2. **分区**（Partition）：重新排列数组，使得所有小于基准的元素位于基准左侧，所有大于基准的元素位于基准右侧。
3. **递归排序**（Recursive Sort）：递归地对基准左侧和右侧的子数组进行排序。

优化：

* `选择随机基准`：通过随机选择基准元素，可以减少最坏情况（即每次选择的基准都是数组的最大或最小元素）的概率。

时间复杂度：

| 情况       | 时间复杂度  | 描述                                                                 |
|------------|-------------|----------------------------------------------------------------------|
| 平均情况   | O(n log n)  | 基准元素将数组平衡地分成两个大小近似相等的子数组。                     |
| 最坏情况   | O(n^2)      | 基准元素每次都选择了最小或最大的元素，导致数组没有有效分区。             |
| 最优情况   | O(n log n)  | 基准元素每次都将数组完美地分成两个大小完全相等的子数组。                 |

```typescript
function quickSort<T>(arr: T[]): T[] {
    if (arr.length <= 1) {
        return arr;
    }
    
    const pivotIndex = Math.floor(Math.random() * arr.length);
    const pivot = arr[pivotIndex];
    const left: T[] = [];
    const right: T[] = [];
    
    for (let i = 0; i < arr.length; i++) {
        if (i === pivotIndex) continue;
        if (arr[i] < pivot) {
            left.push(arr[i]);
        } else {
            right.push(arr[i]);
        }
    }
    
    return [...quickSort(left), pivot, ...quickSort(right)];
}

const arr = [3, 6, 8, 10, 1, 2, 1];
console.log(quickSort(arr)); // [1, 1, 2, 3, 6, 8, 10]
```

{{< iframe src="https://www.lumin.tech/tools/sort/?type=quickSort" width="100%" >}}

#### 基数排序 (Radix Sort)

```typescript
function getMax(arr: number[]): number {
  let max = arr[0];
  for (let i = 1; i < arr.length; i++) {
    if (arr[i] > max) {
      max = arr[i];
    }
  }
  return max;
}

function countingSort(arr: number[], exp: number): void {
  let n = arr.length;
  let output: number[] = new Array(n);
  let count: number[] = new Array(10).fill(0);

  for (let i = 0; i < n; i++) {
    let index = Math.floor(arr[i] / exp) % 10;
    count[index]++;
  }

  for (let i = 1; i < 10; i++) {
    count[i] += count[i - 1];
  }

  for (let i = n - 1; i >= 0; i--) {
    let index = Math.floor(arr[i] / exp) % 10;
    output[count[index] - 1] = arr[i];
    count[index]--;
  }

  for (let i = 0; i < n; i++) {
    arr[i] = output[i];
  }
}

function radixSort(arr: number[]): number[] {
  let max = getMax(arr);
  for (let exp = 1; Math.floor(max / exp) > 0; exp *= 10) {
    countingSort(arr, exp);
  }
  return arr;
}
```

#### 堆排序 (Heap Sort)

```typescript
function heapify(arr: number[], n: number, i: number): void {
  let largest = i;
  let left = 2 * i + 1;
  let right = 2 * i + 2;

  if (left < n && arr[left] > arr[largest]) {
    largest = left;
  }

  if (right < n && arr[right] > arr[largest]) {
    largest = right;
  }

  if (largest !== i) {
    let swap = arr[i];
    arr[i] = arr[largest];
    arr[largest] = swap;

    heapify(arr, n, largest);
  }
}

function heapSort(arr: number[]): number[] {
  let n = arr.length;

  for (let i = Math.floor(n / 2) - 1; i >= 0; i--) {
    heapify(arr, n, i);
  }

  for (let i = n - 1; i > 0; i--) {
    let temp = arr[0];
    arr[0] = arr[i];
    arr[i] = temp;

    heapify(arr, i, 0);
  }

  return arr;
}
```

#### 计数排序 (Counting Sort)

```typescript
function countingSort(arr: number[]): number[] {
  let max = Math.max(...arr);
  let min = Math.min(...arr);
  let range = max - min + 1;
  let count: number[] = new Array(range).fill(0);
  let output: number[] = new Array(arr.length).fill(0);

  for (let i = 0; i < arr.length; i++) {
    count[arr[i] - min]++;
  }

  for (let i = 1; i < count.length; i++) {
    count[i] += count[i - 1];
  }

  for (let i = arr.length - 1; i >= 0; i--) {
    output[count[arr[i] - min] - 1] = arr[i];
    count[arr[i] - min]--;
  }

  for (let i = 0; i < arr.length; i++) {
    arr[i] = output[i];
  }

  return arr;
}
```

#### 桶排序 (Bucket Sort)

```typescript
function bucketSort(arr: number[], bucketSize: number = 5): number[] {
  if (arr.length === 0) {
    return arr;
  }

  let minValue = Math.min(...arr);
  let maxValue = Math.max(...arr);

  let bucketCount = Math.floor((maxValue - minValue) / bucketSize) + 1;
  let buckets: number[][] = new Array(bucketCount).fill(null).map(() => []);

  for (let i = 0; i < arr.length; i++) {
    let bucketIndex = Math.floor((arr[i] - minValue) / bucketSize);
    buckets[bucketIndex].push(arr[i]);
  }

  arr.length = 0;
  for (let i = 0; i < buckets.length; i++) {
    if (buckets[i] !== null) {
      insertionSort(buckets[i]);
      arr.push(...buckets[i]);
    }
  }

  return arr;
}
```


