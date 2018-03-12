对接一个保险产品接口的时候，对方用的是基于 Java 的 **AES/ECB/PKCS5Padding 对报文进行加密**（AES 加密方法，ECB 加密模式，PKCS5Padding 填充方式），注意他们还**用 SHA1 哈希算法对密码进行哈希**。我用的是 PHP 的 OpenSSL。代码及注释记录如下。
#Java 代码
下面的 Java 示例代码加密后的字符串是`7C22386998D853F7FC97D0EDCC028B77549F80DDFCE96A4C8BF1111C19139F96`：
```
import java.security.SecureRandom;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;

import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class Demo {

  // constants
  private static final String KEY = "123456";
  
  private static final String DEFAULT_CHARSET = "utf-8";
  private static final String PRNG_ALGORITHM_NAME = "SHA1PRNG";
  private static final String ALGORITHM_NAME = "AES";
  private static final String ALGORITHM_STR = ALGORITHM_NAME + "/ECB/PKCS5Padding";

  private static KeyGenerator keyGen;
  private static Cipher cipher;
  private static boolean isInited = false;

  public static void main(String[] args) {
    try {
      calPrem();
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
  
  public static void calPrem() throws Exception {
      String requestData = "解释器*1234567890 abcd<>"; // 待加密字符串
    requestData = encrypt(requestData, KEY); // 加密
    System.out.println(requestData); //7C22386998D853F7FC97D0EDCC028B77549F80DDFCE96A4C8BF1111C19139F96
  }
  
  private static void init(byte[] key, int mode) throws Exception {   
    keyGen = KeyGenerator.getInstance(ALGORITHM_NAME); // 初始化 keyGen，这里用的是 AES 算法
    SecureRandom secureRandom = SecureRandom.getInstance(PRNG_ALGORITHM_NAME); // 使用 SHA1PRNG 随机数算法
    secureRandom.setSeed(key);  // 用指定的 key 作为随机对象的种子，获取随机数
    keyGen.init(128, secureRandom); // 用随机对象 secureRandom 生成 keyGen，128 位
    SecretKey secretKey = keyGen.generateKey();
    byte[] enCodeFormat = secretKey.getEncoded();
    SecretKeySpec secretKeySpec = new SecretKeySpec(enCodeFormat, ALGORITHM_NAME); // 生成 SecretKeySpec
    cipher = Cipher.getInstance(ALGORITHM_STR);
    cipher.init(mode, secretKeySpec); // 初始化cipher
    isInited = true;
  }

  public static byte[] encrypt(byte[] content, byte[] keyBytes) throws Exception {
    byte[] encryptedText = null;
    init(keyBytes, Cipher.ENCRYPT_MODE);
    encryptedText = cipher.doFinal(content); // 注意加密后是字节数组
    return encryptedText;
  }

  public static String encrypt(String a, String key, String charset) throws Exception {
    byte[] resultByte = encrypt(a.getBytes(charset), key.getBytes(charset));
    return parseByte2HexStr(resultByte);
  }

  public static String encrypt(String a, String key) throws Exception {
    return encrypt(a, key, DEFAULT_CHARSET);
  }

  public static byte[] decrypt(byte[] content, byte[] keyBytes) throws Exception {
    byte[] originBytes = null;
    init(keyBytes, Cipher.DECRYPT_MODE);
    originBytes = cipher.doFinal(content);
    return originBytes;
  }

  public static String decrypt(String a, String key, String charset) throws Exception {
    byte[] inputByte = parseHexStr2Byte(a);
    byte[] resultByte = decrypt(inputByte, key.getBytes(charset));
    return new String(resultByte, charset);
  }

  public static String decrypt(String a, String key) throws Exception {
    return decrypt(a, key, DEFAULT_CHARSET);
  }

  private static String parseByte2HexStr(byte buf[]) throws Exception {
    StringBuffer sb = new StringBuffer();
    for (int i = 0; i < buf.length; i++) {
      String hex = Integer.toHexString(buf[i] & 0xFF);
      if (hex.length() == 1) {
        hex = '0' + hex;
      }
      sb.append(hex.toUpperCase());
    }
    return sb.toString();
  }

  private static byte[] parseHexStr2Byte(String hexStr) {
    if (hexStr.length() < 1)
      return null;
    byte[] result = new byte[hexStr.length() / 2];
    for (int i = 0; i < hexStr.length() / 2; i++) {
      int high = Integer.parseInt(hexStr.substring(i * 2, i * 2 + 1), 16);
      int low = Integer.parseInt(hexStr.substring(i * 2 + 1, i * 2 + 2), 16);
      result[i] = (byte) (high * 16 + low);
    }
    return result;
  }
}
```
#PHP 代码：
```php
<?php 

$str = '解释器*1234567890 abcd<>';
$key = '123456';
// sha1 哈希加密给定的密码，只保留 16 字节（128 位），基本对应 Java 的下面步骤：
/*
  keyGen = KeyGenerator.getInstance(ALGORITHM_NAME); // 初始化 keyGen，这里用的是 AES 算法
  SecureRandom secureRandom = SecureRandom.getInstance(PRNG_ALGORITHM_NAME); // 使用 SHA1PRNG 随机数算法
  secureRandom.setSeed(key);  // 用指定的 key 作为随机对象的种子，获取随机数
  keyGen.init(128, secureRandom); // 用随机对象 secureRandom 生成 keyGen，128 位
  SecretKey secretKey = keyGen.generateKey();
  byte[] enCodeFormat = secretKey.getEncoded();
  SecretKeySpec secretKeySpec = new SecretKeySpec(enCodeFormat, ALGORITHM_NAME); // 生成 SecretKeySpec
*/
$key = substr(openssl_digest(openssl_digest($key, 'sha1', true), 'sha1', true), 0, 16);

// 加密，对应下面的 Java 代码：
/*
  cipher.init(mode, secretKeySpec); // 初始化cipher
  encryptedText = cipher.doFinal(content); // 得到字节数组
*/
$t = openssl_encrypt($str, 'AES-128-ECB', $key, 1);
echo strToHex($t);

// 转为 16 进制表示，其中 ord 函数将字符转为对应的 ASCII 码，dechex 函数将数字从十进制转为十六进制
function strToHex($string)
{
    $hex = "";
    for ($i = 0; $i < strlen($string); $i++) {
        $tmp = dechex(ord($string[$i]));
        if (strlen($tmp) == 1) $tmp = '0'.$tmp; // 一个字节必须占 2 个十六进制位，不够则补 0
        $hex .= $tmp;
    }
    $hex = strtoupper($hex);
    return $hex;
}
```