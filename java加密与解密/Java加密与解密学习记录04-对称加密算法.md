# 概述
对称加密算法是应用较早的加密算法，技术成熟。在对称加密算法中，数据发信方将明文（原始数据）和加密密钥（mi yao）一起经过特殊加密算法处理后，使其变成复杂的加密密文发送出去。收信方收到密文后，若想解读原文，则需要使用加密用过的密钥及相同算法的逆算法对密文进行解密，才能使其恢复成可读明文。在对称加密算法中，使用的密钥只有一个，发收信双方都使用这个密钥对数据进行加密和解密，这就要求解密方事先必须知道加密密钥。

# 数据加密标准-DES
DES算法和DESade算法统称为DES系列算法。DES算法是对称加密算法领域中的典型算法，为后续对称加密算法的发展奠定了坚实的基础。DESede算法基于DES算法进行三重迭代，增加其算法安全性。
DES算法秘钥片段，仅有56位，迭代次数偏少，导致安全性很低。


| 秘钥长度| 秘钥长度默认值| 工作模式|填充方式|备注|
| ------------- |-------------|-----|-----|-----|
| 56 | 56 | ECB、CBC、PCBC、CTR、CTS、CFB、CFB8至CFB128、OFB、OFB8至OFB128 |NoPadding、PKCS5Padding、ISO10126Padding|Java 6实现
| 64 | 56| 同上|PKCS7Padding、ISO10126d2Padding、X932Padding、ISO7816d4Padding、ZeroBytePadding|Bouncy Castle实现


## des 在Java中的实现

加密解密类
```
 import java.security.Key;  
import java.security.SecureRandom;  
  
import javax.crypto.Cipher;  
import javax.crypto.KeyGenerator;  
import javax.crypto.SecretKey;  
import javax.crypto.SecretKeyFactory;  
import javax.crypto.spec.DESKeySpec;  

import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;
  
  
/** 
 * DES安全编码组件 
 *  
 * <pre> 
 * 支持 DES、DESede(TripleDES,就是3DES)、AES、Blowfish、RC2、RC4(ARCFOUR) 
 * DES                  key size must be equal to 56 
 * DESede(TripleDES)    key size must be equal to 112 or 168 
 * AES                  key size must be equal to 128, 192 or 256,but 192 and 256 bits may not be available 
 * Blowfish             key size must be multiple of 8, and can only range from 32 to 448 (inclusive) 
 * RC2                  key size must be between 40 and 1024 bits 
 * RC4(ARCFOUR)         key size must be between 40 and 1024 bits 
 * 具体内容 需要关注 JDK Document http://.../docs/technotes/guides/security/SunProviders.html 
 * </pre> 
 *  
 * @author 梁栋 
 * @version 1.0 
 * @since 1.0 
 */  
public abstract class DESCoder {  
	
	/** 
     * BASE64解密 
     *  
     * @param key 
     * @return 
     * @throws Exception 
     */  
    public static byte[] decryptBASE64(String key) throws Exception {  
        return (new BASE64Decoder()).decodeBuffer(key);  
    }  
  
    /** 
     * BASE64加密 
     *  
     * @param key 
     * @return 
     * @throws Exception 
     */  
    public static String encryptBASE64(byte[] key) throws Exception {  
        return (new BASE64Encoder()).encodeBuffer(key);  
    } 
	
    
    /**
     * 密钥算法
     */
    public static final String ALGORITHM = "DES";  
    
    /**
     * 加密、解密算法 /工作模式/填充
     */
    public static final String CIPHER_ALGORITHM = "DES/ECB/PKCS5Padding";
  
    /** 
     * 转换密钥<br> 
     *  
     * @param key 
     * @return 
     * @throws Exception 
     */  
    private static Key toKey(byte[] key) throws Exception {  
        DESKeySpec dks = new DESKeySpec(key);  
        SecretKeyFactory keyFactory = SecretKeyFactory.getInstance(ALGORITHM);  
        SecretKey secretKey = keyFactory.generateSecret(dks);  
  
        // 当使用其他对称加密算法时，如AES、Blowfish等算法时，用下述代码替换上述三行代码  
        // SecretKey secretKey = new SecretKeySpec(key, ALGORITHM);  
  
        return secretKey;  
    }  
  
    /** 
     * 解密 
     *  
     * @param data 
     * @param key 
     * @return 
     * @throws Exception 
     */  
    public static byte[] decrypt(byte[] data, String key) throws Exception {  
        Key k = toKey(decryptBASE64(key));  
  
        Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);  
        cipher.init(Cipher.DECRYPT_MODE, k);  
  
        return cipher.doFinal(data);  
    }  
  
    /** 
     * 加密 
     *  
     * @param data 
     * @param key 
     * @return 
     * @throws Exception 
     */  
    public static byte[] encrypt(byte[] data, String key) throws Exception {  
        Key k = toKey(decryptBASE64(key));  
        Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);  
        cipher.init(Cipher.ENCRYPT_MODE, k);  
  
        return cipher.doFinal(data);  
    }  
  
    /** 
     * 生成密钥 
     *  
     * @return 
     * @throws Exception 
     */  
    public static String initKey() throws Exception {  
    	 KeyGenerator kg = KeyGenerator.getInstance(ALGORITHM);  
         kg.init(56);  
         SecretKey secretKey = kg.generateKey();  
   
         return encryptBASE64(secretKey.getEncoded());  
    }  
}  
```

