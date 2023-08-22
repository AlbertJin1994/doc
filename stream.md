# Stream流

## 1、常用操作

### 1.1、创建流

#### 1.1.1、单列集合

```java
List<Author> authors = getAuthors();
Stream<Author> stream = authors.stream();
```

#### 1.1.2、数组

```java
Integer[] arr = {1, 2, 3, 4, 5,};
Stream<Integer> stream1 = Arrays.stream(arr);
Stream<Integer> Stream2 = Stream.of(arr);
```

#### 1.1.3、双列集合

转换成单列集合后才能创建Stream对象：

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
map.put("C", 3);
Stream<Map.Entry<String, Integer>> stream3 = map.entrySet().stream();
```

### 1.2、中间操作

#### 1.2.1、filter

打印所有姓名长度大于1的作家的姓名：

```java
List<Author> authors = getAuthors();
authors.stream()
        .filter(x -> x.getName().length() > 1)
        .forEach(System.out::println);
```

#### 1.2.2、map

可对流中的元素进行计算或转换，提取你想要的数据（可看作重新装配或构造）。

第一个参数必须与流中成员同类型，第二个参数则是想提取的目标数据的类型。

```java
List<Author> authors = getAuthors();
authors.stream()
        .map(new Function<Author, Object>() {
            @Override
            public Object apply(Author author) {
                return null;
            }
        }).forEach(System.out::println);
```

打印所有作家的姓名：

```java
List<Author> authors = getAuthors();
authors.stream()
        .map(x -> x.getName())
        .forEach(System.out::println);
```

#### 1.2.3、distinct

distinct方法是依赖Object的equals方法来判断是否是相同对象的，所以需要注意重写equals方法。

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode//用于后期的去重使用
public class Author implements Comparable {
	// ...
}
```

打印所有作家的姓名，并且要求其中不能有重复元素：

```java
List<Author> authors = getAuthors();
authors.stream()
        .distinct()
        .forEach(x-> System.out.println(x.getName()));
```

#### 1.2.4、sorted

如果调用空参的`sorted()`方法，需要流中的元素是实现了`Comparable`接口，重写过排序规则。

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode//用于后期的去重使用
public class Author implements Comparable {
    // ...
    @Override
    public int compareTo(Object o) {
        int age = 0;
        if(o instanceof Author){
            Author author = (Author)o;
            age = author.getAge();
        }
        return age -this.age;
    }
}
```

对流中的元素按照年龄进行降序排序，并且要求不能有重复的元素：

```java
List<Author> authors = getAuthors();
authors.stream()
        .distinct()
        .sorted((o1, o2) -> o2.getAge() - o1.getAge())
        .forEach(System.out::println);
```

#### 1.2.5、limit

可以设置流的最大长度，超过的部分将被抛弃。

对流中的元素按照年龄进行降序排序，并且要求不能有重复的元素，然后打印其中年龄最大的两个作家的姓名：

```java
List<Author> authors = getAuthors();
authors.stream()
        .distinct()
        .sorted((o1, o2) -> o2.getAge() - o1.getAge())
        .limit(2)
        .forEach(x -> System.out.println(x.getName()));
```

#### 1.2.6、skip

跳过流中的前n个元素，返回剩下的元素。

打印除了年龄最大的作家外的其他作家，要求不能有重复元素，并且按照年龄降序排序：

```java
List<Author> authors = getAuthors();
authors.stream()
        .distinct()
        .sorted((o1, o2) -> o2.getAge() - o1.getAge())
        .skip(1l)
        .forEach(System.out::println);
```

#### 1.2.7、flatMap

map只能把一个对象转换成另一个对象来作为流中的元素。

```java
List<Author> authors = getAuthors();
authors.stream()
        .map(new Function<Author, List<Book>>() {
            @Override
            public List<Book> apply(Author author) {
                return author.getBooks();
            }
        })
        .forEach(System.out::println);
```

![image-20230820193109013](./assets/image-20230820193109013.png)

而flatMap可以把一个对象转换成多个对象作为流中的元素。

注意：使用flatMap的时候需要返回一个Stream类型的数据。

```java
List<Author> authors = getAuthors();
authors.stream()
        .flatMap(x -> x.getBooks().stream())
        .forEach(System.out::println);
```

![image-20230820193645649](./assets/image-20230820193645649.png)

打印所有书籍的名字，要求对重复的元素进行去重：

```java
List<Author> authors = getAuthors();
authors.stream()
        .flatMap(x -> x.getBooks().stream())
        .distinct()
        .forEach(x -> System.out.println(x.getName()));
```

打印所有书籍的所有分类，要求对分类进行去重。不能出现这种格式：“哲学,爱请”：

```java
List<Author> authors = getAuthors();
authors.stream()
        .flatMap(x -> x.getBooks().stream())
        .flatMap(x -> Arrays.stream(x.getCategory().split(",")))
        .distinct()
        .forEach(System.out::println);
