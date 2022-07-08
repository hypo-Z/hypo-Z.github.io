---
title: Algorithm一天一道算法
author: hypo
img: medias/featureimages/55.jpeg
top: false
cover: false
toc: true
mathjax: false
date: 2022-07-08 11:12:42
coverImg:
password:
summary: 为了使自己每日进步一点，每天更新一道leetcode上的算法题，及理解其原理
categories: Golang
tags:
- Golang
- 算法
---
# 一天一道算法题

## 1.两数之和

给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 的那 两个 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

你可以按任意顺序返回答案。

示例 1：

输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
示例 2：

输入：nums = [3,2,4], target = 6
输出：[1,2]
示例 3：

输入：nums = [3,3], target = 6
输出：[0,1]

```go
//解法一
func twoSum1(nums []int, target int) []int {
	//遍历所有的加法组合，直到找到符合相加得到目标数的两个数,并且两数不是同一个数
   l:=len(nums)
   for i:=0;i<l-1;i++{
      for j:=1;j<l;j++{
         if target==nums[i]+nums[j]&&i!=j{
            return []int{i,j}
         }
      }
   }
   return nil
}
//这种方法比较笨，且较慢，时间复杂度O(n^2)

//解法二
func twoSum2(nums []int, target int) []int {
   //创建一个map存储遍历后的数组对，键值对个数为数组长度
   num2index := make(map[int]int, len(nums))
   //遍历数组
   for i, num := range nums {
      //核心是用目标值减去循环遍历出来的值，判断是否是数组内的值
      pair := target - num
      //
      if j, ok := num2index[pair]; ok && i != j {
         // 注意返回值顺序，向后遍历 nums，所以 i 在 j 后
         return []int{j, i}
      }
      //循环一边就存入map
      //第一遍map[2]=0,没有相同
      //第二遍map[7]=1,没有相同且存在pair := target - num

      num2index[num] = i
   }
   return nil
}
//这种方法虽牺牲空间但速度快，时间复杂度O(log2n)
```



## 2.翻转字符串

给定一串字符串，将字符串进行翻转

示例1：输入"abc",输出"cba";

示例2：输入"1313adfasf",输出"fsada3131";



```go
//解法
func Reverse(s string) string {
    //[]rune符文是int32的别名，在所有方面都与int32等效。按照惯例，它用于区分字符值和整数值。
   a := []rune(s)
    //从两端开始进行转变位置
   for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
       //go语言内部会构造一个虚拟变量
      a[i], a[j] = a[j], a[i]
   }
   return string(a)
}
```



## 3.回文判断

给你一个整数 x ，如果 x 是一个回文整数，返回 true ；否则，返回 false 。

回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

例如，121 是回文，而 123 不是。

```go
//我的解法
func isPalindrome(x int) bool {
    //负数不是回文
   if x<0 {
      return false
   }
    //提示：转为字符串方便
   a:=strconv.Itoa(x)
   s:=[]rune(a)
   l:=len(s)
    //由翻转想到的灵感，不是最好
   for i,j:=0,l-1;i<j;i,j=i+1,j-1{
      if s[i]!=s[j]{
         return false
      }
   }
   return true
}
```



