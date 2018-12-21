# XSS 防禦

將cookie設成httponly可以讓js沒辦法存取它，是否這麼做將取決於網站本身的功能是否需要js來存取cookie ~~有點廢話~~


XSS Filter是個很複雜的東西，如果要做到完全防範的話，必須依據狀況來encode輸入(常見的有htmlencode跟jsencode兩種)，否則就可能發生如下的狀況:

`<a href=# onclick="alert('$var')">test</a>`
若執行賦值 `$var = htmlencode("');alert('2")` 
//這裡的htmlencode是用[這個網站](https://mothereff.in/html-entities)的，與原文的不同之處在於它並沒有將括弧()轉義
於是原本的link就變成寫到網頁上
`<a href=# onclick="alert('&#x27;);alert(&#x27;2')">test</a>`

當瀏覽器接收到render出來的網頁後，會先跑htmlparser，把html entity decode成純文字，變成
`<a href=# onclick="alert('');alert('2')">test</a>`
接著誘導使用者點了連結後跑javascript，就成功XSS了

<!--
如果全部的東西都會被html decode過一遍，那這樣要怎麼在網頁上顯示純文字的tag 例如`<a href="aaa">aaa</a>`
答案是 `&#x3C;a  href="aaa">aaa&#x3C;/a>`(其中一種作法)
因為html parser在做的事情其實就是把html entity decode後變成純文字，所以它不再是html的語意了-->

那麼我們該怎麼預防剛才講到的`onclick="jscode"`這類型的XSS?
就我的理解而言，只要在輸出時確保jscode是被正確地encode過就好，例如把剛剛的htmlencode換成jsencode，那麼`');alert('2`就會變成`\');alert(\'2`，而jsparser接收到輸出後會再把它decode後當成純文字，因此jscode就不會被執行(這個方法同樣也適用於`<script>jscode</script>`) 

p.s. jsencode的方法可以看[這裡](https://www.freeformatter.com/javascript-escape.html#ad-output) **待補充 jsencode好像可以分成三種?**

另外針對URL的XSS舉一個例子
如果是`<a href="http://xxx.com/?test=$var">test</a>`
只要把`$var`經過URL encode就可以了
但是如果是`<a href="$var">test</a>`
把整個`$var`encode後會造成`http://`也一起被encode掉，所以要特判掉

接著要講一個比較特殊的XSS，稱作DOM-Based XSS，上面的方法對這個是不管用的
舉個例子，如果這段code本來就存在於網站中
```
<script>
    document.write("<a href='" + $var + "'>test</a>");
</script>
```
這樣一看就知道可以XSS，因為它並沒有對$var做encode

但是如果這樣對他做jsencode
```
<script>
    $var = "\x27\x20\x6f\x6e\x63\x6c\x69\x63\x6b\x3d\x27\x61\x6c\x65\x72\x74\x28\x31\x29";
    document.write("<a href='" + $var + "'>test</a>");
</script>
```
流程是這樣的 瀏覽器接收到這個網頁->跑jsparser使得那一串編碼變成
純文字`' onclick='alert(1)`->開始執行<script></script>裡面的東西，也就是`document.write(<a href='' onclick='alert(1)'>test</a>)`->誘導使用者點連結->XSS
所以這個方法防禦不了，同樣地，做html encode也是行不通的



**總而言之，確保輸出是被encode這點非常重要，並且輸出後是先執行htmlparser再執行jsparser，而parser的輸出是純文字(不會被執行)**


reference: 白帽子講web安全
