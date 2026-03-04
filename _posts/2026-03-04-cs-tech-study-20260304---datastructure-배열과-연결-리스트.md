---
layout: post
title: "[CS Tech Study] 2026.03.04 - DataStructure: 배열과 연결 리스트"
date: 2026-03-04 16:43:53
excerpt: "한 줄 핵심 정의"
tags: [DataStructure, cs-study]
image: "/assets/images/thumbnails/2026-03-04-cs-tech-study-20260304---datastructure-배열과-연결-리스트.svg"
series: "HoneyByte"
---
> 한 줄 핵심 정의
배열은 메모리 상에 연속적으로 할당되어 빠른 접근(O(1))이 가능하지만, 삽입/삭제 시 요소 이동으로 비효율적입니다(O(n)). 연결 리스트는 노드들이 포인터로 연결되어 삽입/삭제(O(1))가 빠르지만, 접근 시 순차 탐색이 필요하여 느립니다(O(n)). 각 자료구조는 특정 연산에 최적화되어 있어, 사용 사례에 따라 적합한 것을 선택해야 합니다.

## TL;DR

배열은 메모리 상에 연속적으로 할당되어 빠른 접근(O(1))이 가능하지만, 삽입/삭제 시 요소 이동으로 비효율적입니다(O(n)). 연결 리스트는 노드들이 포인터로 연결되어 삽입/삭제(O(1))가 빠르지만, 접근 시 순차 탐색이 필요하여 느립니다(O(n)). 각 자료구조는 특정 연산에 최적화되어 있어, 사용 사례에 따라 적합한 것을 선택해야 합니다.

## 개념 이해

### 배열 (Array)
메모리 상에 연속적인 공간에 데이터를 저장하는 자료구조입니다. 각 데이터는 인덱스를 통해 고유하게 접근할 수 있습니다.

*   **정적 배열 (Static Array)**: 컴파일 시점에 크기가 고정됩니다.
*   **동적 배열 (Dynamic Array)**: 런타임에 크기를 조절할 수 있습니다. 내부적으로는 보통 고정 크기 배열을 사용하며, 용량이 부족하면 더 큰 새 배열을 할당하고 기존 데이터를 복사합니다. (예: Python `list`, Java `ArrayList`)

**장점**:
*   인덱스를 통한 임의 접근(Random Access)이 매우 빠릅니다 (O(1)).

**단점**:
*   요소 삽입 또는 삭제 시, 해당 위치 이후의 모든 요소를 이동시켜야 하므로 비효율적입니다 (O(n)).
*   크기가 고정된 경우, 저장 공간이 낭비되거나 확장 시 오버헤드가 발생합니다.

### 연결 리스트 (Linked List)
각 데이터 요소(노드)가 데이터와 다음 노드를 가리키는 포인터(또는 주소)로 구성됩니다. 노드들이 메모리 상에 흩어져 있어도 순차적으로 연결될 수 있습니다.

*   **단일 연결 리스트 (Singly Linked List)**: 각 노드는 데이터와 다음 노드를 가리키는 포인터만 가집니다.
*   **이중 연결 리스트 (Doubly Linked List)**: 각 노드는 데이터, 다음 노드 포인터, 이전 노드 포인터를 모두 가집니다. 양방향 탐색이 가능합니다.
*   **원형 연결 리스트 (Circular Linked List)**: 마지막 노드의 다음 포인터가 첫 번째 노드를 가리키거나, 이중 연결 리스트에서 첫 번째 노드의 이전 포인터가 마지막 노드를 가리키는 등 순환 구조를 가집니다.

**장점**:
*   요소 삽입 및 삭제가 효율적입니다 (O(1), 단, 삽입/삭제 위치를 알아야 함).
*   크기가 동적으로 조절됩니다.

**단점**:
*   특정 위치의 데이터에 접근하려면 처음부터 순차적으로 탐색해야 하므로 느립니다 (O(n)).
*   각 노드마다 추가적인 메모리(포인터)가 필요합니다.

## 시각화

```mermaid
graph TD
    subgraph 배열 (Array)
        A[Index 0] --> B(Index 1)
        B --> C(Index 2)
        C --> D(...)
    end

    subgraph 단일 연결 리스트 (Singly Linked List)
        N1[Data | Next] --> N2(Data | Next)
        N2 --> N3(Data | Next)
        N3 --> N4(...)
        N4 --> Null
    end

    subgraph 이중 연결 리스트 (Doubly Linked List)
        P1[Prev | Data | Next] <--> P2(Prev | Data | Next)
        P2 <--> P3(Prev | Data | Next)
        P3 <--> P4(...)
        P4 --> Null
    end

    subgraph 원형 연결 리스트 (Circular Linked List)
        C1[Data | Next] --> C2(Data | Next)
        C2 --> C3(Data | Next)
        C3 --> C1
    end
```

