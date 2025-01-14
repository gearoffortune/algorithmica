---
title: Связность и остовные деревья
weight: 22
---

Рассмотрим следующую задачу:

> Авиакомпания содержит $m$ рейсов между $n$ городами, $i$-ый из них обходится в $w_i$ рублей, причём из любого города можно добраться до любого другого. В стране наступил кризис, и нужно отказаться от как можно большего числа из них таким образом, что содержание оставшиъся рейсов будет наиболее дешевым.

Иными словами, нужно найти дерево минимального веса, которое является подграфом данного неориентированного графа. Почему дерево? Потому что в противном случае там был бы цикл, из которого можно удалить какое-то ребро и получить более оптималный ответ. А если это больше, чем одно дерево, то какие-то две вершины остаются несвязны.

Такие деревья называют *остовами* (*каркас*, *скелет*; ударение на первый слог, но так мало кто произносит; англ. *minimum spanning tree* — дословно, минимальное покрывающее дерево).

В этой статье мы рассмотрим алгоритмы нахождения остовных деревьев и классические задачи на них.
