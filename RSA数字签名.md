### RSA数字签名

> 流程：
>
> * 加密时如果是多数据则进行排序
>   |->元数据 -> 生成MD5摘要 -> 私钥加密 -> <>数字签名</>
>   	|->请求时:元数据+数字签名
> * 接收时：
>
>   |->数字签名 -> 公钥解密 -> MD5摘要
>
>   |->明文 -> 生成MD5摘要
>   	|->摘要比较

#### 安装openssl

下载安装设置环境变量即可使用http://slproweb.com/products/Win32OpenSSL.html

进入openssl安装目录

>1024 生成1024长度私钥
>
>openssl genrsa -out rsa_private.key` 
>
>生成公钥
>
>`openssl rsa -in rsa_private.key -out rsa_public.key -pubout` 
>
>转换私钥为PKCS#8 如果不转换为PKCS#1 编码规范
>
>`openssl pkcs8 -topk8 -in rsa_private.key -out pkcs8_rsa_private.key -nocrypt `

#### 服务端代码

> 公钥使用pem格式X509标准,私钥使用der格式PKCS8标准
>
> 两者格式编码不同，下面使用两种方式进行加解密

```java
import cn.hutool.core.io.FileUtil;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.serializer.SerializerFeature;
import com.google.common.collect.Lists;
import com.google.common.collect.Maps;
import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;

import java.io.*;
import java.security.*;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;
public class CyperUtil {

    private static final String CHARSET = "UTF-8";

    private static final String RSA = "RSA";

    private static final String PUBLIC_KEY_PATH = CyperUtil.class.getResource("/").getPath()+ "sign/rsa_pub_key.pem";
    private static final String PRIVATE_KEY_PATH = CyperUtil.class.getResource("/").getPath()+ "sign/pkcs8_rsa_pri_key.der";
    /**
     * 用私钥对信息生成数字签名
     * @param data 数据
     * @param privateKey 私钥
     * @return base64加密后数字签名
     * @throws Exception
     */
    public static String sign(String data, PrivateKey privateKey) throws
            Exception {
        //MD5摘要
        ByteArrayOutputStream byteArrayOut = new ByteArrayOutputStream();

        DataOutputStream dataOut = new DataOutputStream(byteArrayOut);

        String digestData = digest(data);

        dataOut.write(digestData.getBytes(CHARSET));
        //私钥加密
        Signature sig = Signature.getInstance("SHA1WithRSA");

        sig.initSign(privateKey);

        sig.update(byteArrayOut.toByteArray());

        byte[] signatureBytes = sig.sign();
        //base64加密后一行超过64会添加换行符
        return new BASE64Encoder().encode(signatureBytes).replaceAll("[\\s*\t\n\r]", "");
    }

    /**
     * 校验数字签名
     * @param data 数据
     * @param sign 数字签名
     * @return true 相同 false不同
     * @throws Exception
     *
     */
    public static boolean verify(String data, String sign)
            throws Exception {
        PublicKey publicKey = readPublicKeyFromFile(PUBLIC_KEY_PATH);
        //MD5摘要
        ByteArrayOutputStream byteArrayOut = new ByteArrayOutputStream();

        DataOutputStream dataOut = new DataOutputStream(byteArrayOut);

        String digestData = digest(data);

        dataOut.write(digestData.getBytes(CHARSET));
        //公钥解密
        Signature sig = Signature.getInstance("SHA1WithRSA");
        
        sig.initVerify(publicKey);

        sig.update(byteArrayOut.toByteArray());
        //base64解密并验签
        return sig.verify(new BASE64Decoder().decodeBuffer(sign));
    }

    /**
     * 从文件加载公钥
     * @param path 文件路劲
     * @return PublicKey 公钥
     * @throws IOException, NoSuchAlgorithmException,
     *             InvalidKeySpecException
     */
    public static PublicKey readPublicKeyFromFile (String path)
            throws IOException, NoSuchAlgorithmException,
            InvalidKeySpecException

    {
        String publicKeyPEM = FileUtil.readString(new File(path), CHARSET);

        publicKeyPEM = publicKeyPEM
                .replace("-----BEGIN PUBLIC KEY-----", "")
                .replace("-----END PUBLIC KEY-----", "");

        byte[] publicKeyDER = new BASE64Decoder().decodeBuffer(publicKeyPEM);

        KeyFactory keyFactory = KeyFactory.getInstance(RSA);
        PublicKey publicKey = keyFactory.generatePublic(new X509EncodedKeySpec(publicKeyDER));

        return publicKey;
    }


    /**
     * 从文件加载私钥
     * @param path 文件路劲
     * @return PrivateKey 私钥
     * @throws IOException, NoSuchAlgorithmException,
     *             InvalidKeySpecException
     */
    public static PrivateKey readPrivateKeyFromFile (String path)
            throws IOException, NoSuchAlgorithmException,
            InvalidKeySpecException

    {
        //针对pem文件后缀
        /*String privateKeyPEM = FileUtil.readString(
                new File(path), CHARSET);

        privateKeyPEM = privateKeyPEM
                .replace("-----BEGIN PRIVATE KEY-----", "")
                .replace("-----END PRIVATE KEY-----", "");*/
        //针对der文件后缀
        FileInputStream in = new FileInputStream(path);

        ByteArrayOutputStream out = new ByteArrayOutputStream();

        byte[] byteArray = new byte[2048];

        int count = 0;

        while (-1 != (count = in.read(byteArray))) {
            out.write(byteArray, 0, count);
        }

        //byte[] privateKeyDER = new BASE64Decoder().decodeBuffer(privateKeyPEM);

        PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(out.toByteArray());

        KeyFactory keyFactory = KeyFactory.getInstance(RSA);

        PrivateKey privateKey = keyFactory.generatePrivate(keySpec);

        return privateKey;
    }


