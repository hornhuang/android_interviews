# Map



#### Map的线程安全？

- Java中平时用的最多的Map集合就是HashMap了，它是线程不安全的

  - 当用在方法内的局部变量时，局部变量属于当前线程级别的变量，其他线程访问不了，所以这时也不存在线程安全不安全的问题了。

  - 当用在单例对象成员变量的时候呢？这时候多个线程过来访问的就是同一个HashMap了，对同个HashMap操作这时候就存在线程安全的问题了。



- 下面来总结下有哪些线程安全的Map呢？

  - SynchronizedMap

    ```java
    private Map<String, Object> map = Collections.synchronizedMap(new HashMap<String, Object>());
    ```

    - 这种是直接使用工具类里面的方法创建SynchronizedMap，把传入进行的HashMap对象进行了包装同步而已，来看看它的源码。

    ```java
    private static class SynchronizedMap<K,V>
        implements Map<K,V>, Serializable {
        private static final long serialVersionUID = 1978198479659022715L;
    
        private final Map<K,V> m;     // Backing Map
        final Object      mutex;        // Object on which to synchronize
    
        SynchronizedMap(Map<K,V> m) {
            this.m = Objects.requireNonNull(m);
            mutex = this;
        }
    
        SynchronizedMap(Map<K,V> m, Object mutex) {
            this.m = m;
            this.mutex = mutex;
        }
    
        public int size() {
            synchronized (mutex) {return m.size();}
        }
        public boolean isEmpty() {
            synchronized (mutex) {return m.isEmpty();}
        }
        public boolean containsKey(Object key) {
            synchronized (mutex) {return m.containsKey(key);}
        }
        public boolean containsValue(Object value) {
            synchronized (mutex) {return m.containsValue(value);}
        }
        public V get(Object key) {
            synchronized (mutex) {return m.get(key);}
        }
    
        public V put(K key, V value) {
            synchronized (mutex) {return m.put(key, value);}
        }
    ```

  - ConcurrentHashMap - 推荐

    ```java
    private Map<String, Object> map = new ConcurrentHashMap<>();
    ```

    - 这个也是最推荐使用的线程安全的Map，也是实现方式最复杂的一个集合，每个版本的实现方式也不一样
    - 在jdk8之前是使用分段加锁的一个方式，分成16个桶，每次只加锁其中一个桶，而在jdk8又加入了红黑树和CAS算法来实现。

    ![ConcurrentHashMap ](https://github.com/FishInWater-1999/PictureRepository/blob/master/Interviews/hashmap.jpg)

  - HashTable

    ```Java
    private Map<String, Object> map = new Hashtable<>();
    ```

    ```java
    @SuppressWarnings("unchecked")
    public synchronized V get(Object key) {
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                return (V)e.value;
            }
        }
        return null;
    }
    ```

    ```java
    @SuppressWarnings("unchecked")
    public synchronized V get(Object key) {
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                return (V)e.value;
            }
        }
        return null;
    }
    ```

    - HashTable 的 get/put 方法都被 synchronized 关键字修饰，说明它们是方法级别阻塞的。
    - 它们占用共享资源锁，所以导致同时只能一个线程操作 get 或者 put ，而且 get/put 操作不能同时执行，所以这种同步的集合效率非常低，一般不建议使用这个集合。



#### 读多写少选哪个集合？

- CopyOnWrite
前置条件（volatile Synchronized ReentrantLock）
