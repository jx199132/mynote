# 概述
 1、与对称加密算法的主要差别在于，加密和解密的密钥不相同，一个公开（公钥），一个保密（私钥）。主要解决了对称加密算法密钥分配管理的问题，提高了算法安全性。
  2、非对称加密算法的加密、解密的效率比较低。在算法设计上，非对称加密算法对待加密的数据长度有着苛刻的要求。例如RSA算法要求待加密的数据不得大于53个字节。
  3、非对称加密算法主要用于 交换对称加密算法的密钥，而非数据交换
  4、java6提供实现了DH和RSA两种算法。Bouncy Castle提供了E1Gamal算法支持。除了上述三种算法还有一个ECC算法，目前没有相关的开源组件提供支持

# 模型分析
我们还是以甲乙双方发送数据为模型进行分析
1、甲方（消息发送方，下同）构建密钥对（公钥+私钥），甲方公布公钥给乙方（消息接收方，下同）
2、乙方以甲方发送过来的公钥作为参数构造密钥对（公钥+私钥），将构造出来的公钥公布给甲方
3、甲方用“甲方的私钥+乙方的公钥”构造本地密钥
4、乙方用“乙方的私钥+甲方的公钥”构造本地的密钥
5、这个时候，甲乙两方本地新构造出来的密钥应该一样。然后就可以使用AES这类对称加密算法结合密钥进行数据的安全传送了。传送过程参考AES的相关算法

# 密钥交互加密算法-DH

运行代码如果出现  java.security.InvalidKeyException: Illegal key size or default parameters异常信息

原因如下：

当密钥大于128时，代码会抛出Java.security.InvalidKeyException: Illegal key size or default parameters

Illegal key size or default parameters是指密钥长度是受限制的，java运行时环境读到的是受限的policy文件。文件位于${java_home}/jre/lib/security

这种限制是因为美国对软件出口的控制。

解决方案：
需要下载Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files.

下载包的readme.txt 有安装说明。就是替换${java_home}/jre/lib/security/ 下面的local_policy.jar和US_export_policy.jar

下载地址  ：  jdk官网 找到对应jdk版本   页面会有 **Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files for JDK/JRE** 这样的字样 对应的链接

