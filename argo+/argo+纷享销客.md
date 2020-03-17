## argo与crm打通（纷享销客案例）


**方案**：纷享销客提供了事件回调功能，我们可以在项目中创建事件监听服务，监听纷享销客线索信息，然后将信息同步到argo。


方案确定后，实现该方案只需要完成以下3个步骤
1. 在纷享销客中开启事件回调服务。
2. 在自己项目中添加监听服务，接收纷享销客回调信息
3. 将回调信息同步到argo




##### 第一步：在纷享销客中开启事件回调服务
 可通过观看纷享销客文档，在纷享销客中完成回调事件的配置
>  文档地址：  http://open.fxiaoke.com/wiki.html#artiId=233


##### 第二步：添加监听服务，接收纷享销客回调信息
在项目中添加监听服务，接收纷享销客回调消息，并将纷享销客信息同步到argo

```
/**
 * 纷享销客监听服务
 */
@RestController
@RequestMapping("/fxk")
public class FxkController {

	// 纷享销客-事件回调-解析密钥
	public static final String fxkCallbackToken = "XXX";
	@Resource
	private FxkHandler fxkHandler;

	/**
	 * 接收推送过来的消息的方法
	 *
	 * @param vo
	 * @see MsgReceiveParamVO
	 * @return "success" 表示接收成功,其它表示失败 方法需保证在3秒内返回，否则开平认为推送失败会重试一次
	 *         如果存在复杂的逻辑处理不能保证3秒内返回，需考虑先返回然后开线程去处理业务
	 */
	@RequestMapping(value = "/callback", method = RequestMethod.POST, produces = "application/json;charset=UTF-8")
	public FxkResult decode(@RequestBody MsgReceiveParamVO vo) {
		try {

			// 1、验证签名
			boolean validateResult = SigUtils.verifyMsgReq(vo, fxkCallbackToken);
			if (validateResult) {

				// 2、通过AES算法及约定的秘钥进行解密，获取回调消息
				String content = SigUtils.aesDecrypt(vo.getEncryptedContent(), fxkCallbackToken);
				RecMessageBody body = JSON.parseObject(content, RecMessageBody.class);


				// 3、回调消息处理---线索
				if ("LeadsObj".equals(body.getApiName())) {
					// 4、调用纷享销客-查询对象数据接口，获得线索信息
					JSONObject jsonObject = fxkHandler.getDataById(body);
					if (jsonObject.getJSONArray("owner") != null) {
						// 5、调用纷享销客-获取用户信息接口，获得线索所有人用户信息，取得线索所有人名称
						JSONObject user = fxkHandler.getUserById(jsonObject.getJSONArray("owner").getString(0));
						jsonObject.put("owner_name", user.getString("name"));

						// 6、将纷享销客线索上报到argo
						String userId = "argo用户ID";
						LogSDK.getInstance().profileSet(userId,true, jsonObject);
					}
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		return FxkResult.success(fxkCallbackToken);
	}

}
```


