# SQL injection

magic_quotes_gpc=on 跟 addslashes() 的區別在於前者只會在執行SQL時對`'` `"` `\` `NULL`加上`\`，並且前者從php 5.4開始預設為off

addslash()有個漏洞 可以參考這篇 寫的很清楚
http://shiflett.org/blog/2006/addslashes-versus-mysql-real-escape-string
但這漏洞的條件是server那邊要把client的編碼設成GBK想要安全的使用addslash
想要安全的使用addslash 可以參考 http://www.t086.com/article/5027 最下面

mysql_real_escape_string貌似也有類似的漏洞 但沒搞懂 ~~也懶得搞懂~~

總而言之addslash()跟mysql_real_escape_string()都存在著漏洞，所以現在大家都用mysqli_real_escape_string

使用預編譯基本上也可以很好的防範SQL injection
![](https://i.imgur.com/vUn4Rlm.png)

</br>

接下來列出幾種繞過的方法

</br>

有的時候可以用類似這種形式
```http://localhost/injection/user.php?username=CHAR(97,100, 109, 105, 110, 35)```
來繞過字串驗證

當and or被轉義時
可以嘗試用&& || 代替

strstr會區分大小寫
所以如果有人寫`strstr(id, "union")`
就可以用 uNion之類的繞過 ~~廢話~~

過濾空格的話可以用`/**/`或是`%0A`(windows則用`%0A%0D`)

有的時候過濾不會寫在php之類的後端語言上，如果是用C/C++寫的話
可以用null byte斷開string
![](https://i.imgur.com/goUIEsl.png)


reference: https://www.anquanke.com/post/id/86005
