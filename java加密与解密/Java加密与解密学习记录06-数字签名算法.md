# 概述
在超时购买东西刷银行卡支付时，我们都会签下自己的名字，这种行为称之为 手写签名 。签名与使用者一一绑定，称为防止使用者否认的一种手段。如果使用者对自己的行为表示质疑，可以鉴定签名是否一致，判断签名是否有效。
数字签名是计算机软件应用中的一种体现，手写签名用于纸张，数字签名用于数据。

数字签名算法可以看做是一种带有密钥的消息摘要算法，并且这种密钥包含了公钥和私钥。也就是说，数字签名算法是非对称加密算法和消息摘要算法的结合体。

# 作用

	1. 验证数据的完整性 
	2. 认证数据来源 
	3. 抗否认


数字签名遵循：私钥签名，公钥验证 

# RSA

```
package rsa_qm;
import java.security.Key;
import java.security.KeyFactory;  
import java.security.KeyPair;  
import java.security.KeyPairGenerator;  
import java.security.PrivateKey;  
import java.security.PublicKey;  
import java.security.Signature;  
import java.security.interfaces.RSAPrivateKey;  
import java.security.interfaces.RSAPublicKey;  
import java.security.spec.PKCS8EncodedKeySpec;  
import java.security.spec.X509EncodedKeySpec;  
import java.util.HashMap;  
import java.util.Map;  
  
import org.apache.commons.codec.binary.Base64;  
  
/** 
 * 经典的数字签名算法RSA 
 * 数字签名 
 * @author kongqz 
 * */  
public class RSACoder {  
    //数字签名，密钥算法  
    public static final String KEY_ALGORITHM="RSA";  
      
    /** 
     * 数字签名 
     * 签名/验证算法 
     * */  
    public static final String SIGNATURE_ALGORITHM="MD5withRSA";  
      
    /** 
     * RSA密钥长度，RSA算法的默认密钥长度是1024 
     * 密钥长度必须是64的倍数，在512到65536位之间 
     * */  
    private static final int KEY_SIZE=512;  
    //公钥  
    private static final String PUBLIC_KEY="RSAPublicKey";  
      
    //私钥  
    private static final String PRIVATE_KEY="RSAPrivateKey";  
      
    /** 
     * 初始化密钥对 
     * @return Map 甲方密钥的Map 
     * */  
    public static Map<String,Object> initKey() throws Exception{  
        //实例化密钥生成器  
        KeyPairGenerator keyPairGenerator=KeyPairGenerator.getInstance(KEY_ALGORITHM);  
        //初始化密钥生成器  
        keyPairGenerator.initialize(KEY_SIZE);  
        //生成密钥对  
        KeyPair keyPair=keyPairGenerator.generateKeyPair();  
        //甲方公钥  
        RSAPublicKey publicKey=(RSAPublicKey) keyPair.getPublic();  
        //甲方私钥  
        RSAPrivateKey privateKey=(RSAPrivateKey) keyPair.getPrivate();  
        //将密钥存储在map中  
        Map<String,Object> keyMap=new HashMap<String,Object>();  
        keyMap.put(PUBLIC_KEY, publicKey);  
        keyMap.put(PRIVATE_KEY, privateKey);  
        return keyMap;  
          
    }  
      
      
    /** 
     * 签名 
     * @param data待签名数据 
     * @param privateKey 密钥 
     * @return byte[] 数字签名 
     * */  
    public static byte[] sign(byte[] data,byte[] privateKey) throws Exception{  
          
        //取得私钥  
        PKCS8EncodedKeySpec pkcs8KeySpec=new PKCS8EncodedKeySpec(privateKey);  
        KeyFactory keyFactory=KeyFactory.getInstance(KEY_ALGORITHM);  
        //生成私钥  
        PrivateKey priKey=keyFactory.generatePrivate(pkcs8KeySpec);  
        //实例化Signature  
        Signature signature = Signature.getInstance(SIGNATURE_ALGORITHM);  
        //初始化Signature  
        signature.initSign(priKey);  
        //更新  
        signature.update(data);  
        return signature.sign();  
    }  
    /** 
     * 校验数字签名 
     * @param data 待校验数据 
     * @param publicKey 公钥 
     * @param sign 数字签名 
     * @return boolean 校验成功返回true，失败返回false 
     * */  
    public static boolean verify(byte[] data,byte[] publicKey,byte[] sign) throws Exception{  
        //转换公钥材料  
        //实例化密钥工厂  
        KeyFactory keyFactory=KeyFactory.getInstance(KEY_ALGORITHM);  
        //初始化公钥  
        //密钥材料转换  
        X509EncodedKeySpec x509KeySpec=new X509EncodedKeySpec(publicKey);  
        //产生公钥  
        PublicKey pubKey=keyFactory.generatePublic(x509KeySpec);  
        //实例化Signature  
        Signature signature = Signature.getInstance(SIGNATURE_ALGORITHM);  
        //初始化Signature  
        signature.initVerify(pubKey);  
        //更新  
        signature.update(data);  
        //验证  
        return signature.verify(sign);  
    }  
    /** 
     * 取得私钥 
     * @param keyMap 密钥map 
     * @return byte[] 私钥 
     * */  
    public static byte[] getPrivateKey(Map<String,Object> keyMap){  
        Key key=(Key)keyMap.get(PRIVATE_KEY);  
        return key.getEncoded();  
    }  
    /** 
     * 取得公钥 
     * @param keyMap 密钥map 
     * @return byte[] 公钥 
     * */  
    public static byte[] getPublicKey(Map<String,Object> keyMap) throws Exception{  
        Key key=(Key) keyMap.get(PUBLIC_KEY);  
        return key.getEncoded();  
    }  
    /** 
     * @param args 
     * @throws Exception  
     */  
    public static void main(String[] args) throws Exception {  
        //初始化密钥  
        //生成密钥对  
        Map<String,Object> keyMap=RSACoder.initKey();  
        //公钥  
        byte[] publicKey=RSACoder.getPublicKey(keyMap);  
          
        //私钥  
        byte[] privateKey=RSACoder.getPrivateKey(keyMap);  
        System.out.println("公钥：\n"+Base64.encodeBase64String(publicKey));  
        System.out.println("私钥：\n"+Base64.encodeBase64String(privateKey));  
          
        System.out.println("================密钥对构造完毕,甲方将公钥公布给乙方，开始进行加密数据的传输=============");  
        String str="RSA数字签名算法";  
        System.out.println("原文:"+str);  
        //甲方进行数据的加密  
        byte[] sign=RSACoder.sign(str.getBytes(), privateKey);  
        System.out.println("产生签名："+Base64.encodeBase64String(sign));  
        //验证签名  
        boolean status=RSACoder.verify(str.getBytes(), publicKey, sign);  
        System.out.println("状态："+status+"\n\n");  
          
          
    }  
}  
```

