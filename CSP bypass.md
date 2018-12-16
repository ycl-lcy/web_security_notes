# CSP bypass

CSP就是為了防止瀏覽器去跑惡意的js而設下的規則，只能執行符合特定條件的js


`Content-Security-Policy: default-src 'self '; script-src http://127.0.0.1/static/ `
如果static裡面有個可控的302 php，那麼就可以跳轉到放著我們想要執行的js的頁面

對於那些根據input來執行的js(並且這個js會引發xss)來說 CSP是沒有作用的

`script-src *` ，預設還是會擋掉 inline script

`Content-Security-Policy: default-src 'self '; script-src 'self' 'unsafe-inline'`
這是最普遍的規則 unsafe-inline代表可以跑`<script>blablabla</script>`
可以利用`<script>在dom上append一些元素 ex: img style(有src屬性的) 或是link prefetch</script>` 來把你想傳出的資料(cookie)送出去
除此之外也可以用跳轉頁面的方式 以及jsonp和CORS(後者沒看懂 也沒試過)

Reference: https://paper.seebug.org/423/

因為以上種種繞CSP的方式，於是之後CSP採用了用nonce或是hash來驗證js的合法性，以下就簡單說明一下


nonce: 
Content-Security-Policy: script-src 'nonce-EDNnf03nceIOfn39fn3e9h3sdfa'
`<script nonce=EDNnf03nceIOfn39fn3e9h3sdfa></script>`
只有上面的script(指定了nonce)才會被執行，並且每次頁面刷新時都會變更nonce
此外，CSP允許具有正確nonce的script去加載沒有nonce的script，

`script-src 'strict-dynamic'` 代表任何除了nonce和hash的白名單都將被忽略 ex: self, http://127.0.0.1/static/, unsafe-inline

如何bypass nonce＆strict-dynamic:
```
<meta http-equiv="Content-Security-Policy" content="default-src 'none';script-src 'nonce-secret' 'strict-dynamic'">
<script data-main="data:,alert(1)"></script>
<script nonce="secret" src="require.js"></script>
```
條件是對方有用指定正確nonce的require.js，接著它就會去data-main裡面執行script，即使這個script沒有指定nonce
Reference的第二篇有介紹如何在沒有require.js的狀況下利用firefox的漏洞bypass nonce＆strict-dynamic(沒看懂)

這篇也是在講怎麼bypass nonce＆strict-dynamic
http://sirdarckcat.blogspot.com/2016/12/how-to-bypass-csp-nonces-with-dom-xss.html

總而言之，用黑名單會比白名單穩一些

Reference: 
https://itw01.com/QY8WSEN.html, https://www.anquanke.com/post/id/146180
