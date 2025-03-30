## 数据结构 (一) - 单向链表 (Singly Linked List) 
### 简介

在计算机科学中， 一个`链表`是数据元素的`线性集合`，元素的线性顺序不是由它们在内存中的物理位置给出的。相反，每个元素指向下一个元素。它是由一组节点组成的数据结构，这些节点一起，表示序列。

在最简单的形式下，每个节点由数据和到序列中下一个节点的引用(换句话说，链接)组成。这种结构遍历时，能在任何位置极为方便得`插入`、`删除`元素。

链表的`访问时间是线性的`。

如果想快速访问到链表的某个节点位置是困难的。与链表相比，数组具有更好的缓存位置，它能通过索引快速访问到节点位置。

相比数组，链表在遍历阶段删除节点`无需调整`后面节点的位置信息。

但在遍历数组时删除节点，如果想保持数组结构，就必须把后面的节点进行索引重计算操作。而这样的操作带来的风险是计算过程中原有对数组节点的引用将全部失效。所以通常在数组遍历过程中删除原来的数组元素是被禁止的。

<img src='linked-list.svg'>

### 基本操作的伪代码

最简单的链表结构只需要维护一个节点变量即可，所有其他节点都可以通过该变量的引用来获取。

但是为了对链表结构操作的便携，通常链表通过两个分别表意`头部节点`和`尾部节点`的变量作为链表的基本属性。

#### 插入

```text
Add(value)
  Pre: value is the value to add to the list
  Post: value has been placed at the tail of the list
  n ← node(value)
  if head = ø
    head ← n
    tail ← n
  else
    tail.next ← n
    tail ← n
  end if
end Add
```

1. 若链表为空，则新节点变成链表的头部和尾部；
2. 若链表非空，则`尾部.next`指向新节点，新节点变成尾部。

```text
Prepend(value)
 Pre: value is the value to add to the list
 Post: value has been placed at the head of the list
 n ← node(value)
 n.next ← head
 head ← n
 if tail = ø
   tail ← n
 end
end Prepend
```

1. 新节点指向头部，头部变成新节点；
2. 若链表为空，尾部变成新节点。

#### 搜索

```text
Contains(head, value)
  Pre: head is the head node in the list
       value is the value to search for
  Post: the item is either in the linked list, true; otherwise false
  n ← head
  while n != ø and n.value != value
    n ← n.next
  end while
  if n = ø
    return false
  end if
  return true
end Contains
```

1. 从头部开始，线性遍历链表；
2. 若对比节点值相等，搜索结束。

#### 删除

```text
Remove(head, value)
  Pre: head is the head node in the list
       value is the value to remove from the list
  Post: value is removed from the list, true, otherwise false
  if head = ø
    return false
  end if
  n ← head
  if n.value = value
    if head = tail
      head ← ø
      tail ← ø
    else
      head ← head.next
    end if
    return true
  end if
  while n.next != ø and n.next.value != value
    n ← n.next
  end while
  if n.next != ø
    if n.next = tail
      tail ← n
    end if
    n.next ← n.next.next
    return true
  end if
  return false
end Remove
```

1. 链表空，停止删除操作；
2. 校验头部是否需要删除？从头部遍历，若头部为要删除的节点，则把头部变成`头部.next` (删除操作) ，直到头部为非删除的值；
3. 从`头部.next`开始遍历，若发现遍历的`当前节点.next` 为需要删除的节点，则把`当前节点.next.next`赋值给`当前节点.next` (删除操作) ;
4. 校验尾部是否需要删除？若为要删除的节点，则把步骤 3 遍历结束的`当前节点`变成尾部。

#### 遍历

```text
Traverse(head)
  Pre: head is the head node in the list
  Post: the items in the list have been traversed
  n ← head
  while n != ø
    yield n.value
    n ← n.next
  end while
end Traverse
```

1. 线性遍历

#### 反向遍历

```text
ReverseTraversal(head, tail)
  Pre: head and tail belong to the same list
  Post: the items in the list have been traversed in reverse order
  if tail != ø
    curr ← tail
    while curr != head
      prev ← head
      while prev.next != curr
        prev ← prev.next
      end while
      yield curr.value
      curr ← prev
    end while
   yield curr.value
  end if
end ReverseTraversal
```

1. 若链表为空，停止操作；
2. 从尾部开始遍历，直到发现`当前节点`为头部时，停止遍历。 (外层遍历执行一次时，内层遍历就要线性查找上一个节点)

单向链表的反向遍历复杂度为 $O(n^2)$，随着链表长度的增长，这种单向链表结构不再适合反向遍历。可以考虑`双向链表结构`，因为它的每个节点都保存着上一个节点的引用。

### 复杂度

#### 时间复杂度

|  访问  |  搜索  |  插入  |  删除  |
| :----: | :----: | :----: | :----: |
| $O(n)$ | $O(n)$ | $O(1)$ | $O(1)$ |

#### 空间复杂度

$O(n)$

参考资料：

\> [https://en.wikipedia.org/wiki/Linked_list](https://en.wikipedia.org/wiki/Linked_list)

\> [https://www.youtube.com/watch?v=njTh_OwMljA&index=2&t=1s&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8](https://www.youtube.com/watch?v=njTh_OwMljA&index=2&t=1s&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)