测试类

```
	@Test  
    public void test() throws Exception {  
    	String inputStr = "DES";  
        String key = DESCoder.initKey();  
        System.err.println("原文:\t" + inputStr);  
        System.err.println("密钥:\t" + key);  
        byte[] inputData = inputStr.getBytes();  
        inputData = DESCoder.encrypt(inputData, key);  
        System.err.println("加密后:\t" + DESCoder.encryptBASE64(inputData));  
        byte[] outputData = DESCoder.decrypt(inputData, key);  
        String outputStr = new String(outputData);  
        System.err.println("解密后:\t" + outputStr);  
  
        assertEquals(inputStr, outputStr);  
    }  
```

# 三重DES-DESede
DESede是DES算法的一种改良，该算法针对其密钥长度偏短，迭代次数偏少的问题做了相应改进，提高了安全度。但是DESede算法处理速度较慢，密钥计算时间长，加密效率不高也是存在的问题。这里就不贴代码了

# 高级加密标准-AES
DES算法漏洞的发现加速了对称加密算法的改进，通过对DES算法的简单改造得到的三重DES算法虽然一定程度上提升了算法安全强度。但是三重DES算法低效的加密实现和缓慢的处理速度 存在问题，AES算法正是基于这些缘由而诞生

AES算法建立时间短，灵敏性好、内存需求低等优点，在各个领域得到广泛的研究与应用。
| 秘钥长度| 秘钥长度默认值| 工作模式|填充方式|备注|
| ------------- |-------------|-----|-----|-----|
| 128,192,256| 128| ECB、CBC、PCBC、CTR、CTS、CFB、CFB8至CFB128、OFB、OFB8至OFB128 |NoPadding、PKCS5Padding、ISO10126Padding|Java 6实现
| 128,192,256| 128| 同上|PKCS7Padding、ZeroBytePadding|Bouncy Castle实现

## AES 在 Java中的实现