## 구현 코드

### Python
```python
# 동적 배열 (Python list)
print(\"---\
--- Python Dynamic Array ---\")
my_list = [10, 20, 30]
print(f\"Initial list: {my_list}\")

# Access (O(1))
print(f\"Element at index 1: {my_list[1]}\")

# Append (amortized O(1))
my_list.append(40)
print(f\"After append(40): {my_list}\")

# Insert at index (O(n))
my_list.insert(1, 15)
print(f\"After insert(1, 15): {my_list}\")

# Remove by value (O(n))
my_list.remove(20)
print(f\"After remove(20): {my_list}\")

# Remove by index (O(n))
del my_list[0]
print(f\"After del my_list[0]: {my_list}\")


# 연결 리스트 (Singly Linked List)
print(\"\
---\
--- Python Singly Linked List ---\")
class Node:
    def __init__(self, data=None):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None

    def append(self, data):
        new_node = Node(data)
        if not self.head:
            self.head = new_node
            return
        last_node = self.head
        while last_node.next:
            last_node = last_node.next
        last_node.next = new_node

    def display(self):
        elements = []
        current = self.head
        while current:
            elements.append(current.data)
            current = current.next
        print(f\"List: {elements}\")

    def insert_after(self, prev_node_data, new_data):
        current = self.head
        while current:
            if current.data == prev_node_data:
                new_node = Node(new_data)
                new_node.next = current.next
                current.next = new_node
                return
            current = current.next
        print(f\"Node with data {prev_node_data} not found.\")

    def delete_node(self, key):
        current = self.head
        if current and current.data == key:
            self.head = current.next
            return
        prev = None
        while current and current.data != key:
            prev = current
            current = current.next
        if current is None:
            print(f\"Node with data {key} not found.\")
            return
        prev.next = current.next

ll = LinkedList()
ll.append(1)
ll.append(2)
ll.append(3)
ll.display() # List: [1, 2, 3]

ll.insert_after(2, 2.5)
ll.display() # List: [1, 2, 2.5, 3]

ll.delete_node(2.5)
ll.display() # List: [1, 2, 3]

ll.delete_node(1)
ll.display() # List: [2, 3]
```

### Java
```java
// Dynamic Array (Java ArrayList)
import java.util.ArrayList;
import java.util.List;

class DynamicArrayExample {
    public static void main(String[] args) {
        System.out.println(\"---\
--- Java Dynamic Array ---\");
        List<Integer> myList = new ArrayList<>();
        myList.add(10);
        myList.add(20);
        myList.add(30);
        System.out.println(\"Initial list: \" + myList);

        // Access (O(1))
        System.out.println(\"Element at index 1: \" + myList.get(1));

        // Append (amortized O(1))
        myList.add(40);
        System.out.println(\"After add(40): \" + myList);

        // Insert at index (O(n))
        myList.add(1, 15);
        System.out.println(\"After add(1, 15): \" + myList);

        // Remove by index (O(n))
        myList.remove(0);
        System.out.println(\"After remove(0): \" + myList);

        // Remove by value (O(n))
        myList.remove(Integer.valueOf(20));
        System.out.println(\"After remove(20): \" + myList);
    }
}

// Singly Linked List
class SinglyLinkedList {
    Node head;

    static class Node {
        int data;
        Node next;

        Node(int d) {
            data = d;
            next = null;
        }
    }

    // Method to insert a new node at the end of the list
    public void append(int new_data) {
        Node new_node = new Node(new_data);
        if (head == null) {
            head = new_node;
            return;
        }
        Node last = head;
        while (last.next != null) {
            last = last.next;
        }
        last.next = new_node;
    }

    // Method to delete a node with a given key
    public void deleteNode(int key) {
        Node current = head;
        Node prev = null;

        // If head node itself holds the key
        if (current != null && current.data == key) {
            head = current.next; // Changed head
            return;
        }

        // Search for the key to be deleted, keep track of the previous node
        while (current != null && current.data != key) {
            prev = current;
            current = current.next;
        }

        // If the key was not present in linked list
        if (current == null) return;

        // Unlink the node from the linked list
        prev.next = current.next;
    }

    // Method to print the linked list
    public void printList() {
        Node current = head;
        System.out.print(\"List: [\");
        boolean first = true;
        while (current != null) {
            if (!first) {
                System.out.print(\", \");
            }
            System.out.print(current.data);
            current = current.next;
            first = false;
        }
        System.out.println(\"]\");
    }

    public static void main(String[] args) {
        System.out.println(\"\
---\
--- Java Singly Linked List ---\");
        SinglyLinkedList list = new SinglyLinkedList();

        list.append(1);
        list.append(2);
        list.append(3);
        list.printList(); // List: [1, 2, 3]

        list.deleteNode(2);
        list.printList(); // List: [1, 3]

        list.deleteNode(1);
        list.printList(); // List: [3]
    }
}
```

