### 操作JSON

> JSON和XML一样，都可以作为服务器端和客户端传输数据的方式，而由于JSON没有像XML那样规范的开始结束标签，所以在内容上更加简短，而且读写和传输速度更快且更容易解析，是现如今很流行的数据传输的格式。操作JSON的工具包也有很多，常用的是org.json的工具包，net.sf.json包，还有现在流行的称之为最快的JSON处理工具包：FastJSON 是有阿里巴巴开发的。

#### 使用FastJSON工具包

> FastJSON工具包是专门针对JAVA解析JSON提供的工具支持，稍微分了一些版本，支持java ee端的F功能最全，支持Android版的由于Dalvik虚拟机的原因，一小部分功能不支持，被剔除，但是核心功能都在，而且这样也使针对Android版的FastJson更加轻小。在Android端使用FastJSON时，需要引入fastjson-1.1.43.android.jar 中间的数字是版本号，可以自行选择最新或最好用的版本。引入以后，就可以在Android项目中直接使用了。最常使用的就是JSON对象中的那些静态方法，还有JSONObject和JSONArray对象。

需要注意的是：在Android项目中Android jar包默认会包含org.json的工具包，所以在使用以下对象时，导入的包都必须是阿里巴巴的包：

	import com.alibaba.fastjson.JSON;
	import com.alibaba.fastjson.JSONArray;
	import com.alibaba.fastjson.JSONObject;

##### 解析：JSON String to Object

	public void paseObj()
	{
		String json = "{version:'2.3',url:'http://www.baidu.com',isNew:true,person:{name:'ding',age:25,sex:'man'}}";
	
		// 直接使用JSON中的静态方法parse解析json 字符串
		JSONObject obj = (JSONObject) JSON.parse(json);
		String version = obj.getString("version");
		String url = obj.getString("url");
		boolean isNew = obj.getBooleanValue("isNew");
		System.out.println("version:"+version+"\t url:"+url+"\t isNew:"+isNew);

		// 从解析出的JSONObject对象中再次获取以key="person"的JSONObject对象
		JSONObject person = obj.getJSONObject("person");
		String pName = person.getString("name");
		System.out.println("person name:"+pName);
	}

执行结果：

	version:2.3	 url:http://www.baidu.com	 isNew:true
	person name:ding

JSON对象中有很多parse(....)和toJSONString(...)的重构方法，根据的参数的不同分别完成不同类型的解析和生成操作。

##### 解析：JSON String to javaBean

	/**
	 * 从一个对应javaBean的json串中解析出对象
	 */
	public void paseObject()
	{
		String json = "{id:3,name:'qing',age:25}";
		PersonBean pb = JSON.parseObject(json, PersonBean.class);
		System.out.println(pb.toString());
	}

执行结果：

	PersonBean [id=3, name=qing, age=25]

只要解析的json字符串中的字段和定义的JavaBean中的能够对应上，就可以直接将json串解析为一个JavaBean对象。

##### 解析： JSON String to List<T>

	/**
	 * 从包含对象的List集合中解析出对象
	 */
	public void paseListJson()
	{
		String jsons = "[{\"age\":25,\"id\":2,\"name\":\"qing\"},{\"age\":25,\"id\":2,\"name\":\"ding\"}]";
		System.out.println("json:"+jsons);
		List<PersonBean> list = JSONArray.parseArray(jsons, PersonBean.class);
		for (PersonBean personBean : list) {
			System.out.println(personBean.toString());
		}
	}

执行结果：

	PersonBean [id=2, name=qing, age=25]
	PersonBean [id=2, name=ding, age=25]

##### 生成JSON String： attr to JSON String

	/**
	 * 将一些属性字段放入map集合中，解析成json串
	 */
	public void toJsonStringByObj()
	{
		Map<String, Object> map = new HashMap<String, Object>();
		map.put("number", 2);
		map.put("isUser", true);
		map.put("version", "2.3");
		
		String jsonStr = JSON.toJSONString(map);
		System.out.println("jsonStr:"+jsonStr);
	}

执行结果：

	jsonStr:{"isUser":true,"number":2,"version":"2.3"}

可以看到JSON.toJSONString()方法可以把java基本数据类型都能正确的转换成jsonString，而且toJSONString()方法有很多重构方法，用于接收不同的对象，将其转换成JSON格式的String.


##### 生成JSON String : attr and JavaBean to JSON String

	/**
	 * 将基本数据类型和javaBean对象一起转换成JSON String
	 */
	public void fromMapTest()
	{
		Map<String, Object> map = new HashMap<String, Object>();
		map.put("number", 2);
		map.put("isUser", true);
		
		PersonBean pb = new PersonBean();
		pb.setId(2);
		pb.setAge(25);
		pb.setName("qing");
		map.put("person", pb);
		
		String jsonStr = JSON.toJSONString(map);
		System.out.println("jsonStr:"+jsonStr);
	}