```
import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.KeyGenerator;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;

public class Encrypt{
	
	  private static final String password = "xxxxxxx";
	  
	  private static final int KEY_LENGTH = 128;
	  
	  public static void main(String[] args){
	    String content = "fasdfa";
	    
	    System.out.println("加密前" + content);
	    byte[] encryptResult = encrypt(content);
	    String encryptResultStr = parseByte2HexStr(encryptResult);
	    System.out.println("加密后" + encryptResultStr);
	    
	    byte[] decryptFrom = parseHexStr2Byte(encryptResultStr);
	    byte[] decryptResult = decrypt(decryptFrom);
	    System.out.println("解密后" + new String(decryptResult));
	  }
	  
	  public static byte[] parseHexStr2Byte(String hexStr)
	  {
	    if (hexStr.length() < 1) {
	      return null;
	    }
	    byte[] result = new byte[hexStr.length() / 2];
	    for (int i = 0; i < hexStr.length() / 2; i++){
	      int high = Integer.parseInt(hexStr.substring(i * 2, i * 2 + 1), 16);
	      int low = Integer.parseInt(hexStr.substring(i * 2 + 1, i * 2 + 2), 16);
	      result[i] = ((byte)(high * 16 + low));
	    }
	    return result;
	  }
	  
	  public static String parseByte2HexStr(byte[] buf){
	    StringBuffer sb = new StringBuffer();
	    for (int i = 0; i < buf.length; i++){
	      String hex = Integer.toHexString(buf[i] & 0xFF);
	      if (hex.length() == 1) {
	        hex = '0' + hex;
	      }
	      sb.append(hex.toUpperCase());
	    }
	    return sb.toString();
	  }
	  
	  public static byte[] encrypt(String content){
	    try{
	      KeyGenerator kgen = KeyGenerator.getInstance("AES");
	      kgen.init(KEY_LENGTH, new SecureRandom(password.getBytes()));
	      SecretKey secretKey = kgen.generateKey();
	      byte[] enCodeFormat = secretKey.getEncoded();
	      SecretKeySpec key = new SecretKeySpec(enCodeFormat, "AES");
	      Cipher cipher = Cipher.getInstance("AES");
	      byte[] byteContent = content.getBytes("utf-8");
	      cipher.init(1, key);
	      return cipher.doFinal(byteContent);
	    }catch (NoSuchAlgorithmException e){
	      e.printStackTrace();
	    }catch (NoSuchPaddingException e){
	      e.printStackTrace();
	    }catch (InvalidKeyException e){
	      e.printStackTrace();
	    }catch (UnsupportedEncodingException e){
	      e.printStackTrace();
	    }catch (IllegalBlockSizeException e){
	      e.printStackTrace();
	    }catch (BadPaddingException e){
	      e.printStackTrace();
	    }
	    return null;
	  }
	  
	  public static byte[] decrypt(byte[] content){
	    try
	    {
	      KeyGenerator kgen = KeyGenerator.getInstance("AES");
	      kgen.init(KEY_LENGTH, new SecureRandom(password.getBytes()));
	      SecretKey secretKey = kgen.generateKey();
	      byte[] enCodeFormat = secretKey.getEncoded();
	      SecretKeySpec key = new SecretKeySpec(enCodeFormat, "AES");
	      Cipher cipher = Cipher.getInstance("AES");
	      cipher.init(2, key);
	      return cipher.doFinal(content);
	    }catch (NoSuchAlgorithmException e){
	      e.printStackTrace();
	    }catch (NoSuchPaddingException e){
	      e.printStackTrace();
	    }catch (InvalidKeyException e){
	      e.printStackTrace();
	    }catch (IllegalBlockSizeException e){
	      e.printStackTrace();
	    }catch (BadPaddingException e){
	      e.printStackTrace();
	    }
	    return null;
	  }
}
```

# 国际加密标准-IDEA
IDEA（international Data encryption algorithm，国际数据加密标准）算法是旅居瑞士中国青年学者来学家和著名密码专家J.Massey于1990年提出的。它在1990年正式公布并在以后得到增强。这种算法是在DES算法的基础上发展出来的，类似于三重DES，和DES一样IDEA也是属于对称密钥算法。发展IDEA也是因为感到DES具有密钥太短等缺点，已经过时。IDEA的密钥为128位，这么长的密钥在今后若干年内应该是安全的。 

## IDEA在Java中的实现
java 6 中没有提供IDEA算法的相应实现，若需要使用该算法可以通过Bouncy Castle来完成。Bouncy Castele不仅仅提供了 IDEA算法实现，包括其他Java6 未能支持的对称加密算法也可以通过Bouncy Castle实现。

下载地址  https://downloads.bouncycastle.org/java/bcprov-jdk15on-157.jar

