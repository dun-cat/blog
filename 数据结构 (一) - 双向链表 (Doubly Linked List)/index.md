## 数据结构 (一) - 双向链表 (Doubly Linked List) 
### 双向链表

**双向链表(doubly linked list)** 是**单向链表**的扩展结构。单向链表的节点只包含下一个节点的引用及节点值，而双向链表的节点则多了一个包含上一个节点的引用。

<img src='doubly-linked-list.svg' />

两个节点链接允许在任一方向上遍历列表。

在双向链表中进行`添加`或者`删除`节点时,需做的链接更改要比单向链表复杂得多。这种操作在单向链表中更简单高效,因为不需要关注一个节点 (除第一个和最后一个节点以外的节点) 的两个链接,而只需要关注一个链接即可。

### 基础操作的伪代码

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
    n.previous ← tail
    tail.next ← n
    tail ← n
  end if
end Add
```

1. 若链表为空，则新节点变成链表的头部和尾部；
2. 若链表非空，则`尾部.next`指向新节点并把`新节点.previous`指向尾部，最后新节点变成尾部。

#### 删除

```text
Remove(head, value)
  Pre: head is the head node in the list
       value is the value to remove from the list
  Post: value is removed from the list, true; otherwise false
  if head = ø
    return ø
  end if

  current ← head

  while current != ø 
    if current.value === value
      if current == head
        delete head
      else if current == tail
        delete tail
      else
        delete current
    end if
    current ← current.next;
  end while

  return  
end Remove
```

1. 链表空，停止删除操作；
2. 从头部开始遍历，若值匹配：
   1. 若是头部，删除头部；
   2. 若是尾部，删除尾部；
   3. 否则，删除当前节点。
3. 否则，遍历继续。

#### 反向遍历

```text
ReverseTraversal(tail)
  Pre: tail is the node of the list to traverse
  Post: the list has been traversed in reverse order
  n ← tail
  while n = ø
    yield n.value
    n ← n.previous
  end while
end Reverse Traversal
```

1. 线性反向遍历。

#### 复杂度

#### 时间复杂度

|  访问  |  搜索  |  插入  |  删除  |
| :----: | :----: | :----: | :----: |
| $O(n)$ | $O(n)$ | $O(1)$ | $O(1)$ |

#### 空间复杂度

$O(n)$

参考资料：

\> [https://en.wikipedia.org/wiki/Doubly_linked_list](https://en.wikipedia.org/wiki/Doubly_linked_list)

\> [https://www.youtube.com/watch?v=JdQeNxWCguQ&t=7s&index=72&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8](https://www.youtube.com/watch?v=JdQeNxWCguQ&t=7s&index=72&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)
