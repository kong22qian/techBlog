[TOC]

# IT base info

## 计算机组成

> https://blog.csdn.net/weixin_48720080/article/details/124962455



## 网络基础

> https://blog.csdn.net/qq_47200222/article/details/123155329

![img](https://img-blog.csdnimg.cn/img_convert/e4e293ca89167cab225838ce6f25b941.png)

## ==http & https:==

> https://blog.csdn.net/fedorafrog/article/details/113991947

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019091510262488.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppbmppbmlhbzE=,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190915102641902.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppbmppbmlhbzE=,size_16,color_FFFFFF,t_70)



> 使用对称密钥的好处是解密的效率比较快，使用非对称密钥的好处是可以使得传输的内容不能被破解，因为就算你拦截到了数据，但是没有对应的私钥，也是不能破解内容的。就比如说你抢到了一个保险柜，但是没有保险柜的钥匙也不能打开保险柜。那我们就将对称加密与非对称加密结合起来,充分利用两者各自的优势，**在交换密钥环节使用非对称加密方式，之后的建立通信交换报文阶段则使用对称加密方式**。
>
> 具体做法是：**发送密文的一方使用对方的公钥进行加密处理“对称的密钥”，然后对方用自己的私钥解密拿到“对称的密钥”，这样可以确保交换的密钥是安全的前提下，使用对称加密方式进行通信**。所以，HTTPS采用对称加密和非对称加密两者并用的混合加密机制.



## HTTP和HTTPS简单介绍(TCP==三次握手四次挥手==和==TLS 2或4 次握手==)

> ==**`https://blog.csdn.net/weixin_43113679/article/details/124311474`**==