```

### 1.3、终结操作

#### 1.3.1、forEach

略

#### 1.3.2、count

获取当前流中元素的个数。

打印这些作家的所出书籍的数目，注意删除重复元素：

```java
List<Author> authors = getAuthors();
long count = authors.stream()
        .flatMap(x -> x.getBooks().stream())
        .distinct()
        .count();
System.out.println(count);
```

#### 1.3.3、max

分别获取这些作家的所出书籍的最高分和最低分并打印：

```java
List<Author> authors = getAuthors();
Optional<Integer> max = authors.stream()
        .flatMap(x -> x.getBooks().stream())
        .map(x -> x.getScore())
        .max((x1, x2) -> x1 - x2);
System.out.println(max.get());
```

#### 1.3.4、min

与max同理。

#### 1.3.5、collect

把当前流转换成一个集合。

获取一个存放所有作者名字的List集合：

```java
List<Author> authors = getAuthors();
List<String> list = authors.stream()
        .map(x -> x.getName())
        .collect(Collectors.toList());
System.out.println(list);
```

获取一个所有书名的Set集合：

```java
List<Author> authors = getAuthors();
Set<String> set = authors.stream()
        .flatMap(x -> x.getBooks().stream())
        .map(x -> x.getName())
        .collect(Collectors.toSet());
System.out.println(set);
```

获取一个Map集合，Map的key为作者名，value为书籍集合：

```java
List<Author> authors = getAuthors();
Map<String, Object> map = authors.stream()
        .distinct()
        .collect(Collectors.toMap(new Function<Author, String>() {
            @Override
            public String apply(Author author) {
                return author.getName();
            }
        }, new Function<Author, Object>() {
            @Override
            public List<Book> apply(Author author) {
                return author.getBooks();
            }
        }));
System.out.println(map);
```

#### 1.3.6、查找与匹配

##### 1.3.6.1、anyMatch

可以用来判断是否有任意符合匹配条件的元素，结果为布尔型。

判断是否有年龄在29以上的作家：

```java
List<Author> authors = getAuthors();
boolean match = authors.stream()
        .anyMatch(new Predicate<Author>() {
            @Override
            public boolean test(Author author) {
                return author.getAge() > 29;
            }
        });
System.out.println(match == true ? "有" : "没有");
```

##### 1.3.6.2、allMatch

可以判断流中的元素是否都不符合匹配条件，结果为布尔型。

判断作家是否都没有超过100岁：

```java
List<Author> authors = getAuthors();
boolean match = authors.stream()
        .allMatch(author -> author.getAge() < 100);
System.out.println(match == true ? "都小于100岁" : "有大于100岁的");
```

##### 1.3.6.3、findAny

获取流中的任意一个元素。该方法没有办法保证获取的一定是流中的第一个元素。

获取任意一个年龄大于18的作家，如果存在就输出他的名字：

```java
List<Author> authors = getAuthors();
Optional<Author> any = authors.stream()
        .filter(x -> x.getAge() > 18)
        .findAny();
any.ifPresent(x -> System.out.println(x.getName()));
```

##### 1.3.6.4、findFirst

获取流中第一个元素。

获取一个年龄最小的作家，并输出他的名字：

```java
List<Author> authors = getAuthors();
Optional<Author> first = authors.stream()
        .sorted((o1, o2) -> o1.getAge() - o2.getAge())
        .findFirst();
System.out.println(first.get());
```

#### 1.3.7、reduce

归并 or 缩减 操作。

reduce的作用是把stream中的元素给组合起来。我们可以传入一个初始值，它会按照我们的计算方式依次拿流中的元素和初始值进行计算，计算结果再和后面的元素计算。

reduce中具有3个重载方法，适用于不同场景，第三个方法比较复杂，适合在并行流中使用。

![image-20230821220207517](./assets/image-20230821220207517.png)

求所有作者年龄的和：

```java
List<Author> authors = getAuthors();
Integer result = authors.stream()
        .map(Author::getAge)
	    // .reduce(0, (result1, element) -> result1 + element);
        .reduce(0, new BinaryOperator<Integer>() {
            @Override
            public Integer apply(Integer result, Integer element) {
                return result + element;
            }
        });
System.out.println(result);
```

求所有作者中年龄的最大值：

```java
List<Author> authors = getAuthors();
Optional<Integer> max = authors.stream()
        .map(Author::getAge)
        // .reduce((result, element) -> result < element ? element : result);
        .reduce(new BinaryOperator<Integer>() {
            @Override
            public Integer apply(Integer result, Integer element) {
                return result < element ? element : result;
            }
        });