```
/**
 * 纷享销客工具类
 * 
 */
@Service
public class FxkHandler {

	private static final String domain = "https://open.fxiaoke.com/cgi/";
	private static final String appId = "纷享销客-企业应用ID";
	private static final String appSecret = "纷享销客-企业应用凭证密钥";
	private static final String permanentCode = "纷享销客-企业应用获得的公司永久授权码";
	private static final String openUserId = "纷享销客-当前操作人OpenUserID";

	private static final String ACCESS_TOKEN_URL = "/corpAccessToken/get/V2";
	private static final String GET_DATA_URL = "crm/v2/data/get";
	private static final String USER_GET_URL = "/user/get";

	private static final String CORP_ACCESS_TOKEN = "corpAccessToken";
	private static Map<String, CorpAccessToken> accessTokenMap = Maps.newConcurrentMap();

	/**
	 * 与纷享销客建立连接
	 * 文档地址：http://open.fxiaoke.com/wiki.html#artiId=215
	 */
	public CorpAccessToken getAccessToken() {
		// 从缓存中获取数据
		String key = CORP_ACCESS_TOKEN;
		CorpAccessToken corpAccessToken = accessTokenMap.get(key);
		if (corpAccessToken != null) {
			long expiresIn = corpAccessToken.getExpiresIn();
			if (System.currentTimeMillis() < expiresIn) {
				return corpAccessToken;
			}
		}

		accessTokenMap.remove(key);
		synchronized (this) {
			// 多线程环境下，其他线程可能已经获得最新appAccessToken，直接返回
			corpAccessToken = accessTokenMap.get(key);
			if (corpAccessToken != null) {
				return corpAccessToken;
			}

			// 调用接口获取token
			CorpAccessTokenArg corpAccessTokenArg = new CorpAccessTokenArg();
			corpAccessTokenArg.setAppId(appId);
			corpAccessTokenArg.setAppSecret(appSecret);
			corpAccessTokenArg.setPermanentCode(permanentCode);
			JSONObject json = post(ACCESS_TOKEN_URL, corpAccessTokenArg);
			// 封装返回数据
			corpAccessToken = new CorpAccessToken();
			corpAccessToken.setCorpAccessToken(json.getString("corpAccessToken"));
			corpAccessToken.setCorpId(json.getString("corpId"));
			long expiresIn = json.getLong("expiresIn");
			// 减去2分钟，以免过时
			corpAccessToken.setExpiresIn((expiresIn - 2 * 60) * 1000 + System.currentTimeMillis());
			accessTokenMap.put(key, corpAccessToken);
			return corpAccessToken;
		}
	}

	/**
	 * 查询对象数据列表
	 * 文档地址：http://open.fxiaoke.com/wiki.html#artiId=218
	 */
	public JSONObject getDataById(RecMessageBody body) {
		CorpAccessToken token = getAccessToken();
		CrmDataArg crmDataArg = new CrmDataArg();
		crmDataArg.setCorpId(token.getCorpId());
		crmDataArg.setCorpAccessToken(token.getCorpAccessToken());
		crmDataArg.setCurrentOpenUserId(openUserId);
		Map<String, Object> getMap = Maps.newHashMap();
		getMap.put("dataObjectApiName", body.getApiName());
		getMap.put("objectDataId", body.getDataId());
		crmDataArg.setData(getMap);
		JSONObject jsonObject = post(GET_DATA_URL, crmDataArg);
		return jsonObject.getJSONObject("data");
	}

	/**
	 * 获取用户信息
	 * 文档地址：http://open.fxiaoke.com/wiki.html#artiId=23
	 */
	public JSONObject getUserById(String openUserId) {
		CorpAccessToken token = getAccessToken();
		UserGetArg userGetArg = new UserGetArg();
		userGetArg.setCorpId(token.getCorpId());
		userGetArg.setCorpAccessToken(token.getCorpAccessToken());
		userGetArg.setOpenUserId(openUserId);
		return post(USER_GET_URL, userGetArg);
	}

	private JSONObject post(String url, Arg arg) {
		String param = JSONObject.toJSONString(arg);
		RequestBuilder requestBuilder = RequestBuilder.post(domain + url);
		requestBuilder.setEntity(new StringEntity(param, Charset.forName("UTF-8")));
		ThreeTuple<Integer, String, String> postResult =
		        HttpClientProxyManager.getHttpClientProxy().executeJsonResult(requestBuilder);
		JSONObject json = JSONObject.parseObject(postResult.getThird());
		if (!"0".equals(json.getString("errorCode"))) {
			throw new RuntimeException(json.getString("errorMessage"));
		}
		return json;
	}

}
```


##### 第三步：将回调信息同步到argo

```
import java.util.Map;
import cn.analysys.javasdk.AnalysysException;
import cn.analysys.javasdk.AnalysysJavaSdk;
import cn.analysys.javasdk.DEBUG;
import cn.analysys.javasdk.SyncCollecter;
/**
 * argo工具类
 */
public class LogSDK {
	private static LogSDK logSDK = null;
	private static AnalysysJavaSdk analysysJavaSdk = null;
	public static final String ANALYSYS_SERVICE_URL = "方舟serviceUrl";
	public static final String APP_KEY = "方舟appkey";
	/**
	 * 提供单例
	 *
	 * @return
	 */
	public static LogSDK getInstance() {
		if (logSDK == null) {
			logSDK = new LogSDK();
		}
		initSDKInstance();
		return logSDK;
	}

	/**
	 * 初始化java-sdk对象
	 */
	public static void initSDKInstance() {
		if (analysysJavaSdk == null) {
			try {
				analysysJavaSdk = new AnalysysJavaSdk(new SyncCollecter(ANALYSYS_SERVICE_URL), APP_KEY);
				analysysJavaSdk.setDebugMode(DEBUG.OPENANDSAVE);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}

	/**
	 * Description：记录事件
	 *
	 * @param userId     用户ID,长度大于0且小于255字符
	 * @param isLogin    用户ID是否是登录 ID
	 * @param properties 事件属性
	 */
	public void profileSet(String userId, boolean isLogin, Map<String, Object> properties)
			throws AnalysysException {
		if (analysysJavaSdk == null) {
			return;
		}
		analysysJavaSdk.profileSet(userId, isLogin, properties, "JS");
	}
}
```

