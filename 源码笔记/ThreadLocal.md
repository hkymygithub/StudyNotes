# ThreadLocal

```java
/*
 * Copyright (c) 1997, 2013, Oracle and/or its affiliates. All rights reserved.
 * DO NOT ALTER OR REMOVE COPYRIGHT NOTICES OR THIS FILE HEADER.
 *
 * This code is free software; you can redistribute it and/or modify it
 * under the terms of the GNU General Public License version 2 only, as
 * published by the Free Software Foundation.  Oracle designates this
 * particular file as subject to the "Classpath" exception as provided
 * by Oracle in the LICENSE file that accompanied this code.
 *
 * This code is distributed in the hope that it will be useful, but WITHOUT
 * ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
 * FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License
 * version 2 for more details (a copy is included in the LICENSE file that
 * accompanied this code).
 *
 * You should have received a copy of the GNU General Public License version
 * 2 along with this work; if not, write to the Free Software Foundation,
 * Inc., 51 Franklin St, Fifth Floor, Boston, MA 02110-1301 USA.
 *
 * Please contact Oracle, 500 Oracle Parkway, Redwood Shores, CA 94065 USA
 * or visit www.oracle.com if you need additional information or have any
 * questions.
 */

package java.lang;

import java.lang.ref.*;
import java.util.Objects;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.function.Supplier;

/**
 * This class provides thread-local variables.  These variables differ from
 * their normal counterparts in that each thread that accesses one (via its
 * {@code get} or {@code set} method) has its own, independently initialized
 * copy of the variable.  {@code ThreadLocal} instances are typically private
 * static fields in classes that wish to associate state with a thread (e.g.,
 * a user ID or Transaction ID).
 *
 * <p>For example, the class below generates unique identifiers local to each
 * thread.
 * A thread's id is assigned the first time it invokes {@code ThreadId.get()}
 * and remains unchanged on subsequent calls.
 * <pre>
 * import java.util.concurrent.atomic.AtomicInteger;
 *
 * public class ThreadId {
 *     // Atomic integer containing the next thread ID to be assigned
 *     private static final AtomicInteger nextId = new AtomicInteger(0);
 *
 *     // Thread local variable containing each thread's ID
 *     private static final ThreadLocal&lt;Integer&gt; threadId =
 *         new ThreadLocal&lt;Integer&gt;() {
 *             &#64;Override protected Integer initialValue() {
 *                 return nextId.getAndIncrement();
 *         }
 *     };
 *
 *     // Returns the current thread's unique ID, assigning it if necessary
 *     public static int get() {
 *         return threadId.get();
 *     }
 * }
 * </pre>
 * <p>Each thread holds an implicit reference to its copy of a thread-local
 * variable as long as the thread is alive and the {@code ThreadLocal}
 * instance is accessible; after a thread goes away, all of its copies of
 * thread-local instances are subject to garbage collection (unless other
 * references to these copies exist).
 *
 * @author Josh Bloch and Doug Lea
 * @since 1.2
 */
public class ThreadLocal<T> {
    /**
     * ThreadLocals rely on per-thread linear-probe hash maps attached
     * to each thread (Thread.threadLocals and
     * inheritableThreadLocals).  The ThreadLocal objects act as keys,
     * searched via threadLocalHashCode.  This is a custom hash code
     * (useful only within ThreadLocalMaps) that eliminates collisions
     * in the common case where consecutively constructed ThreadLocals
     * are used by the same threads, while remaining well-behaved in
     * less common cases.
     */
    private final int threadLocalHashCode = nextHashCode();

    /**
     * 下一个要给出的哈希码。原子更新。数值从 0 开始
     */
    private static AtomicInteger nextHashCode =
            new AtomicInteger();

    /**
     * 哈希增量
     * 将连续生成的哈希码之间的差异 - 将顺序的本地线程 ID 隐式地转换为接近最佳传播的乘法哈希值，用于二次幂大小的表。
     */
    private static final int  HASH_INCREMENT = 0x61c88647;

    /**
     * 返回下一个哈希值
     */
    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }

    /**
     * 返回当前线程局部变量的“初始值”。
     * 该方法将在线程第一次使用 get 方法访问变量时调用，如果线程先前调用了 set 方法，本方法不会调用。
     * 通常，每个线程最多调用一次此方法，但在调用 remove 方法后再次调用 get方法 的情况下，它可能会再次调用。
     *
     * 这个实现只返回 null
     * 如果希望线程局部变量具有其它初始值，则必须将 ThreadLocal 子类化，并重写此方法。通常，将使用匿名内部类的方式实现。
     *
     * @return 返回这个 thread-local 的初始值
     */
    protected T initialValue() {
        return null;
    }

    /**
     * 创建一个 thread local 实例。 变量的初始值被定义在 Supplier 的 get 方法中
     *
     * @param <S> thread local 值的类型
     * @param supplier 定义初始值的 supplier
     * @return 返回一个 thread local 实例
     * @throws NullPointerException 如果 supplier 为 null 则返回空指针异常
     * @since 1.8
     */
    public static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier) {
        return new SuppliedThreadLocal<>(supplier);
    }

    /**
     * 创建一个 thread local 实例。
     * 无参构造器
     * @see #withInitial(java.util.function.Supplier)
     */
    public ThreadLocal() {
    }

    /**
     * 返回当前线程中 thread-local 里的值
     * 如果这个 thread-local 没有值则返回调用 initialValue 方法的结果，即返回初始值
     *
     * @return 返回当前线程在 thread-local 中的值
     */
    public T get() {
        Thread t = Thread.currentThread();// 获取当前线程
        ThreadLocalMap map = getMap(t);// 获取当前线程的 ThreadLocalMap
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);// 以本实例为 key 获取 ThreadLocalMap 中对应的 Entry
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T) e.value;
                return result;// 返回 Entry 中的 value
            }
        }
        return setInitialValue();// 如果没找到则返回初始值
    }

    /**
     * set() 的变体，用于建立初始值。如果用户重写了 set() 方法，则使用它代替 set()。
     *
     * @return 初始 value
     */
    private T setInitialValue() {
        T value = initialValue();// 获取初始值
        Thread t = Thread.currentThread();// 获取当前线程
        ThreadLocalMap map = getMap(t);// 获取当前线程的 ThreadLocalMap
        if (map != null)// Map 不空则直接设置初始值
            map.set(this, value);
        else// Map 为空则创建 Map 并设置初始值
            createMap(t, value);
        return value;
    }

    /**
     * 设置当前线程 ThreadLocal 的值。
     * 大多数子类不需要重写此方法，仅依靠 initialValue 方法来设置线程局部变量的值。
     *
     * @param value 要存入的值
     */
    public void set(T value) {
        Thread t = Thread.currentThread();// 获取当前线程
        ThreadLocalMap map = getMap(t);// 获取当前线程的 ThreadLocalMap
        if (map != null)// Map 不空则直接设置值
            map.set(this, value);
        else// Map 为空则创建 Map 并设置值
            createMap(t, value);
    }

    /**
     * 
     * 删除此线程 ThreadLocalMap 中以本实例为 key 的 value。
     * 如果这个 ThreadLocal 随后被当前线程调用了 get 方法，它的值将通过调用它的 initialValue 方法重新初始化，除非它的值提前被 set 。
     * 因此在当前线程中可能会多次调用此 ThreadLocal 实例的 initialValue 方法。
     *
     * @since 1.5
     */
    public void remove() {
        ThreadLocalMap m = getMap(Thread.currentThread());// 获取当前线程的 ThreadLocalMap
        if (m != null)// 在 ThreadLocalMap 中删除本以实例为 key 的 value
            m.remove(this);
    }

    /**
     * 获取与 ThreadLocal 关联的 ThreadLocalMap。
     * 本方法在会在 InheritableThreadLocal 中重写。
     *
     * @param  t 当前线程
     * @return the map
     */
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    /**
     * 创建与 ThreadLocal 关联的 ThreadLocalMap。
     * 本方法在会在 InheritableThreadLocal 中重写。
     *
     * @param t 当前线程
     * @param firstValue 初始值
     */
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

    /**
     * 创建 InheritableThreadLocal 对应的 Map 的工厂方法
     * 设计上仅被 Thread 构造函数调用。
     *
     * @param  parentMap 父线程的 ThreadLocalMap
     * @return 包含父级可继承绑定的 Map
     */
    static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {
        return new ThreadLocalMap(parentMap);
    }

    /**
     * 方法 childValue 可见定义在子类 InheritableThreadLocal 中。
     * 在这里内部定义是为了提供 createInheritedMap 工厂方法，而不需要在 InheritableThreadLocal 中继承 map 类。
     * 这种技术优于在方法中嵌入 instanceof 测试的替代方法。
     */
    T childValue(T parentValue) {
        throw new UnsupportedOperationException();
    }

    /**
     * ThreadLocal 的扩展，从指定的 Supplier 获取其初始值。
     * 此类重写了 initialValue 方法，通过构造器中传入 Supplier 来自定义 ThreadLocal 的初始值
     */
    static final class SuppliedThreadLocal<T> extends ThreadLocal<T> {

        private final Supplier<? extends T> supplier;

        SuppliedThreadLocal(Supplier<? extends T> supplier) {
            this.supplier = Objects.requireNonNull(supplier);
        }

        @Override
        protected T initialValue() {
            return supplier.get();
        }
    }

    /**
     * ThreadLocalMap 是一个定制的哈希表，仅适用于维护线程本地值。
     * 在 ThreadLocal 类之外不会导出任何操作。该类是包私有的，以允许在类 Thread 中声明字段。
     * 为了帮助处理非常大且长期存在的使用，哈希表条目使用 WeakReferences 作为键。
     * 由于不使用引用队列，因此只有在表开始空间不足时才能保证删除过时的条目
     */
    static class ThreadLocalMap {

        /**
         * 此哈希表中的元素( Entry )扩展了 WeakReference，使用其主 ref 字段作为键（始终是 ThreadLocal 对象）。
         * 请注意，空键（即 entry.get() == null）意味着不再引用该键，因此可以从表中删除该条目。
         * 在之后的代码中，此类条目称为“陈旧条目”。
         */
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }

        /**
         * 初始容量 -- 必须是 2 的幂次
         */
        private static final int INITIAL_CAPACITY = 16;

        /**
         * Entry 数组作为哈希表，表的长度必须是 2 的幂次
         */
        private Entry[] table;

        /**
         * 表中的条目数量
         */
        private int size = 0;

        /**
         * resize 的阈值
         */
        private int threshold; // Default to 0

        /**
         * Set the resize threshold to maintain at worst a 2/3 load factor.
         */
        private void setThreshold(int len) {
            threshold = len * 2 / 3;
        }

        /**
         * 增量 i % len.
         */
        private static int nextIndex(int i, int len) {
            return ((i + 1 < len) ? i + 1 : 0);
        }

        /**
         * 减少 i % len.
         */
        private static int prevIndex(int i, int len) {
            return ((i - 1 >= 0) ? i - 1 : len - 1);
        }

        /**
         * 构造一个有初始内容的 Map (firstKey, firstValue).
         * ThreadLocalMaps 是懒加载的，所以只在有一个值的时候进行加载
         */
        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];// 初始容量 16
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);// 确定在 table 中的位置
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);// 初始阈值 16 * 2 / 3
        }

        /**
         * 从给定的父 Map 构造一个包含所有可继承 ThreadLocals 的新 Map。
         * 仅由 createInheritedMap 调用
         *
         * @param parentMap the map associated with parent thread.
         */
        private ThreadLocalMap(ThreadLocalMap parentMap) {
            Entry[] parentTable = parentMap.table;// 复制父 Map 参数
            int len = parentTable.length;
            setThreshold(len);
            table = new Entry[len];

            for (int j = 0; j < len; j++) {// 重新映射
                Entry e = parentTable[j];
                if (e != null) {
                    @SuppressWarnings("unchecked")
                    ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
                    if (key != null) {
                        Object value = key.childValue(e.value);
                        Entry c = new Entry(key, value);
                        int h = key.threadLocalHashCode & (len - 1);
                        while (table[h] != null)
                            h = nextIndex(h, len);
                        table[h] = c;
                        size++;
                    }
                }
            }
        }

        /**
         * 获取与 key 关联的条目。此方法本身仅处理快速路径：直接寻找现有键。
         * 否则它会执行 getEntryAfterMiss 方法。
         * 这旨在最大限度地提高直接命中的性能，部分原因是使该方法易于内联。
         *
         * @param  key the thread local object
         * @return the entry associated with key, or null if no such
         */
        private Entry getEntry(ThreadLocal<?> key) {
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.get() == key)// 如果直接命中则返回
                return e;
            else// 如果没有直接命中
                return getEntryAfterMiss(key, i, e);
        }

        /**
         * 当在其直接哈希表中找不到 key 时使用的 getEntry 方法版本。
         *
         * @param  key the thread local object
         * @param  i the table index for key's hash code
         * @param  e the entry at table[i]
         * @return the entry associated with key, or null if no such
         */
        private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
            Entry[] tab = table;
            int len = tab.length;

            while (e != null) {// e 不为空说明要找到 key 可能在 e 后面
                ThreadLocal<?> k = e.get();
                if (k == key)// 找到则返回
                    return e;
                if (k == null)// key 为 null 说明这个条目过时了，可以删除
                    expungeStaleEntry(i);
                else// 找下一个位置
                    i = nextIndex(i, len);
                e = tab[i];
            }
            return null;
        }

        /**
         * 根据 key 设置 value
         *
         * @param key the thread local object
         * @param value the value to be set
         */
        private void set(ThreadLocal<?> key, Object value) {

            // 我们不像 get() 那样使用快速路径，因为使用 set() 创建新条目与替换现有条目一样普遍，在这种情况下，快速路径往往会失败.

            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len - 1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {// 如果 key 已存在 则替换 value
                    e.value = value;
                    return;
                }

                if (k == null) {// 如果 key 等于 null,说明此位置条目已过期，则替换此条目
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }
            // 在表中没有找到 key 并且没有过期的项，则往后招一个空位放入
            tab[i] = new Entry(key, value);
            int sz = ++size;// size 加一
            if (!cleanSomeSlots(i, sz) && sz >= threshold)// 如果没有清除过期条目并且表大小超过阈值则重新哈希
                rehash();
        }

        /**
         * 根据传入的 key 删除元素
         */
        private void remove(ThreadLocal<?> key) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len - 1);// 确定表中的位置
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                if (e.get() == key) {
                    e.clear();// 将此条目过期
                    expungeStaleEntry(i);// 删除过期条目
                    return;
                }
            }
        }

        /**
         * Replace a stale entry encountered during a set operation
         * with an entry for the specified key.  The value passed in
         * the value parameter is stored in the entry, whether or not
         * an entry already exists for the specified key.
         *
         * As a side effect, this method expunges all stale entries in the
         * "run" containing the stale entry.  (A run is a sequence of entries
         * between two null slots.)
         *
         * @param  key the key
         * @param  value the value to be associated with key
         * @param  staleSlot index of the first stale entry encountered while
         *         searching for key.
         */
        private void replaceStaleEntry(ThreadLocal<?> key, Object value,
                                       int staleSlot) {
            Entry[] tab = table;
            int len = tab.length;
            Entry e;

            // Back up to check for prior stale entry in current run.
            // We clean out whole runs at a time to avoid continual
            // incremental rehashing due to garbage collector freeing
            // up refs in bunches (i.e., whenever the collector runs).
            int slotToExpunge = staleSlot;
            for (int i = prevIndex(staleSlot, len);
                 (e = tab[i]) != null;
                 i = prevIndex(i, len))
                if (e.get() == null)
                    slotToExpunge = i;

            // Find either the key or trailing null slot of run, whichever
            // occurs first
            for (int i = nextIndex(staleSlot, len);
                 (e = tab[i]) != null;
                 i = nextIndex(i, len)) {
                ThreadLocal<?> k = e.get();

                // If we find key, then we need to swap it
                // with the stale entry to maintain hash table order.
                // The newly stale slot, or any other stale slot
                // encountered above it, can then be sent to expungeStaleEntry
                // to remove or rehash all of the other entries in run.
                if (k == key) {
                    e.value = value;

                    tab[i] = tab[staleSlot];
                    tab[staleSlot] = e;

                    // Start expunge at preceding stale entry if it exists
                    if (slotToExpunge == staleSlot)
                        slotToExpunge = i;
                    cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
                    return;
                }

                // If we didn't find stale entry on backward scan, the
                // first stale entry seen while scanning for key is the
                // first still present in the run.
                if (k == null && slotToExpunge == staleSlot)
                    slotToExpunge = i;
            }

            // If key not found, put new entry in stale slot
            tab[staleSlot].value = null;
            tab[staleSlot] = new Entry(key, value);

            // If there are any other stale entries in run, expunge them
            if (slotToExpunge != staleSlot)
                cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
        }

        /**
         * 在解决哈希冲突过程中删除过时条目
         * 通过重新散列位于 staleSlot 和下一个空槽之间的任何可能冲突的条目来删除过时的条目。
         * 这也会删除在尾随 null 之前遇到的任何其他陈旧条目。见 Knuth，第 6.4 节
         *
         * @param staleSlot index of slot known to have null key
         * @return 返回过期条目之后的下一个空槽的索引（所有在过期条目和这个槽之间的都将被检查是否被删除）。
         */
        private int expungeStaleEntry(int staleSlot) {
            Entry[] tab = table;
            int len = tab.length;

            // 删除过时条目，value会被垃圾回收器回收
            tab[staleSlot].value = null;
            tab[staleSlot] = null;
            size--;

            // 重新散列，直到遇到 null
            Entry e;
            int i;
            for (i = nextIndex(staleSlot, len);
                 (e = tab[i]) != null;
                 i = nextIndex(i, len)) {
                ThreadLocal<?> k = e.get();
                if (k == null) {// 条目过时，进行删除操作
                    e.value = null;
                    tab[i] = null;
                    size--;
                } else {
                    int h = k.threadLocalHashCode & (len - 1);
                    if (h != i) {
                        tab[i] = null;

                        // 与 Knuth 6.4 算法 R 不同，我们必须扫描到 null，因为多个条目可能已经过时。
                        while (tab[h] != null)
                            h = nextIndex(h, len);
                        tab[h] = e;
                    }
                }
            }
            return i;
        }

        /**
         * 启发式地扫描一些单元格以查找过时的条目。
         * 当添加新元素或删除另一个陈旧元素时调用此方法。
         * 它执行对数扫描，作为不扫描（快速但保留垃圾）和与元素数量成比例的扫描数量之间的平衡，
         * 这将找到所有垃圾但会导致一些插入花费 O(n) 时间。
         *
         * @param i a position known NOT to hold a stale entry. The
         * scan starts at the element after i.
         *
         * @param n scan control: {@code log2(n)} cells are scanned,
         * unless a stale entry is found, in which case
         * {@code log2(table.length)-1} additional cells are scanned.
         * When called from insertions, this parameter is the number
         * of elements, but when from replaceStaleEntry, it is the
         * table length. (Note: all this could be changed to be either
         * more or less aggressive by weighting n instead of just
         * using straight log n. But this version is simple, fast, and
         * seems to work well.)
         *
         * @return 如果有条目被删除了则返回 true
         */
        private boolean cleanSomeSlots(int i, int n) {
            boolean removed = false;
            Entry[] tab = table;
            int len = tab.length;
            do {
                i = nextIndex(i, len);
                Entry e = tab[i];
                if (e != null && e.get() == null) {// 说明找到了一个过期条目
                    n = len;
                    removed = true;// 表示已经删除部分内容
                    i = expungeStaleEntry(i);// 具体删除操作
                }
            } while ((n >>>= 1) != 0);// 并非遍历每个元素，索引 i 每次除 2，遍历时间大约为 log(n)
            return removed;
        }

        /**
         * 重新打包和/或重新调整表格大小。首先扫描整个表删除过时的条目。如果这不能充分缩小表格的大小，则将表格大小加倍。
         */
        private void rehash() {
            expungeStaleEntries();// 清除过期条目

            // 使用较低的加倍阈值以避免滞后
            if (size >= threshold - threshold / 4)
                resize();
        }

        /**
         * 将表的容量乘 2
         */
        private void resize() {
            Entry[] oldTab = table;
            int oldLen = oldTab.length;
            int newLen = oldLen * 2;
            Entry[] newTab = new Entry[newLen];
            int count = 0;

            for (int j = 0; j < oldLen; ++j) {
                Entry e = oldTab[j];
                if (e != null) {
                    ThreadLocal<?> k = e.get();
                    if (k == null) {
                        e.value = null; // Help the GC
                    } else {
                        int h = k.threadLocalHashCode & (newLen - 1);
                        while (newTab[h] != null)
                            h = nextIndex(h, newLen);
                        newTab[h] = e;
                        count++;
                    }
                }
            }

            setThreshold(newLen);
            size = count;
            table = newTab;
        }

        /**
         * Expunge all stale entries in the table.
         */
        private void expungeStaleEntries() {
            Entry[] tab = table;
            int len = tab.length;
            for (int j = 0; j < len; j++) {
                Entry e = tab[j];
                if (e != null && e.get() == null)
                    expungeStaleEntry(j);
            }
        }
    }
}
```