执行结果：

	jsonStr:{"isUser":true,"number":2,"person":{"age":25,"id":2,"name":"qing"}}

##### 生成JSON String: attrs and List<JaveBean> to JSON String

	/**
	 * 将基本数据类型和javeBean集合一起转换成JSON String
	 */
	public void listTest() {
		List<PersonBean> list = new ArrayList<PersonBean>();
		PersonBean pb = new PersonBean();
		pb.setId(2);
		pb.setAge(25);
		pb.setName("qing");

		PersonBean pb2 = new PersonBean();
		pb2.setId(2);
		pb2.setAge(25);
		pb2.setName("ding");

		list.add(pb);
		list.add(pb2);
		
		Map<String,Object> map = new HashMap<String,Object>();
		map.put("version:", "6.0");
		map.put("desc:", "这是最新的版本...");
		map.put("data", list);
		
		String listStr = JSONArray.toJSONString(map);
		System.out.println("listStr:"+listStr);
	}

执行结果：

	listStr:{"data":[{"age":25,"id":2,"name":"qing"},{"age":25,"id":2,"name":"ding"}],"desc:":"这是最新的版本...","version:":"6.0"}


#### 使用org.json的工具包

> 在Android项目中使用org.json工具包时，不需要引入其jar包，Android类库中，已经包含了这个jar包，所以可以直接使用，在导包时，导的是org.json下的类。

##### 解析： JSON String to attrs and javeBean

	public void parseJson() {
		String json = "{version:'2.3',url:'http://www.baidu.com',isNew:true,person:{id:3,name:'ding',age:25}}";

		try {
			JSONObject jsonObj = new JSONObject(json);
			String version = jsonObj.getString("version");
			String url = jsonObj.getString("url");
			System.out.println("version:"+version+"\t url:"+url);
			
			
			//使用JSONObject解析javeBean不能直接得到，只能一一获取其属性值后，在set进去，这种就不方便了
			JSONObject person = jsonObj.getJSONObject("person");			

			int id = person.getInt("id");
			String name = person.getString("name");
			int age = person.getInt("age");
			
			PersonBean pb = new PersonBean();
			pb.setId(id);
			pb.setName(name);
			pb.setAge(age);
			
			System.out.println(pb.toString());

		} catch (JSONException e) {
			e.printStackTrace();
		}
	}

执行结果：

	version:2.3	 url:http://www.baidu.com
	PersonBean [id=3, name=ding, age=25]

从上面的代码可以看出，使用org.json包下的JSONObject解析JSON String时，直接获取基本类型的数据值是方便的，但是要想直接获得JavaBean对象，是办不到的，只能先得到javaBean对象对应的JSONObject，然后在一一去取得javaBean中的属性，在set给javaBean对象，然后才能获取该javaBean，这样相比FastJSON，就有不及之处了。


Andorid中带的org.json功能并不强大，只能做一些简单的json操作。

#### 使用net.sf.json包

要使程序可以运行必须引入JSON-lib包，JSON-lib包同时依赖于以下的JAR包：

	1.commons-lang.jar
	2.commons-beanutils.jar
	3.commons-collections.jar
	4.commons-logging.jar 
	5.ezmorph.jar
	6.json-lib-2.2.2-jdk15.jar

JSON-lib包是一个beans,collections,maps,java arrays 和XML和JSON互相转换的包。在本例中，我们将使用JSONObject类创建JSONObject对象，然后我们打印这些对象的值。为了使用 JSONObject对象，我们要引入"net.sf.json"包。为了给对象添加元素，我们要使用put()方法。


从上面可以看到，使用net.sf.json包需要使用到很多其他关联jar包，这在Android客户端应用中是不好的，引入了太多jar包，会导致app包过大，所以不适合用在Android客户端，可以用在服务端。


