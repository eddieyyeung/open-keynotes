# Concurrent (to be continued)

## Compare-and-swap

[compare-and-swap](https://en.wikipedia.org/wiki/Compare-and-swap) (CAS) 是一个用于多线程来实现同步的原子操作。CAS 会将内存位置上的值跟给定的值进行比较，只有相等的时候，才会将内存位置上的内容更新为给定的值，整个操作是一个**原子性操作**。如果期间被其他线程更改了，会发生比较后出现值不一样的情况，则更新会失败。

[ABA 问题](https://en.wikipedia.org/wiki/ABA_problem)
