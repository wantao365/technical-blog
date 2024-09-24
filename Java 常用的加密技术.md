# AES 对称加密算法的加密和解密
> AES（Advanced Encryption Standard，高级加密标准）是一种对称加密算法，加密和解密使用相同的密钥。
> 
> Java 代码实现 AES 加密/解密 一般步骤：先根据原始的密码（字节数组/字符串）生成 AES密钥对象；再使用 AES密钥对象 加密/解密 数据。
> 
```java
package com.wantao.aes;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import java.security.SecureRandom;

public class Main {

    public static void main(String[] args) throws Exception {
        String content = "Hello world!";        // 原文内容
        String key = "123456";                  // AES加密/解密用的原始密码

        // 加密数据, 返回密文
        byte[] cipherBytes = encrypt(content.getBytes(), key.getBytes());

        // 解密数据, 返回明文
        byte[] plainBytes = decrypt(cipherBytes, key.getBytes());

        // 输出解密后的明文: "Hello world!"
        System.out.println(new String(plainBytes));
    }

    /**
     * 生成密钥对象
     */
    private static SecretKey generateKey(byte[] key) throws Exception {
        // 根据指定的 RNG 算法, 创建安全随机数生成器
        SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
        // 设置 密钥key的字节数组 作为安全随机数生成器的种子
        random.setSeed(key);

        // 创建 AES算法生成器
        KeyGenerator gen = KeyGenerator.getInstance("AES");
        // 初始化算法生成器
        gen.init(128, random);

        // 生成 AES密钥对象, 也可以直接创建密钥对象: return new SecretKeySpec(key, ALGORITHM);
        return gen.generateKey();
    }

    /**
     * 数据加密: 明文 -> 密文
     */
    public static byte[] encrypt(byte[] plainBytes, byte[] key) throws Exception {
        // 生成密钥对象
        SecretKey secKey = generateKey(key);

        // 获取 AES 密码器
        Cipher cipher = Cipher.getInstance("AES");
        // 初始化密码器（加密模型）
        cipher.init(Cipher.ENCRYPT_MODE, secKey);

        // 加密数据, 返回密文
        return cipher.doFinal(plainBytes);
    }

    /**
     * 数据解密: 密文 -> 明文
     */
    public static byte[] decrypt(byte[] cipherBytes, byte[] key) throws Exception {
        // 生成密钥对象
        SecretKey secKey = generateKey(key);

        // 获取 AES 密码器
        Cipher cipher = Cipher.getInstance("AES");
        // 初始化密码器（解密模型）
        cipher.init(Cipher.DECRYPT_MODE, secKey);

        // 解密数据, 返回明文
        return cipher.doFinal(cipherBytes);
    }

}
```
>为了方便直接使用，将 AES 加密/解密相关方法封装成工具类，并且支持对文件的 AES 加密/解密。 一共封装 4 个公共静态方法：
```java
// 数据加密: 明文 -> 密文
byte[] encrypt(byte[] plainBytes, byte[] key)
// 数据解密: 密文 -> 明文
byte[] decrypt(byte[] cipherBytes, byte[] key)

// 加密文件: 明文输入 -> 密文输出
void   encryptFile(File plainIn, File cipherOut, byte[] key)
// 解密文件: 密文输入 -> 明文输出
void   decryptFile(File cipherIn, File plainOut, byte[] key)
```
AESUtils.java完整代码：
```java
package com.wantao.aes;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import java.io.Closeable;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.security.SecureRandom;

/**
 * AES 对称算法加密/解密工具类
 */
public class AESUtils {

    /** 密钥长度: 128, 192 or 256 */
    private static final int KEY_SIZE = 128;

    /** 加密/解密算法名称 */
    private static final String ALGORITHM = "AES";

    /** 随机数生成器（RNG）算法名称 */
    private static final String RNG_ALGORITHM = "SHA1PRNG";

    /**
     * 生成密钥对象
     */
    private static SecretKey generateKey(byte[] key) throws Exception {
        // 创建安全随机数生成器
        SecureRandom random = SecureRandom.getInstance(RNG_ALGORITHM);
        // 设置 密钥key的字节数组 作为安全随机数生成器的种子
        random.setSeed(key);

        // 创建 AES算法生成器
        KeyGenerator gen = KeyGenerator.getInstance(ALGORITHM);
        // 初始化算法生成器
        gen.init(KEY_SIZE, random);

        // 生成 AES密钥对象, 也可以直接创建密钥对象: return new SecretKeySpec(key, ALGORITHM);
        return gen.generateKey();
    }

    /**
     * 数据加密: 明文 -> 密文
     */
    public static byte[] encrypt(byte[] plainBytes, byte[] key) throws Exception {
        // 生成密钥对象
        SecretKey secKey = generateKey(key);

        // 获取 AES 密码器
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        // 初始化密码器（加密模型）
        cipher.init(Cipher.ENCRYPT_MODE, secKey);

        // 加密数据, 返回密文
        byte[] cipherBytes = cipher.doFinal(plainBytes);

        return cipherBytes;
    }

    /**
     * 数据解密: 密文 -> 明文
     */
    public static byte[] decrypt(byte[] cipherBytes, byte[] key) throws Exception {
        // 生成密钥对象
        SecretKey secKey = generateKey(key);

        // 获取 AES 密码器
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        // 初始化密码器（解密模型）
        cipher.init(Cipher.DECRYPT_MODE, secKey);

        // 解密数据, 返回明文
        byte[] plainBytes = cipher.doFinal(cipherBytes);

        return plainBytes;
    }

    /**
     * 加密文件: 明文输入 -> 密文输出
     */
    public static void encryptFile(File plainIn, File cipherOut, byte[] key) throws Exception {
        aesFile(plainIn, cipherOut, key, true);
    }

    /**
     * 解密文件: 密文输入 -> 明文输出
     */
    public static void decryptFile(File cipherIn, File plainOut, byte[] key) throws Exception {
        aesFile(plainOut, cipherIn, key, false);
    }

    /**
     * AES 加密/解密文件
     */
    private static void aesFile(File plainFile, File cipherFile, byte[] key, boolean isEncrypt) throws Exception {
        // 获取 AES 密码器
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        // 生成密钥对象
        SecretKey secKey = generateKey(key);
        // 初始化密码器
        cipher.init(isEncrypt ? Cipher.ENCRYPT_MODE : Cipher.DECRYPT_MODE, secKey);

        // 加密/解密数据
        InputStream in = null;
        OutputStream out = null;

        try {
            if (isEncrypt) {
                // 加密: 明文文件为输入, 密文文件为输出
                in = new FileInputStream(plainFile);
                out = new FileOutputStream(cipherFile);
            } else {
                // 解密: 密文文件为输入, 明文文件为输出
                in = new FileInputStream(cipherFile);
                out = new FileOutputStream(plainFile);
            }

            byte[] buf = new byte[1024];
            int len = -1;

            // 循环读取数据 加密/解密
            while ((len = in.read(buf)) != -1) {
                out.write(cipher.update(buf, 0, len));
            }
            out.write(cipher.doFinal());    // 最后需要收尾

            out.flush();

        } finally {
            close(in);
            close(out);
        }
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
AESUtils 工具类的使用实例：
```java
package com.wantao.aes;

