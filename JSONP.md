# JSONP 相關漏洞

關於JSONP的調用方法可以參考這篇
https://xz.aliyun.com/t/224

基本上如果要利用這個拿到使用者的從server取回的東西，都必須配合XSS或是CSRF

XSS的話就是單純構造出一個可以替你傳資料回來的script
ex: `http://127.0.0.1/getUsers.php?callback=<script>alert(/xss/)</script>`
防禦方法基本上就是server那邊要過濾callback，不過也有其他辦法，在reference那邊裡，還沒看懂

### JSONP+CSRF
```
<script>
function wooyun(v){
    alert(v.username);
}
</script>
<script src="http://js.login.360.cn/?o=sso&m=info&func=wooyun"></script>
```
我們可以做一個惡意網站，讓使用者連進來以後執行最下面的script，這時候我們就可以利用使用者擁有登入權限這點，讓他把用JSOP收到的資料v，用我們自己的script接收(如果我們不假以使用者的身分，會沒辦法從那個網站收到資料)

防禦方法:
1. 讓server端檢查referer，就會檢查到referer是個惡意的網站，但如果檢查的不夠周全的話，我們仍然可以構造出bypass這個檢查的網站(詳細可見reference的 "Referer 过滤（正则）不严谨")(其實我們也可以讓referer為空來bypass)
2. 讓server隨機產出token，然後放到前端網頁中，讓使用者拿到這個token，夾帶這個token去做JSONP，攻擊者一般來說是拿不到這個token的

reference: http://blog.knownsec.com/2015/03/jsonp_security_technic/



