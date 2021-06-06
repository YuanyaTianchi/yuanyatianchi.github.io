

+++

title = "crypto"
description = "crypto"
tags = ["it","go","lib"]

+++

# base

一次简单但完整的 SSL/TLS 实现：http://www.cnblogs.com/JeffreySun/archive/2010/06/24/1627247.html

- client 向 server 请求数字证书
- server 向 client 发送数字证书
- client 对 server 证书进行验证，验证正确后向 server 发送一个随机字符串 `randString`
  - 数字证书内容主要包括：
    - Issuer：证书的发布机构
    - Valid from , Valid to：证书的有效期
    - Public key：公钥
    - Subject：主体。证书的所有者，一般是某个人或者某个公司名称、机构的名称、公司网站的网址等
    - Signature algorithm：签名算法。指这个数字证书的数字签名所使用的加密算法
    - Thumbprint, Thumbprint algorithm：指纹 & 指纹算法。指纹即证书的 hash 值，指纹算法即所使用的 hash 算法。为了防止篡改，证书的发布机构将使用其私钥进行加密后的密文写入证书。即证书中此项实际写入内容为`server证书的hash值 & 所使用的hash算法 通过CA机构的私钥加密得到的密文`，也表示 CA 机构在 server 证书上的签名
  - 注意：client 持有一系列世界上值得信赖的 CA 机构的证书（操作系统或浏览器自带），证书中包含有 CA 机构的公钥
  - 获取 CA 证书：通过 server 证书的 Issuer 信息，在本地（操作系统或浏览器）获取该 CA 机构的证书信息（如果有的话）
  - 校验 server 证书
    - 知道了 CA 机构对 server 证书所使用的签名算法`Signature algorithm`，就可以使用 CA 证书的`Public key`对`Thumbprint, Thumbprint algorithm`进行解密，得到`server证书的hash值 & 所使用的hash算法`
    - 通过该 hash 算法对 server 证书进行 hash 值计算，如果`解密得到的hash值 == 计算得到的hash值`，表示该证书正确
- server 对 randString 使用 server 私钥加密，并发送给 client
- client 收到后进行校验，如果`使用server公钥对server私钥加密的randString密文解密 == client自己发送的randString`，则确认 server 正确。然后 client 生成一个对称加密算法及其密钥，并通过server证书上Signature algorithm指定的签名算法使用Public key加密后发送给 server
- server 使用其私钥解密获取到接下来通信使用的对称加密算法&密钥
- client & server 使用对称加密开始安全通信......



## 常识

### 网络数据传输威胁

- 数据窃取：机密性。对称加密，非对称加密
- 数据篡改：完整性。解决：单向三里函数，消息认证码，数字签名
- 身份伪装：伪装成别人，认证。解决：消息认证码，数字签名
- 身份否认：否认某件事不是自己所为，不可否认性。解决：数字签名



### 密码信息安全常识

- 不要使用保密的密码算法，公开的著名的加密算法久经考验，且即使被破解也能得到万众之力的解决
- 任何密码总有一天都会被破解（非对称加密算法由英国安全局1973发明，后由RSA于1977公开）
- 密码只是信息安全的一部分（社会工程学，钓鱼）



### 位

#### 单位

1. 位：bit，0或1，最小的单位。
2. 字节：Byte，1Byte = 8bit
3. 千字节：KByte，1KB = 1024B
4. 兆字节：MByte，1MB = 1024KB
5. 1 GB = 1024MB
6. 1TB = 1024GB
7. 1PB = 1024TB

#### 位运算

异或：xor，异为1，同为0。0 ⊕ 0 = 0，0 ⊕ 1 = 1，1 ⊕ 0 = 1，1 ⊕ 1 = 0。类似于奇偶加法，0为偶，1为奇，偶+偶=偶，偶+奇=奇，奇+奇=偶。所以用⊕



## 对称加密

- 举例：凯撒密码，通过将明文中所使用的字母表按照一定的字数“平移”来进行加解密的
- 过程
  - 发送方：明文 + 密钥 + 加密算法 = 密文
  - 接收方：密文 + 密钥 + 解密算法 = 明文
- 特点：相对非对称加密，效率较高
- 问题：必须约定好密钥，但密钥又不能通过网络传输，密钥本身无法保证不被第三方窃取。不能解决数据窃取问题



#### 分组模式

- 分组模式：分组密码的模式，分组密码是如何迭代的。分组密码与分组模式可以任意组合
  - 分组：以一定bit长度的明文（比特序列，通常与密钥长度一致）作为一个单位进行加密，该单位称为一个分组。以分组为单位进行处理的密码算法称为**分组密码（blockcipher）**
  - 模式：一个明文可能有很多分组，分组密码每次只能加密固定长度的明文（分组），如果需要加密任意长度的明文，就需要对分组密码进行迭代，迭代的具体方式称为模式（mode）
    - 填充：对于需要填充的模式，要按照分组长度来填充
    - 初始化向量iv：对于需要iv的，iv的长度要与分组一致



##### ECB

- Electronic Code Book mode（电子密码本模式）
- 加密：每个分组被单独加密，互不干涉。加密前需要把明文数据填充到块大小的整倍数
  1. 明文分组1 + 加密算法 = 密文分组1
  2. 明文分组n + 加密算法 = 密文分组n
- 解密：
  1. 密文分组1 + 解密算法 = 明文分组1
  2. 密文分组n + 解密算法 = 明文分组n
- 使用： 适用于数据较少的情形。安全性比较差，已淘汰，不建议使用



##### CBC

- Cipher Block Chaining mode（密文分组链接模式）。

- 加密：先异或，后加密。操作全程如链接状。

  1. iv ⊕ 明文分组1 -> 加密算法 => 密文分组1
  2. 密文分组1 ⊕ 明文分组2 -> 加密算法 => 密文分组2
  3. 密文分组n-1 ⊕ 明文分组n -> 加密算法 => 密文分组n