import java.io.File;

public class Main {

    public static void main(String[] args) throws Exception {
        String content = "Hello world!";        // 原文内容
        String key = "123456";                  // AES加密/解密用的原始密码

        // 加密数据, 返回密文
        byte[] cipherBytes = AESUtils.encrypt(content.getBytes(), key.getBytes());
        // 解密数据, 返回明文
        byte[] plainBytes = AESUtils.decrypt(cipherBytes, key.getBytes());
        // 输出解密后的明文: "Hello world!"
        System.out.println(new String(plainBytes));

        /*
         * AES 对文件的加密/解密
         */
        // 将 文件demo.jpg 加密后输出到 文件demo.jpg_cipher
        AESUtils.encryptFile(new File("demo.jpg"), new File("demo.jpg_cipher"), key.getBytes());
        // 将 文件demo.jpg_cipher 解密后输出到 文件demo.jpg_plain
        AESUtils.decryptFile(new File("demo.jpg_cipher"), new File("demo.jpg_plain"), key.getBytes());

        // 对比 原文件demo.jpg 和 解密得到的文件demo.jpg_plain 两者的 MD5 将会完全相同
    }

}
```

# RSA 非对称加密算法的签名与验签
> RSA 非对称加密算法，除了用来加密/解密数据外，还可以用于对数据（文件）的 签名 和 验签，可用于确认数据或文件的完整性与签名者（所有者）。
>
> RSA 密钥对的生成与加密/解密见上一篇:Java 实现 AES 对称加密算法的加密和解密

RSA 密钥对在使用时通常:

加密/解密：通常使用 私钥加密，公钥解密。
签名/验签：通常使用 私钥签名，公钥验签。
Android 安装包 APK 文件的签名，是 RSA 签名验签的典型应用：Android 打包后，用私钥对 APK 文件进行签名，并把公钥和签名结果放到 APK 包中。下次客户端升级 APK 包时，根据新的 APK 包和包内的签名信息，用 APK 包内的公钥验签校验是否和本地已安装的 APK 包使用的是同一个私钥签名，如果是，则允许安装升级。

# 异或(xor)算法的加密和解密