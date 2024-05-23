5月13日：

## BUUCTF-Crypto-看我回旋踢

ROT13编码
ROT13（回转13位，rotate by 13 places，有时中间加了个连字符称作ROT-13）是一种简易的替换式密码。

ROT13被描述成“杂志字谜上下颠倒解答的Usenet点对点体”。ROT13 也是过去在古罗马开发的凯撒加密的一种变体。

特点：
套用ROT13到一段文字上仅仅只需要检查字元字母顺序并取代它在13位之后的对应字母， 有需要超过时则重新绕回26英文字母开头即可。

A换成N、B换成O、依此类推到M换成Z，然后序列反转：N换成A、O换成B、最后Z换成M。
只有这些出现在英文字母里头的字元受影响；
数字、符号、空白字元以及所有其他字元都不变。

ROT13函数是它自己的逆反

ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz
NOPQRSTUVWXYZABCDEFGHIJKLMnopqrstuvwxyzabcdefghijklm
解题：
看到题目我傻眼了一会儿，这嘛玩意儿？？
自己实在是太菜了

题目：synt{5pq1004q-86n5-46q8-o720-oro5on0417r1}

直接百度可疑字符：synt编码
看到搜索栏里跳出rot13
在线解密（贼好用）：http://www.rot13.de/index.php
得到flag：flag{5cd1004d-86a5-46d8-b720-beb5ba0417e1}
（不知道靠不靠谱的）小结：
遇到synt{xx-xx-xx} 判定他为rot13；
题目一般都是提示信息，“回旋踢”就暗示了rot13的特点，会重新绕回；
好耶！

## buuctf:  变异凯撒

凯撒密码作为最古老的密码体制之一,相信大多数人都知道,即简单的移位操作,那么问题来了,变异的凯撒密码又会是什么呢?
拿到题先比对acsii码值对比表:acsii码
通过上面的acsii码值对比表可以看到第一个字符向后移了5,第二个向后移了6,第三个向后移了7,以此类推,很容易想到变异凯撒即每个向后移的位数是前一个加1:
str="afZ_r9VYfScOeO_UL^RWUc"
k=5
for i in str:
    print(chr(ord(i)+k),end='')
    k+=1

## buuctf:rabbit

rabbit解密：

