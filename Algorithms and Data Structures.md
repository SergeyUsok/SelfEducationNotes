# Data Structures

### Односвязный Linked List
- RemoveLast() на самом деле операция O(n), посокльку требуется проход по **всему** List'у для того, чтобы найти **предпоследний**
нод, из которого нужно удалить ссылку на последний.

### Binary Tree
- В двоичном дереве **левый** нод всегда меньше, чем **root** и **правый** нод больше.

![Binary Tree Example](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_Example.JPG)


# Algorithms

### Binary Tree Adding node
- Проходим по дереву рекурсивно, сравнивая добавляемый нод с существующими, и добавляем его как лист (leaf) в один из нодов. При прохождении по дереву нод будет может определяться и как правый и как левый, в зависимости от результата сравнения.
- Случай равенства трактуется как "больше", то есть добаление идет в правую сторону.

Пример добавления 5:

![Binary Tree Example](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_AddingNode.JPG)

### Binary Tree Searhing node
- Осуществляется также, как и добавление, то есть рекурсивно просматриваем дерево, пока не найдем или пока дерево не закончится.

### Binary Tree Removing node 
- Если удалаемый нод является **листом** (leaf), просто удалить ссылку на него из родительского нода.
- Если нод НЕ является листом, существет 3 сценария:
  - Удаляемый нод **не имеет правого** потомка
  - Удаляемый нод **не имеет левого** потомка
  - Удаляемый нод иммет и левого, и правого потомков.

1) Удаляем 8:

![Binary Tree Example](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_Removing_Case1.JPG)

2) Удаляем 6:

![Binary Tree Example](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_Removing_Case2.JPG)

3) Удаляем 6:

![Binary Tree Example](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_Removing_Case3.JPG)
