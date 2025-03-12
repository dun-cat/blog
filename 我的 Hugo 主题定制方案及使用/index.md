## 我的 Hugo 主题定制方案及使用 
### Markdown 

#### 使用 shortcodes 创建 notice 提示

通常在文章需要特殊说明的地方可以使用 `notice 提示`，同时它支持 `dark` 和 `light` 两种主题。如下面的效果：

**Note**

{{< notice note  >}}
A notice disclaimer
{{< /notice >}}

**Info**

{{< notice info >}}
An information disclaimer
{{< /notice >}}

**Tip**

{{< notice tip >}} A tip disclaimer {{< /notice >}}

**Warning**

{{< notice warning >}} This is a warning notice. Be warned! {{< /notice >}}

{{% notice warning %}}
爱情是生命的甜蜜旋律，
心灵的交汇，情感的奇迹。
在暗夜里，你是明亮的星辰，
在寂寞中，你是温暖的怀抱。

你的笑容如春风拂面，
我的世界因你而充满了色彩。
你的眼睛，那如潭清水，
深情地注视，让我陷入迷醉。

爱情是岁月的礼物和诺言，
我们一起前行，无惧风雨。
手牵手，心贴心，不离不弃，
这份深情，永远不会凋零。

在你的怀抱中，我找到家，
在你的微笑里，我感到温馨。
爱情的力量，永不衰退，
你是我生命中最美的风景。

在黎明的曙光中，我们一同醒来，
在星光的璀璨下，我们共舞。
爱情的旋律，永不停歇，
因为你，我的爱，永远屹立。

{{% /notice %}}

你可以在[这里](https://github.com/martignoni/hugo-notice/tree/master/layouts/shortcodes)看到它的 shortcode 源码，并根据[官方文档](https://gohugo.io/templates/shortcode-templates/)来创建自己的 shortcode 或调整你想要的样式。



### 代码内部部分高亮

通过设置 `{hl_lines="16-22"}`

``` js {hl_lines="11-19"}
function quickSort(arr) {
    // 如果数组长度小于等于 1，直接返回该数组，因为它已经是有序的
    if (arr.length <= 1) {
        return arr;
    }
    // 选择数组的第一个元素作为基准元素
    const pivot = arr[0];
    const left = [];
    const right = [];
    // 从第二个元素开始遍历数组
    for (let i = 1; i < arr.length; i++) {
        if (arr[i] < pivot) {
            // 如果当前元素小于基准元素，将其放入左数组
            left.push(arr[i]);
        } else {
            // 如果当前元素大于等于基准元素，将其放入右数组
            right.push(arr[i]);
        }
    }
    // 递归地对左数组和右数组进行快速排序，并将结果合并
    return [...quickSort(left), pivot, ...quickSort(right)];
}

// 示例用法
const unsortedArray = [3, 6, 8, 10, 1, 2, 1];
const sortedArray = quickSort(unsortedArray);
console.log(sortedArray); 

```