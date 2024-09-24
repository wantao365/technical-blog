## 1. 异或（xor）加密原理

一个整数 a 和任意一个整数 b 异或两次，得到的结果是整数 a 本身，即: `a == a ^ b ^ b`。

这里的 a 就是需要加密的原数据，b 则是密钥。a ^ b 就是加密过程，异或的结果就是加密后的密文。密文 (a ^ b) 再与密钥 b 异或，就是解密过程，得到的结果就是原数据 a 本身。

```java
a = 原数据
b = 密钥

// 一次异或, 加密得到密文
c = a ^ b

// 二次异或, 解密得到原数据（d == a）
d = c ^ b
```

> 异或加密如果同时知道原文和密文，则对比原文和密文可以推算出密钥，因此异或加密安全性较低，一般只用于简单的加密。

## 2. 异或加密代码实例

### 2.1 异或加密工具类封装：XORUtils

`XORUtils.java`完整源码：

```java
package com.wantao.xor;

import java.io.BufferedOutputStream;
import java.io.Closeable;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;

/**
 * 异或(xor)算法加密/解密工具
 */
public class XORUtils {

    /**
     * 异或算法加密/解密
     *
     * @param data 数据（密文/明文）
     * @param key 密钥
     * @return 返回解密/加密后的数据
     */
    public static byte[] encrypt(byte[] data, byte[] key) {
        if (data == null || data.length == 0 || key == null || key.length == 0) {
            return data;
        }

        byte[] result = new byte[data.length];

        // 使用密钥字节数组循环加密或解密
        for (int i = 0; i < data.length; i++) {
            // 数据与密钥异或, 再与循环变量的低8位异或（增加复杂度）
            result[i] = (byte) (data[i] ^ key[i % key.length] ^ (i & 0xFF));
        }

        return result;
    }

    /**
     * 对文件异或算法加密/解密
     *
     * @param inFile 输入文件（密文/明文）
     * @param outFile 结果输出文件
     * @param key 密钥
     */
    public static void encryptFile(File inFile, File outFile, byte[] key) throws Exception {
        InputStream in = null;
        OutputStream out = null;

        try {
            // 文件输入流
            in = new FileInputStream(inFile);
            // 结果输出流, 异或运算时, 字节是一个一个读取和写入, 这里必须使用缓冲流包装,
            // 等缓冲到一定数量的字节（10240字节）后再写入磁盘（否则写磁盘次数太多, 速度会非常慢）
            out = new BufferedOutputStream(new FileOutputStream(outFile), 10240);

            int b = -1;
            long i = 0;

            // 每次循环读取文件的一个字节, 使用密钥字节数组循环加密或解密
            while ((b = in.read()) != -1) {
                // 数据与密钥异或, 再与循环变量的低8位异或（增加复杂度）
                b = (b ^ key[(int) (i % key.length)] ^ (int) (i & 0xFF));
                // 写入一个加密/解密后的字节
                out.write(b);
                // 循环变量递增
                i++;
            }
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

异或加密和解密用的方法都是同一个，`XORUtils`类中有两个公开静态方法：

```java
// 加密/解密 byte 数组数据
static byte[] encrypt(byte[] data, byte[] key)

// 加密/解密 文件
static void encryptFile(File inFile, File outFile, byte[] key)
```

### 2.2 XORUtils 工具类的使用

```java
package com.wantao.xor;

import java.io.File;

public class Main {

    public static void main(String[] args) throws Exception {
        String content = "Hello world!";        // 原文内容
        String key = "123456";                  // XOR 加密/解密用的原始密码

        // 加密数据, 返回密文
        byte[] cipherBytes = XORUtils.encrypt(content.getBytes(), key.getBytes());
        // 解密数据, 返回明文
        byte[] plainBytes = XORUtils.encrypt(cipherBytes, key.getBytes());
        // 输出解密后的明文: "Hello world!"
        System.out.println(new String(plainBytes));

        /*
         * XOR 对文件的加密/解密
         */
        // 将 文件demo.java 加密后输出到 文件demo.jpg_cipher
        XORUtils.encryptFile(new File("demo.jpg"), new File("demo.jpg_cipher"), key.getBytes());
        // 将 文件demo.jpg_cipher 解密输出到 文件demo.jpg_plain
        XORUtils.encryptFile(new File("demo.jpg_cipher"), new File("demo.jpg_plain"), key.getBytes());

        // 对比 原文件demo.jpg 和 解密得到的文件demo.jpg_plain 两者的 MD5 将会完全相同
        
        //加密的字符串含有中文的话，再次异或之后得不到原字符串?
    }

}
```

