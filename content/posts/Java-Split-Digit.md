---
title: "Java Split Digit"
date: 2019-11-28T09:22:17+08:00
draft: false
---

這是水文😂



真的是一篇水文 

主要是我想不到要寫甚麼了

<!--more-->

# 😀

主要是AMC10有一道題，1000到2013中，多少數字前三位（千，百，十）相加之和爲個位數

然後。。。

```java
import java.util.*;
public class Tester {

	public static void main(String[] args) {
		int count = 0;
		for (int i=1000;i<2014;i++) {
			ArrayList<Integer> num  = new ArrayList<Integer>();
			int number = i;
			while (number != 0) {
				num.add(number % 10);
				number = number / 10;
			}
			if ((num.get(3)+num.get(2)+num.get(1))==num.get(0)) {
					count ++;
					System.out.println(num);
			}
		}
		System.out.println(count);
	}
}

```



