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
  - Удаляемый нод **не имеет правого** потомка (1)
  - Удаляемый нод **не имеет левого** потомка (2)
  - Удаляемый нод иммет и левого, и правого потомков (3)

1) Удаляем 8:
- Найти удаляемый нод
- Видим, что он не имеет правого потомка
- Подниимаем вверх его левый потомок со всем его деревом

![Binary Tree Example](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_Removing_Case1.JPG)

2) Удаляем 6:
- Найти удаляемый нод
- Видим, что он не имеет левого потомка
- Подниимаем вверх его правый потомок со всем его деревом

![Binary Tree Example](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_Removing_Case2.JPG)

3) Удаляем 6:
- Найти удаляемый нод
- Видим, что он имеет оба потомка
- Находим самый **последний левый потомок (left-most child)**
- Заменяем удаляемый нод на найденный самый **последний левый потомок (left-most child)**

Мы поднимаем самый левый нод **(left-most child)**, так как мы знаем, что он самый меньший нод.

![Binary Tree Example](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_Removing_Case3.JPG)
