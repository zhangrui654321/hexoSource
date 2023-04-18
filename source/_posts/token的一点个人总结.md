## token的个人总结

#### token的组成

1.header 元数据，定义token的类型和加密算法

2.payload token的具体数据，比如userid就是放在payload中

3.sign 签名 服务器通过`Payload`、`Header`和一个密钥(`secret`)使用 `Header` 里面指定的签名算法（默认是 HMAC SHA256）生成。

### 生成token方法

调用 jwt.creat()方法生成，具体如下，先用SHA256算法将sercret加密，放入sign中，将userid放入payload中，定义header的加密算法和token类型

![](images/220708-1.png)

### 解析token方法，并获取userid

主要是先根据sign获取到JWTVerifier对象，再获取DecodeJwt对象，最后将payload中uerid取出

![](images/220708-2.png)

总之，token是根据签名的加密算法生成sign，再根据sign反解析得到JWTVerifier，并可以把userid放入clamims中

### token的一些好处

1.无状态

2.有效避免了CSRF 攻击（大部分情况下存放在 local storage ）

