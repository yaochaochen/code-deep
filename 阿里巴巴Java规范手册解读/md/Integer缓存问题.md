# Integer缓存问题

```
【强制】所有整型包装类对象之间的比较，全部使用equals方法比较。
说明: 对于Integer var = ？ 在-128至127范围内赋值， Integer 对象是在 IntegerCache.cache产生的，会复用已有对象，这个区间内的Integer值可以直接使用 == 判断，但是这个分区以外的所有数据，都会在堆上产生不会复用已有的对象，推荐使用equals方法进行判断。


```

## Integer缓存问题分析

```java
 public static void main(String[] args) {

        Integer a = 100, b = 100, c = 150, d = 150;
        System.out.println(a == b);
        System.out.println(c == d);
        
    }

```

输出结果: **true** **flase**

答案: 因为缓存了-128至127之间的数值，但是为什么缓存这一段区间的数值呢？缓存的值又如何修改呢，其他包装类有没有类似的存在？

## 源码分析

我们知道 Integer var = ？ 形式声明变量，会通过 `java.lang.Integer#valueOf(int)` 来构建 Integer对象。

```java
  /**
     * Returns an {@code Integer} instance representing the specified
     * {@code int} value.  If a new {@code Integer} instance is not
     * required, this method should generally be used in preference to
     * the constructor {@link #Integer(int)}, as this method is likely
     * to yield significantly better space and time performance by
     * caching frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  i an {@code int} value.
     * @return an {@code Integer} instance representing {@code i}.
     * @since  1.5
     */
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

从源码上看，不在范围内会new一个整数对象

```
如果想减少内存占用，提高程序运算率，可以将对象缓存起来，需要时直接取
```

### 那么Integer的缓存能被修改吗？

```java
/**
 * Cache to support the object identity semantics of autoboxing for values between
 * -128 and 127 (inclusive) as required by JLS.
 *
 * The cache is initialized on first usage.  The size of the cache
 * may be controlled by the {@code -XX:AutoBoxCacheMax=<size>} option.
 * During VM initialization, java.lang.Integer.IntegerCache.high property
 * may be set and saved in the private system properties in the
 * sun.misc.VM class.
 */

private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }
```

从源码上可以看出来最小值是固定，最大值根据 ` String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high")`

虚拟机的参数来调整的