```
package idea;
import org.apache.commons.codec.binary.Base64;
import org.bouncycastle.jce.provider.BouncyCastleProvider;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.security.Key;
import java.security.Security;

/**
 * 功能
 * <p>
 * Created by zhangyuxin on 2016/8/18.
 * version
 */
public class IDEACode {
    /**
     * 密钥算法
     * */
    public static final String KEY_ALGORITHM="IDEA";

    /**
     * 加密/解密算法/工作模式/填充方式
     * */
    public static final String CIPHER_ALGORITHM="IDEA/ECB/ISO10126Padding";
    /**
     *
     * 生成密钥，只有bouncycastle支持
     * @return byte[] 二进制密钥
     * */
    public static byte[] initkey() throws Exception{
        //加入bouncyCastle支持
        Security.addProvider(new BouncyCastleProvider());

        //实例化密钥生成器
        KeyGenerator kg=KeyGenerator.getInstance(KEY_ALGORITHM);
        //初始化密钥生成器，IDEA要求密钥长度为128位
        kg.init(128);
        //生成密钥
        SecretKey secretKey=kg.generateKey();
        //获取二进制密钥编码形式
        return secretKey.getEncoded();
    }
    /**
     * 转换密钥
     * @param key 二进制密钥
     * @return Key 密钥
     * */
    private static Key toKey(byte[] key) throws Exception{
        //实例化DES密钥
        //生成密钥
        SecretKey secretKey=new SecretKeySpec(key,KEY_ALGORITHM);
        return secretKey;
    }

    /**
     * 加密数据
     * @param data 待加密数据
     * @param key 密钥
     * @return byte[] 加密后的数据
     * */
    private static byte[] encrypt(byte[] data,byte[] key) throws Exception{
        //加入bouncyCastle支持
        Security.addProvider(new BouncyCastleProvider());
        //还原密钥
        Key k=toKey(key);
        //实例化
        Cipher cipher=Cipher.getInstance(CIPHER_ALGORITHM);
        //初始化，设置为加密模式
        cipher.init(Cipher.ENCRYPT_MODE, k);
        //执行操作
        return cipher.doFinal(data);
    }
    /**
     * 解密数据
     * @param data 待解密数据
     * @param key 密钥
     * @return byte[] 解密后的数据
     * */
    private static byte[] decrypt(byte[] data,byte[] key) throws Exception{
        //加入bouncyCastle支持
        Security.addProvider(new BouncyCastleProvider());
        //还原密钥
        Key k =toKey(key);
        Cipher cipher=Cipher.getInstance(CIPHER_ALGORITHM);
        //初始化，设置为解密模式
        cipher.init(Cipher.DECRYPT_MODE, k);
        //执行操作
        return cipher.doFinal(data);
    }
    public static String getKey(){
        String result = null;
        try {
            result = Base64.encodeBase64String(initkey());
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
    public static String ideaEncrypt(String data, String key) {
        String result = null;
        try {
            byte[] data_en = encrypt(data.getBytes(), Base64.decodeBase64(key));
            result = Base64.encodeBase64String(data_en);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    public static String ideaDecrypt(String data, String key) {
        String result = null;
        try {
            byte[] data_de =IDEACode.decrypt(Base64.decodeBase64(data), Base64.decodeBase64(key));;
            result = new String(data_de);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
    public static void main(String[] args) throws Exception {
        String data = "test string";
        String key = getKey();
        System.out.println("密钥：" + key);
        String data_en = ideaEncrypt(data, key);
        System.out.println("密文："+data_en);
        String data_de = ideaDecrypt(data_en, key);
        System.out.println("原文："+data_de);
    }
}
```


# 基于口令加密-PBE
前面描述的几种对称加密算法，这些算法的应用模型几乎一样。但并不是所有的对称加密算法都是如此，PBE算法综合了前面描述的对称加密算法和消息摘要算法的优势，形成了一个对称加密算法的特例。

