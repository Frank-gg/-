## PriorityQueue

### 概念

PriorityQueue：一个基于优先级的无界优先级队列。优先级队列的元素按照其**自然顺序**进行排序，或者根据构造队列时提供的 **Comparator** 进行排序，具体取决于所使用的构造方法。该队列不允许使用 null 元素，也不允许插入不可比较的对象（没有实现Comparable接口的对象）。

PriorityQueue 队列的头：指排序规则最小或最大那个元素，默认自然排序是最小元素。如果多个元素都是最小  / 大值则随机选一个。

PriorityQueue 是一个无界队列，但是初始的容量（实际是一个Object[]），随着不断向优先级队列添加元素，其容量会自动扩容，无需指定容量增加策略的细节。

![img](https://upload-images.jianshu.io/upload_images/807144-b61b8bc5b4557d5c?imageMogr2/auto-orient/strip|imageView2/2/w/451/format/webp)

### 基本使用

PriorityQueue使用跟普通队列一样，唯一区别是PriorityQueue会根据排序规则决定谁在队头，谁在队尾。

> 举例1：往队列中添加可比较的对象String
>
> ```java
> public class App {
>     public static void main(String[] args) {
>         PriorityQueue<String> q = new PriorityQueue<String>();
>         //入列
>         q.offer("1");
>         q.offer("2");
>         q.offer("5");
>         q.offer("3");
>         q.offer("4");
> 
>         //出列
>         System.out.println(q.poll());  //1
>         System.out.println(q.poll());  //2
>         System.out.println(q.poll());  //3
>         System.out.println(q.poll());  //4
>         System.out.println(q.poll());  //5
>     }
> }
> ```
>
> 观察打印结果， 入列：12534， 出列是12345， 也是说出列时做了相关判断，将最小的值返回。默认情况下PriorityQueue使用自然排序法，最小元素先出列。

> 举例2：自定义排序规则
>
> ```java
> public class Student {
>     private String name;  //名字
>     private int score;    //分数
>    //省略getter/setter
> }
> ```
>
> ```java
> public class App {
>     public static void main(String[] args) {
>         //通过改造器指定排序规则
>         PriorityQueue<Student> q = new PriorityQueue<Student>(new Comparator<Student>() {
>             public int compare(Student o1, Student o2) {
>                 //按照分数低到高，分数相等按名字
>                 if(o1.getScore() == o2.getScore()){
>                     return o1.getName().compareTo(o2.getName());
>                 }
>                 return o1.getScore() - o2.getScore();
>             }
>         });
>         //入列
>         q.offer(new Student("dafei", 20));
>         q.offer(new Student("will", 17));
>         q.offer(new Student("setf", 30));
>         q.offer(new Student("bunny", 20));
> 
>         //出列
>         System.out.println(q.poll());  //Student{name='will', score=17}
>         System.out.println(q.poll());  //Student{name='bunny', score=20}
>         System.out.println(q.poll());  //Student{name='dafei', score=20}
>         System.out.println(q.poll());  //Student{name='setf', score=30}
>     }
> }
> ```

PriorityQueue优先级规则可以由我们根据具体需求而定制， 方式有2种：

- 添加元素自身实现了Comparable接口，确保元素是可排序的对象

- 如果添加元素没有实现Comparable接口，可以在创建PriorityQueue队列时直接指定比较器Comparator。

  ```java
  PriorityQueue<Student> q = new PriorityQueue<Student>(new Comparator<Student>() {
      public int compare(Student o1, Student o2) {
          //按照分数低到高，分数相等按名字
          if(o1.getScore() == o2.getScore()){
              return o1.getName().compareTo(o2.getName());
          }
          return o1.getScore() - o2.getScore();
      }
  });
  ```

### 源码解析

PriorityQueue 是怎么实现优先级队列的呢？

```java
public class PriorityQueue<E> extends AbstractQueue<E>
    implements java.io.Serializable {
    transient Object[] queue;    //队列容器， 默认是11
    private int size = 0;  //队列长度
    private final Comparator<? super E> comparator;  //队列比较器， 为null使用自然排序
    //....
}
```

#### 入列

```java
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1);      //当队列长度大于等于容量值时，自动拓展
    size = i + 1;
    if (i == 0)
        queue[0] = e;
    else
        siftUp(i, e); //
    return true;
}
```

```java
private void siftUp(int k, E x) {
    if (comparator != null)
        siftUpUsingComparator(k, x);   //指定比较器
    else
        siftUpComparable(k, x);   //没有指定比较器，使用默认的自然比较器
}
```

```java
private void siftUpComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>) x;
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
        if (key.compareTo((E) e) >= 0)
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = key;
}
```

```cpp
private void siftUpUsingComparator(int k, E x) {
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
        if (comparator.compare(x, (E) e) >= 0)
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = x;
}
```

这里自作简单比较，使用选择排序法将入列的元素放左边或者右边。

从源码上看PriorityQueue的入列操作并没对所有加入的元素进行优先级排序。仅仅保证数组第一个元素是最小的即可。

#### 出列

```cpp
public E poll() {
    if (size == 0)
        return null;
    int s = --size;
    modCount++;
    E result = (E) queue[0];
    E x = (E) queue[s];
    queue[s] = null;
    if (s != 0)
        siftDown(0, x);
    return result;
}
```

```csharp
private void siftDown(int k, E x) {
    if (comparator != null)
        siftDownUsingComparator(k, x);  //指定比较器
    else
        siftDownComparable(k, x);    //默认比较器
}
```

```cpp
private void siftDownComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>)x;
    int half = size >>> 1;        // loop while a non-leaf
    while (k < half) {
        int child = (k << 1) + 1; // assume left child is least
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
            c = queue[child = right];
        if (key.compareTo((E) c) <= 0)
            break;
        queue[k] = c;
        k = child;
    }
    queue[k] = key;
}

@SuppressWarnings("unchecked")
    private void siftDownUsingComparator(int k, E x) {
    int half = size >>> 1;
    while (k < half) {
        int child = (k << 1) + 1;
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            comparator.compare((E) c, (E) queue[right]) > 0)
            c = queue[child = right];
        if (comparator.compare(x, (E) c) <= 0)
            break;
        queue[k] = c;
        k = child;
    }
    queue[k] = x;
}
```

上面源码，当第一个元素出列之后，对剩下的元素进行再排序，挑选出最小的元素排在数组第一个位置。

通过上面源码，也可看出PriorityQueue并**不是线程安全队列**，因为offer/poll都没有对队列进行锁定，所以，如果要拥有线程安全的优先级队列，需要额外进行加锁操作。

### 总结

- PriorityQueue是一种无界的，线程不安全的队列
- PriorityQueue是一种通过数组实现的，并拥有优先级的队列
- PriorityQueue存储的元素要求必须是可比较的对象， 如果不是就必须明确指定比较器

------

------

## Java中比较器的使用

### Comparable与Comparator的区别

- Comparable 和 Comparator：都是用来实现集合中元素的比较、排序的。

- Comparable 是在**集合内部**定义的方法实现的排序，Comparator 是在**集合外部**实现的排序。

- 想实现排序，就需要在集合外定义`Comparator`接口的方法`compare()`或在集合内实现 `Comparable`接口的方法`compareTo()`，Comparator位于包java.util下，而Comparable位于包java.lang下。

- 两种方法各有优劣， 用Comparable 简单， 只要实现Comparable 接口的对象直接就成为一个可以比较的对象，但是需要修改源代码。 用Comparator 的好处是不需要修改源代码， 而是另外实现一个比较器， 当某个自定义的对象需要作比较的时候，把比较器和对象一起传递过去就可以比大小了， 并且在Comparator 里面用户可以自己实现复杂的可以通用的逻辑，使其可以匹配一些比较简单的对象。

### 两种比较器的实现

#### 实现java.lang.Comparable接口

第一种是实现java.lang.Comparable接口，使你的类天生具有比较的能力，此接口很简单，只有一个`compareTo`一个方法。若一个类实现了Comparable接口，就意味着该类支持排序。实现了Comparable接口的类的对象的列表或数组可以通过**Collections.sort**或**Arrays.sort**进行自动排序。

此外，实现此接口的对象可以用作有序映射中的键或有序集合中的集合，无需指定比较器。该接口定义如下：

```java
package java.lang;
import java.util.*;
public interface Comparable<T> {
    public int compareTo(T o);
}
```

`x.compareTo(y)` ：比较x和y的大小。若返回“负数”，意味着“x比y小”；返回“零”，意味着“x等于y”；返回“正数”，意味着“x大于y”。

> 举例：
>
> ```java
> class Person implements Comparable<Person> {
>     @Override
>      public int compareTo(Person person) {
>           return name.compareTo(person.name);
>           //return this.name - person.name;
>      }
> }
> ArrayList<Person> list = new ArrayList<Person>();
> // 添加对象到ArrayList中
> list.add(new Person("aaa", 10));
> list.add(new Person("bbb", 20));
> list.add(new Person("ccc", 30));
> list.add(new Person("ddd", 40));
> Collections.sort(list); //这里会自动调用Person中重写的compareTo方法。
> ```

#### 实现java.util.Comparator接口

Comparator是比较接口，我们如果需要控制某个类的次序，而该类本身不支持排序（即没有实现Comparable接口），那么我们就可以建立一个“该类的比较器”来进行排序，这个“比较器”只需要实现Comparator接口即可。也就是说，我们可以通过实现Comparator来新建一个比较器，然后通过这个比较器对类进行排序。该接口定义如下：

```java
package java.util;
public interface Comparator<T>
 {
    int compare(T o1, T o2);
    boolean equals(Object obj);
 }
```

> 举例：
>
> ```java
> public class ComparatorDemo {
>     public static void main(String[] args) {
>         List<Person> people = Arrays.asList(
>                 new Person("Joe", 24),
>                 new Person("Pete", 18),
>                 new Person("Chris", 21)
>         );
>         Collections.sort(people, new LexicographicComparator());
>         System.out.println(people);
>         //[{name=Chris, age=21}, {name=Joe, age=24}, {name=Pete, age=18}]
>         Collections.sort(people, new Comparator<Person>() {
>             @Override
>             public int compare(Person a, Person b) {
>                 // TODO Auto-generated method stub
>                  return a.age < b.age ? -1 : a.age == b.age ? 0 : 1;
>             }
>         });
>         System.out.println(people);
>         //[{name=Pete, age=18}, {name=Chris, age=21}, {name=Joe, age=24}]
>     }
> }
> 
> // 比较器
> class LexicographicComparator implements Comparator<Person> {
>     @Override
>     public int compare(Person a, Person b) {
>         return a.name.compareToIgnoreCase(b.name);
>     }
> }
> 
> class Person {
>     String name;
>     int age;
>     Person(String n, int a) {
>         name = n;
>         age = a;
>     }
>     @Override
>     public String toString() {
>         return String.format("{name=%s, age=%d}", name, age);
>     }
> }
> ```

