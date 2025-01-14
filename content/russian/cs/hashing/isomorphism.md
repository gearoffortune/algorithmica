---
title: Проверки на изоморфизм
weight: 4
authors:
- Александр Гришутин
- Сергей Слотин
date: 2021-08-25
---

Абстрактно, *изоморфизмом* называют обратимое отображение (биекцию) между двумя множествами, наделенными какой-то структурой, с сохранением этой структуры.

Например, два графа называются изоморфными, если вершинам одного графа можно сопоставить вершины другого графа, так чтобы соединённым вершинам первого графа соответствовали соединённые вершины второго графа и наоборот. Иными словами, два графа изоморфны, если они «одинаковы» с точностью до переименования вершин.

Хороший общий способ выполнять проверки на изоморфизм — посчитать от них какую-то хеш-функцию, значение которой не изменяется от переименования объектов, и сравнить её значение.

## Множества

Пусть мы хотим научиться сравнивать множества чисел или строк на равенство с
точностью до перестановки.

Например:

$$
   \{1, 2, 3\} = \{2, 1, 3\}
\\ \{"lal", "abc"\} = \{"abc", "lal"\}
\\ \{4, 5\} \neq \{5, 6\}
\\ \{"lal", "abc"\} \neq \{"abc", "all"\}
$$

Эту задачу можно решить, если отсортировать это множество и посчитать полиномиальный хеш от получившейся последовательности. Результат не будет зависеть от изначальной перестановки элементов, и таким образом мы решим задачу за $O(n \log n)$. Но задачу можно решить и за линейное время.

Сопоставим всем возможным ключам различные 64-битные случайные числа (будем называть их хешом ключа), а хешом множества тогда будем считать `xor`-сумму хешей всех входящих в него ключей. Тогда хеши равных с точностью до перестановки множеств будут одинаковые из-за коммутативности `xor`, а вероятность коллизии будет в точности равна $\frac{1}{2^{64}}$.

Сопоставить хеши ключам можно либо хеш-функцией, либо, если количество различных значений ключей невелико, предпосчитанной таблицей случайных чисел. В обоих случаях подсчет хеша множества потребует подсчет $O(n)$ хешей ключей.

Этот метод также примечатален тем, что пересчет хешей от множеств можно проводить за $O(1)$ при определенных изменениях, таких как добавление элемента в множество, удаление элемента из множества или слияние двух непересекающися множеств.

## Игры

Последний метод можно обобщить для хеширования позиций в абстрактных играх вроде шахмат или шашек.

Быстрый способ пересчета хеша позиций критически важен для реализации компьютерных стратегий для этих игр, потому что большинство из них основываются на переборе всех возможных ходов, который будет работать быстрее, если запоминать результаты и не запускать перебор дважды из одной и той же вершины. Для этого нужно завести хеш-таблицу, в которую нужно складывать хеши и посчитанные результаты от посещенных позиций, для чего в свою очередь нужно уметь их быстро пересчитывать.

![Дерево дебютов «сициалианской защиты». В некоторые дебюты можно попасть более чем одним способом](../img/sicilian.png)

На примере шахмат, у нас есть $8 \times 8 = 64$ поля, каждое из которых может быть в одном из $6 \times 2 + 1 = 13$ возможных состояний (на нем может стоять король, ферзь, ладья, конь, слон или пешка одного из двух цветов или ничего). Сгенерируем $64 \times 13 = 832$ случайных 64-битных чисел — каждое будет соответствовать факту, что на такой-то позиции стоит такая-то фигура. Теперь пройдемся по всем полям и посчитаем `xor`-сумму. Получившееся 64-битное число и будет хешом позиции.

Помимо того, что его легко посчитать (64 раз посмотреть в lookup-таблицу быстрее, чем считать полиномиальные хеши по модулю), его ещё можно очень быстро пересчитывать, если изменение позиции было небольшим. А именно, если сделан один ход, то будут затронуты всего две ячейки, и хеш позиции нужно проксорить с ровно 4 числами (двумя старыми, чтобы их удалить, и двумя новыми, чтобы их добавить).

