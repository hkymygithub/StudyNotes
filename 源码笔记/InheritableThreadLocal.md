# InheritableThreadLocal

```java
/*
 * Copyright (c) 1998, 2012, Oracle and/or its affiliates. All rights reserved.
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

/**
 * 此类继承于 ThreadLocal 以提供从父线程到子线程的值继承
 * 当创建子线程时，子线程的初始值为父线程所有可继承线程局部变量的值。通常，子线程的值和父线程相同；
 * 但是，通过覆盖此类中的 childValue 方法，可以使子线程的值变成任意值。
 * 当在变量中维护的每个线程属性（例如，用户 ID、事务 ID）必须自动传输到创建的任何子线程时
 * InheritableThreadLocal 中的值应该优先于 ThreadLocal 使用.
 *
 * @author Josh Bloch and Doug Lea
 * @see     ThreadLocal
 * @since 1.2
 */

public class InheritableThreadLocal<T> extends ThreadLocal<T> {
    /**
     * 计算此可继承线程局部变量的子项初始值，作为创建子线程时父项值的函数。
     * 在启动子线程之前，从父线程中调用此方法。 
     * 此方法仅返回其输入参数，如果需要不同的行为，则应将其覆盖。
     *
     * @param parentValue the parent thread's value
     * @return the child thread's initial value
     */
    protected T childValue(T parentValue) {
        return parentValue;
    }

    /**
     * 获取与 ThreadLocal 关联的 Map。
     * 此处获取的是线程中的 inheritableThreadLocals 属性，即继承的 ThreadLocalMap
     *
     * @param t the current thread
     */
    ThreadLocalMap getMap(Thread t) {
        return t.inheritableThreadLocals;
    }

    /**
     * 为线程的 inheritableThreadLocals 属性创建一个 ThreadLocalMap。
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the table.
     */
    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}

```