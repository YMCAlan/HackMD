# 在學校不會教的C語言的宣告與解讀


## 簡介

C語言為強型別的語言，對於資料型態規範非常嚴格，因此對於C語言中變數的宣告，要能夠解讀與分析，這樣才能夠理解每一個變數或函數或陣列所使用的資料型態是甚麼。


## Right-left Rule
Right-left Rule指得是解讀C語言宣告式的一個法則，

以下有3個基本的運算子，

* `*`讀為 "指標指向的是`as pointer to ...`只會在 (辨識字) 左邊出現
* `[]`讀為 "陣列元素為`array of ...` 只會在 (辨識字) 右邊出現
* `()`讀為 "函數回傳值`function (parameter) returning ...` 只會在 (辨識字) 右邊出現

辨識子就是函數名稱或變數名稱喔!

## Step步驟

1. 找到辨識字，解讀為 `x`是 或者 `x` is/as
2. 往辨識字的右邊看，如果看到
   * `[]` 解讀為 `x`是陣列，其元素為...，`x` is the array of ...
   * `()` 解讀為 `x`是函數，其回傳值為...，`x` is the function returning ...
   直到遇到 單獨的 `)`
   * 如果遇到`(`則代表中間有其他東西要解讀，基本上就是遇到函數，要解讀其參數為優先。
   * 如果遇到`)`則馬上往左走進行解讀!
3. 接著往辨識字的左邊看
   * `*` 解讀為指向的是，pointer to ...
   * `datatype`就照念就好。


## 以下為不合法的C語言宣告
* `[]()` 沒有陣列中的元素是函數
* `()()` 沒有函數中回傳函數
* `()[]` 沒有函數回傳陣列
但我們可以將其變成指標來使其合法


## Example
`int *ptr[]`
En:declare `ptr` as array of pointer to `int`
En:`ptr` is array of pointer to `int`
Ch:`ptr`是一個陣列，元素內容為指標，指向的是`int`
Ch: `ptr`是一個指標陣列，指向的是`int`
:::info
中文真的很玄
:::


`int (*compare)(void*, void*)`
En：declare `compare` as pointer to function (pointer to `void`, pointer to `void`) returning `int`
Ch：宣告一個`compare`的指標函數，其參數為(`*void`, `*void`),其回傳值為`int`



`void **(*d) (int &, char **(*)(char *, char **))`
英文：declare `d` as pointer to function(ref to `int`, pointer to function (pointer to `char`, pointer to pointer to `char`) returning pointer to pointer to `char`) returning pointer to pointer to `void`.



`int ptr[]()`
不合法的宣告，宣告一個ptr為陣列其元素為函數，要修改為其元素為指標函數。  
`int (*ptr[])()`  declare ptr as the array of pointer to function returning int


## Reference
> [C 語言:輕鬆讀懂複雜的宣告式 (Define and Read the complex declarations)](https://magicjackting.pixnet.net/blog/post/60889356#note4)
> [所不知道的C語言：指標篇](https://hackmd.io/@sysprog/c-pointer) 
> [cdecl](https://cdecl.org/) 