## 복잡도 분석

| Operation         | Array (Dynamic) | Singly Linked List | Doubly Linked List |
| :---------------- | :-------------- | :----------------- | :----------------- |
| Access by Index   | O(1)            | O(n)               | O(n)               |
| Insertion (Begin) | O(n)            | O(1)               | O(1)               |
| Insertion (End)   | Amortized O(1)  | O(n) (O(1) if tail ptr) | O(1)               |
| Insertion (Middle)| O(n)            | O(1) (if prev node known) | O(1) (if prev node known) |
| Deletion (Begin)  | O(n)            | O(1)               | O(1)               |
| Deletion (End)    | O(1)            | O(n)               | O(1)               |
| Deletion (Middle) | O(n)            | O(1) (if prev node known) | O(1) (if prev node known) |
| Space Complexity  | O(n)            | O(n)               | O(n)               |

*   **Note on Linked List Insertion/Deletion**: The O(1) complexity assumes you already have a reference to the node *before* the insertion/deletion point (for singly linked list) or the node itself (for doubly linked list). Finding that node by index still takes O(n).

## 실무 적용

*   **Arrays**:
    *   `ArrayList` in Java, `list` in Python, `std::vector` in C++ are dynamic arrays used extensively.
    *   Implementing lookup tables, buffers, or fixed-size data structures.
    *   Graphics programming for storing vertex data.
*   **Linked Lists**:
    *   Implementing stacks and queues.
    *   Undo/redo functionality in text editors or software.
    *   Memory management (allocating and deallocating memory blocks).
    *   Browser history (back/forward navigation).
    *   Task scheduling in operating systems.

## 관련 문제

*   **LeetCode**:
    *   \"Merge Two Sorted Lists\" (Easy) - Linked List manipulation.
    *   \"Remove Duplicates from Sorted List\" (Easy) - Linked List manipulation.
    *   \"Container With Most Water\" (Medium) - Array problem with two-pointer approach.
    *   \"3Sum\" (Medium) - Array problem often solved with sorting and two pointers.
*   **Baekjoon**:
    *   10828. 스택 (Stack) - Linked List or Array based.
    *   10773. 제로 (Stack) - Uses stack logic.
    *   11650. 좌표 정렬하기 (Sorting Array) - Requires sorting based on array elements.

## 정리

| Item            | Array                                   | Linked List                               |
| :-------------- | :-------------------------------------- | :---------------------------------------- |
| **Structure**   | Contiguous memory                       | Nodes with pointers                       |
| **Access**      | O(1) (by index)                         | O(n) (sequential)                         |
| **Insertion**   | O(n) (middle/beginning), Amortized O(1) (end) | O(1) (if position known)                  |
| **Deletions**   | O(n) (middle/beginning), O(1) (end)     | O(1) (if position known)                  |
| **Memory**      | Efficient (no overhead per element)     | Overhead per node (pointers)              |
| **Use Cases**   | Frequent access, less frequent inserts/deletes | Frequent inserts/deletes, sequential access |

## 관련 포스트

*   (No previous related posts found in memory)

## 레퍼런스

### 영상
*   [Arrays and Linked Lists - Data Structures & Algorithms](https://www.youtube.com/watch?v=T02o0_0_l5k) - freeCodeCamp.org, Comprehensive explanation with diagrams.
*   [Arrays vs Linked Lists - Computerphile](https://www.youtube.com/watch?v=2rs0f8e3oP4) - Computerphile, Explains the fundamental differences.

### 문서 & 기사
*   [Arrays vs. Linked Lists: Which is best for your needs?](https://www.freecodecamp.org/news/arrays-vs-linked-lists-which-is-best-for-your-needs/) - freeCodeCamp, Detailed comparison and use cases.
*   [Array - Wikipedia](https://en.wikipedia.org/wiki/Array_data_structure) - Wikipedia, In-depth information on arrays.
*   [Linked List - Wikipedia](https://en.wikipedia.org/wiki/Linked_list) - Wikipedia, In-depth information on linked lists.

---
*This post is part of the [HoneyByte](https://blog.honeybarrel.co.kr) series.*