# DSA

```
package dsa;
import java.security.Key;
import java.security.KeyFactory;  
import java.security.KeyPair;  
import java.security.KeyPairGenerator;  
import java.security.PrivateKey;  
import java.security.PublicKey;  
import java.security.SecureRandom;  
import java.security.Signature;  
import java.security.interfaces.DSAPrivateKey;  
import java.security.interfaces.DSAPublicKey;  
import java.security.spec.PKCS8EncodedKeySpec;  
import java.security.spec.X509EncodedKeySpec;  
import java.util.HashMap;  
import java.util.Map;  
import org.apache.commons.codec.binary.Base64;  
  
/** 
 * 常用数字签名算法DSA 
 * 数字签名 
 * @author kongqz 
 * */  
public class DSACoder {  
    //数字签名，密钥算法  
    public static final String KEY_ALGORITHM="DSA";  
      
    /** 
     * 数字签名 
     * 签名/验证算法 
     * */  
    public static final String SIGNATURE_ALGORITHM="SHA1withDSA";  
      
    /** 
     * DSA密钥长度，RSA算法的默认密钥长度是1024 
     * 密钥长度必须是64的倍数，在512到1024位之间 
     * */  
    private static final int KEY_SIZE=1024;  
    //公钥  
    private static final String PUBLIC_KEY="DSAPublicKey";  
      
    //私钥  
    private static final String PRIVATE_KEY="DSAPrivateKey";  
      
    /** 
     * 初始化密钥对 
     * @return Map 甲方密钥的Map 
     * */  
    public static Map<String,Object> initKey() throws Exception{  
        //实例化密钥生成器  
        KeyPairGenerator keyPairGenerator=KeyPairGenerator.getInstance(KEY_ALGORITHM);  
        //初始化密钥生成器  
        keyPairGenerator.initialize(KEY_SIZE,new SecureRandom());  
        //生成密钥对  
        KeyPair keyPair=keyPairGenerator.generateKeyPair();  
        //甲方公钥  
        DSAPublicKey publicKey=(DSAPublicKey) keyPair.getPublic();  
        //甲方私钥  
        DSAPrivateKey privateKey=(DSAPrivateKey) keyPair.getPrivate();  
        //将密钥存储在map中  
        Map<String,Object> keyMap=new HashMap<String,Object>();  
        keyMap.put(PUBLIC_KEY, publicKey);  
        keyMap.put(PRIVATE_KEY, privateKey);  
        return keyMap;  
          
    }  
      
      
    /** 
     * 签名 
     * @param data待签名数据 
     * @param privateKey 密钥 
     * @return byte[] 数字签名 
     * */  
    public static byte[] sign(byte[] data,byte[] privateKey) throws Exception{  
          
        //取得私钥  
        PKCS8EncodedKeySpec pkcs8KeySpec=new PKCS8EncodedKeySpec(privateKey);  
        KeyFactory keyFactory=KeyFactory.getInstance(KEY_ALGORITHM);  
        //生成私钥  
        PrivateKey priKey=keyFactory.generatePrivate(pkcs8KeySpec);  
        //实例化Signature  
        Signature signature = Signature.getInstance(SIGNATURE_ALGORITHM);  
        //初始化Signature  
        signature.initSign(priKey);  
        //更新  
        signature.update(data);  
        return signature.sign();  
    }  
    /** 
     * 校验数字签名 
     * @param data 待校验数据 
     * @param publicKey 公钥 
     * @param sign 数字签名 
     * @return boolean 校验成功返回true，失败返回false 
     * */  
    public static boolean verify(byte[] data,byte[] publicKey,byte[] sign) throws Exception{  
        //转换公钥材料  
        //实例化密钥工厂  
        KeyFactory keyFactory=KeyFactory.getInstance(KEY_ALGORITHM);  
        //初始化公钥  
        //密钥材料转换  
        X509EncodedKeySpec x509KeySpec=new X509EncodedKeySpec(publicKey);  
        //产生公钥  
        PublicKey pubKey=keyFactory.generatePublic(x509KeySpec);  
        //实例化Signature  
        Signature signature = Signature.getInstance(SIGNATURE_ALGORITHM);  
        //初始化Signature  
        signature.initVerify(pubKey);  
        //更新  
        signature.update(data);  
        //验证  
        return signature.verify(sign);  
    }  
    /** 
     * 取得私钥 
     * @param keyMap 密钥map 
     * @return byte[] 私钥 
     * */  
    public static byte[] getPrivateKey(Map<String,Object> keyMap){  
        Key key=(Key)keyMap.get(PRIVATE_KEY);  
        return key.getEncoded();  
    }  
    /** 
     * 取得公钥 
     * @param keyMap 密钥map 
     * @return byte[] 公钥 
     * */  
    public static byte[] getPublicKey(Map<String,Object> keyMap) throws Exception{  
        Key key=(Key) keyMap.get(PUBLIC_KEY);  
        return key.getEncoded();  
    }  
    /** 
     * @param args 
     * @throws Exception  
     */  
    public static void main(String[] args) throws Exception {  
        //初始化密钥  
        //生成密钥对  
        Map<String,Object> keyMap=DSACoder.initKey();  
        //公钥  
        byte[] publicKey=DSACoder.getPublicKey(keyMap);  
          
        //私钥  
        byte[] privateKey=DSACoder.getPrivateKey(keyMap);  
        System.out.println("公钥：\n"+Base64.encodeBase64String(publicKey));  
        System.out.println("私钥：\n"+Base64.encodeBase64String(privateKey));  
          
        System.out.println("================密钥对构造完毕,甲方将公钥公布给乙方，开始进行加密数据的传输=============");  
        String str="DSA数字签名算法";  
        System.out.println("原文:"+str);  
        //甲方进行数据的加密  
        byte[] sign=DSACoder.sign(str.getBytes(), privateKey);  
        System.out.println("产生签名："+Base64.encodeBase64String(sign));  
        //验证签名  
        boolean status=DSACoder.verify(str.getBytes(), publicKey, sign);  
        System.out.println("状态："+status+"\n\n"); 
          
          
    }  
}  
```
