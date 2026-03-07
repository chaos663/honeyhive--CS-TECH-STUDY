---
layout: post
title: "DataStructure: 배열과 연결 리스트"
date: 2026-03-04 08:00:00 +0900
excerpt: "배열은 연속 메모리로 O(1) 접근이 빠르지만 삽입/삭제가 O(n) / 연결 리스트는 포인터 기반으로 삽입/삭제 O(1)이지만 접근이 O(n) / 용도에 따라 적합한 자료구조를 선택하는 것이 핵심"
tags: [DataStructure, cs-study]
image: "/assets/images/thumbnails/2026-03-04-datastructure-배열과-연결-리스트.svg"
series: "HoneyByte"
---
> 배열은 연속 메모리 할당으로 빠른 인덱스 접근을, 연결 리스트는 포인터 기반으로 유연한 삽입/삭제를 제공하는 기본 선형 자료구조이다.

## 핵심 요약 (TL;DR)

- 배열은 메모리 상에 연속적으로 할당되어 인덱스 기반 접근이 O(1)로 빠르지만, 중간 삽입/삭제 시 요소 이동이 필요해 O(n)입니다.
- 연결 리스트는 노드가 포인터로 연결되어 삽입/삭제가 O(1)로 빠르지만, 특정 위치 접근 시 순차 탐색이 필요해 O(n)입니다.
- 빈번한 접근이 필요하면 배열, 빈번한 삽입/삭제가 필요하면 연결 리스트를 선택하세요.

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

| 연산 | 동적 배열 | 단일 연결 리스트 | 이중 연결 리스트 |
|------|----------|----------------|----------------|
| 인덱스 접근 | O(1) | O(n) | O(n) |
| 맨 앞 삽입 | O(n) | O(1) | O(1) |
| 맨 뒤 삽입 | 평균 O(1) | O(n) (tail 포인터 있으면 O(1)) | O(1) |
| 중간 삽입 | O(n) | O(1) (이전 노드 알 때) | O(1) (이전 노드 알 때) |
| 맨 앞 삭제 | O(n) | O(1) | O(1) |
| 맨 뒤 삭제 | O(1) | O(n) | O(1) |
| 중간 삭제 | O(n) | O(1) (이전 노드 알 때) | O(1) (해당 노드 알 때) |
| 공간 복잡도 | O(n) | O(n) | O(n) |

> **참고**: 연결 리스트의 삽입/삭제 O(1)은 이미 해당 위치의 노드 참조를 갖고 있을 때의 복잡도입니다. 인덱스로 해당 노드를 찾는 데는 여전히 O(n)이 소요됩니다.

## 실무 적용

* **배열 기반 자료구조**:
    * Java의 `ArrayList`, Python의 `list`, C++의 `std::vector`가 대표적인 동적 배열 구현체입니다.
    * 룩업 테이블, 버퍼, 고정 크기 데이터 구조에 활용됩니다.
    * 그래픽 프로그래밍에서 정점(vertex) 데이터 저장에 사용됩니다.
* **연결 리스트 기반 자료구조**:
    * 스택과 큐의 내부 구현에 활용됩니다.
    * 텍스트 에디터의 실행 취소/다시 실행(Undo/Redo) 기능에 사용됩니다.
    * 운영체제의 메모리 관리(할당/해제 블록 관리)에 활용됩니다.
    * 브라우저 방문 기록(뒤로/앞으로 탐색)에 사용됩니다.
    * 운영체제의 태스크 스케줄링에 활용됩니다.

## 관련 문제

### 기초
* LeetCode: [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) (Easy) — 두 정렬된 연결 리스트를 병합하는 기본 문제
* LeetCode: [Remove Duplicates from Sorted List](https://leetcode.com/problems/remove-duplicates-from-sorted-list/) (Easy) — 정렬된 연결 리스트에서 중복 제거
* 백준: [10828 - 스택](https://www.acmicpc.net/problem/10828) — 배열 또는 연결 리스트 기반 스택 구현
* 백준: [10773 - 제로](https://www.acmicpc.net/problem/10773) — 스택 논리를 활용한 기본 문제

### 심화
* LeetCode: [Container With Most Water](https://leetcode.com/problems/container-with-most-water/) (Medium) — 배열에서 투 포인터를 활용한 최적화
* LeetCode: [3Sum](https://leetcode.com/problems/3sum/) (Medium) — 정렬 + 투 포인터 기법의 배열 문제
* 백준: [11650 - 좌표 정렬하기](https://www.acmicpc.net/problem/11650) — 배열 원소 기반 정렬 문제

## 정리

| 항목 | 배열 | 연결 리스트 |
|------|------|-----------|
| **구조** | 연속 메모리 | 노드 + 포인터 |
| **접근** | O(1) (인덱스) | O(n) (순차 탐색) |
| **삽입** | O(n) (앞/중간), 평균 O(1) (끝) | O(1) (위치를 알 때) |
| **삭제** | O(n) (앞/중간), O(1) (끝) | O(1) (위치를 알 때) |
| **메모리** | 효율적 (요소당 오버헤드 없음) | 노드당 포인터 오버헤드 |
| **적합한 상황** | 빈번한 접근, 적은 삽입/삭제 | 빈번한 삽입/삭제, 순차 접근 |

## 관련 포스트

* [OS: 프로세스와 스레드](/2026/03/03/os-프로세스와-스레드/) — 프로세스 제어 블록(PCB)이 연결 리스트로 관리되는 사례

## 레퍼런스

### 영상
* [Arrays and Linked Lists - Data Structures & Algorithms](https://www.youtube.com/watch?v=T02o0_0_l5k) — freeCodeCamp.org, 다이어그램과 함께 배열과 연결 리스트를 종합적으로 설명
* [Arrays vs Linked Lists - Computerphile](https://www.youtube.com/watch?v=2rs0f8e3oP4) — Computerphile, 두 자료구조의 근본적 차이를 설명

### 문서 & 기사
* [Arrays vs. Linked Lists: Which is best for your needs?](https://www.freecodecamp.org/news/arrays-vs-linked-lists-which-is-best-for-your-needs/) — freeCodeCamp, 상세 비교 및 활용 사례
* [Array - Wikipedia](https://en.wikipedia.org/wiki/Array_data_structure) — 배열 자료구조 상세 정보
* [Linked List - Wikipedia](https://en.wikipedia.org/wiki/Linked_list) — 연결 리스트 상세 정보

---
*이 포스트는 [HoneyByte](https://blog.honeybarrel.co.kr) 시리즈의 일부입니다.*
