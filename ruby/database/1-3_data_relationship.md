## 認識資料的三種關係
* 辨識出關聯式資料庫裡，資料間的關係是屬於哪一個種類
> JC 大這章節有特別講！

資料間的三種關係：  
* 一對一 One to One
* 一對多 One to Many
* 多對多 Many to Many

## One to One 一對一的關係
- `一個使用者`只會有`一個購物車`
___
## One to Many 一對多的關係
- `一個分類`會有`很多商品`  
- `一本相簿`擁有`多個照片`
___
## Many to Many 多對多的關係
### 多對多會有兩個一對多的關係
- `同一個使用者`可以參加`很多個活動`
- `同一個活動`可以有`很多使用者`參加
>使用者與活動就是多對多的關係

___
## 如何辨識資料之間的關係？
資料之間的關係會根據 app 的需求而不同，如何辨識關係的種類  
最基本的思考方式是詢問目前app 需求的情境下：  
* 資料A是否有多筆資料 B？
* 資料B是否有多筆資料 A？

>如果兩個問題都是 NO，代表資料 A 和 B 是一對一的關係。

>如果其中一個是 NO，而另一個是 YES，那麼就是一對多的關係，看哪個資料是多的那一方。

>如果兩個問題都是 YES，那麼就是多對多的關係。

___


<!-- 到此時間累計爲 2hr, 本章節耗時36mins -->