max.ifPresent(System.out::println);
```

## 2、注意事项

- 惰性求值（如果没有终结操作，没有中间操作是不会得到执行的）；

- 流是一次性的（一旦一个流对象经过一个终结操作后，这个流就不能再被使用）；

- 不会影响数据（我们在流中可以对数据做很多处理，但是正常情况下是不会影响原来集合中的元素的，这也是我们期望的）；

  - 不改变原数据：

    ```java
    List<Author> authors = getAuthors();
    authors.stream()
            .map(x->x.getAge())
            .map(age->age+10)
            .forEach(System.out::println);
    ```

  - 以下就会改变原数据了：

    ```java
    List<Author> authors = getAuthors();
    authors.stream()
            .map(new Function<Author, Author>() {
                @Override
                public Author apply(Author author) {
                    author.setAge(author.getAge() + 10);
                    return author;
                }
            }).forEach(System.out::println);
    ```

# Optional

## 1、概述

我们在编写代码的时候出现最多的就是空指针异常。所以在很多情况下我们需要做各种非空的判断。

过多的非空判断会让我们的代码显得臃肿不堪，所以在JDK8中引入了Optional。

养成使用Optional的习惯后你可以写出更优雅的代码来避免空指针异常。

并且在很多函数式编程相关的API中也都用到了Optional。

Optional就好像是包装类，可以把我们的具体数据封装到Optional对象内部，然后我们去使用Optional中封装好的方法操作封装进去的数据就可以非常优雅的避免空指针异常。

## 2、使用

### 2.1、创建对象

#### 2.1.1、Optional.ofNullable(x)

一般使用这种ofNullable来把数据封装成一个Optional对象，无论传入的参数是否为null都不会出现问题。

在实际开发中，很多数据都是从数据库获取的。而且MyBatis从3.5版本开始支持Optional，可以直接把Dao方法的返回值类型定义成Optional类型，MyBatis会自己把数据封装成Optional对象返回，封装的过程也不需要我们自己操作。

```java
Author author = getAuthor();
Optional<Author> authorOptional = Optional.ofNullable(author);
```

ofNullable的底层实现：

```java
public static <T> Optional<T> ofNullable(T value) {
    return value == null ? empty() : of(value);
}

public static<T> Optional<T> empty() {
    @SuppressWarnings("unchecked")
    Optional<T> t = (Optional<T>) EMPTY;
    return t;
}

private static final Optional<?> EMPTY = new Optional<>();

private Optional() {
    this.value = null;
}
```

#### 2.1.2、Optional.of(x)

如果确定一个对象不为空则可以使用of来把数据封装成Optional对象，但是如果传入的参数为null，则会出现空指针异常。

```java
Optional<Author> authorOptional = Optional.of(null);
```

```tex
Exception in thread "main" java.lang.NullPointerException
	at java.util.Objects.requireNonNull(Objects.java:203)
	at java.util.Optional.<init>(Optional.java:96)
	at java.util.Optional.of(Optional.java:108)
	at org.yeahicode.StreamNewDemo.test25(StreamNewDemo.java:47)
	at org.yeahicode.StreamNewDemo.main(StreamNewDemo.java:40)
```

#### 2.1.3、Optional.empty()

将null包装成Optional对象。

```java
Optional<Object> empty = Optional.empty();
```

底层实现：

```java
public static<T> Optional<T> empty() {
    @SuppressWarnings("unchecked")
    Optional<T> t = (Optional<T>) EMPTY;
    return t;
}

private static final Optional<?> EMPTY = new Optional<>();

private Optional() {
    this.value = null;
}
```

### 2.2、安全消费值 - ifPresent

```java
Author author = getAuthor();
Optional<Author> authorOptional = Optional.ofNullable(author);
// 当封装的数据不为空时才执行ifPresent中的代码
authorOptional.ifPresent(System.out::println);
```

### 2.3、不安全获取值 - get

当Optional内部的数据为空的时候，通过get获取值会出现异常：

```java
authorOptional.get();
```

```tex
Exception in thread "main" java.util.NoSuchElementException: No value present
	at java.util.Optional.get(Optional.java:135)
	at org.yeahicode.StreamNewDemo.test25(StreamNewDemo.java:49)
	at org.yeahicode.StreamNewDemo.main(StreamNewDemo.java:40)
```

### 2.4、安全获取值

#### 2.4.1、orElseGet

![image-20230822231244916](./assets/image-20230822231244916.png)

存在值则直接返回，不存在则返回orElseGet中特别设置的默认值（可返回子类）：

```java
// Author author1 = authorOptional.orElseGet(() -> new Author());
Author author1 = authorOptional.orElseGet(new Supplier<Author>() {
    @Override
    public Author get() {
        return new Author();
    }
});
```

#### 2.4.2、orElseThrow
