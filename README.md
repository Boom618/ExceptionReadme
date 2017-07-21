# ExceptionReadme
### 总结项目中出现的异常

## java.util.ConcurrentModificationException
>**异常描述**：该异常表示并发修改异常.迭代器迭代过程中，迭代的对象发生了改变，如数据项增加或删除.
>com.czcg.gwt.util.LogOut.exitAll(LogOut.java:72)

>**解决思路**：由于迭代对象不是线程安全，在迭代的过程中，会检查 modCount 是否和初始 modCount 即 expectedModCount 一致，如果不一致，则认为数据有变化，迭代终止并抛出异常。常出现的场景是，两个线程同时对集合进行操作，线程 1 对集合进行遍历，而线程 2 对集合进行增加、删除操作，此时将会发生 ConcurrentModificationException 异常。

>**解决方法**：多线程访问时要增加同步锁，或者建议使用线程安全的集合：

>1. 使用 ConcurrentHashMap 替换 HashMap，CopyOnWriteArrayList 替换 ArrayList。
>2. 或者使用使用 Vector 替换 ArrayList，Vector 是线程安全的。Vector 的缺点：大量数据操作时，由于线程安全，性能比 ArrayList低。

## java.lang.ArrayIndexOutOfBoundsException
>**异常描述**：该异常表示数组越界.
>
>**解决思路**：这种情况一般要在数组循环前做好 length 判断，index 超出 length 上限和下限时都会报错。举例如下：一个数组 int test[N]，一共有 N 个元素分别是test[0]~test[N-1]，如果调用 test[N]，将会报错。建议读取时，不要超过数组的长度（array.length）。
Android 中一种常见情形就是上拉刷新中 header 也会作为 listview 的第 0 个位置，如果判断失误很容易造成越界。

>**异常情况**：TextView 中 ellipsize 使用引发 Crash，该问题为 Android 系统 bug，存在于 Android 5.0 及以下设备，[问题描述参考](https://code.google.com/p/android/issues/detail?id=33868)
>
>**解决方法**：使用 android:singleLine="true" 代替 android:lines="1" 和 android:maxLines="1"

## java.lang.IllegalStateException
>**异常描述**：状态异常
>
>**解决思路** 错误类型大致为以下几种:
>
* java.lang.IllegalStateException：Cannot forward a response that is   already committed(无法转发已经提交的响应)
* IllegalStateException：response already commited(响应已经提交)
* IllegalStateException：getOutputStream() has already been called for this request(已经为此请求调用了getOutputStream )
> ...............
> 
>例1：IllegalStateException: Can not perform this action after onSaveInstanceState(在onSaveInstanceState之后无法执行此操作)

>**错误原因**：
>该异常表示，当前对客户端的响应已经结束，不能在响应已经结束（或说消亡）后再向客户端（实际上是缓冲区）输出任何内容。Object is no longer valid to operate on. Was it deleted by another thread?该异常表示，realmObject对象在其他线程已被删除，在这个线程中使用的时候抛出的异常。
>
>**解决方法**：
onSaveInstanceState方法是在该Activity即将被销毁前调用，来保存Activity数据的，如果在保存玩状态后再给它添加Fragment就会出错。解决办法就是把commit（）方法替换成 commitAllowingStateLoss()

>**具体分析**：
>首先解释下flush()，我们知道在使用读写流的时候数据先被读入内存这个缓冲区中， 然后再写入文件，但是当数据读完时不代表数据已经写入文件完毕，因为可能还有一部分仍未写入文件而留在内存中，这时调用 flush() 方法就会把缓冲区的数据强行清空输出，因此 flush() 的作用就是保证缓存清空输出。response 是服务端对客户端请求的一个响应，其中封装了响应头、状态码、内容等，服务端在把 response 提交到客户端之前，会向缓冲区内写入响应头和状态码，然后将所有内容 flush。这就标志着该次响应已经 committed。对于当前页面中已经 committed 的 response，就不能再使用这个 response 向缓冲区写任何东西（注：同一个页面中的 response.XXX()是同一个 response 的不同方法，只要其中一个已经导致了 committed，那么其它类似方式的调用都会导致 IllegalStateException 异常）。
[参考1](http://my.oschina.net/guhai2004/blog/187041)、[参考2](https://github.com/realm/realm-java/issues/1206)

>例2: java.lang.IllegalStateException
Can't change tag of fragment d{e183845 #0 d{e183845}}: was d{e183845} now d{e183845 #0 d{e183845}}

>**错误原因**：
经查，我在显示fragment的代码中使用了fragment.show(getSupportFragmentManager, fragment.toString())
而这里是因为两次 toString()结果不同，导致不同的 tag 指向的是同一个 fragment。

>**解决方法**：
获取 fragment 的 tag 的正确方法应该是使用其提供的 fragment.getTag()方法。

## java.lang.NullPointerException
>**异常描述**：空指针
>
>**解决思路**：使用了一个空对象引用，建议您检查引用的对象是否为空
>
>**解决方法**：这种异常通常是调用一个对象的方法抛出的，凡是调用一个对象的方法之前，一定要进行判空或者进行try-catch，这样基本可以规避大部分空指针异常。

## java.lang.IllegalArgumentException
>**异常描述**：参数不匹配异常，通常由于传递了不正确的参数导致。
>
>**解决思路**：检测传入的参数
>
>**解决方法**：常见于：
>
* Activity、Service状态异常
* 非法URL
* UI线程操作
* Fragment中嵌套了子Fragment，Fragment 被销毁，而内部 Fragment 未被销毁，所以导致再次加载时重复，在 onDestroyView() 中将内部 Fragment 销毁即可
* 在请求网络的回调中使用了glide.into(view),view 已经被销毁会导致该错误