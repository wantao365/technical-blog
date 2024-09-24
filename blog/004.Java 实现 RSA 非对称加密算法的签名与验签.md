## 1. RSA 签名/验签 简介

RSA 非对称加密算法，除了用来加密/解密数据外，还可以用于对数据（文件）的 **签名** 和 **验签**，可用于确认数据或文件的完整性与签名者（所有者）。

RSA 密钥对的生成与加密/解密见上一篇:Java 实现 AES 对称加密算法的加密和解密

RSA 密钥对在使用时通常:

- 加密/解密：通常使用 **私钥加密，公钥解密**。
- 签名/验签：通常使用 **私钥签名，公钥验签**。

Android 安装包 APK 文件的签名，是 RSA 签名验签的典型应用：Android 打包后，用私钥对 APK 文件进行签名，并把公钥和签名结果放到 APK 包中。下次客户端升级 APK 包时，根据新的 APK 包和包内的签名信息，用 APK 包内的公钥验签校验是否和本地已安装的 APK 包使用的是同一个私钥签名，如果是，则允许安装升级。

## 2. RSA 签名/验签 代码实例

```java
package com.wantao.rsa;

import sun.misc.BASE64Encoder;

import java.security.*;

/**
 * @author xietansheng
 */
public class Main {

    public static void main(String[] args) throws Exception {
        /*
         * 1. 先生成一对 RSA 密钥, 用于测试
         */
        // 随机生成一对 RAS 密钥（包含公钥和私钥）
        KeyPair keyPair = generateKeyPair();
        // 获取 公钥 和 私钥
        PublicKey pubKey = keyPair.getPublic();
        PrivateKey priKey = keyPair.getPrivate();

        /*
         * 2. 原始数据
         */
        String data = "你好, World";

        /*
         * 3. 私钥签名: 对数据进行签名, 计算签名结果
         */
        // 根据指定算法获取签名工具
        Signature sign = Signature.getInstance("Sha1WithRSA");
        // 用私钥初始化签名工具
        sign.initSign(priKey);
        // 添加要签名的数据
        sign.update(data.getBytes());
        // 计算签名结果（签名信息）
        byte[] signInfo = sign.sign();
        // 输出签名结果的 Base64 字符串
        System.out.println(new BASE64Encoder().encode(signInfo));

        /*
         * 4. 公钥验签: 用公钥校验数据的签名是否来自指定的私钥
         */
        // 根据指定算法获取签名工具
        sign = Signature.getInstance("Sha1WithRSA");
        // 用公钥初始化签名工具
        sign.initVerify(pubKey);
        // 添加要校验的数据
        sign.update(data.getBytes());
        // 校验数据的签名信息是否正确,
        // 如果返回 true, 说明该数据的签名信息来自该公钥对应的私钥,
        // 同一个私钥的签名, 数据和签名信息一一对应, 只要其中有一点修改, 则用公钥无法校验通过,
        // 因此可以用私钥签名, 然后用公钥来校验数据的完整性与签名者（所有者）
        boolean verify = sign.verify(signInfo);
        System.out.println(verify);
    }

    /**
     * 随机生成 RSA 密钥对（包含公钥和私钥）
     */
    private static KeyPair generateKeyPair() throws Exception {
        // 获取指定算法的密钥对生成器
        KeyPairGenerator gen = KeyPairGenerator.getInstance("RSA");
        // 初始化密钥对生成器（指定密钥长度, 使用默认的安全随机数源）
        gen.initialize(2048);
        // 随机生成一对密钥（包含公钥和私钥）
        return gen.generateKeyPair();
    }

}
```

## 3. RSA 签名/验签工具类: RSASignUtils.java

为了方便在代码中直接使用 RSA 签名/验签，将 RSA 签名/验签步骤封装成工具类。

`RSASignUtils.java`工具类完整代码：