- 解密

  1. 密文分组1 -> 解密算法 ⊕ iv => 明文分组1
  2. 密文分组2 -> 解密算法 ⊕ 密文分组1 => 明文分组2
  3. 密文分组n -> 解密算法 ⊕ 密文分组n-1 => 明文分组n

  ![1538842682058](https://ws3.sinaimg.cn/large/006tNc79ly1fyzch4ouwuj30hp0exac6.jpg)

- 特点

  - 因为有直接对明文进行加密操作，加密前需要把明文数据填充到块大小的整倍数

- 常用，推荐



##### CFB

- Cipher FeedBack mode（密文反馈模式）。

- 加密：先加密，后异或。前一个密文分组，反馈反当前分组密码算法作输入。

  1. 密文分组0（iv） & 加密算法 => 加密输出1，      加密输出1 ⊕ 明文分组1 => 密文分组1
  2. 密文分组1           & 加密算法 => 加密输出2，      加密输出2 ⊕ 明文分组2 => 密文分组2
  3. 密文分组n-1        & 加密算法 => 加密输出n，      加密输出n ⊕ 明文分组n => 密文分组n

  ![1538727477811](https://ws2.sinaimg.cn/large/006tNc79ly1fyzchdhedaj30g808e754.jpg)

- 解密：注意仍然是使用加密算法

  1. 密文分组0（iv） & 加密算法 => 加密输出1，      加密输出1 ⊕ 密文分组1 => 明文分组1
  2. 密文分组1           & 加密算法 => 加密输出2，      加密输出2 ⊕ 密文分组2 => 明文分组2
  3. 密文分组n-1        & 加密算法 => 加密输出n，      加密输出n ⊕ 密文分组n => 明文分组n

  ![1538727501371](https://ws4.sinaimg.cn/large/006tNc79ly1fyzchap416j30ga08hq3t.jpg)

- 特点

  - 整个明文操作过程中没有被直接进行过加密操作，因此无须填充。

- 不推荐使用，建议CTR代替



##### OFB

- Output FeedBack mode（输出反馈模式）

- 加密：上一个分组密码算法的输出，反馈给当前分组密码算法作输入。

  1. 加密输出0（iv） & 加密算法 => 加密输出1，      加密输出1 ⊕ 明文分组1 => 密文分组1
  2. 加密输出1           & 加密算法 => 加密输出2，      加密输出2 ⊕ 明文分组2 => 密文分组2
  3. 加密输出n-1        & 加密算法 => 加密输出n，      加密输出n ⊕ 明文分组n => 密文分组n

- 解密：注意仍然是使用加密算法

  1. 加密输出0（iv） & 加密算法 => 加密输出1，      加密输出1 ⊕ 密文分组1 => 明文分组1
  2. 加密输出1           & 加密算法 => 加密输出2，      加密输出2 ⊕ 密文分组2 => 明文分组2
  3. 加密输出n-1        & 加密算法 => 加密输出n，      加密输出n ⊕ 密文分组n => 明文分组n

  ![1538728323577](https://ws2.sinaimg.cn/large/006tNc79ly1fyzchfcj2aj30gg0g940f.jpg)

- 特点

  - 运算是速度快。密钥流可以提前生成，xor操作顺序任意，可并行进行xor运算
  - 整个明文操作过程中没有被直接进行过加密操作，因此无须填充
  - 建议CTR代替



##### CTR

- CounTeR mode（计数器模式）

- 加密：通过将逐次累加的计数器进行加密，来生成密钥流的流密码

  1. CTR     & 加密算法=>加密输出1，      加密输出1 ⊕ 明文分组1 => 密文分组1
  2. CTR+1 & 加密算法=>加密输出2，      加密输出2 ⊕ 明文分组2 => 密文分组2
  3. CTR+n & 加密算法=>加密输出n，      加密输出n ⊕ 明文分组n => 密文分组n

  ![1538711701763](https://ws4.sinaimg.cn/large/006tNc79ly1fyzchb5a06j30f60bydh3.jpg)

- 解密：注意仍然是使用加密算法

  1. CTR     & 加密算法=>加密输出1，      加密输出1 ⊕ 密文分组1 => 明文分组1
  2. CTR+1 & 加密算法=>加密输出2，      加密输出2 ⊕ 密文分组2 => 明文分组2
  3. CTR+n & 加密算法=>加密输出n，      加密输出n ⊕ 密文分组n => 明文分组n

  ![1538711715415](https://ws1.sinaimg.cn/large/006tNc79ly1fyzch5h2a7j30fa0c10u1.jpg)

- 特点

  - 模拟生成随机的比特序列。每次加密时都会生成一个不同的值（nonce）来作为计数器的初始值，每次所生成的密钥流不同，得到的密文也不同。
  - 当分组长度为128比特（16字节）时，计数器的初始值可能是像下面这样的形式。其中前8个字节为nonce（随机数），这个值在每次加密时必须都是不同的，后8个字节为分组序号，这个部分是会逐次累加的。在加密的过程中，计数器的值会产生如下变化：
    1. 66 1F 98 CD 37 A3 8B 4B 00 00 00 00 00 00 00 01，明文分组1的计数器(初始值)
    2. 66 1F 98 CD 37 A3 8B 4B 00 00 00 00 00 00 00 02，明文分组2的计数器
    3. 66 1F 98 CD 37 A3 8B 4B 00 00 00 00 00 00 00 03，明文分组3的计数器
  - 运算是速度快。密钥流可以提前生成，xor操作顺序任意，可并行进行xor运算
  - 整个明文操作过程中没有被直接进行过加密操作，因此无须填充
  - 常用，建议使用



#### 分组密码

##### DES

- Data Encryption Standard（数据加密标准）
- 内容
  - 密钥：64bit（8byte），每隔7比特设置一个用于错误检查的比特，实质加密位为56bit
  - 分组：64bit
  - 模式
- 特点：已经可以暴力破解，不安全

```go
/*DES & CBC 加密*/
func DesEncrypt_CBC(src, key []byte) []byte { //src明文，key秘钥(8byte)
	block, _ := des.NewCipher(key)                      //创建DES算法的cipher.Block对象
	src = Pkcs5Padding(src, block.BlockSize())          //填充最后一个明文分组
	blackMode := cipher.NewCBCEncrypter(block, key[:8]) //创建CBC分组、DES算法的BlockMode对象
	dst := make([]byte, len(src))
	blackMode.CryptBlocks(dst, src) //加密
	fmt.Println("密文: ", dst)
	return dst
}

/*DES & CBC 解密*/
func DesDecrypt_CBC(src, key []byte) []byte { //src密文，key秘钥(8byte)
	block, _ := des.NewCipher(key)                      //创建DES算法的cipher.Block对象
	blockMode := cipher.NewCBCDecrypter(block, key[:8]) //创建CBC分组、DES算法的BlockMode对象
	dst := make([]byte, len(src))
	blockMode.CryptBlocks(src, dst) //解密
	dst = Pkcs5UnPadding(dst)       //去填充
	fmt.Println("明文: ", dst)
	return dst
}
```



##### 3DES

- triple-DES
- 内容
  - 密钥：24byte（64bit * 3），实质加密位为56bit * 3
  - 分组：8byte（64bit）
  - 模式：密钥分为3段，每段64bit，依次对同一段64bit明文，进行共3次加（解）密操作
  - 加密：先用密钥第1段进行DES加密，再用密钥第2段DES解密（以解密的形式加密），最后用密钥第3段DES加密。3段操作都是对**同一个**分组（64bit）进行的
  - 解密：密钥1段解密，密钥2段加密，密钥3段解密
- 特点：相对DES安全。多次加密效率太低。加-解-加 的方式是为了兼容DES，当密钥1段与密钥3段相同，则相当于一般的DES

```go
/*3DES & CBC 加密*/
func DesEncrypt_CBC(src, key []byte) []byte { //src明文，key秘钥(8byte)
	block, _ := des.NewTripleDESCipher(key)             //创建3DES算法的cipher.Block对象
	src = Pkcs5Padding(src, block.BlockSize())          //填充
	blackMode := cipher.NewCBCEncrypter(block, key[:8]) //创建CBC分组、3DES算法的BlockMode对象
	dst := make([]byte, len(src))
	blackMode.CryptBlocks(dst, src) //加密
	fmt.Println("密文: ", dst)
	return dst
}

/*3DES & CBC 解密*/
func DesDecrypt_CBC(src, key []byte) []byte { //src密文，key秘钥(8byte)
	block, _ := des.NewTripleDESCipher(key)             //创建3DES算法的cipher.Block对象
	blockMode := cipher.NewCBCDecrypter(block, key[:8]) //创建CBC分组、3DES算法的BlockMode对象
	dst := make([]byte, len(src))
	blockMode.CryptBlocks(src, dst) //解密
	dst = Pkcs5UnPadding(dst)       //去填充
	fmt.Println("明文: ", dst)
	return dst
}
```





##### AES

- Advanced Encryption Standard（高级加密标准），Rijndael（密钥生成算法）

- 内容

  - 秘钥：128bit（16byte），192bit（24byte），256bit（32byte）

  - 分组：16字节（128比特）

  - 模式：

  - 加密：SubBytes：字节代换，ShiftRows：行移位代换，MixColumns：列混淆，AddRoundKey：轮密钥加。

    ​		和DES—样，AES算法也是由多个轮所构成的，下图展示了每一轮的大致计算步骤。DES使用Feistel网络作为其基本结构，而AES没有使用Feistel网络，而是使用了SPN Rijndael的输人分组为128比特，也就是16字节。首先，需要逐个字节地对16字节的输入数据进行SubBytes处理。所谓SubBytes,就是以每个字节的值（0～255中的任意值）为索引，从一张拥有256个值的替换表（S-Box）中查找出对应值的处理，也是说，将一个1字节的值替换成另一个1字节的值。

    SubBytes之后需要进行ShiftRows处理，即将SubBytes的输出以字节为单位进行打乱处理。从下图的线我们可以看出，这种打乱处理是有规律的。

    ​		ShiftRows之后需要进行MixCo1umns处理，即对一个4字节的值进行比特运算，将其变为另外一个4字节值。

    ​		最后，需要将MixColumns的输出与轮密钥进行XOR，即进行AddRoundKey处理。到这里，AES的一轮就结東了。实际上，在AES中需要重复进行10 ~ 14轮计算。

    ​		通过上面的结构我们可以发现输入的所有比特在一轮中都会被加密。和每一轮都只加密一半输人的比特的Feistel网络相比，这种方式的优势在于加密所需要的轮数更少。此外，这种方式还有一个优势，即SubBytes，ShiftRows和MixColumns可以分别按字节、行和列为单位进行并行计算。

    ![1538731104755](https://ws3.sinaimg.cn/large/006tNc79ly1fyzcdxdh18j30fb0c375z.jpg)

  - 解密：InvSubBytes：逆字节替代，InvShiftRows：逆行移位，InvMixColumns：逆列混淆

    ​		SubBytes、ShiftRows、MixColumns分别存在反向运算InvSubBytes、InvShiftRows、InvMixColumns，这是因为AES不像Feistel网络一样能够用同一种结构实现加密和解密

    ![1538731285324](https://ws3.sinaimg.cn/large/006tNc79ly1fyzcdxuhkgj30fy0c40uf.jpg)

```go
/*AES & CBC 加密*/
func AESEncrypt(src, key []byte) []byte {
	block, _ := aes.NewCipher(key)                                      //创建AES加密的cipher.Block对象
	src = Pkcs5Padding(src, block.BlockSize())                          //填充
	blockMode := cipher.NewCBCEncrypter(block, key[:block.BlockSize()]) //创建CBC分组、AES加密的BlockMode对象
	dst := make([]byte, len(src))
	blockMode.CryptBlocks(dst, src) //加密
	return dst
}

/*AES & CBC 解密*/
func AESDecrypt(src, key []byte) []byte {
	block, _ := aes.NewCipher(key)                                      //创建AES加密的cipher.Block对象
	blockMode := cipher.NewCBCDecrypter(block, key[:block.BlockSize()]) //创建CBC分组、AES加密的BlockMode对象
	dst := make([]byte, len(src))
	blockMode.CryptBlocks(dst, src) //解密
	dst = Pkcs5UnPadding(dst)       //去填充
	return dst
}
```



#### 填充

##### PKCS5

```go
/*Pkcs5填充*/
func Pkcs5Padding(ciphertext []byte, blockSize int) []byte {
	padding := blockSize - (len(ciphertext) % blockSize)        //计算最后一个分组缺少字节数
	paddingText := bytes.Repeat([]byte{byte(padding)}, padding) //创建一个大小为padding的切片, 每个字节的值为padding
	newCiphertext := append(ciphertext, paddingText...)         //将paddingText添加到原始数据的后
	return newCiphertext
}

/*Pkcs5去填充*/
func Pkcs5UnPadding(origData []byte) []byte {
	length := len(origData)             //计算数据的总长度
	number := int(origData[length-1])   //根据填充的字节值得到填充的次数
	return origData[:(length - number)] //将尾部填充的number个字节去掉
}
```

##### PKCS7



## 非对称加密

- 过程：公钥加密私钥解，私钥加密公钥解
  - A方：A公钥（公开），A私钥 （A私有），B公钥
    - 发送：明文 + B公钥 = B公钥加密密文
    - 接收：A公钥加密密文 + A私钥 = 明文
    - 签名：信息 + A私钥 = A签名信息
  - B方：B公钥（公开），B私钥（B私有），A公钥
    - 发送：明文 + A公钥 =  A公钥加密密文
    - 接收：B公钥加密密文 + B私钥 = 明文
    - 签名：信息 + B私钥 = B签名信息
  - 第三方：A公钥，B公钥
    - 即使截取到密文，也不能解密任何信息
- 分工
  - 公钥：加密，签名（对方公钥加密也可以用于用于身份验证，验证过程是：A用B的公钥加密数据后将密文传输给B，B用自己的私钥进行解密并将明文发送回给A，A对比B返回的明文和自己加密前的明文一致则表示对B完成了身份验证，通过SSH进行免密钥登录时就是通过这种方式来完成用户身份验证的）
  - 私钥：加密，解密，签名（私钥加密，你的公钥能解开，证明信息属于你）
- 特点：相对对称加密，加密效率较低，一般不作大量数据使用
- 问题：
- 使用：
  - **公钥加密算法很少用于数据加密，它通常只是用来做身份认证**，因为它的密钥太长，加密速度太慢--公钥加密算法的速度甚至比对称加密算法的速度慢上3个数量级（1000倍）
    1. 建立连接之初，通过非对称加密协商对称加密的密钥和算法
    2. 之后通过对称加密通信
  - http：验证服务器，数字证书
  - 签名：哈希+非对称加密
  - 网银U盾：验证client，U盾相当于私钥，公钥在服务端
  - github ssh(secure shell)登录

#### base64编码

- 简介：Base64编码，是我们程序开发中经常使用到的编码方法。因为base64编码的字符串，更适合不同平台、不同语言的传输（一个字符可能其他的系统没有）。它是一种基于用64个可打印字符来表示二进制数据的表示方法。它通常用作存储、传输一些二进制数据编码方法，一句：将二进制数据文本化（转成ASCII）。

- 作用：由于某些系统中只能使用ASCII字符。Base64就是用来将非ASCII字符的数据转换成ASCII字符的一种方法。

- 原理：Base64编码要求把3个8位字节（3\*8=24）转化为4个6位的字节（4*6=24），之后在6位的前面补两个0，形成8位一个字节的形式。 如果剩下的字符不足3个字节，则用0填充，输出字符使用'='，因此编码后输出的文本末尾可能会出现1或2个'=’。

- 由来：为了保证所输出的编码位可读字符，Base64制定了一个编码表，以便进行统一转换。编码表的大小为2^6=64，这也是Base64名称的由来。

- 编码表：![Picture1](https://ws2.sinaimg.cn/large/0069RVTdly1fuv29ohtc1j30l70edtc5.jpg)

- 3字节情况：![Picture1](https://ws4.sinaimg.cn/large/0069RVTdly1fuv2ak55sxj30l8049aav.jpg)

- 不足3字节情况：![Picture1](https://ws2.sinaimg.cn/large/0069RVTdly1fuv2b07jcwj30l707a40e.jpg)

- 特点：

  - base64就是一种基于64个可打印字符来表示二进制数据的方法。
  - Z（26）、a-z（26）、0-9（10）、+/（2）：共计64个字符。
  - 编码后便于传输，尤其是不可见字符或特殊字符，对端接收后解码即可复原。
  - base64只是编码，并不具有加密作用。

- 代码：

  ```go
  func main() {
  	input := []byte("区块链学员中有一群神秘的大佬，他们家里有矿！")
  	encodeString := base64.StdEncoding.EncodeToString(input) //base64编码
  	fmt.Println(encodeString)
  	decodeBytes, _ := base64.StdEncoding.DecodeString(encodeString) //base64解码
  	fmt.Println(string(decodeBytes))
  
  	uEnc := base64.URLEncoding.EncodeToString([]byte(input)) //如果要用在url中，需要使用URLEncoding
  	fmt.Println(uEnc)
  	uDec, _ := base64.URLEncoding.DecodeString(uEnc)
  	fmt.Println(string(uDec))
  }
  ```

#### 

#### RSA

```go
/*生成RSA密钥对*/
func GenerateRsaKey(bits int) error { //bits指定生成的秘钥的长度，单位: bit
	privateKey, _ := rsa.GenerateKey(rand.Reader, bits) //GenerateKey(Reader是一个全局共享密码用强随机数生成器,秘钥的位数)生成RSA密钥对

	//生成私钥文件
	derStream := x509.MarshalPKCS1PrivateKey(privateKey) //MarshalPKCS1PrivateKey将rsa私钥序列化为ASN.1 PKCS#1 DER编码
	block := pem.Block{ //Block代表PEM编码的结构, 对其进行设置
		Type:  "RSA PRIVATE KEY", //"RSA PRIVATE KEY",
		Bytes: derStream,
	}
	privateKeyFile, _ := os.Create("private.pem") //创建文件
	err := pem.Encode(privateKeyFile, &block)     //使用pem编码, 并将数据写入文件中
	if err != nil {
		return err
	}
	defer privateKeyFile.Close()

	//生成公钥文件
	publicKey := privateKey.PublicKey
	derPkix, _ := x509.MarshalPKIXPublicKey(&publicKey)
	block = pem.Block{
		Type:  "RSA PUBLIC KEY", //"PUBLIC KEY",
		Bytes: derPkix,
	}
	publicKeyFile, _ := os.Create("public.pem")
	err = pem.Encode(publicKeyFile, &block) //编码公钥, 写入文件
	if err != nil {
		panic(err)
		return err
	}
	defer publicKeyFile.Close()

	return nil
}

/*RSA公钥加密*/
func RSAEncrypt(src, filename []byte) []byte {
	//根据文件名将文件内容从文件中读出
	file, _ := os.Open(string(filename))
	info, _ := file.Stat()
	allText := make([]byte, info.Size())
	file.Read(allText)
	file.Close()

	block, _ := pem.Decode(allText) //从数据中查找到下一个PEM格式的块
	if block == nil {
		return nil
	}
	publicInterface, _ := x509.ParsePKIXPublicKey(block.Bytes) //解析一个DER编码的公钥
	publicKey := publicInterface.(*rsa.PublicKey)
	result, _ := rsa.EncryptPKCS1v15(rand.Reader, publicKey, src) //公钥加密
	return result
}

/*RSA私钥解密*/
func RSADecrypt(src, filename []byte) []byte {
	//根据文件名将文件内容从文件中读出
	file, _ := os.Open(string(filename))
	info, _ := file.Stat()
	allText := make([]byte, info.Size())
	file.Read(allText)
	file.Close()

	block, _ := pem.Decode(allText)                                //从数据中查找到下一个PEM格式的块
	privateKey, _ := x509.ParsePKCS1PrivateKey(block.Bytes)        //解析一个pem格式的私钥
	result, _ := rsa.DecryptPKCS1v15(rand.Reader, privateKey, src) //私钥解密
	return result
}
```

- 内容：在RSA中，明文、密钥和密文都是数字

  ![1538732995510](https://ws4.sinaimg.cn/large/006tNc79ly1fyzci1t18sj30e705jq37.jpg)

  - 加密：**明文的E次方modN**，mod指求余

    - 公式：
      $$
      密文=明文 ^ E mod N（RSA加密）
      $$

    - 公钥：数E（encryption加密）、数N的组合就是公钥

  - 解密：**明文的D次方modN**，D和E是有着紧密联系的

    - 公式：
      $$
      明文=密文^DmodN（RSA解密）
      $$

    - 私钥：数D（Decryption解密）、数N的组合即私钥

- 密钥生成：

  - x509：https://baike.baidu.com/item/x509/1240109
  - pem：https://blog.csdn.net/crjmail/article/details/79095385
  - base64：
    1. x509证书规范、pem、base64
       - pem编码规范 - 数据加密
       - base64 - 对数据编码, 可逆
         - 不管原始数据是什么, 将原始数据使用64个字符来替代
           - a-z  A-Z 0-9 + /
    2. ASN.1抽象语法标记
    3. PKCS1标准

#### ECC

- ECC：Elliptic curve cryptography，椭圆曲线密码体制

- 简介：ECC比其它的方法使用更小的密钥，提供相当或更高等级的安全。

  - 对比：ECC 164位的密钥，安全级相当于RSA 1024位的密钥，且计算量较小、处理速度更快、存储空间较少、传输带宽占用较少。
  - 举例
    - 目前我国居民二代身份证正在使用 256 位的ECC
    - 虚拟货币`比特币`也选择ECC作为加密算法

- 公式：
  $$
  y^2=x^3+ax+b
  $$

- 图解

  - 普通曲线：<img src="https://ws4.sinaimg.cn/large/006tNc79ly1fzbjmwbgh4j30ot0ow3zd.jpg" style="zoom: 25%;" />

  1. 画一条斜线，会产生三个点P，Q，R：<img src="http://7fvhfe.com1.z0.glb.clouddn.com/wp-content/uploads/2015/07/%E6%9B%B2%E7%BA%BF.png" alt="1" style="zoom: 67%;" />

  2. 在椭圆中P + Q + R = 0，（等价于传统意义上的零值）：<img src="http://7fvhfe.com1.z0.glb.clouddn.com/wp-content/uploads/2015/07/%E6%9B%B2%E7%BA%BF-2.png" alt="2" style="zoom: 67%;" />

  3. 在椭圆曲线中，点的相加等同于从该点画切线找到与曲线相交的另一 点，然后翻折到x轴。

     即，当P == Q时，P+Q+R = 2P = -R：<img src="https://ws4.sinaimg.cn/large/006tNc79ly1fzbjz6eckbg308c08ctff.gif" alt="animation-point-doubling" style="zoom:67%;" />

  4. K = n * G，可以看成：K= n * G = G + G + G …(相加n次）

     那么，如果想得到最终的K值，我们需要对G点在曲线上做切线，获取交点，重复做n次。<img src="https://ws3.sinaimg.cn/large/006tNc79ly1fzbk0gsqkfj30p00p3gnh.jpg" alt="4" style="zoom: 33%;" />

  5. 在公式K = n* G中，n是私钥，G和K组成公钥（曲线上的两个点）==



#### PKCS标准

> PKCS 全称是 Public-Key Cryptography Standards ，是由 RSA 实验室与其它安全系统开发商为促进公钥密码的发展而制订的一系列标准。
>
> 可以到官网上看看 [What is PKCS](http://www.rsa.com/rsalabs/node.asp?id=2308)
>
> http://www.rsa.com/rsalabs/node.asp?id=2308

 PKCS 目前共发布过 15 个标准

1. **PKCS#1：RSA加密标准。==PKCS#1定义了RSA公钥函数的基本格式标准，特别是数字签名==。它定义了数字签名如何计算，包括待签名数据和签名本身的格式；它也定义了PSA公/私钥的语法。**
2. PKCS#2：涉及了RSA的消息摘要加密，这已被并入PKCS#1中。
3. PKCS#3：Diffie-Hellman密钥协议标准。PKCS#3描述了一种实现Diffie- Hellman密钥协议的方法。
4. PKCS#4：最初是规定RSA密钥语法的，现已经被包含进PKCS#1中。
5. PKCS#5：基于口令的加密标准。PKCS#5描述了使用由口令生成的密钥来加密8位位组串并产生一个加密的8位位组串的方法。PKCS#5可以用于加密私钥，以便于密钥的安全传输（这在PKCS#8中描述）。
6. PKCS#6：扩展证书语法标准。PKCS#6定义了提供附加实体信息的X.509证书属性扩展的语法（当PKCS#6第一次发布时，X.509还不支持扩展。这些扩展因此被包括在X.509中）。
7. PKCS#7：密码消息语法标准。PKCS#7为使用密码算法的数据规定了通用语法，比如数字签名和数字信封。PKCS#7提供了许多格式选项，包括未加密或签名的格式化消息、已封装（加密）消息、已签名消息和既经过签名又经过加密的消息。
8. PKCS#8：私钥信息语法标准。PKCS#8定义了私钥信息语法和加密私钥语法，其中私钥加密使用了PKCS#5标准。
9. PKCS#9：可选属性类型。PKCS#9定义了PKCS#6扩展证书、PKCS#7数字签名消息、PKCS#8私钥信息和PKCS#10证书签名请求中要用到的可选属性类型。已定义的证书属性包括E-mail地址、无格式姓名、内容类型、消息摘要、签名时间、签名副本（counter signature）、质询口令字和扩展证书属性。
10. PKCS#10：证书请求语法标准。PKCS#10定义了证书请求的语法。证书请求包含了一个唯一识别名、公钥和可选的一组属性，它们一起被请求证书的实体签名（证书管理协议中的PKIX证书请求消息就是一个PKCS#10）。
11. PKCS#11：密码令牌接口标准。PKCS#11或“Cryptoki”为拥有密码信息（如加密密钥和证书）和执行密码学函数的单用户设备定义了一个应用程序接口（API）。智能卡就是实现Cryptoki的典型设备。注意：Cryptoki定义了密码函数接口，但并未指明设备具体如何实现这些函数。而且Cryptoki只说明了密码接口，并未定义对设备来说可能有用的其他接口，如访问设备的文件系统接口。
12. PKCS#12：个人信息交换语法标准。PKCS#12定义了个人身份信息（包括私钥、证书、各种秘密和扩展字段）的格式。PKCS#12有助于传输证书及对应的私钥，于是用户可以在不同设备间移动他们的个人身份信息。
13. PDCS#13：椭圆曲线密码标准。PKCS#13标准当前正在完善之中。它包括椭圆曲线参数的生成和验证、密钥生成和验证、数字签名和公钥加密，还有密钥协定，以及参数、密钥和方案标识的ASN.1语法。
14. PKCS#14：伪随机数产生标准。PKCS#14标准当前正在完善之中。为什么随机数生成也需要建立自己的标准呢？PKI中用到的许多基本的密码学函数，如密钥生成和Diffie-Hellman共享密钥协商，都需要使用随机数。然而，如果“随机数”不是随机的，而是取自一个可预测的取值集合，那么密码学函数就不再是绝对安全了，因为它的取值被限于一个缩小了的值域中。因此，安全伪随机数的生成对于PKI的安全极为关键。
15. **PKCS#15：密码令牌信息语法标准。**PKCS#15通过定义令牌上存储的密码对象的通用格式来增进密码令牌的互操作性。在实现PKCS#15的设备上存储的数据对于使用该设备的所有应用程序来说都是一样的，尽管实际上在内部实现时可能所用的格式不同。PKCS#15的实现扮演了翻译家的角色，它在卡的内部格式与应用程序支持的数据格式间进行转换。

## 单向散列（Hash）

- 概念：单向散列函数，将任意长度消息的比特序列，快速计算出固定长度的散列值。消息的散列值，类似于人的指纹
- 性质：
  - 单向：计算不可逆，无法通过散列值计算出消息的比特序列
  - 抗碰撞：难以发现碰撞的性质（或指不可能被人为地发现碰撞）。
    - 碰撞：collision，也叫Hash冲突，两个不同的消息产生同一个散列值的情况称为碰撞
  - 固定：对于同一个单向散列函数，任意长度消息的比特序列，计算出的散列值都是固定长度的
  - 快速
- 应用：验证消息的散列值，即类似于验证人的指纹
  - 校验软件篡改：官方把通过单向散列函数计算出的散列值公布，用户在下载到软件之后，可以自行计算散列值，然后与官方网站上公布的散列值进行对比。通过散列值，用户可以确认自己所下载到的文件与软件作者所提供的文件是否一致
  - 消息认证码：消息认证码是将“发送者和接收者之间的共享密钥”和“消息，进行混合后计算出的散列值。使用消息认证码可以检测并防止通信过程中的错误、篡改以及伪装。消息认证码在SSL/TLS中也得到了运用
  - 数字签名：**私钥对文件签名时，并不会对文件本身做签名，而是对这个文件的哈希值进行签名**。数字签名是现实社会中的签名（sign）和盖章这样的行为在数字世界中的实现。数字签名的处理过程非常耗时，因此一般不会对整个消息内容直接施加数字签名，而是先通过单向散列函数计算出消息的散列值，然后再对这个散列值施加数字签名。
  - 伪随机数生成器：使用单向散列函数可以构造伪随机数生成器。密码技术中所使用的随机数需要具备“事实上不可能根据过去的随机数列预测未来的随机数列”这样的性质。为保证不可预测性，可以利用单向散列函数的单向性。
  - 一次性口令：one-time password。一次性口令经常被用于服务器对客户端的合法性认证。在这种方式中，通过使用单向散列函数可以保证口令只在通信链路上传送一次（one-time），因此即使窃听者窃取了口令，也无法使用
  - 密码存储：数据库中，对密码的存储并不是密码的明文，而是密码的哈希值，每次登录时，会对密码进行哈希处理，然后与数据库对比，即使数据库被盗，黑客也无法拿到用户的密码，保证用户账户安全
- 常见Hash函数
  - MD：Message Digest，消息摘要
    - MD4：散列值长度128bit。MD4是由Rivest于1990年设计的单向散列函数，能够产生128比特的散列值。随着Dobbertin提出寻找MD4散列碰撞的方法，已经不安全
    - MD5：散列值长度128bit。MD5是由Rwest于1991年设计的单项散列函数，能够产生128比特的散列值。MD5的强抗碰撞性已经被攻破，能够产生具备相同散列值的两条不同的消息，已经不安全
  - SHA：
    - SHA-1：160bit散列值。SHA-1的消息长度存在上限，但这个值接近于2^64^比特，是个非常巨大的数值，因此在实际应用中没有问题。强抗碰撞性已于2005年被攻破
    - SHA-2：尚未被攻破
      - SHA-224：散列值长度224bit，消息长度上限接近于 2^64^ bit
      - **SHA-256**：散列值长度256bit，消息长度上限接近于 2^64^ bit
      - SHA-384：散列值长度384bit，消息长度上限接近于 2^128^ bit
      - SHA-512：散列值长度512bit，消息长度上限接近于 2^128^ bit



#### MD5

```go
func main() {
	fmt.Println("getMD5_1:", getMD5_1([]byte("ltc"))) //getMD5_1: 90d9d5d7e72b9f5f385555845ecb8a25
	fmt.Println("getMD5_2:", getMD5_2([]byte("ltc"))) //getMD5_2: 90d9d5d7e72b9f5f385555845ecb8a25
}

func getMD5_1(str []byte) string {
	result := md5.Sum(str) //返回MD5校验和
	fmt.Println(result)    //[144 217 213 215 231 43 159 95 56 85 85 132 94 203 138 37]。结果

	result16 := fmt.Sprintf("%x", result) //将数据格式化为16进制格式字符串
	fmt.Println(result16)                 //90d9d5d7e72b9f5f385555845ecb8a25

	return result16
}

func getMD5_2(str []byte) string {
	md5Hash := md5.New()       //创建一个使用MD5校验的Hash对象，hash.Hash是一个被所有hash函数实现的公共接口
	md5Hash.Write([]byte(str)) //通过io操作将数据写入hash对象中，等同于io.WriteString(md5Hash, str)
	result := md5Hash.Sum(nil) //Sum(添加的数据：将该数据进行hash并添加到原始数据前)返回MD5校验和
	fmt.Println(result)        //[144 217 213 215 231 43 159 95 56 85 85 132 94 203 138 37]

	result16slice := hex.EncodeToString(result[:]) //转换为16进制格式字符串，另一种方式
	fmt.Println(result16slice)                     //90d9d5d7e72b9f5f385555845ecb8a25
	return result16slice
}
```



#### SHA-1

```go
func getSha1(filePath string) string {
	fp, _ := os.Open(filePath)    //打开文件
	myHash := sha1.New()          //创建基于sha1算法的Hash对象
	num, _ := io.Copy(myHash, fp) //将文件数据拷贝给哈希对象
	fmt.Println("文件大小: ", num)
	tmp1 := myHash.Sum(nil)            //计算文件哈希值
	result := hex.EncodeToString(tmp1) //转换为16进制格式字符串
	fmt.Println(result)
	return result
}
```

## 消息认证码

- 消息认证码：MAC，message authentication code

- 解决问题：消息被正确传送了吗？

  - **消息完整性**：integrity，指“消息没有被篡改"，也叫一致性。确认汇款请求的内容与Alice银行所发出的内容完全一致

  - **消息认证**：authentication，指“消息来自正确的发送者”，识别发送者身份是否被伪装。确认汇款请求确实来自A银行

    A和B分别使用不同的银行，A向B发起汇款，内容是：*账户A->账户B，汇款100万元*。那么B银行所收到的汇款请求内容，必须与A银行所发送的内容是完全一致的，且是确认是A发送的

    - 如果，主动攻击者C在中途将A银行发送的汇款请求进行了篡改，那么B银行就必须要能够识别出这种篡改，否则如果C将收款账户改成了自己的账户，那么100万元就会被盗走
    - 另外，这条汇款请求到底是不是A银行发送的呢？有可能Alice银行根本就没有发送过汇款请求，而是由主动攻击者Mallory伪装成Alice银行发送的。如果汇款请求不是来自A银行，那么就绝对不能执行汇款

- 过程：共享密钥依然需要通过非对称加密协商

  - 发送方：共享密钥、请求信息
    1. 计算：请求信息 & 共享密钥 => MAC值
    2. 发送：请求信息、MAC值

  - 接收方：共享密钥、请求信息、发送放MAC值
    1. 计算：请求信息 & 共享密钥 => 接收方MAC值
    2. 对比：发送放MAC值==接收方MAC值 ? 认证成功 : 认证失败

- 实现方式：

  - 对称加密MAC
  - HMAC

- 问题：根本原因是共享密钥，因为共享，所以无法验证。出具借条的场景中，将无法单纯使用消息验证码

  - 身份认证：C没有公共密钥，无法通过C证明消息是A发的；假设C也收到了密钥，也无法证明消息是A发的，因为还可能是B发的
  - 防止否认：A向B发送消息后，A可以说消息不是自己发的，而是B自导自演

#### HMAC

- HMAC：HashMAC，使用单向散列函数来构造消息认证码的方法
- 过程：最后得到的MAC值，是一个和输入的消息以及密钥都相关的长度固定的比特序列![1538733763591](https://ws3.sinaimg.cn/large/006tNc79ly1fyzciosd4ij30ex0gwwfo.jpg)

```go
/*HMAC*/
func GenerateHMAC(src, key []byte) []byte {
	myHmac := hmac.New(sha256.New, key) //创建一个底层采用sha256算法的 hash.Hash 接口
	myHmac.Write(src) //添加测试数据
	result := myHmac.Sum(nil) //计算结果
	return result
}
/*HMAC验证*/
func VerifyHMAC(res, src, key []byte) bool {
	myHmac := hmac.New(sha256.New, key) //创建一个底层采用sha256算法的 hash.Hash 接口
	myHmac.Write(src) //添加测试数据
	result := myHmac.Sum(nil) //计算结果
	return hmac.Equal(res, result) //比较结果
}
```

## 数字签名

- 数字签名：digital signature
- 解决问题：消息是谁写的？
  - **身份认证**
  - **事后否认**
- 行为：在数字签名技术中，出现了两种行为。从以下两个行为不难看出，非对称加密可以用来完成数字签名
  - **生成消息签名的行为**：发送者A“对消息签名”，根据消息内容计算数字签名的值，这个行为意味着，“发送者认可该消息的内容”，**签名密钥只能由签名的人持有**。
  - **验证消息签名的行为**：接收者B（或需要验证消息的第三方C），检查该消息的签名是否真的属于发送者A，**验证密钥则是任何需要验证签名的人都可以持有**
- 过程：非对称加密方式。一般不直接对消息进行签名，而是对消息的散列值进行签名
  - A：A私钥、A公钥
    - 签名：信息 + A私钥 = A签名信息
  - Other：A公钥
    - 验证：可以用A的公钥验证对应签名
- 方法：
  - 对称加密MAC
  - HMAC

#### RSA

```go
/*数字签名，RSA实现*/
func SignatureRSA() ([]byte, error) {
	privateFile, _ := os.Open("private.pem") //打开私钥文件
	fileInfo, _ := privateFile.Stat()        //获取文件信息
	all := make([]byte, fileInfo.Size())
	_, _ = privateFile.Read(all) //读取文件内容到all
	defer privateFile.Close()

	block, _ := pem.Decode(all)                             //将数据解析成pem格式的数据块
	privateKey, _ := x509.ParsePKCS1PrivateKey(block.Bytes) //解析pem数据块, 得到私钥

	message := []byte("渡远荆门外，来从楚国游。山随平野尽，江入大荒流。月下飞天境，云生结海楼。仍怜故乡水，万里送行舟。") //消息
	sha256Hash := sha256.New()                                            //sha256进行Hash
	sha256Hash.Write(message)
	hashSum := sha256Hash.Sum(nil) //Hash值

	sign, _ := rsa.SignPKCS1v15(rand.Reader, privateKey, crypto.SHA256, hashSum) //签名
	return sign, nil
}

/*数字签名验证，RSA实现*/
func DigitalSignatureVerificationRSA(data, sign []byte) error {
	publicFile, _ := os.Open("public.pem")
	fileInfo, _ := publicFile.Stat()
	fileContent := make([]byte, fileInfo.Size())
	publicFile.Read(fileContent)
	defer publicFile.Close()

	block, _ := pem.Decode(fileContent)                     //将公钥数据解析为pem格式的数据块
	pubInterface, _ := x509.ParsePKIXPublicKey(block.Bytes) //将公钥从pem数据块中提取出来
	publicKey := pubInterface.(*rsa.PublicKey)              //公钥接口转换为公钥对象
	sha256Hash := sha256.New()
	sha256Hash.Write(data) //hash
	hashSum := sha256Hash.Sum(nil)

	err := rsa.VerifyPKCS1v15(publicKey, crypto.SHA256, hashSum, sign) //验证
	if err != nil {
		return err
	}
	fmt.Println("数字签名验证成功, 恭喜o(*￣︶￣*)o恭喜")
	return nil
}
```

#### ECC

```go
/*ECC（椭圆曲线）生成密钥对*/
func generateEccKeyPair() {
	curve := elliptic.P224()                               //选择一个椭圆曲线（elliptic包）
	privateKey, _ := ecdsa.GenerateKey(curve, rand.Reader) //使用ecdsa包创建私钥
	privateText, _ := x509.MarshalECPrivateKey(privateKey) //使用x509进行编码
	block := pem.Block{
		Type:    "ECC PRIVATE KEY",
		Headers: nil,
		Bytes:   privateText, //写入pem.Block中
	}
	eccPrivateFile, _ := os.Create("eccPrivateKey.pem")
	defer eccPrivateFile.Close()
	err := pem.Encode(eccPrivateFile, &block) //pem.Encode
	if err != nil {
		panic(err)
	}
	publicKey := privateKey.PublicKey //创建公钥
	publicText, _ := x509.MarshalPKIXPublicKey(&publicKey)
	block1 := pem.Block{
		Type:    "ECC PUBLIC KEY",
		Headers: nil,
		Bytes:   publicText,
	}
	eccPublicFile, _ := os.Create("eccPublicKey.pem")
	defer eccPublicFile.Close()
	err = pem.Encode(eccPublicFile, &block1)
	if err != nil {
		panic(err)
	}
}

//自己定义的签名结构
type Signature struct {
	r *big.Int
	s *big.Int
}

/*ECC签名*/
func eccSignData(filename string, src []byte) (Signature, error) {
	info, _ := ioutil.ReadFile(filename)               //读取私钥
	block, _ := pem.Decode(info)                       //返回：pem.bloc,rest参加是未解码完的数据，存储在这里
	derText := block.Bytes                             //得到block中的der编码数据
	privateKey, err := x509.ParseECPrivateKey(derText) //解码der，得到私钥
	if err != nil {
		return Signature{}, err
	}
	hash := sha256.Sum256(src)                                //hash。[32]byte，使用私钥对任意长度的hash值（必须是较大信息的hash结果）进行签名
	r, s, err := ecdsa.Sign(rand.Reader, privateKey, hash[:]) //使用私钥签名。返回签名结果：一对大整数。私钥的安全性取决于密码读取器
	if err != nil {
		return Signature{}, err
	}
	sig := Signature{r, s}
	return sig, nil
}

/*ECC签名验证*/
func eccVerifySig(filename string, src []byte, sig Signature) error {
	info, _ := ioutil.ReadFile(filename)
	block, _ := pem.Decode(info)
	derText := block.Bytes
	publicKeyInterface, _ := x509.ParsePKIXPublicKey(derText)
	publicKey, ok := publicKeyInterface.(*ecdsa.PublicKey)
	if !ok {
		return errors.New("断言失败，非ecds公钥!\n")
	}
	hash := sha256.Sum256(src)
	isValid := ecdsa.Verify(publicKey, hash[:], sig.r, sig.s) //使用公钥验证
	if !isValid {
		return errors.New("校验失败!")
	}
	return nil
}
```



## 数字证书

### 一、证书

一个故事讲明白加密与数字证书：https://www.cnblogs.com/franson-2016/p/5530671.html

- 证书：certificate。为公钥加上数字签名，公钥证书，PKC，Public-Key Certificate

  - windows查看证书：certmgr.msc

- CA：Certification Authority，认证机构。

  - 颁发证书：CA为某人的公钥加上数字签名，则表示该CA认定该公钥的确属于此人。这关系到社会信任链。
  - 举例：比如阿里是一个CA，你信任阿里，那么阿里信任的人或者网站（被阿里颁发证书），也会被你信任。

- 过程：发送者A向接收者B发送一则message（前3步仅新注册时需要）

  1. B生成密钥对，并发送给CA进行注册

  2. CA审核：身份确认和认证业务准则

     认证机构确认"本人"身份的方法和认证机构的认证业务准则（CertificatePractice Statement, CPS，的内容有关。如果认证机构提供的是测试用的服务，那么可能完全不会进行任何身份确认。如果是政府部门运營的认证机构，可能就需要根据法律规定来进行身份确认。如果是企业面向内部设立的认证机构，那就可能会给部门负责人打电话直接确认。

     例如，VeriSign的认证业务准则中将身份确认分为Class1 ~ 3共三个等级

     - Class1：通过向邮箱发送件来确认本人身份
     - Class2：通过第三方数据库来确认本人身份
     - Class3：通过当面认证和身份证明来确认本人身份

     等级越高，身份确认越严格。

  3. CA颁发证书：CA私钥 & B公钥 => B证书。B将公钥发送给CA进行注册，CA用自己的私钥对B公钥签名

  4. A验证证书：A从CA获取到B证书验证签名，验证是某CA的签名，A信任这个CA，则信任是B公钥，确认了B公钥的合法性。

  5. A发送消息：用B公钥加密消息并发送给B

  6. B接收消息：用B私钥解密消息

![1538736605294](https://ws2.sinaimg.cn/large/006tNc79ly1fyzcj57qkqj30h00940tx.jpg)

### 三、证书标准规范X.509

> 证书是由认证机构颁发的，使用者需要对证书进行验证，因此如果证书的格式千奇百怪那就不方便了。于是，人们制定了证书的标准规范，其中使用最广泛的是由ITU（International TelecommumcationUnion，国际电信联盟）和ISO（IntemationalOrganizationforStandardization, 国际标准化组织）制定的X.509规范。很多应用程序都支持x.509并将其作为证书生成和交换的标准规范。
>
> X.509是一种非常通用的证书格式。所有的证书都符合ITU-T X.509国际标准，因此(理论上)为一种应用创建的证书可以用于任何其他符合X.509标准的应用。X.509证书的结构是用ASN1(Abstract Syntax Notation One)进行描述数据结构，并使用ASN.1语法进行编码。 
>
> 在一份证书中，必须证明公钥及其所有者的姓名是一致的。对X.509证书来说，认证者总是[CA](https://baike.baidu.com/item/CA)或由CA指定的人，一份X.509证书是一些标准字段的集合，这些字段包含有关用户或设备及其相应公钥的信息。X.509标准定义了证书中应该包含哪些信息，并描述了这些信息是如何编码的(即数据格式)
>
> 一般来说，一个数字证书内容可能包括基本数据（版本、序列号) 、所签名对象信息（ 签名算法类型、签发者信息、有效期、被签发人、签发的公开密钥）、CA的数字签名，等等。

##### 1. 证书规范

> 前使用最广泛的标准为ITU和ISO联合制定的X.509的 v3版本规范 (RFC5280）, 其中定义了如下证书信息域：
>
> - <font color="red">版本号(Version Number）</font>：规范的版本号，目前为版本3，值为0x2；
> - <font color="red">序列号（Serial Number）</font>：由CA维护的为它所发的每个证书分配的一的列号，用来追踪和撤销证书。只要拥有签发者信息和序列号，就可以唯一标识一个证书，最大不能过20个字节；
>
> - <font color="red">签名算法（Signature Algorithm）</font>：数字签名所采用的算法，如：
>   - sha256-with-RSA-Encryption 
>   - ecdsa-with-SHA2S6；
>
> - <font color="red">颁发者（Issuer）</font>：发证书单位的标识信息，如 ” C=CN，ST=Beijing, L=Beijing, O=org.example.com，CN=ca.org。example.com ”；
>
> - <font color="red">有效期(Validity)</font>:  证书的有效期很，包括起止时间。
> - <font color="red">主体(Subject)</font> : 证书拥有者的标识信息（Distinguished Name），如：" C=CN，ST=Beijing, L=Beijing, CN=person.org.example.com”；
>
> - <font color="red">主体的公钥信息(SubJect Public Key Info）</font>：所保护的公钥相关的信息：
>   - 公钥算法 (Public Key Algorithm）公钥采用的算法；
>   - ==主体公钥（Subject Unique Identifier）：公钥的内容。==(服务器的公钥)
> - <font color="red">颁发者唯一号（Issuer Unique Identifier）</font>：代表颁发者的唯一信息，仅2、3版本支持，可选；
> - <font color="red">主体唯一号（Subject Unique Identifier）</font>：代表拥有证书实体的唯一信息，仅2，3版本支持，可选：
>
> - <font color="red">扩展（Extensions，可选）</font>: 可选的一些扩展。中可能包括：
>   - Subject Key Identifier：实体的秘钥标识符，区分实体的多对秘钥；
>   - Basic Constraints：一指明是否属于CA;
>   - Authority Key Identifier：证书颁发者的公钥标识符；
>   - CRL Distribution Points: 撤销文件的颁发地址；
>   - Key Usage：证书的用途或功能信息。
>
> ==此外，证书的颁发者还需要对证书内容利用自己的私钥添加签名， 以防止别人对证书的内容进行篡改。==(重要)

##### 2. 证书格式

> X.509规范中一般推荐使用PEM(Privacy Enhanced Mail）格式来存储证书相关的文件。证书文件的文件名后缀一般为 .crt 或 .cer 。对应私钥文件的文件名后缀一般为 .key。证书请求文件的文件名后綴为 .csr 。有时候也统一用pem作为文件名后缀。
>
> PEM格式采用文本方式进行存储。一般包括首尾标记和内容块，内容块采用Base64进行编码。
>
> **编码格式总结:**
>
> - **X.509 DER(Distinguished Encoding Rules)编码，后缀为：.der .cer .crt**
> - **X.509 BASE64编码(PEM格式)，后缀为：.pem .cer .crt**
>
> 例如，一个PEM格式（base64编码）的示例证书文件内容如下所示：

```go
-----BEGIN CERTIFICATE-----
MIIDyjCCArKgAwIBAgIQdZfkKrISoINLporOrZLXPTANBgkqhkiG9w0BAQsFADBn
MSswKQYDVQQLDCJDcmVhdGVkIGJ5IGh0dHA6Ly93d3cuZmlkZGxlcjIuY29tMRUw
EwYDVQQKDAxET19OT1RfVFJVU1QxITAfBgNVBAMMGERPX05PVF9UUlVTVF9GaWRk
bGVyUm9vdDAeFw0xNzA0MTExNjQ4MzhaFw0yMzA0MTExNjQ4MzhaMFoxKzApBgNV
BAsMIkNyZWF0ZWQgYnkgaHR0cDovL3d3dy5maWRkbGVyMi5jb20xFTATBgNVBAoM
DERPX05PVF9UUlVTVDEUMBIGA1UEAwwLKi5iYWlkdS5jb20wggEiMA0GCSqGSIb3
DQEBAQUAA4IBDwAwggEKAoIBAQDX0AM198jxwRoKgwWsd9oj5vI0and9v9SB9Chl
gZEu6G9ZA0C7BucsBzJ2bl0Mf6qq0Iee1DfeydfEKyTmBKTafgb2DoQE3OHZjy0B
QTJrsOdf5s636W5gJp4f7CUYYA/3e1nxr/+AuG44Idlsi17TWodVKjsQhjzH+bK6
8ukQZyel1SgBeQOivzxXe0rhXzrocoeKZFmUxLkUpm+/mX1syDTdaCmQ6LT4KYYi
soKe4f+r2tLbUzPKxtk2F1v3ZLOjiRdzCOA27e5n88zdAFrCmMB4teG/azCSAH3g
Yb6vaAGaOnKyDLGunW51sSesWBpHceJnMfrhwxCjiv707JZtAgMBAAGjfzB9MA4G
A1UdDwEB/wQEAwIEsDATBgNVHSUEDDAKBggrBgEFBQcDATAWBgNVHREEDzANggsq
LmJhaWR1LmNvbTAfBgNVHSMEGDAWgBQ9UIffUQSuwWGOm+o74JffZJNadjAdBgNV
HQ4EFgQUQh8IksZqcMVmKrIibTHLbAgLRGgwDQYJKoZIhvcNAQELBQADggEBAC5Y
JndwXpm0W+9SUlQhAUSE9LZh+DzcSmlCWtBk+SKBwmAegbfNSf6CgCh0VY6iIhbn
GlszqgAOAqVMxAEDlR/YJTOlAUXFw8KICsWdvE01xtHqhk1tCK154Otci60Wu+tz
1t8999GPbJskecbRDGRDSA/gQGZJuL0rnmIuz3macSVn6tH7NwdoNeN68Uj3Qyt5
orYv1IFm8t55224ga8ac1y90hK4R5HcvN71aIjMKrikgynK0E+g45QypHRIe/z0S
/1W/6rqTgfN6OWc0c15hPeJbTtkntB5Fqd0sfsnKkW6jPsKQ+z/+vZ5XqzdlFupQ
29F14ei8ZHl9aLIHP5s=
-----END CERTIFICATE-----
```

证书中的解析出来的内容：

```go
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            10:e6:fc:62:b7:41:8a:d5:00:5e:45:b6
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=BE, O=GlobalSign nv-sa, CN=GlobalSign Organization Validation CA-SHA256-G2
        Validity
            Not Before: Nov 21 08:00:00 2016 GMT
            Not After : Nov 22 07:59:59 2017 GMT
        Subject: C=US, ST=California, L=San Francisco, O=Wikimedia Foundation, Inc., CN=*.wikipedia.org
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub: 
                    04:c9:22:69:31:8a:d6:6c:ea:da:c3:7f:2c:ac:a5:
                    af:c0:02:ea:81:cb:65:b9:fd:0c:6d:46:5b:c9:1e:
                    ed:b2:ac:2a:1b:4a:ec:80:7b:e7:1a:51:e0:df:f7:
                    c7:4a:20:7b:91:4b:20:07:21:ce:cf:68:65:8c:c6:
                    9d:3b:ef:d5:c1
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Agreement
            Authority Information Access: 
                CA Issuers - URI:http://secure.globalsign.com/cacert/gsorganizationvalsha2g2r1.crt
                OCSP - URI:http://ocsp2.globalsign.com/gsorganizationvalsha2g2

            X509v3 Certificate Policies: 
                Policy: 1.3.6.1.4.1.4146.1.20
                  CPS: https://www.globalsign.com/repository/
                Policy: 2.23.140.1.2.2

            X509v3 Basic Constraints: 
                CA:FALSE
            X509v3 CRL Distribution Points: 

                Full Name:
                  URI:http://crl.globalsign.com/gs/gsorganizationvalsha2g2.crl

            X509v3 Subject Alternative Name: 
                DNS:*.wikipedia.org, DNS:*.m.mediawiki.org, DNS:*.m.wikibooks.org, DNS:*.m.wikidata.org, DNS:*.m.wikimedia.org, DNS:*.m.wikimediafoundation.org, DNS:*.m.wikinews.org, DNS:*.m.wikipedia.org, DNS:*.m.wikiquote.org, DNS:*.m.wikisource.org, DNS:*.m.wikiversity.org, DNS:*.m.wikivoyage.org, DNS:*.m.wiktionary.org, DNS:*.mediawiki.org, DNS:*.planet.wikimedia.org, DNS:*.wikibooks.org, DNS:*.wikidata.org, DNS:*.wikimedia.org, DNS:*.wikimediafoundation.org, DNS:*.wikinews.org, DNS:*.wikiquote.org, DNS:*.wikisource.org, DNS:*.wikiversity.org, DNS:*.wikivoyage.org, DNS:*.wiktionary.org, DNS:*.wmfusercontent.org, DNS:*.zero.wikipedia.org, DNS:mediawiki.org, DNS:w.wiki, DNS:wikibooks.org, DNS:wikidata.org, DNS:wikimedia.org, DNS:wikimediafoundation.org, DNS:wikinews.org, DNS:wikiquote.org, DNS:wikisource.org, DNS:wikiversity.org, DNS:wikivoyage.org, DNS:wiktionary.org, DNS:wmfusercontent.org, DNS:wikipedia.org
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 Subject Key Identifier: 
                28:2A:26:2A:57:8B:3B:CE:B4:D6:AB:54:EF:D7:38:21:2C:49:5C:36
            X509v3 Authority Key Identifier: 
                keyid:96:DE:61:F1:BD:1C:16:29:53:1C:C0:CC:7D:3B:83:00:40:E6:1A:7C

    Signature Algorithm: sha256WithRSAEncryption
         8b:c3:ed:d1:9d:39:6f:af:40:72:bd:1e:18:5e:30:54:23:35:
         ...
```

##### 3. CA证书

> 证书是用来证明某某东西确实是某某东西的东西（是不是像绕口令？）。通俗地说，证书就好比上文里面的公章。通过公章，可以证明对应的证件的真实性。
>
> 理论上，人人都可以找个证书工具，自己做一个证书。那如何防止坏人自己制作证书出来骗人捏？请看后续 CA 的介绍。
>
> CA是Certificate Authority的缩写，也叫“证书授权中心”。
>
> 它是负责管理和签发证书的第三方机构, 好比一个可信任的中介公司。一般来说，CA必须是所有行业和所有公众都信任的、认可的。因此它必须具有足够的权威性。就好比A、B两公司都必须信任C公司，才会找 C 公司作为公章的中介。

- CA证书

  > CA 证书，顾名思义，就是CA颁发的证书。
  >
  > 前面已经说了，人人都可以找工具制作证书。但是你一个小破孩制作出来的证书是没啥用处的。因为你不是权威的CA机关，你自己搞的证书不具有权威性。
  >
  > 比如，某个坏人自己刻了一个公章，盖到介绍信上。但是别人一看，不是受信任的中介公司的公章，就不予理睬。坏蛋的阴谋就不能得逞啦。

- 证书信任链

  > 证书直接是可以有信任关系的, 通过一个证书可以证明另一个证书也是真实可信的. 实际上，证书之间的信任关系，是可以嵌套的。比如，C 信任 A1，A1 信任 A2，A2 信任 A3......这个叫做证书的信任链。只要你信任链上的头一个证书，那后续的证书，都是可以信任滴。 
  >
  > 假设 C 证书信任 A 和 B；然后 A 信任 A1 和 A2；B 信任 B1 和 B2。则它们之间，构成如下的一个树形关系（一个倒立的树）。 

  ![1533293270573](https://ws1.sinaimg.cn/large/006tNc79ly1fyzcj3rh43j30d606e3yr.jpg)

  > 处于最顶上的树根位置的那个证书，就是“**根证书**”。除了根证书，其它证书都要依靠上一级的证书，来证明自己。那谁来证明“根证书”可靠捏？实际上，根证书自己证明自己是可靠滴（或者换句话说，根证书是不需要被证明滴）。
  >
  > 聪明的同学此刻应该意识到了：根证书是整个证书体系安全的根本。所以，如果某个证书体系中，根证书出了问题（不再可信了），那么所有被根证书所信任的其它证书，也就不再可信了。

- 证书有啥用

  > 1. **验证网站是否可信（针对HTTPS）** 
  >
  >    通常，我们如果访问某些敏感的网页（比如用户登录的页面），其协议都会使用 HTTPS 而不是 HTTP。因为 HTTP 协议是明文的，一旦有坏人在偷窥你的网络通讯，他/她就可以看到网络通讯的内容（比如你的密码、银行帐号、等）；而 HTTPS 是加密的协议，可以保证你的传输过程中，坏蛋无法偷窥。
  >
  >    但是，千万不要以为，HTTPS 协议有了加密，就可高枕无忧了。俺再举一个例子来说明，光有加密是不够滴。假设有一个坏人，搞了一个假的网银的站点，然后诱骗你上这个站点。假设你又比较单纯，一不留神，就把你的帐号，口令都输入进去了。那这个坏蛋的阴谋就得逞鸟。
  >
  >    为了防止坏人这么干，HTTPS 协议除了有加密的机制，还有一套证书的机制。通过证书来确保，某个站点确实就是某个站点。
  >
  >    有了证书之后，当你的浏览器在访问某个 HTTPS 网站时，会验证该站点上的 CA 证书（类似于验证介绍信的公章）。如果浏览器发现该证书没有问题（证书被某个根证书信任、证书上绑定的域名和该网站的域名一致、证书没有过期），那么页面就直接打开；否则的话，浏览器会给出一个警告，告诉你该网站的证书存在某某问题，是否继续访问该站点？下面给出 IE 和 Firefox 的抓图： 
  >
  >    ![img](https://ws2.sinaimg.cn/large/006tNc79ly1fyzcj2qqjzj30e609pq3w.jpg) 
  >
  >    ![img](https://ws4.sinaimg.cn/large/006tNc79ly1fyzcj5px3zj30mq0hk0ve.jpg) 
  >
  >    大多数知名的网站，如果用了 HTTPS 协议，其证书都是可信的（也就不会出现上述警告）。所以，今后你如果上某个知名网站，发现浏览器跳出上述警告，你就要小心啦！ 
  >
  > 2. **验证某文件是否可信（是否被篡改）** 
  >
  >    证书除了可以用来验证某个网站，还可以用来验证某个文件是否被篡改。具体是通过证书来制作文件的数字签名。制作数字签名的过程太专业，咱就不说了。后面专门告诉大家如何验证文件的数字签名。考虑到大多数人用 Windows 系统，俺就拿 Windows 的例子来说事儿。
  >
  >    比如，俺手头有一个 Google Chrome的安装文件（带有数字签名）。当俺查看该文件的属性，会看到如下的界面。眼神好的同学，会注意到到上面有个“**数字签名**”的标签页。如果没有出现这个标签页，就说明该文件没有附带数字签名。
  >
  >    ![1533294208392](https://ws4.sinaimg.cn/large/006tNc79ly1fyzcj1tyvvj30d20i3ab0.jpg)
  >
  >    一般来说，签名列表中，有且仅有一个签名。选中它，点“**详细信息**”按钮。跳出如下界面：
  >
  >    通常这个界面会显示一行字：“**该数字签名正常**”（图中红圈标出）。如果有这行字，就说明该文件从出厂到你手里，中途没有被篡改过（是原装滴、是纯洁滴）。如果该文件被篡改过了（比如，感染了病毒、被注入木马），那么对话框会出现一个警告提示“**该数字签名无效**” 
  >
  >    ![1533294414623](https://ws1.sinaimg.cn/large/006tNc79ly1fyzcj46npvj30d20h50ty.jpg)
  >
  >    不论签名是否正常，你都可以点“查看证书”按钮。这时候，会跳出证书的对话框。如下： 
  >
  >    ![1533294685323](https://ws3.sinaimg.cn/large/006tNc79ly1fyzcj3cx3kj30d20ic3zn.jpg)
  >
  >    ![1533294711691](https://ws4.sinaimg.cn/large/006tNc79ly1fyzcj4m7hnj30d20icdgr.jpg)
  >
  >    从后一个界面，可以看到刚才说的证书信任链。图中的信任链有3层：
  >
  >    - 第1层是根证书（verisign）。
  >    - 第2层是 symantec 专门用来签名的证书。
  >    - 第3层是 Google自己的证书。 
  >
  >    目前大多数知名的公司（或组织机构），其发布的可执行文件（比如软件安装包、驱动程序、安全补丁），都带有数字签名。你可以自己去看一下。
  >
  >    建议大伙儿在安装软件之前，都先看看是否有数字签名？如果有，就按照上述步骤验证一把。一旦数字签名是坏的，那可千万别装。

### 四、公钥基础设施（PKI）

> 仅制定证书的规范还不足以支持公钥的实际运用，我们还需要很多其他的规范，例如证书应该由谁来颁发，如何颁发，私钥泄露时应该如何作废证书，计算机之间的数据交换应采用怎样的格式等。这一节我们将介绍能够使公钥的运用更加有效的公钥基础设施。

##### 1. 什么是公钥基础设施

> 公钥基础设施（Public-Key infrastructure）是为了能够更有效地运用公钥而制定的一系列规范和规格的总称。公钥基础设施一般根据其英语缩写而简称为PKI。
>
> PKI只是一个总称，而并非指某一个单独的规范或规格。例如，RSA公司所制定的PKCS（Public-Key Cryptography Standards，公钥密码标准）系列规范也是PKI的一种，而互联网规格RFC（Requestfor Comments）中也有很多与PKI相关的文档。此外，X.509这样的规范也是PKI的一种。在开发PKI程序时所使用的由各个公司编写的API（Application Programming Interface, 应用程序编程接口）和规格设计书也可以算是PKI的相关规格。
>
> 因此，根据具体所采用的规格，PKI也会有很多变种，这也是很多人难以整体理解PKI的原因之一。
>
> 为了帮助大家整体理解PKI,我们来简单总结一下PKI的基本组成要素（用户、认证机构、仓库）以及认证机构所负责的工作。

##### 2. PKI的组成要素

###### - 概述

PKI的组成要素主要有以下三个：

- 用户 --- 使用PKI的人
- 认证机构 --- 颁发证书的人
- 仓库 --- 保存证书的数据库

![1538758063013](https://ws4.sinaimg.cn/large/006tNc79ly1fyzcj2aivdj30fh0do0tg.jpg)

###### - 用户

> 用户就是像Alice、Bob这样使用PKI的人。用户包括两种：一种是希望使用PKI注册自己的公钥的人，另一种是希望使用已注册的公钥的人。我们来具体看一下这两种用户所要进行的操作。

- **注册公钥的用户所进行的操作**

  - 生成密钥对（也可以由认证机构生成）
  - 在认证机构注册公钥
  - 向认证机构申请证书
  - 根据需要申请作废已注册的公钥
  - 解密接收到的密文
  - 对消息进行数字签名

- **使用已注册公钥的用户所进行的操作**

  - 将消息加密后发送给接收者
  - 验证数字签名

  ```c++
  /* 
  ==================== 小知识点 ==================== 
  浏览器如何验证SSL证书
  1. 在IE浏览器的菜单中点击“工具 /Internet选项”，选择“内容”标签，点击“证书”按钮，然后就可以看到IE
     浏览器已经信任了许多“中级证书颁发机构”和“受信任的根证书颁发机 构。当我们在访问该网站时，浏览器
     就会自动下载该网站的SSL证书，并对证书的安全性进行检查。
  2. 由于证书是分等级的，网站拥有者可能从根证书颁发机构领到证书，也可能从根证书的下一级（如某个国家
     的认证中心，或者是某个省发出的证书）领到证书。假设我们正在访问某个使用 了 SSL技术的网站，IE浏
     览器就会收到了一个SSL证书，如果这个证书是由根证书颁发机构签发的，IE浏览器就会按照下面的步骤来
     检查：浏览器使用内 置的根证书中的公钥来对收到的证书进行认证，如果一致，就表示该安全证书是由可信
     任的颁证机构签发的，这个网站就是安全可靠的；如果该SSL证书不是根服 务器签发的，浏览器就会自动检
     查上一级的发证机构，直到找到相应的根证书颁发机构，如果该根证书颁发机构是可信的，这个网站的SSL证
     书也是可信的。
  */
  ```

###### - 认证机构（CA）

> 认证机构（Certification Authority，CA）是对证书进行管理的人。上面的图中我们给它起了一个名字叫作Trent。认证机构具体所进行的操作如下：

- **生成密钥对 (也可以由用户生成)**

  > 生成密钥对有两种方式：一种是由PKI用户自行生成，一种是由认证机构来生成。在认证机构生成用户密钥对的情况下，认证机构需要将私钥发送给用户，这时就需要使用PKCS#12（Personal Information Exchange Syntax Standard）等规范。

- 在注册公钥时对本人身份进行认证, 生成并颁发证书

  > 在用户自行生成密钥对的情况下，用户会请求认证机构来生成证书。申请证书时所使用的规范是由PKCS#10（Certification Request Syntax Standard）定义的。
  >
  > 认证机构根据其认证业务准则（Certification Practice Statement，CPS）对用户的身份进行认证，并生成证书。在生成证书时，需要使用认证机构的私钥来进行数字签名。生成的证书格式是由PKCS#6 （Extended-Certificate Syntax Standard）和 X.509定义的。

- 作废证书

  > 当用户的私钥丢失、被盗时，认证机构需要对证书进行作废（revoke）。此外，即便私钥安然无恙，有时候也需要作废证书，例如用户从公司离职导致其失去私钥的使用权限，或者是名称变更导致和证书中记载的内容不一致等情况。
  >
  > 纸质证书只要撕毁就可以作废了，但这里的证书是数字信息，即便从仓库中删除也无法作废，因为用户会保存证书的副本，但认证机构又不能人侵用户的电脑将副本删除。
  >
  > 要作废证书，认证机构需要制作一张证书==**作废清单（Certificate Revocation List),简称为CRL**==。
  >
  > CRL是认证机构宣布作废的证书一览表，具体来说，是一张已作废的证书序列号的清单，并由认证机构加上数字签名。证书序列号是认证机构在颁发证书时所赋予的编号，在证书中都会记载。
  >
  > PKI用户需要从认证机构获取最新的CRL,并查询自己要用于验证签名（或者是用于加密）的公钥证书是否已经作废这个步骤是非常重要的。
  >
  > 假设我们手上有Bob的证书，该证书有合法的认证机构签名，而且也在有效期内，但仅凭这些还不能说明该证书一定是有效的，还需要查询认证机构最新的CRL，并确认该证书是否有效。一般来说，这个检查不是由用户自身来完成的，而是应该由处理该证书的软件来完成，但有很多软件并没有及时更能CRL。

> 认证机构的工作中，公钥注册和本人身份认证这一部分可以由注册机构（Registration Authority，RA) 来分担。这样一来，认证机构就可以将精力集中到颁发证书上，从而减轻了认证机构的负担。不过，引入注册机构也有弊端，比如说认证机构需要对注册机构本身进行认证，而且随着组成要素的增加，沟通过程也会变得复杂，容易遭受攻击的点也会增。

###### - 仓库

> 仓库（repository）是一个保存证书的数据库，PKI用户在需要的时候可以从中获取证书．它的作用有点像打电话时用的电话本。在本章开头的例子中，尽管没特别提到，但Alice获取Bob的证书时，就可以使用仓库。仓库也叫作证书目录。

##### 3. 各种各样的PKI

> 公钥基础设施（PKI）这个名字总会引起一些误解，比如说“面向公众的权威认证机构只有一个"，或者“全世界的公钥最终都是由一个根CA来认证的"，其实这些都是不正确的。认证机构只要对公钥进行数字签名就可以了，因此任何人都可以成为认证机构，实际上世界上已经有无数个认证机构了。
>
> 国家、地方政府、医院、图书馆等公共组织和团体可以成立认证机构来实现PKI,公司也可以出于业务需要在内部实现PKI,甚至你和你的朋友也可以以实验为目的来构建PKI。
>
> 在公司内部使用的情况下，认证机构的层级可以像上一节中一样和公司的组织层级一一对应，也可以不一一对应。例如，如果公司在东京、大阪、北海道和九州都成立了分公司，也可以采取各个分公司之间相互认证的结构。在认证机构的运营方面，可以购买用于构建PKI的软件产品由自己公司运营，也可以使用VeriSign等外部认证服务。具体要采取怎样的方式，取决于目的和规模，并没有一定之规。 



## https

- https：http + ssl。为了更安全的通信
  - http
  - ssl：Secure Socket Layer， 一个通讯协议，在通讯过程中，使用了数字证书
- 通信：所有的通信不再传输公钥，而是传输数字证书。证书里面包含了公钥，可以由CA机构认证。
  1. 网站提供者会自己生成公钥私钥（也可以不自己生成，全部由CA帮助生成）。
  2. 服务器提供者将公钥发送给选择的CA机构
  3. CA机构也有自己的私钥公钥。CA使用自己的私钥对服务器的公钥进行签名
     - 还有一些其他辅助信息（发行机构，主题，指纹）
     - 公钥
     - 签名
     - CA向服务器颁发一个数字证书
  4. 当用户访问服务器的时候，服务器会将CA证书发送给客户
  5. 在客户的浏览器中，已经随着操作系统预装了知名CA机构的根证书，这里   面包含了CA机构的公钥， 浏览器就会对服务器的证书进行验证
  6. 如果验证成功，说明服务器可靠，可以正常通信（小锁头）
  7. 如果验证失败，显示（Not Secure）， 提示Warning
  8. 证书有效时， 浏览器会将自己支持的对称加密算法（des, 3des, aes）发送给服务器， 生成随机秘钥（对称）， 使用服务器的公钥，对上述信息加密。发送给服务器
  9. 服务器选择一个加密算法，使用对称秘钥加密消息，发送给客户端
  10. 双方达成一致，接下来通信转换为对称加密
- 优点：尽管HTTPS并非绝对安全，掌握根证书的机构、掌握加密算法的组织同样可以进行中间人形式的攻击，但HTTPS仍是现行架构下最安全的解决方案，主要有以下几个好处：
  1. 使用HTTPS协议可认证用户和服务器，确保数据发送到正确的客户机和服务器；
  2. HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，要比http协议安全，可防止数据在传输过程中不被窃取、改变，确保数据的完整性。
  3. HTTPS是现行架构下最安全的解决方案，虽然不是绝对安全，但它大幅增加了中间人攻击的成本。
  4. 谷歌曾在2014年8月份调整搜索引擎算法，并称 “比起同等HTTP网站，采用HTTPS加密的网站在搜索结果中的排名将会更高”。
- 缺点：虽然说HTTPS有很大的优势，但其相对来说，还是存在不足之处的：
  1. HTTPS协议握手阶段比较费时，会使页面的加载时间延长近50%，增加10%到20%的耗电；
  2. HTTPS连接缓存不如HTTP高效，会增加数据开销和功耗，甚至已有的安全措施也会因此而受到影响；
  3. SSL/TLS证书需要钱，功能越强大的证书费用越高，个人网站、小网站没有必要一般不会用。
  4. SSL/TLS证书通常需要绑定IP，不能在同一IP上绑定多个域名，IPv4资源不可能支撑这个消耗。
  5. HTTPS协议的加密范围也比较有限，在黑客攻击、拒绝服务攻击、服务器劫持等方面几乎起不到什么作用。最关键的，SSL证书的信用链体系并不安全，特别是在某些国家可以控制CA根证书的情况下，中间人攻击一样可行
- https://github.com/AdugiBeyond/golang-https-example

##### http

- http：超文本传输协议

- http客户端：即web浏览器

- http服务器：即web服务器

- HTTP协议：是互联网上应用最为广泛的一种网络协议，是一个客户端和服务器端请求和应答的标准（TCP），用于从WWW服务器传输超文本到本地浏览器的传输协议，它可以使浏览器更加高效，使网络传输减少。

  HTTPS：是以安全为目标的HTTP通道，简单讲是HTTP的安全版，即HTTP下加入SSL/TLS层，HTTPS的安全基础是SSL/TLS，因此加密的详细内容就需要SSL/TLS。

  <font color="red">HTTPS协议的主要作用可以分为两种：</font>

  - <font color="red">建立一个信息安全通道，来保证数据传输的安全；</font>
  - <font color="red">确认网站的真实性。</font>

  HTTPS和HTTP的区别主要如下：

  　　1、https协议需要到ca申请证书，一般免费证书较少，因而需要一定费用。

  　　2、http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl/tls加密传输协议。

  　　3、http和https使用的是完全不同的连接方式，==用的端口也不一样，前者是80，后者是443。==

  　　4、http的连接很简单，是无状态的；HTTPS协议是由SSL/TLS+HTTP协议构建的可进行加密传输、

##### SSL/TLS

- SSL/TLS：SSL，secure sockets layer，安全套接层。TLS，transport layer security，传输层安全。TLS是SSL的继任者。

  - 中综合运用了之前所学习的对称密码、消息认证码、公钥密码、数字签名、伪随机数生成器等密码技术。严格来说，SSL（Secure Socket Layer)与TLS（Transport Layer Security）是不同的，TLS相当于是SSL的后续版本。不过，本章中所介绍的内容，大多是SSL和TLS两者兼备的，因此除具体介绍通信协议的部分以外，都统一写作SSL/TLS
  - 在明文的上层和TCP层之间加上一层加密，这样就保证上层信息传输的安全。它存在的唯一目的就是保证上层通讯安全的一套机制

- 解决问题：下面内容找齐了解决问题的方法，只要用一个“框架”（framework）将这些工具组合起来就可以了。SSL/TIS协议其实就扮演了这样一种框架的角色

  - 窃取：机密性问题。对称加密解决

    要确保机密性，可以使用对称加密。由于对称加密算法的密钥不能被攻击者预测，因此我们使用伪随机数生成器来生成密钥。若要将对称加密的密钥发送给通信对象，可以使用非对称加密算法完成密钥交换。要识别篡改，对数据进行认证，可以使用消息认证码。消息认证码是使用单向散列函数来实现的。

  - 篡改：完整性问题。

  - 认证：认证问题。确保通信对象是真正要通信的对象。

    要对通信对象进行认证，可以使用对公钥加上数字签名所生成的证书

- SSL/TLS不仅可以承载HTTP，还可以承载SMTP（Simple Mail Transfer Protocol, 简单邮件传输协议）和接收邮件时使用的POP3（Post Office Protocol，邮局协议）等

- 过程：

  - 第一次

    - 客户端连接服务器
      - 客户端使用的ssl版本, 客户端支持的加密算法
    - 服务器
      - 先将自己支持ssl版本和客户端的支持的版本比较
        - 支持的不一样, 连接断开
        - 支持的一样, 继续
      - 根据得到的客户端支持 的加密算法, 找一个服务器端也同样支持算法, 发送给客户端
      - 需要发送服务器的证书给客户端

  - 第二次
    - 客户端
      - 接收服务器的证书
      - 校验证书的信息
        - 校验证书的签发机构
        - 证书的有效期
        - 证书中支持 的域名和访问的域名是否一致
      - 校验有问题, 浏览器会给提示



##### demo

```sh
git@github.com:AdugiBeyond/golang-https-example.git
```

### 一、单向认证

- client认证server
- client端事先保存server的签发机构根证书，与server建立联系时，server需要向client发送自己的证书
- client无需向server发送证书



server证书生成：pem格式

```sh
openssl req \
    -x509 \
    -nodes \
    -newkey rsa:2048 \
    -keyout server.key \
    -out server.crt \
    -days 3650 \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=Global Security/OU=IT Department/CN=*"
```



##### 1. 写server.go

###### -分析

```go
//1. 创建http server，
//- Addr
//- Handler
//- TLSConfig

//2. server启动服务， 同时加载自己的证书

```

###### - 代码

```go
package main

import (
	"net/http"
	"log"
	"fmt"
)

func main() {
	//1. 创建http server，
	server := http.Server{
		Addr:      ":1234",
		Handler:   &myHandler{},
		TLSConfig: nil,
	}

	//2. 启动服务， 同时加载自己的证书
	err := server.ListenAndServeTLS("server.crt", "server.key")
	if err != nil {
		log.Fatal(err)
	}
}

type myHandler struct{}

func (h *myHandler) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	fmt.Printf("myFunc called!")
	w.Write([]byte("hello world!!!!"))
}

```



浏览器访问!



###### - 补充(可选)

如果handler使用nil，则程序会使用默认的处理器

```go
	server := http.Server{
		Addr:      ":1234",
		Handler:   nil,
		TLSConfig: nil,
	}
```

实现下面的函数即可

```go
	http.HandleFunc("/", func(writer http.ResponseWriter, request *http.Request) {
		fmt.Printf("myFunc called!\n")
		writer.Write([]byte("hello world!!!!"))
	})
```



##### 2. 写client.go

###### - 分析

```go
//1. 注册server的ca证书
//- 读取证书
//- 加载证书到ca池

//2. 配置tls， tls.Config
//- 把ca池注册进去 ===> RootCAs

//3. 创建client ===> Transport， 指定cfg
//4. 发送请求
//5. 接收并打印返回值
```

###### - 代码

```go
package main

import (
	"io/ioutil"
	"log"
	"crypto/x509"
	"crypto/tls"
	"net/http"
	"fmt"
)

func main() {
	//1. 注册server的ca证书
	//- 读取证书
	caCert, err := ioutil.ReadFile("server.crt")
	if err != nil {
		log.Fatal(err)
	}

	//- 加载证书到ca池
	certPool := x509.NewCertPool()

	ok := certPool.AppendCertsFromPEM(caCert)
	if !ok {
		log.Fatal(err)
	}

	//2. 配置tls
	cfg := tls.Config{
		//配置ca池
		RootCAs: certPool,
	}

	//3. 创建client
	client := http.Client{
		Transport: &http.Transport{
			TLSClientConfig: &cfg,
		},
	}

	//4. 发送请求
	res, err := client.Get("https://localhost:1234")
	if err != nil {
		log.Fatal(err)
	}

	info, err := ioutil.ReadAll(res.Body)
	if err != nil {
		log.Fatal(err)
	}

	//5. 接收并打印返回值
	fmt.Printf("body : %s\n", info)
	fmt.Printf("status : %s\n", res.Status)
}
```





### 二、双向认证



生成server证书

```sh
openssl req \
    -x509 \
    -nodes \
    -newkey rsa:2048 \
    -keyout server.key \
    -out server.crt \
    -days 3650 \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=Global Security/OU=IT Department/CN=*"
```



生成client证书

```sh
openssl req \
    -x509 \
    -nodes \
    -newkey rsa:2048 \
    -keyout client.key \
    -out client.crt \
    -days 3650 \
    -subj "/C=GB/ST=ITCAST/L=ITCAST/O=Global Security/OU=IT Department/CN=*"
```



##### 1. server.go

###### - 分析

```go
//1. 注册客户端的ca证书
//- 读取client证书
//- 添加到ca池

//2.创建tls.config， 配置tls通信
//- 指明需要验证客户端， ClientAuth:tls
//- 添加caPool， ClientCAs:

//3. 创建http的server，
//- 端口
//- 处理器  =》 自己实现ServeHTTP函数， 也可以设置nil
//- TLS配置

//4. 启动服务，指定证书
```



###### - 代码

```go
package main

import (
	"io/ioutil"
	"log"
	"crypto/x509"
	"crypto/tls"
	"net/http"
	"fmt"
)

func main() {
	//1. 注册client的ca证书，自己签名的，所以只需要把client的证书注册即可
	//-读取
	//caCert, err := ioutil.ReadFile("client.crt")
	caCert, err := ioutil.ReadFile("client.crt")
	if err != nil {
		log.Fatal("err : ", err)
	}

	//-加载到ca池
	caCerPool := x509.NewCertPool()
	caCerPool.AppendCertsFromPEM(caCert)

	//2. 配置tls通信
	cfg := tls.Config{
		//需要客户端的证书并且验证
		ClientAuth: tls.RequireAndVerifyClientCert,
		ClientCAs:  caCerPool,
	}

	//3. 根据tls配置，创建http的server
	server := http.Server{
		Addr:      ":8843",
		Handler:   &handler{},
		TLSConfig: &cfg,
	}

	//4. 启动server
	err = server.ListenAndServeTLS("server.crt", "server.key")
	fmt.Printf(" err : %s\n", err)
}

type handler struct {
}

func (h *handler) ServeHTTP(ResponseWriter http.ResponseWriter, Request *http.Request) {
	resInfo := []byte("hello world!!!")
	fmt.Println(string(resInfo))
	ResponseWriter.Write(resInfo)
}

```



##### 2. client.go

###### - 分析

```go
//1. 注册server的ca证书
//- 读取证书
//- 加载证书到ca池

//1.5 加载自己的证书和私钥tls.Load  《======= 修改了
//- 证书传递给server
//- 私钥解密数据


//2. 配置tls， tls.Config
//- 把ca池注册进去 ===> RootCAs
//- 提供client自己的证书信息  《===========修改了 
// Certificates: []tls.Certificate{cert},

//3. 创建client ===> Transport， 指定cfg
//4. 发送请求
//5. 接收并打印返回值
```



###### - 代码

```go
package main

import (
	"io/ioutil"
	"log"
	"crypto/x509"
	"crypto/tls"
	"net/http"
	"fmt"
)

//client代码
func main() {
	//1. 注册能够识别server证书的ca, 这个证书是自签名的，所以自己就是ca
	//- 读取ca
	caCert, err := ioutil.ReadFile("server.crt")
	if err != nil {
		log.Fatal(err)
	}

	//- 注册到证书链
	caCertPool := x509.NewCertPool()
	caCertPool.AppendCertsFromPEM(caCert)

	//2. 加载client自己的证书和私钥
	//- 证书是要传递给server
	//- 私钥是为了解开server用client公钥加密的信息
	cert, err := tls.LoadX509KeyPair("client.crt", "client.key")
	if err != nil {
		log.Fatal(err)
	}

	//3. 配置https的client为tls
	cfg := tls.Config{
		RootCAs:      caCertPool,
		Certificates: []tls.Certificate{cert},
	}

	//4. 创建http client
	client := http.Client{
		Transport: &http.Transport{
			TLSClientConfig: &cfg,
		},
	}

	//5. 发起http请求
	resp, err := client.Get("https://localhost:8843")
	if err != nil {
		log.Fatal(err)
	}

	data, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}

	defer resp.Body.Close()

	//6. 打印应答信息
	fmt.Printf("data : %s\n", data)
	fmt.Printf("status code: %s\n", resp.Status)
}
```

## 常见证书格式整理

### 一、数字证书常见标准

- 符合PKI ITU-T X509标准，传统标准（.DER .PEM .CER .CRT）
- 符合PKCS#7 加密消息语法标准(.P7B .P7C .SPC .P7R)
- 符合PKCS#10 证书请求标准(.p10)
- 符合PKCS#12 个人信息交换标准（.pfx *.p12）
  X509是数字证书的基本规范，而P7和P12则是两个实现规范，P7用于数字信封，P12则是带有私钥的证书实现规范。



#####  x509

- 基本的证书格式，只包含公钥。
  x509证书由用户公共密钥和用户标识符组成。
- 此外还包括版本号、证书序列号、CA标识符、签名算法标识、签发者名称、证书有效期等信息。

##### PKCS#7

- Public Key Cryptography Standards #7。
- PKCS#7一般把证书分成两个文件，一个公钥、一个私钥，有PEM和DER两种编码方式。
- PEM比较多见，是纯文本的，一般用于分发公钥，看到的是一串可见的字符串，通常以.crt，.cer，.key为文件后缀。
- DER是二进制编码。
  PKCS#7一般主要用来做数字信封。



##### PKCS#10

证书请求语法。



##### PKCS#12

- Public Key Cryptography Standards #12。

一种文件打包格式，为存储和发布用户和服务器私钥、公钥和证书指定了一个可移植的格式，是一种二进制格式，通常以.pfx或.p12为文件后缀名。
使用OpenSSL的pkcs12命令可以创建、解析和读取这些文件。
P12是把证书压成一个文件，xxx.pfx 。主要是考虑分发证书，私钥是要绝对保密的，不能随便以文本方式散播。所以P7格式不适合分发。.pfx中可以加密码保护，所以相对安全些。



##### PKCS系列标准

实际上PKCS#7、PKCS#10、PKCS#12都是PKCS系列标准的一部分。相互之间并不是替代的关系，而是对不同使用场景的定义。



### 二、证书编码格式

PEM和DER两种编码格式。

##### PEM

- Privacy Enhanced Mail(信封)

- 查看内容，以"-----BEGIN..."开头，以"-----END..."结尾。

- 查看PEM格式证书的信息：

  ```sh
  `Apache和*NIX服务器偏向于使用这种编码格式。
  openssl x509 -in certificate.pem -text -noout
  ```

  

##### DER

- Distinguished Encoding Rules

- 打开看是二进制格式，不可读。

- Java和Windows服务器偏向于使用这种编码格式。

- 查看DER格式证书的信息

  ```sh
  `der是格式，与证书的后缀名没有直接关系
  openssl x509 -in certificate.der -inform der -text -noout  `请试试-pubkey参数
  ```



### 三、各种后缀含义

文件的内容和后缀没有必然的关系，但是一般使用这些后缀来表示这是什么文件。

##### JKS

Java Key Store(JKS)。

##### CSR

- 证书请求文件(Certificate Signing Request)。
- 这个并不是证书，而是向权威证书颁发机构获得签名证书的申请，其核心内容是一个公钥(当然还附带了一些别的个人信息)。
- 查看的办法：openssl req -noout -text -in my.csr，
- DER格式的话加上`-inform der`。  ==<<<<— 重要==



##### CER

一般指使用DER格式的证书。



##### CRT

证书文件。可以是PEM格式。



##### KEY

- 通常用来存放一个公钥或者私钥。

- 查看KEY的办法：

  ```sh
  openssl rsa -in mykey.key -text -noout
  ```


  如果是DER格式的话

  ```sh
  `这是使用RSA算法生成的key这么查看，DSA算法生成的使用dsa参数。
  openssl rsa -in mykey.key -text -noout -inform der
  ```

##### CRL

证书吊销列表 (Certification Revocation List)，是一种包含撤销的证书列表的签名数据结构。