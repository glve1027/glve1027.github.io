---
layout:       post
title:        "重拾算法(Java)"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Algorithm
---

## 最近买了几本算法方面的数，都是Java写的，其实算法重在思维方式，其实用什么语言去实现是无所谓的，之前都是用Swift来实现的，正好抽个机会学习一下Java。

<!-- more -->

> 通过Java实现一个动态数组的功能：

```java
package com.gh;

@SuppressWarnings("unchecked")
public class ArrayList<E> {
	/*
	 * 元素的数量
	 */
	private int size;
	
	/*
	 * 所有的元素
	 */
	private E[] elements;
	
	/*
	 * 默认容量
	 */
	private static final int DEFAULT_CAPACITY = 10;
	private static final int ELEMENT_NOT_FOUND = -1;
	
	/*
	 * 构造函数
	 */
	public ArrayList(int capacity) {
		elements = capacity < DEFAULT_CAPACITY ? (E[]) new Object[DEFAULT_CAPACITY] : (E[]) new Object[capacity];
	} 
	
	/*
	 * 默认构造函数
	 */
	public ArrayList() {
		this(DEFAULT_CAPACITY);
	}
	
	/*
	 * 数组的容量
	 */
	public int size() {
		return size;
	}
	
	/*
	 * 数组是否为空
	 */
	public boolean isEmpty() {
		return size == 0;
	}
	
	/*
	 * 根据index返回元素
	 */
	public E get(int index) {
		checkIndex(index);
		return elements[index];
	}
	
	/*
	 * 设置新的值
	 */
	public E set(int index, E element) {
		E old = get(index);
		elements[index] = element;
		return old;
	}
	
	/*
	 * 查看元素在数组中的位置
	 */
	public int indexOf(E elememt) {
		if (null == elememt) {
			for (int i = 0; i < size; i++) {
				if (elements[i] == null) return i;
			}
	 	} else {
			for (int i = 0; i < size; i++) {
				/**
				 * 如果 elements[i] == element 这样比较的是两个内存地址是否相同
				 * 这里用.equals不会影响对int的判断
				 * 因为这里的element肯定不会为null, 可以放心使用
				 */
				if (elememt.equals(elements[i])) return i;
			}
		}
		return ELEMENT_NOT_FOUND;
	}
	
	/*
	 * 是否包含元素
	 */
	public boolean contains(E element) {
		return indexOf(element) != ELEMENT_NOT_FOUND;
	}
	
	/*
	 * 清除所有的元素
	 */
	public void clear() {
		// 清除指针内存
		for (int i = 0; i < size; i++) {
			elements[i] = null;
		}
		size = 0;
	}
	
	/*
	 * 添加元素
	 */
	public void add(E element) {
		add(size, element);
	}
	
	/*
	 * 删除元素
	 */
	public E remove(int index) {
		checkIndex(index);
		E originValue = elements[index];
//		for (int i = index+1; i < size; i++) {
//			elements[i-1] = elements[i];
//		}
		while(index + 1 < size) {
			elements[index] = elements[index+1];
			index ++;
		}
		// 1. 先减减 2. 再清空数据
		elements[--size] = null;
		return originValue;
	}
	
	/*
	 * 在某和节点位置添加元素
	 */
	public void add(int index, E element) {
		if (index < 0 || index > size) return;
		
		ensureCapacity(size + 1);
		
		for (int i = size - 1; i >= index ; i--) {
			elements[i + 1] = elements[i];
		}
		elements[index] = element;
		size ++; 
	}
	
	@Override
	public String toString() {
		StringBuilder string = new StringBuilder();
		string.append("size=").append(size).append(", [");
		for (int i = 0; i < size; i++) {
			string.append(elements[i]);
			if (i != size - 1) { 
				string.append(", ");
			}
		}
		string.append("]");
		return string.toString();
	}
	
	/**
	 * 保证要有的capacity的容量
	 * @param capacity
	 */
	private void ensureCapacity(int capacity) {
		int oldCapacity = elements.length;
		if (oldCapacity >= capacity) return;
		// 新的容量是旧容量的1.5倍
		int newCapacity = oldCapacity + (oldCapacity >> 1);
		
		E[] newElements = (E[]) new Object[newCapacity];
		for (int i = 0; i < size; i++) {
			newElements[i] = elements[i];
		}
		elements = newElements;
		
		System.out.println(oldCapacity + "扩容为:" + newCapacity);
	}
	
	/*
	 * 检测Index的合法性
	 */
	private void checkIndex(int index) {
		if (index < 0 || index >= size) {
			throw new IndexOutOfBoundsException("数组越界 Index = " + index + ", Size = " + size);
		}
	} 
}
```

> 查看Java源码中可以发现，它的数组动态扩容用的是1.5倍, 这里的写法可以借鉴一下，很赞：int newCapacity = oldCapacity + (oldCapacity >> 1);运用位运算来来实现。

> 发现一个一直以来自己的不会的一个Vim命令,看源码的时候，我们正常需要gd到某个函数的定义或者实现里面去，这里就可以使用control+o就可以返回回去了【记录一下】。