```
package dh;

import java.io.UnsupportedEncodingException;
import java.security.InvalidAlgorithmParameterException;
import java.security.InvalidKeyException;
import java.security.Key;
import java.security.KeyFactory;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.KeyAgreement;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.interfaces.DHPublicKey;
import javax.crypto.spec.DHParameterSpec;
import javax.crypto.spec.SecretKeySpec;

import org.apache.commons.codec.binary.Base64;

/**
 * 基于JDK的DH算法，工作模式采用ECB
 */
public class DHCoder {
    private static final String ENCODING = "UTF-8";
    private static final String FDC_KEY_ALGORITHM = "DH";//非对称加密密钥算法
    private static final String DC_KEY_ALGORITHM = "AES";//产生本地密钥的算法（对称加密密钥算法）
    private static final String CIPHER_ALGORITHM = "AES/ECB/PKCS5Padding";//加解密算法 格式：算法/工作模式/填充模式 注意：ECB不使用IV参数
    private static final int FDC_KEY_SIZE = 512;//非对称密钥长度（512~1024之间的64的整数倍）
    
    /**
     * 生成甲方密钥对
     */
    public static KeyPair initKey() throws NoSuchAlgorithmException{
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(FDC_KEY_ALGORITHM);//密钥对生成器
        keyPairGenerator.initialize(FDC_KEY_SIZE);//指定密钥长度
        KeyPair keyPair = keyPairGenerator.generateKeyPair();//生成密钥对
        return keyPair;
    }
    
    /**
     * 生成乙方密钥对
     * @param key 甲方公钥
     */
    public static KeyPair initKey(byte[] key) throws NoSuchAlgorithmException, 
                                                     InvalidKeySpecException, 
                                                     InvalidAlgorithmParameterException{
        KeyFactory keyFactory = KeyFactory.getInstance(FDC_KEY_ALGORITHM);//密钥工厂
        PublicKey publicKey = keyFactory.generatePublic(new X509EncodedKeySpec(key));//还原甲方公钥
        DHParameterSpec dHParameterSpec = ((DHPublicKey)publicKey).getParams();
        
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(keyFactory.getAlgorithm());//乙方密钥对生成器
        keyPairGenerator.initialize(dHParameterSpec);//使用甲方公钥参数初始化乙方密钥对生成器
        KeyPair keyPair = keyPairGenerator.generateKeyPair();//生成密钥对
        return keyPair;
    }
    
    /**
     * DH加密
     * @param data     带加密数据
     * @param keyByte  本地密钥，由getSecretKey(byte[] publicKey, byte[] privateKey)产生
     */
    public static byte[] encrypt(String data, byte[] keyByte) throws NoSuchAlgorithmException, 
                                                                     NoSuchPaddingException, 
                                                                     InvalidKeyException, 
                                                                     IllegalBlockSizeException, 
                                                                     BadPaddingException, 
                                                                     UnsupportedEncodingException {
        Key key = new SecretKeySpec(keyByte, DC_KEY_ALGORITHM);//生成本地密钥

        Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);
        cipher.init(Cipher.ENCRYPT_MODE, key);//设置加密模式并且初始化key
        return cipher.doFinal(data.getBytes(ENCODING));
    }
    
    /**
     * DH解密
     * @param data        待解密数据为字节数组
     * @param keyByte    本地密钥，由getSecretKey(byte[] publicKey, byte[] privateKey)产生
     */
    public static byte[] decrypt(byte[] data, byte[] keyByte) throws NoSuchAlgorithmException, 
                                                                     NoSuchPaddingException, 
                                                                     InvalidKeyException, 
                                                                     IllegalBlockSizeException, 
                                                                     BadPaddingException {
        Key key = new SecretKeySpec(keyByte, DC_KEY_ALGORITHM);//生成本地密钥
        Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);
        cipher.init(Cipher.DECRYPT_MODE, key);
        return cipher.doFinal(data);
    }
    
    /**
     * 根据本方私钥与对方公钥构建本地密钥（即对称加密的密钥）
     * @param publicKey        对方公钥
     * @param privateKey    本方私钥
     */
    public static byte[] getSecretKey(byte[] publicKey, byte[] privateKey) throws NoSuchAlgorithmException, 
                                                                                  InvalidKeySpecException, 
                                                                                  InvalidKeyException{
        KeyFactory keyFactory = KeyFactory.getInstance(FDC_KEY_ALGORITHM);//密钥工厂
        PublicKey pubkey = keyFactory.generatePublic(new X509EncodedKeySpec(publicKey));//还原公钥
        PrivateKey prikey = keyFactory.generatePrivate(new PKCS8EncodedKeySpec(privateKey));//还原私钥
        
        KeyAgreement keyAgreement = KeyAgreement.getInstance(keyFactory.getAlgorithm());
        keyAgreement.init(prikey);
        keyAgreement.doPhase(pubkey, true);
        return keyAgreement.generateSecret(DC_KEY_ALGORITHM).getEncoded();//生成本地密钥（对称加密的密钥）
    }
    
    /**
     * 获取公钥
     */
    public static byte[] getPublicKey(KeyPair keyPair){
        return keyPair.getPublic().getEncoded();
    }
    
    /**
     * 获取私钥
     */
    public static byte[] getPrivateKey(KeyPair keyPair){
        return keyPair.getPrivate().getEncoded();
    }
    
    /**
     * 测试
     */
    public static void main(String[] args) throws NoSuchAlgorithmException, 
                                                  InvalidKeySpecException, 
                                                  InvalidAlgorithmParameterException, 
                                                  InvalidKeyException, 
                                                  NoSuchPaddingException, 
                                                  IllegalBlockSizeException, 
                                                  BadPaddingException, 
                                                  UnsupportedEncodingException {
        byte[] pubKey1;//甲方公钥
        byte[] priKey1;//甲方私钥
        byte[] key1;//甲方本地密钥
        byte[] pubKey2;//乙方公钥
        byte[] priKey2;//乙方私钥
        byte[] key2;//乙方本地密钥
        
        /*********************测试是否可以正确生成以上6个key，以及key1与key2是否相等*********************/
        KeyPair keyPair1 = DHCoder.initKey();//生成甲方密钥对
        pubKey1 = DHCoder.getPublicKey(keyPair1);
        priKey1 = DHCoder.getPrivateKey(keyPair1);
        
        KeyPair keyPair2 = DHCoder.initKey(pubKey1);//根据甲方公钥生成乙方密钥对
        pubKey2 = DHCoder.getPublicKey(keyPair2);
        priKey2 = DHCoder.getPrivateKey(keyPair2);
        
        key1 = DHCoder.getSecretKey(pubKey2, priKey1);//使用对方公钥和自己私钥构建本地密钥
        key2 = DHCoder.getSecretKey(pubKey1, priKey2);//使用对方公钥和自己私钥构建本地密钥
        
        System.out.println("甲方公钥pubKey1-->"+Base64.encodeBase64String(pubKey1)+"@@pubKey1.length-->"+pubKey1.length);
        System.out.println("甲方私钥priKey1-->"+Base64.encodeBase64String(priKey1)+"@@priKey1.length-->"+priKey1.length);
        System.out.println("乙方公钥pubKey2-->"+Base64.encodeBase64String(pubKey2)+"@@pubKey2.length-->"+pubKey2.length);
        System.out.println("乙方私钥priKey2-->"+Base64.encodeBase64String(priKey2)+"@@priKey2.length-->"+priKey2.length);
        System.out.println("甲方密钥key1-->"+Base64.encodeBase64String(key1));
        System.out.println("乙方密钥key2-->"+Base64.encodeBase64String(key2));
        
        /*********************测试甲方使用本地密钥加密数据向乙方发送，乙方使用本地密钥解密数据*********************/
        System.out.println("甲方-->乙方");
        String data = "找一个好姑娘啊！";
        byte[] encodeStr = DHCoder.encrypt(data, key1);
        System.out.println("甲方加密后的数据-->"+Base64.encodeBase64String(encodeStr));
        byte[] decodeStr = DHCoder.decrypt(encodeStr, key2);
        System.out.println("乙方解密后的数据-->"+new String(decodeStr,"UTF-8"));
        
        /*********************测试乙方使用本地密钥加密数据向甲方发送，甲方使用本地密钥解密数据*********************/
        System.out.println("乙方-->甲方");
        String data2 = "找一个好姑娘啊！";
        byte[] encodeStr2 = DHCoder.encrypt(data2, key2);
        System.out.println("乙方加密后的数据-->"+Base64.encodeBase64String(encodeStr2));
        byte[] decodeStr2 = DHCoder.decrypt(encodeStr, key1);
        System.out.println("甲方解密后的数据-->"+new String(decodeStr2,"UTF-8"));
    }
}
```