[在线Rabbit加密 | Rabbit解密- 在线工具 (sojson.com)](https://www.sojson.com/encrypt_rabbit.html)



## buuctf：rsa

### 知识点：rsa：

RSA算法参数
参数	解释	    公式	                   描述
P、Q	质数	   P*Q=N	          分解模数N后得到的值
N	   公共模数   N=P*Q	          在RAS中进行模运算
E	    公钥指数   gcd(φ,e)=1	  1<e<φ
D	    私钥指数  gcd(φ,d)=1	  1<d<φ
φ	    欧拉公式  φ=(P-1)*(Q-1)	    /
c	    密文	       c=me mod N	    /
d	    明文	      m=cd mod N	    /

#### 加密算法：密文

由上面的参数表可知，RSA加密过程可以用下面这个公式表示

> c ≡ m e mod N

也就是说，RSA的密文是对代表明文的数字 m 的 e 次方对 N 求余的结果

#### 公钥

上述加密算法中出现的两个数—— E 和 N ，到底是什么数呢？由上面的加密算法可知，在已知明文的情况下，只要只要 E 和 N 这两个数，任何人都可以完成对加密的运算。所以说， E 和 N 是RSA加密的密钥，换句话说，E 和 N 的组合就是公钥，表示为公钥是(E,N)

公钥是可以任意公开的

#### **私钥**

参考公钥的解释，私钥也就明了了。在已知密文的情况下，只要只要 D 和 N 这两个数，任何人都可以完成对解密的运算。所以说， D 和 N 是RSA解密的密钥，换句话说，D 和 N 的组合就是私钥，表示为私钥是(D,N)

私钥必须妥善保管，不能告诉任何人

![在这里插入图片描述](https://img-blog.csdnimg.cn/0583fa1d5b7a414ebd8c78c9f2c5376c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6KW_55S15Y2i5pys5Lyf,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 生成密钥对

由上述分析可知，加解密的过程需要三个参数 E、D、N，那么这三个参数该怎么生成呢？由于 E 和 N 是公钥，D 和 N 是私钥，求这三个数的过程就是生成密钥对。生成步骤如下：

1、求N
2、求φ
3、求E
4、求D

##### 求N

首先准备两个很大的质数 P、Q。上述所说的算法安全性与大数分解有关，就体现在这了，我们反过来想，如果 p 和 q 选的很小，那对于 N 的求解将会变得非常简单，密码就容易被破译；但是物极必反，也不能选的太大，太大会使得计算时间大大延长。

准备好之后，N 由以下公式求得

P*Q=N

##### 求φ

φ这个数在加密解密的过程中都不曾出现，它只出现在生成密钥对的过程中。由下列公式求得

φ=(P-1)*(Q-1)

##### 求E

e 的求取基于上述的 φ 值。首先明确 e 的取值范围 1<e<φ，并且 gcd(φ,e)=1（φ 与 e 的最大公约数为1，即两者互质）。之所以要加上1这个条件，是为了保证一定存在解密时需要使用的参数 D

至此，我们生成了密钥对中的公钥

##### 求D

d 的求取基于上述的 e 值和 φ 值。首先明确 d 的取值范围 1<d<φ，并且 gcd(φ,d)=1。d 的求解方式如下

e*d mod φ=1

至此，我们生成了密钥对中的私钥



[RSA加密算法-CSDN博客](https://blog.csdn.net/lbwnbnbnbnbnbnbn/article/details/124173910?ops_request_misc=%7B%22request%5Fid%22%3A%22171560240416800215061091%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=171560240416800215061091&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-124173910-null-null.142^v100^pc_search_result_base5&utm_term=RSA&spm=1018.2226.3001.4187)

# 明天再理解

在一次RSA密钥对生成中，假设p=473398607161，q=4511491，e=17
求解出d作为flga提交

```python
import gmpy2
p = 473398607161
q = 4511491
e = 17
d = int(gmpy2.invert(e, (p-1)*(q-1)))
print(d)
```

## buuctf crypto-丢失的MD5

2.解压压缩包得到一个py文件

3.尝试用pycharm打开文件得到一串py代码

4.解析py代码

python3
print des -格式错误
改为
print(des)
运行

import hashlib   
for i in range(32,127):
    for j in range(32,127):
        for k in range(32,127):
            m=hashlib.md5()
            m.update('TASC'+chr(i)+'O3RJMV'+chr(j)+'WDJKX'+chr(k)+'ZM')
            des=m.hexdigest()
            if 'e9032' in des and 'da' in des and '911513' in des:
                print(des)

发现结果为
原因是



 import hashlib
for i in range(32,127):
    for j in range(32,127):
        for k in range(32,127):
            m=hashlib.md5()
            m.update('TASC'.encode('utf-8')+chr(i).encode('utf-8')+'O3RJMV'.encode('utf-8')+chr(j).encode('utf-8')+'WDJKX'.encode('utf-8')+chr(k).encode('utf-8')+'ZM'.encode('utf-8'))
            des=m.hexdigest()
            if 'e9032' in des and 'da' in des and '911513' in des:
                print (des)

得到结果

e9032994dabac08080091151380478a2
即flag{e9032994dabac08080091151380478a2}

## buuctf:rsarsa

```
import gmpy2
p = 9648423029010515676590551740010426534945737639235739800643989352039852507298491399561035009163427050370107570733633350911691280297777160200625281665378483
q = 11874843837980297032092405848653656852760910154543380907650040190704283358909208578251063047732443992230647903887510065547947313543299303261986053486569407
e = 65537
c = 83208298995174604174773590298203639360540024871256126892889661345742403314929861939100492666605647316646576486526217457006376842280869728581726746401583705899941768214138742259689334840735633553053887641847651173776251820293087212885670180367406807406765923638973161375817392737747832762751690104423869019034
n = p*q
f = (p-1)*(q-1)
d = int(gmpy2.invert(e,f))
m = pow(c,d,n)     #pow(c,d,n)的内部运算
print('明文:',m)
```

[pow(c,d,n)的底层实现原理](https://www.cnblogs.com/linshuhui/p/9885696.html)