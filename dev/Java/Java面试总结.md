## Java基础
* ### 1. for()和foreach()区别
  * 如果只是遍历集合或者数组，用foreach好些，快些。
  * 如果对集合中的值进行修改，就要用for循环了。其实foreach的内部原理其实也是Iterator,但它不能像Iterator一样可以人为的控制，而且也不能调用iterator.remove()；更不能使用下标来访问每个元素，所以不能用于增加，删除等复杂的操作。
## Java集合
* ### 1. list如何边遍历边删除
    list边遍历边删除有三种方式，原理是modCount 和 expectedModCount 的值相等，则不会抛出并发修改异常（foreach循环只在删除倒数第二个元素时，不会抛并发修改异常）：

        a: 使用 Iterator 的 remove() 方法
        b: 使用for 循环正序遍历（注意每删除一次，需要修正下标的值）
        c: 使用 for 循环倒序遍历
* ### 2. hashmap, hashtable,ConcurrentHashMap 

   * hashmap：底层数组+链表实现，可以存储null键和null值，线程不安全。**初始size为16，扩容：newsize = oldsize*2，size一定为2的n次幂。** 扩容针对整个Map，每次扩容时，原来数组中的元素依次重新计算存放位置，并重新插入。插入元素后才判断该不该扩容，有可能无效扩容（插入后如果扩容，如果没有再次插入，就会产生无效扩容）。当Map中元素总数超过Entry数组的75%，触发扩容操作，为了减少链表长度，元素分配更均匀
   * hashtable：底层数组+链表实现，无论key还是value都不能为null，线程安全，实现线程安全的方式是在修改数据时**锁住整个HashTable**，效率低，ConcurrentHashMap做了相关优化 。**初始size为11，扩容：newsize = olesize*2+1**
   * ConcurrentHashMap：底层采用分段的数组+链表实现，线程安全。通过把整个Map分为N个Segment，可以提供相同的线程安全，但是效率提升N倍，**默认提升16倍**。**(读操作不加锁，由于HashEntry的value变量是 volatile的，也能保证读取到最新的值。)** Hashtable的synchronized是针对整张Hash表的，即每次锁住整张表让线程独占，ConcurrentHashMap允许多个修改操作并发进行，其关键在于使用了锁分离技术，有些方法需要跨段，比如size()和containsValue()，它们可能需要锁定整个表而而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放所有段的锁。扩容：段内扩容（段内元素超过该段对应Entry数组长度的75%触发扩容，不会对整个Map进行扩容），插入前检测需不需要扩容，有效避免无效扩容
* ### 3. Set无序的理解
  * hashSet无序不可重复，既不能保证添加元素的顺序，也不能保证元素的自然顺序
  * LinkedHashset : 保证元素添加的自然顺序
  * TreeSet : 保证元素的自然顺序，即a-z的顺序
    * TreeSet实现有序的方式：
        1.  对于基本的数据类型。比如Integer或者是String ，在你把数据添加进去之后，TreeSeth会按照这个类的compareTo方法进行排序
        2.  其他数据类型需要implements Comparator接口中的方法compare  或者 implements Comparable接口 然后重写其中的compareTo方法 ，指定出你想要的排序规则
        3.  在对TreeSet进行实例化的时候，在构造函数中 通过匿名内部类 重写其中的compare方法，在其中指定出具体的排序方法


Eg:
  ```java
      Set<String> set = new HashSet<>();
		set.add("String1");
		set.add("String4");
		set.add("String3");
		set.add("String2");
		set.add("String5");
		set.forEach(e-> System.out.print(e+" "));
		System.out.println();
		
		
		//LinkedHashSet会保证元素的添加顺序
		Set<String> set2 = new LinkedHashSet<>();
		set2.add("String1");
		set2.add("String5");
		set2.add("String3");
		set2.add("String4");
		set2.add("String2");
		set2.forEach(e-> System.out.print(e+" "));
		System.out.println();
		
		
		//TreeSet保证元素自然顺序
		Set<String> set3 = new TreeSet<>();
		set3.add("String1");
		set3.add("String5");
		set3.add("String4");
		set3.add("String2");
		set3.add("String3");
		set3.forEach(e-> System.out.print(e+" "));

    // 输出结果：

    String5 String4 String3 String2 String1       HashSet元素乱序
    String1 String5 String3 String4 String2       LinkedHashSet保证元素添加顺序

    String1 String2 String3 String4 String5       TreeSet元素按自然顺序排序
  ```


* ### 4. 
## JDBC
* ### 1. JDBC使用步骤
```
    1. 注册驱动
    2. 获取连接
    3. 获取预处理对象
    4. SQL语句占位符设置实际参数
    5. 执行SQL语句
    6. 释放资源
```