# 典型非对称加密算法-RSA

```
package rsa;

import java.security.InvalidKeyException;
import java.security.KeyFactory;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.HashMap;
import java.util.Map;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;

import org.apache.commons.codec.binary.Base64;

public abstract class RSACoder {  
    private static final String KEY_ALGORITHM= "RSA";  
    private static final int KEY_SIZE=512;  
    private static final String PUBLIC_KEY = "RSAPublicKey";  
    private static final String PRIVATE_KEY = "RSAPrivateKey";  
      
    public static Map<String,Object> initKey() throws NoSuchAlgorithmException{  
        KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance(KEY_ALGORITHM);  
        keyPairGen.initialize(KEY_SIZE);  
        KeyPair keyPair = keyPairGen.generateKeyPair();  
        //生成密钥当然要生成最丰富的啦  
        RSAPublicKey publicKey = (RSAPublicKey)keyPair.getPublic();  
        RSAPrivateKey privateKey = (RSAPrivateKey)keyPair.getPrivate();  
          
        Map<String,Object> keyMap = new HashMap<String,Object>(2);  
        keyMap.put(PUBLIC_KEY, publicKey);  
        keyMap.put(PRIVATE_KEY, privateKey);  
        return keyMap;  
          
    }  
      
    public static byte[] encryptByPrivateKey(byte[] data,byte[] priKey) throws NoSuchAlgorithmException, InvalidKeySpecException, NoSuchPaddingException, InvalidKeyException, IllegalBlockSizeException, BadPaddingException{  
        PKCS8EncodedKeySpec x509EncodedKeySpec = new PKCS8EncodedKeySpec(priKey);  
        KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);  
        PrivateKey privateKey = keyFactory.generatePrivate(x509EncodedKeySpec);  
          
        Cipher cipher = Cipher.getInstance(KEY_ALGORITHM);  
        cipher.init(Cipher.ENCRYPT_MODE, privateKey);  
        return cipher.doFinal(data);  
          
    }  
      
    public static byte[] decryptByPublicKey(byte[] data,byte[] pubKey) throws NoSuchAlgorithmException, InvalidKeySpecException, NoSuchPaddingException, InvalidKeyException, IllegalBlockSizeException, BadPaddingException{  
        X509EncodedKeySpec pkcs8EncodedKeySpec = new X509EncodedKeySpec(pubKey);  
        KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);  
        PublicKey publicKey = keyFactory.generatePublic(pkcs8EncodedKeySpec);  
          
        Cipher cipher = Cipher.getInstance(KEY_ALGORITHM);  
        cipher.init(Cipher.DECRYPT_MODE, publicKey);  
        return cipher.doFinal(data);  
    }  
      
    public static byte[] getPrivateKey(Map<String,Object> keyMap){  
        RSAPrivateKey privateKey = (RSAPrivateKey) keyMap.get(PRIVATE_KEY);  
        return privateKey.getEncoded();  
    }  
      
    public static byte[] getPublicKey(Map<String,Object> keyMap){  
        RSAPublicKey publicKey = (RSAPublicKey)keyMap.get(PUBLIC_KEY);  
        return publicKey.getEncoded();  
    }
    
    
    public static void main(String[] args) {
    	try {
			byte[] privateKey = null;  
			byte[] publicKey = null; 
			Map<String,Object> keyMap = RSACoder.initKey();  
			publicKey = RSACoder.getPublicKey(keyMap);  
			privateKey = RSACoder.getPrivateKey(keyMap);  
			System.err.println("公钥：\n"+Base64.encodeBase64String(publicKey));  
			System.err.println("私钥：\n"+Base64.encodeBase64String(privateKey));
			
			System.err.println("\n--私钥加密，公钥解密--");  
	        String input1 = "RSA加密算法";  
	        System.err.println("原文：\n"+input1);  
	        byte[] data1 = input1.getBytes();  
	        byte[] encodedata1 = RSACoder.encryptByPrivateKey(data1, privateKey);  
	        System.err.println("加密后：\n"+Base64.encodeBase64String(encodedata1));  
	        byte[] decodedata1 = RSACoder.decryptByPublicKey(encodedata1, publicKey);  
	        System.err.println("解密后：\n"+Base64.encodeBase64String(decodedata1));  
	        String output1 = new String(decodedata1);  
	        System.out.println("结果：\n"+output1);  
	        
	        System.out.println("结果：\n"+ (input1.equals(output1)));  
		} catch (NoSuchAlgorithmException e) {
			e.printStackTrace();
		} catch (InvalidKeyException e) {
			e.printStackTrace();
		} catch (InvalidKeySpecException e) {
			e.printStackTrace();
		} catch (NoSuchPaddingException e) {
			e.printStackTrace();
		} catch (IllegalBlockSizeException e) {
			e.printStackTrace();
		} catch (BadPaddingException e) {
			e.printStackTrace();
		}  
	}
} 
```

