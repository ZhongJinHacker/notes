

### Java8 Optional 总结



**模型类样例**

```java
@Data
private static class User {
  private String name;
  private String age;
}
```

### 用例1

```java
@Test(expected = NoSuchElementException.class)
public void whenCreateEmptyOptional_thenNull() {
  Optional<User> userOpt = Optional.empty();
  userOpt.get();
}
```



####获取一个包含null的Optional对象

```java
Optional<User> userOpt = Optional.empty()
```



#### optional 的get方法

```java
// 这里会抛出NoSuchElementException（extends RuntimeException）
// 因为 userOpt 是一个包含null的Optional对象，因为是非受检异常
// 最好在get之前 先检查一下比如 调用 userOpt.ifPresent()
userOpt.get();
```



### 用例2

```java
@Test(expected = NullPointerException.class)
public void test2() {
  User user = null;
  Optional<User> userOptional = Optional.of(user);
}
```

#### Optional 的==静态方法of== 不能传入null对象，否则==报空指针异常==，

**看源码**

```java
/**
 * Optional 源码
 */
public static <T> Optional<T> of(T value) {
  return new Optional<>(value);
}

private Optional(T value) {
  this.value = Objects.requireNonNull(value);
}

// Objects 源码
public static <T> T requireNonNull(T obj) {
  if (obj == null)
    throw new NullPointerException();
  return obj;
}
```



### 用例3 

```java
@Test
public void test_ofNullable() {
  User user = null;
  Optional<User> userOptional = Optional.ofNullable(user);
}
```

#### Optional 静态方法==ofNullable==可以传入null，传入null会得到一个包含null的optional对象

**源码**

```java
public static <T> Optional<T> ofNullable(T value) {
  return value == null ? empty() : of(value);
}
```



### 用例4

```java
    @Test
    public void test_getOptionalValue() {
        String name = "jim";
        Optional<String> nameOptional = Optional.of(name);
        if (nameOptional.isPresent()) {
            System.out.println(nameOptional.get());
        }
    }
```



#### Optional 实例方法 isPresent，==判断optional 的value是不是null==

**源码**

```java
    public boolean isPresent() {
        return value != null;
    }
```



### 用例5

```java
    @Test
    public void test_ifPresentFunc() {
        String name = "jim";
        Optional<String> nameOptional = Optional.of(name);
        nameOptional.ifPresent(value -> System.out.println("ifPresent=>" + value));
    }
```

####ifPresent(Consumer consumer) ：如果有值就执行一个consumer这个函数

**源码**

```java
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }
```



### 用例6

```java
@Test
public void test_filter() {
  String str = "abcd";
  Optional<String> stringOptional = Optional.of(str);
  stringOptional.filter(value -> value.contains("e"))
    						.ifPresent(value -> System.out.println("过滤后: " + value));
}
```

#### ==filter==用于条件过滤，如果==满足传入的Predicate函数==，则返回this，不满足返回一个包含null的Optional对象(既后续操作不做了,比如上面的<u>ifPresent</u>)

**源码**

```java
    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }
```



### 用例7 map 用法

 ```java
@Test
public void test_map() {
  String str = "abcd";
  Optional<String> stringOptional = Optional.of(str);
  stringOptional.map(String::toUpperCase)
    						.ifPresent(value -> System.out.println("映射后: " + value));
}
 ```



####==map==用于映射，将对象转化成另一个对象，或者是对内容做变形（比如上面的将字符串转换为大写）,并返回一个Optional对象；如果调用map的Optional对象为空值Optional对象==，也会返回一个包含空值的Optional对象



### 用例8 flatMap用法

**flatMap 和 map 的区别就是<u>map</u>==自动==帮你封装成了Optional对象，而<u>flatMap</u>需要你自己==手动封装==成Optional对象(这样看来map更好用)**

```java
@Test
public void test_flatMap() {
  String str = "abcd";
  Optional<String> stringOptional = Optional.of(str);
  stringOptional.flatMap(value -> Optional.ofNullable(value.toUpperCase()))
    .ifPresent(value -> System.out.println("flatMap映射后: " + value));
}
```



### 用例9 orElse用法

```java
    @Test
    public void test_orElse() {
        String str = null;
        Optional<String> stringOptional = Optional.ofNullable(str);
        System.out.println(stringOptional.orElse("qwerty"));
    }
```

**orElse即当调用它的对象是value是null的Optional对象时就执行返回参数的对象引用的函数**



==相对于==<u>orElseGet</u>的==缺点==是：

​	**执行到orElse时，括号内的语句一定会先执行，可能会造成一定的浪费（比如是个大对象）**

==如果需要orElse中临时创建对象，建议使用orElseGet，如果是已经创建好的对象的引用就无所谓了==

**源码**（可以看到other进来的时候已经创建完毕，即使是个null的Optional对象）

```java
    public T orElse(T other) {
        return value != null ? value : other;
    }
```



### 用例10 orElseGet用法

```java
@Test
public void test_orElseGet() {
  String str = null;
  Optional<String> stringOptional = Optional.ofNullable(str);
  System.out.println(stringOptional.orElseGet(() -> "qwerty"));
}
```



源码

```java
public T orElseGet(Supplier<? extends T> other) {
  return value != null ? value : other.get();
}
```

从源码可以看出来 只有value为null时，才会执行传入进来的函数，

==这样当value不为null时，就可以**避免提前创建**处对象所带来的浪费了==



用例11 orElseThrow 

```java
@Test
public void test_orElseThrow() {
  String str = null;
  Optional<String> stringOptional = Optional.ofNullable(str);
  System.out.println(stringOptional.orElseThrow(RuntimeException::new));
}
```



orElseThrow 如果是null的Optional对象调用它，会抛出括号内的异常（==这个异常类必须要有无参构造函数==）

源码

```java
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }
```