PBE(Password Based Encryption，基于口令加密）算法是一种基于口令的加密算法，其特点在于口令由用户自己掌管，采用随机数（也称作“盐”）杂凑多重加密算法等方法保证数据的安全性。

PBE算法没有密钥的概念，把口令当做密钥了。因为密钥长短影响算法安全性，还不方便记忆，这里我们直接换成我们自己常用的口令就大大不同了，便于我们的记忆。但是单纯的口令很容易被字典法给穷举出来，所以我们这里给口令加了点“盐”，这个盐和口令组合，想破解就难了。同时我们将盐和口令合并后用消息摘要算法进行迭代很多次来构建密钥初始化向量的基本材料，使破译更加难了。

## PBE在Java中的实现

参考 : http://desert3.iteye.com/blog/743713

```
package pbe;
import sun.misc.BASE64Encoder;  
  
import javax.crypto.*;  
import javax.crypto.spec.PBEKeySpec;  
import javax.crypto.spec.PBEParameterSpec;  
import java.io.UnsupportedEncodingException;  
import java.security.InvalidAlgorithmParameterException;  
import java.security.InvalidKeyException;  
import java.security.Key;  
import java.security.NoSuchAlgorithmException;  
import java.security.spec.InvalidKeySpecException;  
import java.util.Random;  
  
/** 
 * Created by xiang.li on 2015/2/28. 
 * PBE 加解密工具类 
 */  
public class PBE {  
    /** 
     * 定义加密方式 
     * 支持以下任意一种算法 
     * <p/> 
     * <pre> 
     * PBEWithMD5AndDES 
     * PBEWithMD5AndTripleDES 
     * PBEWithSHA1AndDESede 
     * PBEWithSHA1AndRC2_40 
     * </pre> 
     */  
    private final static String KEY_PBE = "PBEWITHMD5andDES";  
  
    private final static int SALT_COUNT = 100;  
  
    /** 
     * 初始化盐（salt） 
     * 
     * @return 
     */  
    public static byte[] init() {  
        byte[] salt = new byte[8];  
        Random random = new Random();  
        random.nextBytes(salt);  
        return salt;  
    }  
  
    /** 
     * 转换密钥 
     * 
     * @param key 字符串 
     * @return 
     */  
    public static Key stringToKey(String key) {  
        SecretKey secretKey = null;  
        try {  
            PBEKeySpec keySpec = new PBEKeySpec(key.toCharArray());  
            SecretKeyFactory factory = SecretKeyFactory.getInstance(KEY_PBE);  
            secretKey = factory.generateSecret(keySpec);  
        } catch (NoSuchAlgorithmException e) {  
            e.printStackTrace();  
        } catch (InvalidKeySpecException e) {  
            e.printStackTrace();  
        }  
        return secretKey;  
    }  
  
    /** 
     * PBE 加密 
     * 
     * @param data 需要加密的字节数组 
     * @param key  密钥 
     * @param salt 盐 
     * @return 
     */  
    public static byte[] encryptPBE(byte[] data, String key, byte[] salt) {  
        byte[] bytes = null;  
        try {  
            // 获取密钥  
            Key k = stringToKey(key);  
            PBEParameterSpec parameterSpec = new PBEParameterSpec(salt, SALT_COUNT);  
            Cipher cipher = Cipher.getInstance(KEY_PBE);  
            cipher.init(Cipher.ENCRYPT_MODE, k, parameterSpec);  
            bytes = cipher.doFinal(data);  
        } catch (NoSuchAlgorithmException e) {  
            e.printStackTrace();  
        } catch (NoSuchPaddingException e) {  
            e.printStackTrace();  
        } catch (InvalidAlgorithmParameterException e) {  
            e.printStackTrace();  
        } catch (InvalidKeyException e) {  
            e.printStackTrace();  
        } catch (BadPaddingException e) {  
            e.printStackTrace();  
        } catch (IllegalBlockSizeException e) {  
            e.printStackTrace();  
        }  
        return bytes;  
    }  
  
    /** 
     * PBE 解密 
     * 
     * @param data 需要解密的字节数组 
     * @param key  密钥 
     * @param salt 盐 
     * @return 
     */  
    public static byte[] decryptPBE(byte[] data, String key, byte[] salt) {  
        byte[] bytes = null;  
        try {  
            // 获取密钥  
            Key k = stringToKey(key);  
            PBEParameterSpec parameterSpec = new PBEParameterSpec(salt, SALT_COUNT);  
            Cipher cipher = Cipher.getInstance(KEY_PBE);  
            cipher.init(Cipher.DECRYPT_MODE, k, parameterSpec);  
            bytes = cipher.doFinal(data);  
        } catch (NoSuchAlgorithmException e) {  
            e.printStackTrace();  
        } catch (NoSuchPaddingException e) {  
            e.printStackTrace();  
        } catch (InvalidAlgorithmParameterException e) {  
            e.printStackTrace();  
        } catch (InvalidKeyException e) {  
            e.printStackTrace();  
        } catch (BadPaddingException e) {  
            e.printStackTrace();  
        } catch (IllegalBlockSizeException e) {  
            e.printStackTrace();  
        }  
        return bytes;  
    }  
  
    /** 
     * BASE64 加密 
     * 
     * @param key 需要加密的字节数组 
     * @return 字符串 
     * @throws Exception 
     */  
    public static String encryptBase64(byte[] key) throws Exception {  
        return (new BASE64Encoder()).encodeBuffer(key);  
    }  
  
    /** 
     * 测试方法 
     * 
     * @param args 
     */  
    public static void main(String[] args) {  
        // 加密前的原文  
        String str = "hello world !!!";  
        // 口令  
        String key = "qwert";  
        // 初始化盐  
        byte[] salt = init();  
        // 采用PBE算法加密  
        byte[] encData = encryptPBE(str.getBytes(), key, salt);  
        // 采用PBE算法解密  
        byte[] decData = decryptPBE(encData, key, salt);  
        String encStr = null;  
        String decStr = null;  
        try {  
            encStr = encryptBase64(encData);  
            decStr = new String(decData, "UTF-8");  
        } catch (UnsupportedEncodingException e) {  
            e.printStackTrace();  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
        System.out.println("加密前：" + str);  
        System.out.println("加密后：" + encStr);  
        System.out.println("解密后：" + decStr);  
    }  
}
```

 