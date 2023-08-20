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

