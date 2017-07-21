# ExceptionReadme
### 总结项目中出现的异常

## java.util.ConcurrentModificationException
>**异常描述**：该异常表示迭代器迭代过程中，迭代的对象发生了改变，如数据项增加或删除.
>com.czcg.gwt.util.LogOut.exitAll(LogOut.java:72)

>**解决思路**：由于迭代对象不是线程安全，在迭代的过程中，会检查 modCount 是否和初始 modCount 即 expectedModCount 一致，如果不一致，则认为数据有变化，迭代终止并抛出异常。常出现的场景是，两个线程同时对集合进行操作，线程 1 对集合进行遍历，而线程 2 对集合进行增加、删除操作，此时将会发生 ConcurrentModificationException 异常。

>**解决方法**：多线程访问时要增加同步锁，或者建议使用线程安全的集合：
>
1. 使用 ConcurrentHashMap 替换 HashMap，CopyOnWriteArrayList 替换 ArrayList。
2. 或者使用使用 Vector 替换 ArrayList，Vector 是线程安全的。Vector 的缺点：大量数据操作时，由于线程安全，性能比 ArrayList低。


