Thread有一个成员 

```java
ThreadLocal.ThreadLocalMap threadLocals = null;
```

在创建ThreadLocal时会创建这个对象

```java
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
```

```java
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```

精髓尽在==ThreadLocalMap==中

现在相当于==Thread== 一对一==ThreadLocalMap==

而==ThreadLocalMap 以ThreadLocal（的HashCode）为Key==，也就是说 ==ThreadLocalMap== 一对多 ==ThreadLocal==，也可以有结论

==Thread== 一对多==ThreadLocal==

```java
//ThreadLocalMap 方法       
private void set(ThreadLocal<?> key, Object value) {

            // We don't use a fast path as with get() because it is at
            // least as common to use set() to create new entries as
            // it is to replace existing ones, in which case, a fast
            // path would fail more often than not.

            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
```

## 所以一个线程可以有多个ThreadLocal对象，同时每个ThreadLocal都能取到自己对应的值

