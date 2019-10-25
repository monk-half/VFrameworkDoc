---
title: string操作
date: 2019-10-23 19:26:33
tags:
---

# string 类型

## 参数传递
string是C#中比较特殊的数据类型，属于引用类型，但是用法上又跟普通的值类型相似。

```
private void FuncA(string strAIn){
	strAIn = "b";
}

private void FuncB(){
	string strA = "a";
	FuncA(strA);
	print("output: " + strA);
}
```

结果：
```output: a```


### 原理
定义字符串常量时，会在堆中申请相应的空间存放该字符串常量。
strA = "a";执行该语句时，在栈中开辟一个存放字符串地址的变量，即存放指向"a"的指针。
FuncA(strA);执行该语句时，在调用栈中再开辟一个指针strAIn，将strA中存放的地址拷贝给strAIn。
strAIn = "b";执行该语句时，将strAIn中的存放的地址修改为指向"b"的地址。
FuncA(string strAIn);方法结束时，清除调用栈，strAIn变量回收，strA存放的内容不变。

## 使用效率
C# 内置了个Stopwatch的类(System.Diagnostics)，可以用来计算代码块的运行时间。

### const readonly 与普通变量
```
    const string str1 = "str1";
    readonly string str2 = "str2";
    string str3 = "str3";

    private void FuncConst(int iLoopCnt) {
        for (int i=0; i<iLoopCnt; i++){
            string str = str1 + str1;
        }
    }

    private void FuncReadOnly(int iLoopCnt) {
        for (int i = 0; i < iLoopCnt; i++) {
            string str = str2 + str2;
        }
    }

    private void FuncCommon(int iLoopCnt) {
        for (int i = 0; i < iLoopCnt; i++) {
            string str = str3 + str3;
        }
    }

    // Start is called before the first frame update
    void Start()
    {
        Stopwatch sw = new Stopwatch();

        sw.Start();
        FuncConst(1000000);
        sw.Stop();
        print("FuncConst cost: " + sw.ElapsedMilliseconds);
        sw.Reset();

        sw.Start();
        FuncReadOnly(1000000);
        sw.Stop();
        print("FuncReadOnly cost: " + sw.ElapsedMilliseconds);
        sw.Reset();

        sw.Start();
        FuncCommon(1000000);
        sw.Stop();
        print("FuncCommon cost: " + sw.ElapsedMilliseconds);
        sw.Reset();
    }
```
运行结果：
```
FuncConst cost: 4
FuncReadOnly cost: 232
FuncCommon cost: 244
```
> 结论：
> 字符串常量效率 >> 只读字符串变量效率 > 普通字符串变量。
> 字符串常量之所以效率这么高，是因为在C#编译阶段就对const字符串进行了替换。


### StringBuilder 与 string.Format 与 + 短拼接
```
    const string str1 = "str1";
    readonly string str2 = "str2";
    string str3 = "str3";

	private void FuncStringBuilder(int iLoopCnt) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < iLoopCnt; i++) {
            sb.Append(str1).Append(str2).Append(str3);
			sb.Clear();
        }
    }

    private void FuncFormat(int iLoopCnt) {
        string str = string.Empty;
        for (int i = 0; i < iLoopCnt; i++) {
            str = string.Format("{0}{1}{2}", str1, str2, str3);
        }
    }

    private void FuncAdd(int iLoopCnt) {
        string str = string.Empty;
        for (int i = 0; i < iLoopCnt; i++) {
            str = str1 + str2 + str3;
        }
    }

    void Start(){
		sw.Start();
        FuncStringBuilder(100000);
        sw.Stop();
        print("FuncStringBuilder cost: " + sw.ElapsedMilliseconds);
        sw.Reset();

        sw.Start();
        FuncFormat(100000);
        sw.Stop();
        print("FuncFormat cost: " + sw.ElapsedMilliseconds);
        sw.Reset();

        sw.Start();
        FuncAdd(100000);
        sw.Stop();
        print("FuncAdd cost: " + sw.ElapsedMilliseconds);
        sw.Reset();
	}
```
结果：
```
FuncStringBuilder cost: 17
FuncFormat cost: 78
FuncAdd cost: 29
```

如果每次都创建stringbuilder：
```
private void FuncStringBuilder1(int iLoopCnt) {
    for (int i = 0; i < iLoopCnt; i++) {
        StringBuilder sb = new StringBuilder().Append(str1).Append(str2).Append(str3);
    }
}
```
结果：
FuncStringBuilder1 cost: 48

> 结论:
> 拼接次数少的情况下，StringBuilder效率 > + > string.Format.
> 但如果每次都new StringBuilder对象，则 +效率 > StringBuilder > string.Format
> 如果程序在不同地方使用拼接，一般都会new StringBuilder(耗时较高)，所有地方的耗时加起来，效率将会低于直接使用+拼接。
> 综上：拼接次数少的时候，直接使用+号进行拼接即可。

### StringBuilder 与 string.Format 与 + 长拼接

```
    private void FuncStringBuilderLong(int iLoopCnt) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < iLoopCnt; i++) {
            sb.Append(str1);
        }
    }
    private void FuncFormatLong(int iLoopCnt) {
        string str = string.Empty;
        for (int i = 0; i < iLoopCnt; i++) {
            str += string.Format("{0}", str1);
        }
    }

    private void FuncAddLong(int iLoopCnt) {
        string str = string.Empty;
        for (int i = 0; i < iLoopCnt; i++) {
            str += str1;
        }
    }

    void Start(){
        sw.Start();
        FuncStringBuilderLong(10000);
        sw.Stop();
        print("FuncStringBuilderLong cost: " + sw.ElapsedMilliseconds);
        sw.Reset();

        sw.Start();
        FuncFormatLong(10000);
        sw.Stop();
        print("FuncFormatLong cost: " + sw.ElapsedMilliseconds);
        sw.Reset();

        sw.Start();
        FuncAddLong(10000);
        sw.Stop();
        print("FuncAddLong cost: " + sw.ElapsedMilliseconds);
        sw.Reset();
	}
```

结果：
```
FuncStringBuilderLong cost: 0
FuncFormatLong cost: 788
FuncAddLong cost: 2410
```

原因：
str += str1； 执行时，程序先向堆申请新的内存空间，然后将str str1的内容拷贝到新的堆中，最后将str存放的地址替换为新的堆地址。而且操作过程中产生大量垃圾。
sb.Append(str1); 执行时，直接在堆空间中增加内容，若内容长度超过堆空间大小，则进行扩容(扩容大小为初始大小的整数倍)。

> 结论
> 拼接次数多的情况下, StringBuilder效率 >> string.format >> +


## string 使用细节
https://blog.csdn.net/jiankunking/article/details/49702803
## 垃圾回收器原理
https://www.cnblogs.com/nele/p/5673215.html
【备忘】
ref out in等参数传递关键字

## 参考
>[值类型与引用类型原理](https://www.cnblogs.com/boboooo/p/9066831.html)
>[string 与 StringBuilder](https://blog.csdn.net/mrxu_persi/article/details/78642128)
>[C# 字符串操作](https://blog.csdn.net/jiankunking/article/details/49702803)