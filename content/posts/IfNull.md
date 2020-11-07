---
title: "free and null"
date: 2020-11-07T10:03:37+08:00
draft: false
tags: ["C"]
---

前幾天同事問了個問題
```=
FILE* fp = fopen("xxxxxx", "w");

// a lot of code and branch
fclose(fp); 
// more code and branch

// at the end of function
if (fp)
	fclose(fp);
```
這段code的作者早已不可考，但每次都會卡在最後的fclose，同事對此感到不能理解。

我猜測是因為中間的分支太多，作者怕有哪裡沒有fclose，所以在最後做了個檢查。
但這種寫法其實很有問題，fclose 和 free 這類function並不會把pointer指向null，
所以不管有沒有fclose或free，最後的if一定成立，這就導致了double free。

如果一定要檢查有沒有釋放的話，就自己寫個free function，然後把指標指到null
```
void myClose(FILE** p) {
	fclose(*p);
	*p = NULL;
}
```
這樣就能用if去檢查指標有沒有被釋放。

說真的在一個上萬行的陳年專案裡看到這種code真的很毛。