`AES`（Advanced Encryption Standard，高级加密标准）是一种对称加密算法，加密和解密使用相同的密钥。

## 1. AES 加密/解密 代码实例

Java 代码实现 AES 加密/解密 一般步骤：先根据原始的密码（字节数组/字符串）生成 AES密钥对象；再使用 AES密钥对象 加密/解密 数据。

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

## 2. AES 工具类封装: AESUtils.java

为了方便直接使用，将 AES 加密/解密相关方法封装成工具类，并且支持对文件的 AES 加密/解密。

一共封装 4 个公共静态方法：

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

`AESUtils.java`完整代码：

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

> ```java
>  import java.io.IOException;
> import java.io.UnsupportedEncodingException;
> import java.security.InvalidKeyException;
> import java.security.NoSuchAlgorithmException;
> import java.security.SecureRandom;
> import java.util.Scanner;
> 
>  import javax.crypto.BadPaddingException;
> import javax.crypto.Cipher;
> import javax.crypto.IllegalBlockSizeException;
> import javax.crypto.KeyGenerator;
> import javax.crypto.NoSuchPaddingException;
> import javax.crypto.SecretKey;
> import javax.crypto.spec.SecretKeySpec;
> 
>  import Decoder.BASE64Decoder;
> import Decoder.BASE64Encoder;
> 
>  
>  /*
>  * AES对称加密和解密
>  */
> public class SymmetricEncoder {
>   /*
>    * 加密
>    * 1.构造密钥生成器
>    * 2.根据ecnodeRules规则初始化密钥生成器
>    * 3.产生密钥
>    * 4.创建和初始化密码器
>    * 5.内容加密
>    * 6.返回字符串
>    */
>     public static String AESEncode(String encodeRules,String content){
>         try {
>             //1.构造密钥生成器，指定为AES算法,不区分大小写
>             KeyGenerator keygen=KeyGenerator.getInstance("AES");
>             //2.根据ecnodeRules规则初始化密钥生成器
>             //生成一个128位的随机源,根据传入的字节数组
>             keygen.init(128, new SecureRandom(encodeRules.getBytes()));
>               //3.产生原始对称密钥
>             SecretKey original_key=keygen.generateKey();
>               //4.获得原始对称密钥的字节数组
>             byte [] raw=original_key.getEncoded();
>             //5.根据字节数组生成AES密钥
>             SecretKey key=new SecretKeySpec(raw, "AES");
>               //6.根据指定算法AES自成密码器
>             Cipher cipher=Cipher.getInstance("AES");
>               //7.初始化密码器，第一个参数为加密(Encrypt_mode)或者解密解密(Decrypt_mode)操作，第二个参数为使用的KEY
>             cipher.init(Cipher.ENCRYPT_MODE, key);
>             //8.获取加密内容的字节数组(这里要设置为utf-8)不然内容中如果有中文和英文混合中文就会解密为乱码
>             byte [] byte_encode=content.getBytes("utf-8");
>             //9.根据密码器的初始化方式--加密：将数据加密
>             byte [] byte_AES=cipher.doFinal(byte_encode);
>           //10.将加密后的数据转换为字符串
>             //这里用Base64Encoder中会找不到包
>             //解决办法：
>             //在项目的Build path中先移除JRE System Library，再添加库JRE System Library，重新编译后就一切正常了。
>             String AES_encode=new String(new BASE64Encoder().encode(byte_AES));
>           //11.将字符串返回
>             return AES_encode;
>         } catch (NoSuchAlgorithmException e) {
>             e.printStackTrace();
>         } catch (NoSuchPaddingException e) {
>             e.printStackTrace();
>         } catch (InvalidKeyException e) {
>             e.printStackTrace();
>         } catch (IllegalBlockSizeException e) {
>             e.printStackTrace();
>         } catch (BadPaddingException e) {
>             e.printStackTrace();
>         } catch (UnsupportedEncodingException e) {
>             e.printStackTrace();
>         }
> 
>                 //如果有错就返加nulll
>         return null;         
>     }
>     /*
>      * 解密
>      * 解密过程：
>      * 1.同加密1-4步
>      * 2.将加密后的字符串反纺成byte[]数组
>      * 3.将加密内容解密
>      */
>     public static String AESDncode(String encodeRules,String content){
>         try {
>             //1.构造密钥生成器，指定为AES算法,不区分大小写
>             KeyGenerator keygen=KeyGenerator.getInstance("AES");
>             //2.根据ecnodeRules规则初始化密钥生成器
>             //生成一个128位的随机源,根据传入的字节数组
>             keygen.init(128, new SecureRandom(encodeRules.getBytes()));
>               //3.产生原始对称密钥
>             SecretKey original_key=keygen.generateKey();
>               //4.获得原始对称密钥的字节数组
>             byte [] raw=original_key.getEncoded();
>             //5.根据字节数组生成AES密钥
>             SecretKey key=new SecretKeySpec(raw, "AES");
>               //6.根据指定算法AES自成密码器
>             Cipher cipher=Cipher.getInstance("AES");
>               //7.初始化密码器，第一个参数为加密(Encrypt_mode)或者解密(Decrypt_mode)操作，第二个参数为使用的KEY
>             cipher.init(Cipher.DECRYPT_MODE, key);
>             //8.将加密并编码后的内容解码成字节数组
>             byte [] byte_content= new BASE64Decoder().decodeBuffer(content);
>             /*
>              * 解密
>              */
>             byte [] byte_decode=cipher.doFinal(byte_content);
>             String AES_decode=new String(byte_decode,"utf-8");
>             return AES_decode;
>         } catch (NoSuchAlgorithmException e) {
>             e.printStackTrace();
>         } catch (NoSuchPaddingException e) {
>             e.printStackTrace();
>         } catch (InvalidKeyException e) {
>             e.printStackTrace();
>         } catch (IOException e) {
>             e.printStackTrace();
>         } catch (IllegalBlockSizeException e) {
>             e.printStackTrace();
>         } catch (BadPaddingException e) {
>             e.printStackTrace();
>         }
> 
>                 //如果有错就返加nulll
>         return null;         
>     }
> 
>         public static void main(String[] args) {
>         SymmetricEncoder se=new SymmetricEncoder();
>         Scanner scanner=new Scanner(System.in);
>         /*
>          * 加密
>          */
>         System.out.println("使用AES对称加密，请输入加密的规则");
>         String encodeRules=scanner.next();
>         System.out.println("请输入要加密的内容:");
>         String content = scanner.next();
>         System.out.println("根据输入的规则"+encodeRules+"加密后的密文是:"+se.AESEncode(encodeRules, content));
> 
>                /*
>          * 解密
>          */
>         System.out.println("使用AES对称解密，请输入加密的规则：(须与加密相同)");
>          encodeRules=scanner.next();
>         System.out.println("请输入要解密的内容（密文）:");
>          content = scanner.next();
>         System.out.println("根据输入的规则"+encodeRules+"解密后的明文是:"+se.AESDncode(encodeRules, content));
>     }
> 
>  }
> ```