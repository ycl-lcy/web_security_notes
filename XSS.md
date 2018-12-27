# XSS 防禦

XSS Filter是個很複雜的東西，如果要做到完全防範的話，必須依據狀況來escape輸入，否則就可能發生如下的狀況:

`<a href=# onclick="alert('$var')">test</a>`
若執行賦值 `$var = htmlescape("');alert('2")`
於是原本的link就變成寫到網頁上
`<a href=# onclick="alert('&#x27;);alert(&#x27;2')">test</a>`

當瀏覽器接收到render出來的網頁後，會先跑html parser，把html entity unescape成純文字，變成
`<a href=# onclick="alert('');alert('2')">test</a>`
接著誘導使用者點了連結後跑js，就成功XSS了


那麼我們該怎麼預防剛才講到的`onclick="jscode"`這類型的XSS?
就我的理解而言，只要在輸出時確保js code是被正確地escape過就好，例如把剛剛的htmlescape換成jsescape，那麼`');alert('2`就會變成`\');alert(\'2`，而js parser會再把它unescape後當成純文字，因此jscode就不會被執行(這個方法同樣也適用於`<script>jscode</script>`) 

這邊必須注意幾點
1.會先跑html parser再跑js parser
2.每個parser針對unescape的部分，會獨立各自去找對應的prefix，把它unescape後把它視為純文字，但是parser unescape出來的東西，可能會被其他parser當成code執行(這邊舉個例子，不過由於是DOM-Based的XSS，所以放在最下面)

另外針對URL的XSS舉一個例子
如果是`<a href="http://xxx.com/?test=$var">test</a>`
只要把`$var`經過URL escape就可以了
但是如果是`<a href="$var">test</a>`
把整個`$var`escape後會造成`http://`也一起被escape掉，所以要特判掉

**總而言之，確保輸出是被escape，因為被html/js在parse的時候會把這些東西unescape並當成純文字(不過可能會被其他parser當作code)，這點非常重要**

接著要講一個比較特殊的XSS，稱作DOM-Based XSS，上面確保escape的方法對這個是不管用的
舉個例子，如果這段code本來就存在於網站中
```
<script>
    document.write("<a href='" + $var + "'>test</a>");
</script>
```
這樣一看就知道可以XSS(`$var="' onclick='alert(1)"`)，因為它並沒有對$var做escape

但是如果對$var做js escape，嘗試看看能不能防禦
```
<script>
    $var = "\x27\x20\x6f\x6e\x63\x6c\x69\x63\x6b\x3d\x27\x61\x6c\x65\x72\x74\x28\x31\x29"; 
    //$var = "' onclick='alert(1)"
    document.write("<a href='" + $var + "'>test</a>");
</script>
```
流程是這樣的 

1.瀏覽器接收到網頁
2.跑js parser使得那一串編碼被unescape成純文字`' onclick='alert(1)`
3.執行`<script></script>`裡面的東西，也就是`document.write(<a href='' onclick='alert(1)'>test</a>)`
4.再跑一次js parser(因為頁面有東西被append上去了)，不過論unenscape而言有跑等於沒跑，因為沒有unescape後會改變的東西
5.誘導使用者點連結->XSS

所以這個方法防禦不了

至於該怎麼正確的防禦DOM-Based XSS，先暫時留著之後再寫

<!--

在這裡整理一下DOM-Based XSS的正確防禦方法:

1.如果`document.write($XSS)`裡的XSS是利用event或是script標籤，就對`$XSS`做jsencode
2.如果`document.write($XSS)`裡的XSS是利用event或是script標籤，就對`$XSS`做html encode
p.s `$XSS`可控，攻擊者可以決定要用1還是2，我們也可以根據攻擊者的方法來進行相對應的防禦
-->



例子(這個是DOM-Based的XSS)

```
<script>
    x1 = "&apos;&#x20;&#x6F;&#x6E;&#x63;&#x6C;&#x69;&#x63;&#x6B;&equals;&apos;&#x61;&#x6C;&#x65;&#x72;&#x74;&lpar;&#x31;&rpar;&semi;"  // -> ' onclick='alert(1);
    document.write("<a href='" + x1 + "'>test</a>");
    x2 = "&#x31;&rpar;&semi;&#x61;&#x6C;&#x65;&#x72;&#x74;&lpar;&#x32;&rpar;&semi;&sol;&sol;" // -> 1);alert(2);//
    document.write("<a href='' onclick='alert(" + x2 + ")' >test</a>");
</script>
```

x1的部分會在write後被html parser unescape成純文字，因此`' onclick='alert(1);`就因為被視為純文字，所以XSS會失敗
x2的部分雖然說也是在write後被html parser unescape成純文字，但是這只會被html parser視為純文字，對js parser來說`1);alert(2);//`就是一段js code，因此XSS會成功


reference: 白帽子講web安全