    /**
     * 摘要
     * @param input
     * @return String
     * @throws NoSuchAlgorithmException
     * @throws UnsupportedEncodingException
     */
    public static String digest(String input)
            throws NoSuchAlgorithmException, UnsupportedEncodingException {
        MessageDigest messageDigest = MessageDigest.getInstance("MD5");
        byte[] byteArray = messageDigest.digest(input.getBytes(CHARSET));
        String digestData = byteToHexStringSingle(byteArray);
        return digestData;
    }
    /**
     * 独立把byte[]数组转换成十六进制字符串表示形式
     * @param byteArray
     * @return String
     */
    private static String byteToHexStringSingle(byte[] byteArray) {
        StringBuilder builder = new StringBuilder();
        int length = byteArray.length;
        for (int i = 0; i < length; i++) {
            if (1 == Integer.toHexString(0xFF & byteArray[i]).length()) {
                builder.append("0").append(Integer.toHexString(0xFF & byteArray[i]));
            } else {
                builder.append(Integer.toHexString(0xFF & byteArray[i]));
            }
        }
        return builder.toString();
    }

    /**
     * 摘要前多参或对象排序去重转换时间
     * @param obj 摘要的参数
     * @param ignore 忽略的参数
     * @return 序列化的数据
     */
    public static String parseObject(Object obj, String... ignore){
        //将对象中时间类型转换yyyyMMddHHmmss
        Map<String,Object> map = JSONObject.parseObject(JSONObject.toJSONStringWithDateFormat(obj ,"yyyyMMddHHmmss", SerializerFeature.BrowserCompatible), Map.class);
        if(ignore.length > 0){
            ArrayList<String> strings = Lists.newArrayList(ignore);
            HashMap<String, Object> objectObjectHashMap = Maps.newHashMap();
            for(Map.Entry<String, Object> entry : map.entrySet()){
                if(strings.contains(entry.getKey())){// 如果当前属性选择忽略比较，跳到下一次循环
                    continue;
                }
                objectObjectHashMap.put(entry.getKey(),entry.getValue());
            }
            map = objectObjectHashMap;
        }
        //进行排序
        return JSONObject.toJSONString(map, SerializerFeature.SortField.MapSortField);
    }



    public static void main(String[] args) throws Exception{
        //数字签名
        //请求时：元数据MD5摘要算法获取32位摘要信息,在使用私钥进行数字签名
        //验证时：数字签名使用公钥解密,并将入参数据MD5摘要加密进行比对

        String str = "测试1";

        /*PublicKey publickey = CyperUtil.readPublicKeyFromFile
                ("C:\\Users\\Bai\\Desktop\\RSA\\rsa_pub_key.pem");

        PrivateKey privateKey = CyperUtil.readPrivateKeyFromFile
                ("C:\\Users\\Bai\\Desktop\\RSA\\pkcs8_rsa_pri_key.pem");*/

        PrivateKey privateKey = CyperUtil.readPrivateKeyFromFile
                (PRIVATE_KEY_PATH);



        String sign = CyperUtil.sign(str, privateKey);
        System.out.println("sign:" + sign);
        System.out.println("sign length:" + sign.length());
        //比较
        boolean verify = CyperUtil.verify(str, sign);
        System.out.println("verify:" + verify);


    }
```

#### 客户端代码

``` javascript
<script src="jsrsasign-all-min.js"></script>
<script>
  //http://kjur.github.io/jsrsasign/ jsrsasign.js使用地址
  //https://kjur.github.io/jsrsasign/api/symbols/global__.html api
  //注意：
  //1. 密钥对需要带有‘—–BEGIN PRIVATE KEY—–’以及相应的尾标识
  //2. 密钥对格式为PCK#8时使用KEYUTIL.getKey(yourPEMKey)进行解析，而PCK#1则使用rsa.readPrivateKeyFromPEMString
  //3. sig.sign()生成的是一串16进制字符串(hexadecimal)，通过需要转换为base64位使用，通过jsrsasign的全局方法hextob64()进行转换。文档见jsrsasign的global相关方法
  //4. 最后根据情况 需要使用encodeURICompoent(sign)将加密后的密文进行转码，因为一些+号传到后端可能不识别。
  //5.参考：https://www.jianshu.com/p/b32fc387d8ad
  //        https://blog.csdn.net/Sophie_U/article/details/79881872
  window.onload=function show(){

    var rsa = new RSAKey();
    //私钥加密

    //因为后端提供的是pck#8的密钥对，所以这里使用 KEYUTIL.getKey来解析密钥
    var pri = document.getElementById("privkey").value;
    var pub = document.getElementById("pubkey").value;

    rsa = KEYUTIL.getKey(pri);
    // 设定签名以 SHA256 为基准，其他还有sha1等，详见文档
    var sig = new KJUR.crypto.Signature({"alg": "SHA1withRSA"});
    // 初始化
    sig.init(rsa)
    // 传入MD5摘要算法后的字符
    sig.updateString("49d92f4ddd6164663ed8fe915d915dc9")
    // 生成密文并base64编码，注意base64加密后一行超过64会添加换行符，如果出现需要去除换行符
    var sign = hextob64(sig.sign());

    console.log("uri编码前："+sign);
    // 对加密后内容进行URI编码,防止传输过程中出现问题
    sign = encodeURIComponent(sign)

    console.log("uri编码后："+sign);


    }
  </script>
```