```java
package com.wantao.rsa;

import java.io.*;
import java.security.*;

/**
 * RSA 签名验签工具类
 *
 * @author xietansheng
 */
public class RSASignUtils {

    /** 秘钥对算法名称 */
    private static final String ALGORITHM = "RSA";

    /** 密钥长度 */
    private static final int KEY_SIZE = 2048;

    /** 签名算法 */
    private static final String SIGNATURE_ALGORITHM = "Sha1WithRSA";

    /**
     * 随机生成 RSA 密钥对（包含公钥和私钥）
     */
    public static KeyPair generateKeyPair() throws Exception {
        // 获取指定算法的密钥对生成器
        KeyPairGenerator gen = KeyPairGenerator.getInstance(ALGORITHM);

        // 初始化密钥对生成器（指定密钥长度, 使用默认的安全随机数源）
        gen.initialize(KEY_SIZE);

        // 随机生成一对密钥（包含公钥和私钥）
        return gen.generateKeyPair();
    }

    /**
     * 私钥签名（数据）: 用私钥对指定字节数组数据进行签名, 返回签名信息
     */
    public static byte[] sign(byte[] data, PrivateKey priKey) throws Exception {
        // 根据指定算法获取签名工具
        Signature sign = Signature.getInstance(SIGNATURE_ALGORITHM);

        // 用私钥初始化签名工具
        sign.initSign(priKey);

        // 添加要签名的数据
        sign.update(data);

        // 计算签名结果（签名信息）
        byte[] signInfo = sign.sign();

        return signInfo;
    }

    /**
     * 公钥验签（数据）: 用公钥校验指定数据的签名是否来自对应的私钥
     */
    public static boolean verify(byte[] data, byte[] signInfo, PublicKey pubKey) throws Exception {
        // 根据指定算法获取签名工具
        Signature sign = Signature.getInstance(SIGNATURE_ALGORITHM);

        // 用公钥初始化签名工具
        sign.initVerify(pubKey);

        // 添加要校验的数据
        sign.update(data);

        // 校验数据的签名信息是否正确,
        // 如果返回 true, 说明该数据的签名信息来自该公钥对应的私钥,
        // 同一个私钥的签名, 数据和签名信息一一对应, 只要其中有一点修改, 则用公钥无法校验通过,
        // 因此可以用私钥签名, 然后用公钥来校验数据的完整性与签名者（所有者）
        boolean verify = sign.verify(signInfo);

        return verify;
    }

    /**
     * 私钥签名（文件）: 用私钥对文件进行签名, 返回签名信息
     */
    public static byte[] signFile(File file, PrivateKey priKey) throws Exception {
        // 根据指定算法获取签名工具
        Signature sign = Signature.getInstance(SIGNATURE_ALGORITHM);

        // 用私钥初始化签名工具
        sign.initSign(priKey);

        InputStream in = null;

        try {
            in = new FileInputStream(file);

            byte[] buf = new byte[1024];
            int len = -1;

            while ((len = in.read(buf)) != -1) {
                // 添加要签名的数据
                sign.update(buf, 0, len);
            }

        } finally {
            close(in);
        }

        // 计算并返回签名结果（签名信息）
        return sign.sign();
    }

    /**
     * 公钥验签（文件）: 用公钥校验指定文件的签名是否来自对应的私钥
     */
    public static boolean verifyFile(File file, byte[] signInfo, PublicKey pubKey) throws Exception {
        // 根据指定算法获取签名工具
        Signature sign = Signature.getInstance(SIGNATURE_ALGORITHM);

        // 用公钥初始化签名工具
        sign.initVerify(pubKey);

        InputStream in = null;

        try {
            in = new FileInputStream(file);

            byte[] buf = new byte[1024];
            int len = -1;

            while ((len = in.read(buf)) != -1) {
                // 添加要校验的数据
                sign.update(buf, 0, len);
            }

        } finally {
            close(in);
        }

        // 校验签名
        return sign.verify(signInfo);
    }

    private static void close(Closeable c) {
        if (c != null) {
            try {
                c.close();
            } catch (IOException e) {
                // nothing
            }
        }
    }

}
```

`RSASignUtils`类中主要有以下几个公开静态方法：

```java
// 生成 RSA 秘钥对（包含公钥和私钥）
KeyPair generateKeyPair()

// 私钥签名（数据）
byte[]  sign(byte[] data, PrivateKey priKey)
// 公钥验签（数据）
boolean verify(byte[] data, byte[] signInfo, PublicKey pubKey)

// 私钥签名（文件）
byte[]  signFile(File file, PrivateKey priKey)
// 公钥验签（文件）
boolean verifyFile(File file, byte[] signInfo, PublicKey pubKey)
```

工具类的使用：

```java
package com.wantao.rsa;

import sun.misc.BASE64Encoder;

import java.io.File;
import java.security.KeyPair;
import java.security.PrivateKey;
import java.security.PublicKey;

/**
 * @author xietansheng
 */
public class Main {

    public static void main(String[] args) throws Exception {
        // 随机生成一对 RAS 密钥（包含公钥和私钥）
        KeyPair keyPair = RSASignUtils.generateKeyPair();
        // 获取 公钥 和 私钥
        PublicKey pubKey = keyPair.getPublic();
        PrivateKey priKey = keyPair.getPrivate();

        // 原始数据
        String data = "你好, World";

        // 私钥签名（数据）: 对数据进行签名, 返回签名结果
        byte[] signInfo = RSASignUtils.sign(data.getBytes(), priKey);
        System.out.println("数据签名信息:" + new BASE64Encoder().encode(signInfo));

        // 公钥验签（数据）: 用公钥校验数据的签名是否来自公钥对应的私钥
        boolean verify = RSASignUtils.verify(data.getBytes(), signInfo, pubKey);
        System.out.println("数据验签结果:" + verify);

        /*
         * 对文件进行签名和验签
         */
        File file = new File("demo.jpg");

        // 私钥签名（文件）: 对文件进行签名, 返回签名结果
        byte[] fileSignInfo = RSASignUtils.sign(data.getBytes(), priKey);
        System.out.println("文件签名信息:" + new BASE64Encoder().encode(fileSignInfo));

        // 公钥验签（文件）: 用公钥校验文件的签名是否来自公钥对应的私钥
        boolean fileVerify = RSASignUtils.verify(data.getBytes(), signInfo, pubKey);
        System.out.println("文件验签结果:" + fileVerify);
    }

}
```