示例1：

	public class JSONObjectSample {

    // 创建JSONObject对象
    private static JSONObject createJSONObject() {
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("username", "huangwuyi");
        jsonObject.put("sex", "男");
        jsonObject.put("QQ", "413425430");
        jsonObject.put("Min.score", new Integer(99));
        jsonObject.put("nickname", "梦中心境");
        return jsonObject;
    }

    public static void main(String[] args) {
        JSONObject jsonObject = JSONObjectSample.createJSONObject();//静待方法，直接通过类名+方法调用
        // 输出jsonobject对象
        System.out.println("jsonObject：" + jsonObject);

        // 判读输出对象的类型
        boolean isArray = jsonObject.isArray();
        boolean isEmpty = jsonObject.isEmpty();
        boolean isNullObject = jsonObject.isNullObject();
        System.out.println("是否为数组:" + isArray + "， 是否为空:" + isEmpty
                + "， isNullObject:" + isNullObject);

        // 添加属性，在jsonObject后面追加元素。
        jsonObject.element("address", "福建省厦门市");
        System.out.println("添加属性后的对象：" + jsonObject);

        // 返回一个JSONArray对象
        JSONArray jsonArray = new JSONArray();
        jsonArray.add(0, "this is a jsonArray value");
        jsonArray.add(1, "another jsonArray value");
        jsonObject.element("jsonArray", jsonArray);
        //在jsonObject后面住家一个jsonArray
        JSONArray array = jsonObject.getJSONArray("jsonArray");
        System.out.println(jsonObject);
        
        
        System.out.println("返回一个JSONArray对象：" + array);
        // 添加JSONArray后的值
        // {"username":"huangwuyi","sex":"男","QQ":"413425430","Min.score":99,"nickname":"梦中心境","address":"福建省厦门市","jsonArray":["this is a jsonArray value","another jsonArray value"]}
        System.out.println("结果=" + jsonObject);

        // 根据key返回一个字符串
        String username = jsonObject.getString("username");
        System.out.println("username==>" + username);

        // 把字符转换为 JSONObject
        String temp = jsonObject.toString();
        JSONObject object = JSONObject.fromObject(temp);
        // 转换后根据Key返回值
        System.out.println("qq=" + object.get("QQ"));

    }

}

输出结果：

	jsonObject：{"username":"huangwuyi","sex":"男","QQ":"413425430","Min.score":99,"nickname":"梦中心境"}
	是否为数组:false， 是否为空:false， isNullObject:false
	添加属性后的对象：{"username":"huangwuyi","sex":"男","QQ":"413425430","Min.score":99,"nickname":"梦中心境","address":"福建省厦门市"}
	{"username":"huangwuyi","sex":"男","QQ":"413425430","Min.score":99,"nickname":"梦中心境","address":"福建省厦门市","jsonArray":["this is a jsonArray value","another jsonArray value"]}
	返回一个JSONArray对象：["this is a jsonArray value","another jsonArray value"]
	结果={"username":"huangwuyi","sex":"男","QQ":"413425430","Min.score":99,"nickname":"梦中心境","address":"福建省厦门市","jsonArray":["this is a jsonArray value","another jsonArray value"]}
	username==>huangwuyi
	qq=413425430



关于java bean的处理



创建java对象：

	public class Address {
	 private String road;
	 private String streate;
	 private String provience;
	 private String no;
	 public String getRoad() {
	  return road;
	 }
	 public void setRoad(String road) {
	  this.road = road;
	 }
	 public String getStreate() {
	  return streate;
	 }
	 public void setStreate(String streate) {
	  this.streate = streate;
	 }
	 public String getProvience() {
	  return provience;
	 }
	 public void setProvience(String provience) {
	  this.provience = provience;
	 }
	 public String getNo() {
	  return no;
	 }
	 public void setNo(String no) {
	  this.no = no;
	 }
	}

1.将json对象转化为java对象

	 JSONObject jsonObject = JSONObject.fromObject("{\"no\":\"104\",\"provience\":\"陕西\",\"road\":\"高新路\",\"streate\":\"\"}");
	  Address Address  = (Address) JSONObject.toBean(jsonObject,Address.class);
	  log.info(Address.getNo());
	  log.info(Address.getStreate());
	  log.info(Address.getProvience());
	  log.info(Address.getRoad());

 

2.将java对象转化为json对象

    Address address = new Address();
    address.setNo("104");
    address.setProvience("陕西");
    address.setRoad("高新路");
    address.setStreate("");
    JSONArray json = JSONArray.fromObject(address);
    log.info(json.toString());

 

  将java对象list转化为json对象： 

	  Address address = new Address();
	  address.setNo("104");
	  address.setProvience("陕西");
	  address.setRoad("高新路");
	  address.setStreate("");
	  Address address2 = new Address();
	  address2.setNo("105");
	  address2.setProvience("陕西");
	  address2.setRoad("未央路");
	  address2.setStreate("张办");
	  List list = new ArrayList();
	  list.add(address);
	  list.add(address2);
	  JSONArray json = JSONArray.fromObject(list);
	  log.info(json.toString());

3.JSONArray转化为list

	JSONObject jsonObject = JSONObject.fromObject("{\"no\":\"104\",\"provience\":\"陕西\",\"road\":\"高新路\",\"streate\":\"\"}");
	  JSONArray jsonArray = new JSONArray();
	  jsonArray.add("{\"no\":\"104\",\"provience\":\"陕西\",\"road\":\"高新路\",\"streate\":\"\"}");
	  jsonArray.add("{\"no\":\"104\",\"provience\":\"陕西\",\"road\":\"高新路\",\"streate\":\"123\"}");
	  Object object = JSONArray.toList(jsonArray,Address.class);

