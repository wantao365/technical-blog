MD5、SHA-1 等 Hash 值的计算结果通常转换为 16 进制字符串的形式保存。

RSA 等算法的密钥通常转换为 Base64 字符串保存。图片也可以编码为 Base64 字符串直接以文本的形式放到 HTML 中显示图片。

## 1. byte[] <-> 16进制字符串

### 1.1 封装工具类：HexUtils

`HexUtils.java`工具类完整源码：

```java
package com.wantao.data;

/**
 * 16进制字符串 与 byte数组 相互转换工具类
 */
public class HexUtils {

    private static final char[] HEXES = {
            '0', '1', '2', '3',
            '4', '5', '6', '7',
            '8', '9', 'a', 'b',
            'c', 'd', 'e', 'f'
    };

    /**
     * byte数组 转换成 16进制小写字符串
     */
    public static String bytes2Hex(byte[] bytes) {
        if (bytes == null || bytes.length == 0) {
            return null;
        }

        StringBuilder hex = new StringBuilder();

        for (byte b : bytes) {
            hex.append(HEXES[(b >> 4) & 0x0F]);
            hex.append(HEXES[b & 0x0F]);
        }

        return hex.toString();
    }

    /**
     * 16进制字符串 转换为对应的 byte数组
     */
    public static byte[] hex2Bytes(String hex) {
        if (hex == null || hex.length() == 0) {
            return null;
        }

        char[] hexChars = hex.toCharArray();
        byte[] bytes = new byte[hexChars.length / 2];   // 如果 hex 中的字符不是偶数个, 则忽略最后一个

        for (int i = 0; i < bytes.length; i++) {
            bytes[i] = (byte) Integer.parseInt("" + hexChars[i * 2] + hexChars[i * 2 + 1], 16);
        }

        return bytes;
    }

}
```

`HexUtils`类中的两个公开静态方法：

```java
// byte数组 转换成 16进制小写字符串
static String bytes2Hex(byte[] bytes)

// 16进制字符串 转换为对应的 byte数组
static byte[] hex2Bytes(String hex)
```

### 1.2 HexUtils 工具类的使用

```java
package com.wantao.data;

public class Main {

    public static void main(String[] args) {
        String data = "Hello World";

        // byte[] 转换为 16进制字符串
        String hex = HexUtils.bytes2Hex(data.getBytes());
        System.out.println(hex);                    // 输出: 48656c6c6f20576f726c64

        // 16进制字符串 转换为 byte[]
        byte[] bytes = HexUtils.hex2Bytes(hex);
        System.out.println(new String(bytes));      // 输出: Hello World
    }

}
```

## 2. byte[] <-> Base64 字符串

### 2.1 封装工具类：Base64Utils

`Base64Utils.java`工具类完整源码：

```java
package com.wantao.data;

import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;

import java.io.ByteArrayOutputStream;
import java.io.Closeable;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;

/**
 * Base64 转换工具
 */
public class Base64Utils {

    /**
     * byte数组 转换为 Base64字符串
     */
    public static String encode(byte[] data) {
        return new BASE64Encoder().encode(data);
    }

    /**
     * Base64字符串 转换为 byte数组
     */
    public static byte[] decode(String base64) {
        try {
            return new BASE64Decoder().decodeBuffer(base64);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return new byte[0];
    }

    /**
     * 把文件内容编码为 Base64字符串, 只能编码小文件（例如文本、图片等）
     */
    public static String encodeFile(File file) throws Exception {
        InputStream in = null;
        ByteArrayOutputStream bytesOut = null;

        try {
            in = new FileInputStream(file);
            bytesOut = new ByteArrayOutputStream((int) file.length());

            byte[] buf = new byte[1024];
            int len = -1;

            while ((len = in.read(buf)) != -1) {
                bytesOut.write(buf, 0, len);
            }
            bytesOut.flush();

            return encode(bytesOut.toByteArray());

        } finally {
            close(in);
            close(bytesOut);
        }
    }

    /**
     * 把 Base64字符串 转换为 byte数组, 保存到指定文件
     */
    public static void decodeFile(String base64, File file) throws Exception {
        OutputStream fileOut = null;
        try {
            fileOut = new FileOutputStream(file);
            fileOut.write(decode(base64));
            fileOut.flush();
        } finally {
            close(fileOut);
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

`Base64Utils`工具类中的几个公开静态方法：

```java
// byte数组 转换为 Base64字符串
static String encode(byte[] data)
// Base64字符串 转换为 byte数组
static byte[] decode(String base64)

// 把文件内容编码为 Base64字符串, 只能编码小文件（例如文本、图片等）
static String encodeFile(File file)
// 把 Base64字符串 转换为 byte数组, 保存到指定文件
static void decodeFile(String base64, File file)
```

### 2.2 Base64Utils 工具类的使用

```java
package com.wantao.data;

import java.io.File;

public class Main {

    public static void main(String[] args) throws Exception {
        String data = "Hello World";

        // 编码: byte[] 转换为 Base64字符串
        String base64 = Base64Utils.encode(data.getBytes());
        System.out.println(base64);                 // 输出: SGVsbG8gV29ybGQ

        // 解码: Base64字符串 转换为 byte[]
        byte[] bytes = Base64Utils.decode(base64);
        System.out.println(new String(bytes));      // 输出: Hello World

        /*
         * 对文件进行 Base64 编码
         */
        // 编码: 把文件内容编码为 Base64字符串
        String fileBase64Str = Base64Utils.encodeFile(new File("demo.png"));
        System.out.println(fileBase64Str);

        // Base64 字符串格式的图片在 html img 标签中的直接显示格式: 
        // <img src="data:image/png;base64,fileBase64Str"/>

        // 解码: 把 Base64字符串 转换为 byte数组, 保存到指定文件
        Base64Utils.decodeFile(fileBase64Str, new File("demo2.png"));

        // 对比 demo.png 和 demo2.png 两个文件的 MD5 将会完全相同
    }

}
```

