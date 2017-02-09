---
layout: post
title: "对照阿里Java开发规范"
date: 2017-02-09 16:25:15 +0800
comments: true
categories: code
---



[阿里Java开发规范](../images/eb0998bac7664496b2f1af98e07b08e5-Java.pdf)

对照阿里Java开发规范，有些自己没注意的地方：

* 使用索引访问用 String 的 split 方法得到的数组时，需做最后一个分隔符后有无内 容的检查，否则会有抛 IndexOutOfBoundsException 的风险


		String str ="a,b,c,,";
		String[] ary=str.split(",");
		//预期输出4，运行结果为3
		System.out.println(ary.lenght);
		
* ArrayList 的 subList 结果不可强转成 ArrayList，否则会抛出 ClassCastException 异常:java.util.RandomAccessSubList cannot be cast to java.util.ArrayList ; 说明:subList 返回的是 ArrayList 的内部类 SubList，并不是 ArrayList ，而是 ArrayList 的一个视图，对于 SubList 子列表的所有操作最终会反映到原列表上
* 在 JDK7 版本以上，Comparator 要满足自反性，传递性，对称性，不然 Arrays.sort， Collections.sort 会报 IllegalArgumentException 异常。说明:1) 自反性:x，y 的比较结果和 y，x 的比较结果相反。 2) 传递性:x>y,y>z,则 x>z。3) 对称性:x=y,则 x,z 比较结果和 y，z 比较结果相同。  反例:下例中没有处理相等的情况，实际使用中可能会出现异常:	
		new Comparator<Student>() {			@Override			public int compare(Student o1, Student o2) {    			return o1.getId() > o2.getId() ? 1 : -1;		} }