# ElGamal

```
package elGamal;

import java.security.AlgorithmParameterGenerator;
import java.security.AlgorithmParameters;
import java.security.InvalidAlgorithmParameterException;
import java.security.InvalidKeyException;
import java.security.Key;
import java.security.KeyFactory;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.SecureRandom;
import java.security.Security;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.InvalidParameterSpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.HashMap;
import java.util.Map;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.spec.DHParameterSpec;

import org.apache.commons.codec.binary.Base64;
import org.bouncycastle.jce.provider.BouncyCastleProvider;

public abstract class ElGamalCoder {
	public static final String KEY_ALGORITHM = "ElGamal";
	private static final int KEY_SIZE = 256;
	private static final String PUBLIC_KEY = "ElGamalPublicKey";
	private static final String PRIVATE_KEY = "ElGamalPrivateKey";

	public static byte[] decryptByPrivateKey(byte[] data, byte[] key)
			throws NoSuchAlgorithmException, InvalidKeySpecException,
			NoSuchPaddingException, InvalidKeyException,
			IllegalBlockSizeException, BadPaddingException {
		Security.addProvider(new BouncyCastleProvider());
		PKCS8EncodedKeySpec pkc = new PKCS8EncodedKeySpec(key);
		KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
		Key privateKey = keyFactory.generatePrivate(pkc);

		Cipher cipher = Cipher.getInstance(KEY_ALGORITHM);
		cipher.init(Cipher.DECRYPT_MODE, privateKey);
		return cipher.doFinal(data);

	}

	public static byte[] encryptByPublicKey(byte[] data, byte[] key)
			throws NoSuchAlgorithmException, InvalidKeySpecException,
			NoSuchPaddingException, InvalidKeyException,
			IllegalBlockSizeException, BadPaddingException {
		Security.addProvider(new BouncyCastleProvider());
		X509EncodedKeySpec x509 = new X509EncodedKeySpec(key);
		KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
		Key publicKey = keyFactory.generatePublic(x509);

		Cipher cipher = Cipher.getInstance(KEY_ALGORITHM);
		cipher.init(Cipher.ENCRYPT_MODE, publicKey);
		return cipher.doFinal(data);

	}

	public static Map<String, Object> initKey()
			throws NoSuchAlgorithmException, InvalidParameterSpecException,
			InvalidAlgorithmParameterException {
		Security.addProvider(new BouncyCastleProvider());
		AlgorithmParameterGenerator apg = AlgorithmParameterGenerator
				.getInstance(KEY_ALGORITHM);
		apg.init(KEY_SIZE);
		AlgorithmParameters params = apg.generateParameters();
		DHParameterSpec dhParams = params
				.getParameterSpec(DHParameterSpec.class);

		KeyPairGenerator keyPairGene = KeyPairGenerator
				.getInstance(KEY_ALGORITHM);
		keyPairGene.initialize(dhParams, new SecureRandom());

		// keyPairGene.initialize(KEY_SIZE, new SecureRandom());
		KeyPair keyPair = keyPairGene.generateKeyPair();
		PublicKey publicKey = keyPair.getPublic();
		PrivateKey privateKey = keyPair.getPrivate();

		Map<String, Object> keyMap = new HashMap<String, Object>(2);
		keyMap.put(PUBLIC_KEY, publicKey);
		keyMap.put(PRIVATE_KEY, privateKey);
		return keyMap;
	}

	public static byte[] getPrivateKey(Map<String, Object> keyMap) {
		Key key = (Key) keyMap.get(PRIVATE_KEY);
		return key.getEncoded();
	}

	public static byte[] getPublicKey(Map<String, Object> keyMap) {
		Key key = (Key) keyMap.get(PUBLIC_KEY);
		return key.getEncoded();
	}

	public static void main(String[] args) {
		try {
			byte[] privateKey = null;
			byte[] publicKey = null;

			Map<String, Object> keyMap = ElGamalCoder.initKey();
			publicKey = ElGamalCoder.getPublicKey(keyMap);
			privateKey = ElGamalCoder.getPrivateKey(keyMap);
			System.err.println("公钥：\n" + Base64.encodeBase64String(publicKey));
			System.err.println("私钥：\n" + Base64.encodeBase64String(privateKey));

			System.err.println("\n--公钥加密，私钥解密--");
			String input1 = "ElGamal加密算法";
			System.err.println("原文：\n" + input1);
			byte[] data1 = input1.getBytes();
			byte[] encodedata1 = ElGamalCoder.encryptByPublicKey(data1,publicKey);
			System.err.println("加密后：\n" + Base64.encodeBase64String(encodedata1));
			byte[] decodedata1 = ElGamalCoder.decryptByPrivateKey(encodedata1,privateKey);
			System.err.println("解密后：\n" + Base64.encodeBase64String(decodedata1));
			String output1 = new String(decodedata1);
			System.err.println("结果：\n" + output1);
			System.out.println("结果：\n" + (input1.equals(output1)));
		} catch (NoSuchAlgorithmException e) {
			e.printStackTrace();
		} catch (InvalidParameterSpecException e) {
			e.printStackTrace();
		} catch (InvalidAlgorithmParameterException e) {
			e.printStackTrace();
		} catch (InvalidKeyException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (InvalidKeySpecException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (NoSuchPaddingException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IllegalBlockSizeException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (BadPaddingException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}
}
```