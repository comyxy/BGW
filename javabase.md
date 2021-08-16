# 一些基础

## Java 里获取一个对象的方法？

1. 使用new创建对象
2. 通过反射获取类，newInstance()方法。
3. object 的 clone 方法
4. 反序列化
5. Integer 缓存池
  
## 接口与抽象类的区别？

1. 接口只能有方法定义，不能有方法实现，1.8 后可定义 default 方法体实现。
2. 实现接口的关键字为 implements， 一个类可实现多个接口；继承抽象类的关键字为 extends， 一个类只可继承一个抽象类。
3. 接口成员变量默认 public static final，方法默认 public abstract；抽象类成员变量可以任意访问权限，抽象方法被 abstract 修饰，不能被 private、static、native、synchronized 修饰，可以存在非抽象的任意访问权限的方法。
  
## 说一下常见异常？

* Throwable
  * Exception
    * IOException
      * FileNotFoundException
      * UnsupportedEncodingException
    * SQLException
    * RuntimeException
      * NullPointerException
      * IndexOutOfBoundsException
    * ReflectiveOperationException
      * NoSuchMethodException
      * ClassNotFoundException
      * NoSuchFieldException
  * Error
    * OOM
    * StackOverflow
  
## 集合

### HashMap 的 put 过程？

1. 取哈希桶数组 Node[] tab，若为空，则调用 resize() 初始化桶数组。
2. 根据 hash 值定位桶节点 tab[index]（桶大小 2^n - 1）。
   * 若为空，则新建节点 Node，继续步骤 3。
   * 若不为空，则：
      * 查看第一个节点是不是就是 key，是则记录该节点。
      * 否则判断该节点是不是红黑树节点，是则进行红黑树的插入操作。
      * 否则逐个遍历链表，同时记录遍历个数。
      * 当遍历到 null，则在此位置新建节点，并根据遍历个数来决定是否转换成红黑树（树化阈值8 返回链表6）。
      * 当遍历到 key 本身，记录该节点。
      * 若有节点被记录，说明已经存在了该键，更新并返回旧值，否则继续步骤 3。
3. 更新 size，判断是否需要扩容，返回 null。
  
### HashMap 的 get 过程？

流程大致 同 put：判桶空->判第一个节点->判是否为红黑树节点->遍历链表/红黑树。
  
### 讲讲 ArrayList？

动态数组，底层是 `Array(Object[])`。按秩查找时间复杂度为 `O(1)`，内部插入平均 `O(n)`。初始长度为 `0`，第一次插入数据扩容为 `10`，每次扩容最小 1.5 倍。最大长度为 `Integer.MAX_VALUE - 8`，部分VM会利用这8字节存储一些信息，可突破这个上限达到 `Integer.MAX_VALUE`，但对于一些 VM 可能报 OOM。
  
### ArrayList 是否线程安全？有什么线程安全的数组？如何让 ArrayList 变成线程安全的？

线程不安全。add 方法会在秩为 size 的位置赋值，再更新 size；或在 index 处 copy，再赋值，再更新 size，这一系列操作不是原子的，多线程下可能出现数据覆盖。size 的非 cas 更新也无法保证多线程下 size 的准确性。Vector，但由于是早期容器，功能冗余，锁重量，已被弃用。
使用 Collections.synchronizedList(List l) 来使 list 线程安全；使用 juc 包下的 CopyOnWriteArrayList。
  
## 注解

### 注解的原理？

注解的本质是继承了 Annotation 接口的一个**接口**。

1. 创建注解，编译器检查注解位置合法性与作用域，视情况写入 class 文件的类的属性表中。
2. 若是运行时注解，则可通过反射机制调用。AnnotationInvocationHandler 类用于处理注解，使用反射时，会实例化 Handler，将注解的属性的键值对（方法名-返回值，~~明明是属性，却偏要写成一个接口方法~~）存入 map，这个 map 被 一个 Handler 实例持有。
3. 虚拟机为注解接口创建一个动态代理类。反射方法（如 getAnnotations()）实际上是访问到了这个代理类，进一步调用类的各种方法。
4. 动态代理类通过 Handler 实例获得属性值。
  
## Java 8 特性？

1. 接口默认方法
   * 此默认方法无法被 Lambda 表达式重写
2. Lambda 表达式
   * 使用外部局部变量，变量不必显式声明为 final，但这个变量必须和 final 一样不能被修改
3. 函数式接口
   * 只有一个抽象方法的接口
   * 实现类可以和 Lambda 表达式互相转换
4. 方法/构造器引用
   * 类::new
   * 等价于 Lambda 表达式
5. 内置函数接口
   * `Predicate<T>`，**断言型**接口，接收 T，返回布尔值。内有多个默认方法将接口转换为不同的逻辑
   * `Function<T, R>`，接收 R，返回 T。内有多个默认方法来返回多个 Function 的串联
   * `Supplier<T>`，生产者，无参数，返回 T
   * `Consumer<T>`，消费者，参数为 T，无返回值。内有默认方法来复用参数，串联多个 Consumer
   * `Comparator<T>`，比较器。内有默认方法来串联多个 Comparator（本比较器比较出相等后继续执行其他比较器）；取反逻辑比较器等
6. Optional 类，内部处理了空指针。
7. Streams。
   * 源
     * collection.stream()，Arrays.stream(arr)
     * parallelStream()
   * 中间操作
     * filter(predicate)
     * sorted(comparator)
     * map(function)
     * distinct()
     * ...
   * 结束操作
     * foreach(consumer)
     * count()
     * reduce(...)
8. Date API，加入了时区等。
9. 多重注解，同一注解多次使用，会自动生成集合注解。
  
## 部分参考

<https://www.cnblogs.com/yangming1996/p/9295168.html>