##### 补充
以下为上述方法中会使用到的代码


```
import java.io.Serializable;
public interface Arg extends Serializable {

}
```

```
@Data
public class BaseArg implements Arg {

	private static final long serialVersionUID = 6280290415792613761L;

	/**
	 * 第三方应用获得企业授权的凭证
	 */
	protected String corpAccessToken;

	/**
	 * 开放平台派发的公司账号
	 */
	protected String corpId;

}

```

```
@Data
public class UserGetArg extends BaseArg {

	private static final long serialVersionUID = 1L;

	// 开放平台员工帐号
	private String openUserId;

	// 如果为true，则会返回员工主属部门(mainDepartmentId)与附属部门(attachingDepartmentIds);默认值为false
	private Boolean showDepartmentIdsDetail;

}
```


```
/**
 * 企业应用token
 */
@Data
public class CorpAccessToken {

	private static final long serialVersionUID = 1L;

	private String corpId;

	private String corpAccessToken;

	private Long expiresIn;
}
```

```

/**
 * 封装获取CorpAccessToken 请求参数的JaveBean
 */
@Data
public class CorpAccessTokenArg implements Arg {

    private static final long serialVersionUID = 119087883828028381L;
    /**
     * 应用ID
     */
    private String appId;
    /**
     * 应用秘钥
     */
    private String appSecret;
    /**
     * 永久授权码
     */
    private String permanentCode;

}
```

```
@Data
public class CrmDataArg extends BaseArg {

	private static final long serialVersionUID = 1L;

	private String currentOpenUserId; // 当前操作人OpenUserID

	private boolean triggerWorkFlow; // 是否触发工作流（不传时默认为true, 表示触发），该参数对所有对象均有效

	private Object data;
}
```

```
public class FxkResult implements Serializable {
	private static final long serialVersionUID = 1L;
	private String encryptedResult;

	public FxkResult() {
	}

	public String getEncryptedResult() {
		return encryptedResult;
	}

	public void setEncryptedResult(String encryptedResult) {
		this.encryptedResult = encryptedResult;
	}

	public static FxkResult getInstance(String result) {
		FxkResult fxkResult = new FxkResult();
		fxkResult.setEncryptedResult(result);
		return fxkResult;
	}

	public static FxkResult success(String token) {
		FxkResult fxkResult = new FxkResult();
		fxkResult.setEncryptedResult(SigUtils.aesEncrypt("success", token));
		return fxkResult;
	}
}
```

```
@Data
public class MsgReceiveParamVO implements Serializable {

	private static final long serialVersionUID = 3966976690051895927L;

	/**
	 * 消息签名，防截获用。由消息体余下的字段及约定秘钥依次按顺序拼接后再通过SHA1算法加密生成。
	 * SHA1[nonce+messageId+retryTimes(如果存在)+enterpriseAccount+encryptMessage+秘钥(在管理后台中配置)]
	 */
	private String signature;
    /**
     * 请求随机数
     */
    private String nonce;

	/**
	 * 请求时间戳
	 */
	private Long timestamp;

	/**
	 * 消息的Id, 全局唯一
	 */
	private String messageId;
	/**
	 * 重试的次数, 仅当消息产生重试时才会有该字段
	 */
	private Integer retryTimes;
	/**
	 * 企业账号
	 */
	private String enterpriseAccount;
	/**
	 * 加密后的密文内容，加密规则为AES对称加密，秘钥为约定的秘钥 加密模式：CBC
	 */
	private String encryptedContent;

}

```

```
@Data
public class RecMessageBody {

	/**
	 * 监听的CRM对象名称
	 */
	private String apiName;

	/**
	 * 事件类型
	 */
	private String eventType;
	/**
	 * 数据的ID
	 */
	private String dataId;
}
```

