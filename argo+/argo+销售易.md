定时任务

/**
 * Description: 更新销售易销售线索信息
 */
@Component
public class SynXsyLeadTask {

	@Resource
	private XsyHandle xsyHandle;

	/**
	 * 查询出最近一小时销售易有改动的数据，同步到方舟
	 */
	@Scheduled(cron = "0 0 * * * ?")
	public void synSxyToArk() {
		Date endDate = new Date();

		// 每一个小时就同步一次数据，所以开始时间是前一个小时，结束时间是现在
		Calendar instance = Calendar.getInstance();
		instance.setTime(endDate);
		instance.add(Calendar.HOUR_OF_DAY, -1);
		Date startDate = instance.getTime();

		this.synSxyInfoToUserExtend(startDate, endDate);
	}

	private void synSxyInfoToUserExtend(Date startDate, Date endDate) {
		// 查询最近一小时销售易有更新的数据
		JSONArray xsyList = generateExtendRecords(startDate, endDate, 0);

		// 获取销售易线索ID与方舟ID对应关系
		Map<String, String> idMap = getXsyIdAndUserIdMap();

		// 上报SDK
		idMap.keySet().stream().forEach(key -> {
			for (int i = 0; i < xsyList.size(); i++) {
				Map<String, Object> properties = (Map<String, Object>) xsyList.get(i);
				if (idMap.get(key).equals(properties.get("id").toString())) {
					LogSDK.getInstance().profileSet(key, true, properties);
				}
			}
		});
	}

	/**
	 * 分页查询-获取最近一小时销售易有更新的数据
	 */
	private JSONArray generateExtendRecords(Date startDate, Date endDate, int limit) {
		// 获取销售易数据
		JSONObject jsonObject = this.getXsyData(startDate, endDate, limit);
		Integer totalSize = jsonObject.getInteger("totalSize");
		JSONArray jsonArray = jsonObject.getJSONArray("records");
		if (totalSize > 300) {
			int count = totalSize / 300 + 1;
			for (int i = 0; i < count; i++) {
				limit = (i + 1) * 300 - 1;
				// 获取销售易数据
				jsonObject = this.getXsyData(startDate, endDate, limit);
				jsonArray.addAll(jsonObject.getJSONArray("records"));
			}
		}
		return jsonArray;
	}

	/**
	 * 查询销售易
	 */
	private JSONObject getXsyData(Date startDate, Date endDate, int limit) {
		StringBuffer sql = new StringBuffer();
		sql.append("select ");
		sql.append(" id,name,companyName,ownerId, ");
		sql.append(" status,post,depart,recentActivityCreatedBy, ");
		sql.append(" state,email,mobile,createdAt,updatedAt ");
		sql.append(" from lead ");
		sql.append(" where  updatedAt >= " + startDate.getTime());
		sql.append(" and updatedAt <= " + endDate.getTime());
		sql.append(" order by updatedAt limit " + limit + ",300 ");
		JSONObject jsonObject = xsyHandle.queryDataBySql(sql.toString());
		return jsonObject;
	}

	/**
	 * 获取销售易ID与方舟ID关系
	 */
	private Map<String, String> getXsyIdAndUserIdMap() {
		Map<String, String> map = new HashMap<>();
		String userId = "1";
		String xsyId = "1";
		map.put(userId, xsyId);

		userId = "2";
		xsyId = "2";
		map.put(userId, xsyId);

		userId = "3";
		xsyId = "3";
		map.put(userId, xsyId);
		return map;
	}
}
方舟SDK工具类
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
销售易工具类

@Slf4j
@Service
public class XsyHandle {

	private static final int TOKEN_EXP_TIME = 2 * 60 * 60;
	private static final String ACCESS_TOKEN_URL = "/oauth2/token";
	private static final String QUERY_DATA = "/data/v1/query";
	private static final String TOKEN_KEY = "XSY_ACCESS_TOKEN";

	@Autowired
	private XsyConfig xsyConfig;

	private LoadingCache<String, String> tokenCache = CacheBuilder.newBuilder()
	        .expireAfterWrite(TOKEN_EXP_TIME, TimeUnit.SECONDS).build(new CacheLoader<String, String>() {
		        @Override
		        public String load(String key) throws Exception {
			        return requestAccessToken();
		        }
	        });

	/**
	 * 获取URL
	 *
	 * @param method
	 *            方法名
	 * @return
	 */
	private String getXsyUrl(String method) {
		return xsyConfig.getDomain() + method;
	}

	public String getAccessToken() {
		try {
			String token = tokenCache.get(TOKEN_KEY);
			log.info("xsytoken:{}", token);
			return token;
		} catch (Exception e) {
			log.warn("AccessToken为空:{}", e);
			throw new ApiException("AccessToken为空" + e.getMessage());
		}
	}

	public String requestAccessToken() {
		String tokenURL = getXsyUrl(ACCESS_TOKEN_URL);
		try {
			RequestBuilder requestBuilder = RequestBuilder.post(tokenURL);
			requestBuilder.setHeader(HttpClientProxy.FORM_HEADER);
			requestBuilder.addParameter("grant_type", "password");
			requestBuilder.addParameter("client_id", xsyConfig.getClientId());
			requestBuilder.addParameter("client_secret", xsyConfig.getClientSecret());
			requestBuilder.addParameter("redirect_uri", xsyConfig.getRedirectUri());
			requestBuilder.addParameter("username", xsyConfig.getUserName());
			requestBuilder.addParameter("password", xsyConfig.getPassword());
			ThreeTuple<Integer, String, String> postResult =
			        HttpClientProxyManager.getHttpClientProxy().executeStringResult(requestBuilder);
			String result = postResult.getThird();
			JSONObject json = JSONObject.parseObject(result);
			XsyAccessToken token = new XsyAccessToken();
			token.setAccessToken(json.getString("access_token"));
			token.setTokenType(json.getString("token_type"));
			return token.getAccessToken();
		} catch (Exception e) {
			log.error("销售易接口token调用异常,url=" + tokenURL, e);
			return null;
		}
	}


	public JSONObject queryDataBySql(String sql) {
		String token = getAccessToken();
		if (token == null) {
			log.warn("销售易SQL查询失败,AccessToken为空！");
			throw new ApiException("销售易SQL查询失败,AccessToken为空！");
		}
		String url = getXsyUrl(QUERY_DATA);
		RequestBuilder requestBuilder = RequestBuilder.post(url);
		requestBuilder.setHeader("Authorization", "Bearer " + token);
		requestBuilder.setHeader(HttpClientProxy.FORM_HEADER);
		requestBuilder.addParameter("q", sql);
		ThreeTuple<Integer, String, String> postResult =
		        HttpClientProxyManager.getHttpClientProxy().executeJsonResult(requestBuilder);

		String result = postResult.getThird();
		log.info("销售易SQL查询完成。url=" + url + ",param=" + sql + ",result=" + result);
		JSONObject json = JSONObject.parseObject(result);
		if (null != json.getInteger("error_code")) {
			log.warn("销售易SQL查询失败,错误信息为：" + result);
			throw new ApiException("销售易SQL查询失败,错误信息为：" + result);
		}
		return json;
	}
}
