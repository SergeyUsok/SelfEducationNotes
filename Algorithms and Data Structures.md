# Data Structures

### Односвязный Linked List
- RemoveLast() на самом деле операция O(n), посокльку требуется проход по **всему** List'у для того, чтобы найти **предпоследний**
нод, из которого нужно удалить ссылку на последний.

### Binary Tree
- В двоичном дереве **левый** нод всегда меньше, чем **root**, а **правый** нод всегда больше.

![Binary Tree Example](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_Example.JPG)

### AVL Tree
- Правила те же, что и у Binary Trees: меньшие ноды влево, большие или равные вправо.
- В то время как Binary Trees являются **не сбалансированными**, AVL-деревья **сбалансированы**. Это значит, что Binary Tree может превратиться в связный список (добавляем по очереди 1,2,3,4), а поиск стать операцией O(n). Т.о. AVL-tree самобалансируется, а поиск всегда является операцией O(log n).
- AVL-tree вводит четыре новых концепции:
  - Высота (Height) - **не должна** быть больше **1** между левой и правой ветвями дерева (разница между количеством нод слева и справа)
  - Самобалансирование - в приведенном выше примере (добавление 1,2,3,4) при добавлении 3 дерево расбалансируется, т.к. высота становится больше **1**. Значит делаем 2 root нодом вместо 1, а единицу отправляем влево, и высота становится равна **1**
  - Balance Factor
  - Right/Left Heavy

### Hashtable
- При добавлении элемента в коллекцию высчитываем его hash, а затем используем **modulo** операцию по отношению к **размеру** массива для того, чтобы узнать индекс ячейки массива, куда надо поместить данный элемент: **index = hashCode % array.Length**
- Коллизии разрешаются двумя (наиболее популярные) способами:
  - **Открытая адресация:** при коллизии элемент помещается в следующую свободную ячейку. Т.о. его поиск и удаление может стать операцией O(n) даже при хорошо подобранной хэш-фунции.
  - **Chaining:** при коллизии просто добавлем элемент в ту же ячейку в Linked List. Т.о. его поиск и удаление требует прохода только по одному связанному списку, и здесь на производительность влияет только хэш-функция. Но этот метод требует больше памяти, так как надо хранить не просто элементы, а заворачивать их в Linked List Nodes.

# Algorithms

### Binary Tree Adding node
- Проходим по дереву рекурсивно, сравнивая добавляемый нод с существующими, и добавляем его как лист (leaf) в один из нодов. При прохождении по дереву нод будет может определяться и как правый и как левый, в зависимости от результата сравнения.
- Случай равенства трактуется как "больше", то есть добаление идет в правую сторону.

Пример добавления 5:

![Add node](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_AddingNode.JPG)

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

![Removal case 1](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_Removing_Case1.JPG)

2) Удаляем 6:
- Найти удаляемый нод
- Видим, что он не имеет левого потомка
- Подниимаем вверх его правый потомок со всем его деревом

![Removal case 2](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_Removing_Case2.JPG)

3) Удаляем 6:
- Найти удаляемый нод
- Видим, что он имеет оба потомка
- Находим самый **последний левый потомок (left-most child)**
- Заменяем удаляемый нод на найденный самый **последний левый потомок (left-most child)**

Мы поднимаем самый левый нод **(left-most child)**, так как мы знаем, что он самый меньший нод.

![Removal case 3](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_Removing_Case3.JPG)

### Binary Tree Traversal

Существует три порядка посещения дерева:

1) **Pre-Order**:

![Pre Order](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_PreOrder_Traversal.JPG)
  
2) **In-Order**. Этот порядок дает возможность посещать дерево в порядке сортировки. То есть значения возврящаются в отсортированном виде. Это наиболее часто применяемый порядок:

![In Order](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_InOrder_Traversal.JPG)

3) **Post-Order**. Этот порядок используется для расчета математических выражений или, например, компилятором при парсинге исходного кода и построении порядка вызова команд:

![Post Order](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/AlgorithmsAndDataStructures/BinaryTree_PostOrder_Traversal.JPG)