```

import java.security.MessageDigest;

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;

import org.apache.commons.codec.binary.Base64;
import org.apache.commons.lang.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.google.common.base.Charsets;

/**
 * 加解密工具类
 */
public class SigUtils {

	private static final Logger logger = LoggerFactory.getLogger(SigUtils.class);

	private SigUtils() {
	}

	/**
	 * 验证消息推送请求合法性
	 */
	public static boolean verifyMsgReq(MsgReceiveParamVO vo, String token) {
		boolean verifyResult = false;
		try {
			String sha1Str = SigUtils.getSHA1(vo.getTimestamp(), vo.getNonce(), vo.getMessageId(), vo.getRetryTimes(),
			        vo.getEnterpriseAccount(), vo.getEncryptedContent(), token);
			verifyResult = sha1Str.equals(vo.getSignature()) ? true : false;
		} catch (Exception e) {
			verifyResult = false;
			logger.error(" verify signature error, details:", e);
		}
		return verifyResult;
	}

	/**
	 * 用SHA1算法生成安全签名
	 * 
	 * @param timestamp
	 *            时间
	 * @param nonce
	 *            随机数序列
	 * @param messageId
	 *            消息的Id, 全局唯一
	 * @param retryTimes
	 *            重试的次数, 仅当消息产生重试时才会有该字段
	 * @param enterpriseAccount
	 *            enterpriseAccount
	 * @param encryptedContent
	 *            加密后的密文内容，加密规则为AES对称加密，秘钥为约定的秘钥
	 * @param token
	 *            秘钥(在管理后台中配置)
	 * @return 安全签名 @throws NoSuchAlgorithmException @throws
	 */
	private static String getSHA1(Long timestamp, String nonce, String messageId, Integer retryTimes,
	        String enterpriseAccount, String encryptedContent, String token) throws Exception {
		StringBuilder sb = new StringBuilder();
		if (timestamp != null) {
			sb.append(timestamp);
		}
		if (StringUtils.isNotBlank(nonce)) {
			sb.append(nonce);
		}
		if (StringUtils.isNotBlank(messageId)) {
			sb.append(messageId);
		}
		if (retryTimes != null) {
			sb.append(retryTimes);
		}
		if (StringUtils.isNotBlank(enterpriseAccount)) {
			sb.append(enterpriseAccount);
		}
		if (StringUtils.isNotBlank(encryptedContent)) {
			sb.append(encryptedContent);
		}
		sb.append(token);
		String str = sb.toString();
		// SHA1签名生成
		MessageDigest md = MessageDigest.getInstance("SHA-1");
		md.update(str.getBytes());
		byte[] digest = md.digest();

		StringBuilder hexstr = new StringBuilder();
		String shaHex = "";

		for (int i = 0; i < digest.length; i++) {
			shaHex = Integer.toHexString(digest[i] & 0xFF);
			if (shaHex.length() < 2) {
				hexstr.append(0);
			}
			hexstr.append(shaHex);
		}
		return hexstr.toString();
	}

	/**
	 * aes解密
	 */
	public static String aesDecrypt(String ciphertext, String aesKey) {
		byte[] ciphertextBytes = Base64.decodeBase64(ciphertext.getBytes()); // decode加密密文结果
		byte[] aesKeyBytes = Base64.decodeBase64(aesKey.getBytes()); // decode 秘钥
		SecretKeySpec keySpec = new SecretKeySpec(aesKeyBytes, "AES");
		IvParameterSpec iv = new IvParameterSpec(aesKeyBytes, 0, 16); // 初始化向量
		byte[] plaintextBytes;
		try {
			Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding"); // 加密模式为CBC
			cipher.init(Cipher.DECRYPT_MODE, keySpec, iv);
			plaintextBytes = cipher.doFinal(ciphertextBytes);
		} catch (Exception exception) {
			logger.warn("decrypt {} exception! {}", ciphertext, exception);
			return null;
		}
		return new String(plaintextBytes, Charsets.UTF_8); // 指定UTF-8字符集
	}

	/**
	 * 
	 * aes加密
	 */
	public static String aesEncrypt(String ciphertext, String aesKey) {
		byte[] ciphertextBytes = ciphertext.getBytes(); // decode加密密文结果
		byte[] aesKeyBytes = Base64.decodeBase64(aesKey.getBytes()); // decode 秘钥
		SecretKeySpec keySpec = new SecretKeySpec(aesKeyBytes, "AES");
		IvParameterSpec iv = new IvParameterSpec(aesKeyBytes, 0, 16); // 初始化向量
		byte[] plaintextBytes;
		try {
			Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding"); // 加密模式为CBC
			cipher.init(Cipher.ENCRYPT_MODE, keySpec, iv);
			plaintextBytes = cipher.doFinal(ciphertextBytes);
		} catch (Exception exception) {
			logger.warn("decrypt {} exception! {}", ciphertext, exception);
			return null;
		}
		return new String(Base64.encodeBase64(plaintextBytes), Charsets.UTF_8); // 指定UTF-8字符集
	}

}

```