С поправкой на возможность рокировки, взятие на проходе, симметрию цветов, невозможность поместить пешки на первый и последний ряд, и несколько других шахматных нюансов, мы описали то, что называется [хешированием Зобриста](https://en.wikipedia.org/wiki/Zobrist_hashing).

## Подматрицы

**Задача.** Дана матрица $A_{i, j}$ размера $n \times m$, состоящая из чисел. Требуется отвечать на запросы, равны ли подматрицы $A_{x_1:x_2, y_1:y_2}$ и $A_{x_3:x_4, y_3:y_4}$.

Если посмотреть на матрицу как на набор строк, то сравнивать подматрицы это то же самое, что сравнивать набор подстрок на равенство, что мы уже умеем делать хешами. Если предпосчитать массив полиномиальных хешей от каждой строки матрицы и сравнивать хеши за константу, то каждый запрос будет работать за $O(n)$, что не очень быстро, но хотя бы не $O(n^2)$.

### Двумерный хеш

Давайте обобщим идею полиномиального хеширования строк на матрицы. Для этого нам понадобится два разных $k$ — «горизонтальный» $k_h$ и «вертикальный» $k_v$. Положим теперь

$$
h(A_{x_1:x_2, y_1:y_2})
=
\sum_{i=x_1}^{x_2}
\sum_{j=y_1}^{y_2}
A_{i,j} \cdot k_h^i \cdot k_v^j
$$

Предподсчитаем такие хеши для всех *угловых* подматриц (обобщение понятия префикс строки):

$$
h(x, y) = h(A_{:x, :y})
$$

Заметим, что для всех угловых подматриц, которые содержат только первую строку или первый столбец, определение хеша совпадает с определением полиномиального хеша для строки, а значит такие хеши мы умеем предподсчитать за линию.

Теперь заметим, что для всех остальных подматриц

$$
h(x, y) = A_{x, y} k_h^x k_v^y + h(x - 1, y) + h(x, y - 1) - h(x - 1, y - 1)
$$

Таким образом, мы можем посчитать хеш за $O(n^2)$ таким же методом, как для двумерных префиксных сумм.

Теперь мы можем выполнять проверку на равенство за $O(1)$, считая хеш от подматриц через префиксные суммы. Единственным нюансом остается то, что этот хеш будет «сдвинут» на $k_h^{x_1} \cdot k_v^{y_1}$ — это можно исправить, либо поделив по модулю на него, либо домножив хеш от подматриц на $k_h^{n-x_1} \cdot k_v^{m-y_1}$.

**Примечание.** Можно определять не два разных $k_h$ и $k_v$, а одно, если мысленно соединить все строки матрицы в одну. Это эквивалентно тому, что сделать $k_h = k_v^m$.

## Корневые деревья

Теперь мы хотим научиться проверять корневые деревья на изоморфизм, то есть на равенство с точностью до перенумерования вершин, при том, что известно, что корень одного дерева обязательно переходит в корень другого дерева.

### Хеш от вершины

Заметим, что поскольку мы не можем аппелировать к номерам вершин, единственная информация, которую мы можем использовать — это структура нашего дерева.

Положим тогда хешем вершины без детей какую-нибудь константу (например, 179), а для вершины с детьми положим в качестве хеша некоторую функцию от отсортированного (поскольку мы не знаем истинного порядка, в котором дети должны идти, нужно привести их к одинаковому виду) списка хешей детей.

Хешом корневого дерева будем считать хеш корня. Индукцией построению можно показать, что у изоморфных корневых деревьев хеши совпадают.

Единственный нюанс заключается в том, что стандартный полиномиальный хеш для аггрегации хешей от детей не подходит. Например, рассмотрим 2 дерева:

$$
\begin{aligned}
   T_1 &= \{ (1, 2), (1, 3), (1, 4) \}
\\ T_2 &= \{ (1, 2), (1, 3), (3, 4), (3, 5) \}
\end{aligned}
$$

Если в качестве функции от детей взять полиномиальный хеш, то получим:

$$
h(T_1) = 179 + 179p + 179p^2 = 179 + p(179 + 179p) = h(T_2)
$$

В качестве неплохой хеш-функции подойдёт, например

$$
h(v) = 42 + \sum_u \log(h(u))
$$

Для этой хеш-функции может показаться, что можно не сортировать хеши детей, однако это не так, потому что при вычислении чисел с плавающей точкой у нас возникает погрешность, и чтобы это результат суммирования был одинаковый для изоморфных деревьев, суммировать детей надо тоже в одинаковом порядке.

Пример более сложной хеш-функции:

$$
h(v) = \sum_u h(u)^2 + h(u) \cdot p^i + 42 \bmod m $$

В любом случае, асимптотика будет $O(n \log n$, так как всё что нам нужно делать в каждой вершине — это сортировка детей по хешу и суммирование. Так как у нас могут быть два изоморфных ребенка с одинаковым хешом, трюки с `xor`- и другими суммами не работают, и поэтому быстрее нельзя.

### Некорневые деревья

Для некорневых деревьев можно найти центроид и запуститься от него как от корневого. Если центроидов два, то можно запуститься от двух центроидов, посчитать два хеша, упорядочить (свапнуть если `hash_a` меньше `hash_b`) и посчитать мини-хеш от них (`hash = hash_a + k * hash_b`).
