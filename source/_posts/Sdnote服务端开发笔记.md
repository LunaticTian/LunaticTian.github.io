---
title: Sdnote服务端开发笔记
tags:
  - Java
  - Jersey
  - 服务端
  - 笔记
abbrlink: 7c0fc3ab
date: 2018-05-04 19:36:47
---

## 前奏
似乎有大半个月没有啥动静，这段时间一直忙大学生计算机设计大赛，被老师说动参加这项比赛。动心一方面是因为可以有动手的机会，另一方面是对未来阶段有些许帮助。<br>

![](http://img.lunatic.wang/38611917.png)

<!--more-->
回来后就跟同学楠槡提了项目比赛，并且也邀约希望能够一起来做这个项目。没想到楠槡同学欣然答应...我还是挺惊讶的，因为大多数人在听到这种邀约的第一反映都是拒绝，毕竟很多人都有自己的事情，但是大佬还是很爽快的答应让我意想不到。<br>
至于项目做什么，楠槡同学很快给出想法(dalao说他有适合做产品经理)。最开始就跟我讲了这个项目原型，然后基于一个课程所想出来的。主要目的还是想整合我们的日常操作，减轻我们的负担，同时也想让人们回归书写，记住每一刻，也想解决人们对信息临时记录，信息提醒服务的需求。应对当前社会大量碎片化信息堆积导致任务安排不合理的情况。<br>
还有一点就是，我很想尝试在团队中开发产品，一个人没有三头六臂我也想专注的做一件事情。所以在分配的时候我来做后端也就是服务端，楠槡同学做前端。<br>



## 计划
我们计划做全平台的应用，所以理应需要一个接口规范——Restful。通过前端请求api，然后服务端对数据进行处理。至此，我还没有接触过甚至未听说。特别的要用到Jersey的框架。<br>
			
	restful：一种软件架构风格、设计风格，而不是标准，只是提供了一组设计原则和约束条件。它主要用于客户端和服务器交互类的软件。
	基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。
<br>

	Jersey：Jersey RESTful 框架是开源的RESTful框架, 实现了JAX-RS (JSR 311 & JSR 339) 规范。它扩展了JAX-RS 参考实现。
	提供了更多的特性和工具， 可以进一步地简化 RESTful service 和 client 开发。
	尽管相对年轻，它已经是一个产品级的 RESTful service 和 client 框架。与Struts类似，它同样可以和hibernate,spring框架整合。

因为没有经验，所以查了很多资料，不得不说百度作为搜索引擎的确没有Google做的好，搜索了很多资料终于在Google上找到了。
[用Jersey构建RESTful服务](https://www.kancloud.cn/wizardforcel/rest-service-with-jersey/151186)，作者是：[Way Lau](https://github.com/waylau)

## 开始

### 创建项目，引入依赖
首先创建web server项目，然后下载Jersey的依赖[https://jersey.java.net/download.html](https://jersey.java.net/download.html),项目引入jar包，接着引入mysql的jdbc以及json的依赖。<br>
创建一个包，com.wut.connection。这个包中的内容主要写api接口的。
然后在web.xml中加入映射。

	 <servlet>
	    <servlet-name>Way REST Service</servlet-name> <servlet-class>org.glassfish.jersey.servlet.ServletContainer</servlet-class>
	    <init-param>
	    <param-name>jersey.config.server.provider.packages</param-name>
	    <param-value>com.wut.connection</param-value> </init-param>
	    <load-on-startup>1</load-on-startup>
	</servlet>
	
	<servlet-mapping>
	    <servlet-name>Way REST Service</servlet-name>
	    <url-pattern>/api/*</url-pattern>
	</servlet-mapping>

### 功能创建

分析功能可知，Sdnote需要的几大接口：

* 测试API
* 注册
* 登陆
* 修改密码
* 标签笔记
	* 增加
	* 修改
	* 删除
	* 获取所有的标签笔记
* 日程提醒
	* 增加
	* 修改
	* 删除
	* 获取所有的日程提醒
* 图像识别


#### 测试

在com.wut.connection中创建了test.java：


	@Path("/test")
	public class test {
		@GET
		@Produces(MediaType.APPLICATION_JSON)
		public String testString() {
			return "Hello World";
		}
	}
@Path是定义资源的访问路径，也就是访问这个api需要只需要输入

		http://127.0.0.1:8080/Sdnote/api/test

Sdnote是我们的项目名称，api在web.xml中映射到了connection包，而test指向的就是这个test方法。<br>
方法内的@GET需要访问的时候使用get请求，@Produces(MediaType.APPLICATION_JSON)表示此方法向请求返回类型为一个JSON。当然，我们这里返回的是一个字符串。<br>
之所以写这个测试api是为了当服务器开启的时候，测试项目是不是跑起来了，访问路径之类的是否有所错误。


![](http://img.lunatic.wang/%E6%8D%95%E8%8E%B7.PNG)

[dstantside.com]()是我绑定服务器上的域名。

#### 注册

在com.wut.connection中创建了test.java：
##### json
首先需要明确一个定义，就是get的发送是不带body的，也就是请求不带有数据，所以get在保密性上面无法完成登陆的操作。我们需要传递用户名以及密码，这些数据都具有高度的保密性，所以这里要用到post。
接着，我希望在发送请求的时候对方发送的是一个json，因为json是键值对的形式，而且方便解析和传递。
##### api鉴权
OK，当我们收到请求，对方发送了一个JSON里面含有用户名以及密码。我们把数据拿到数据库中进行匹配，然后需要返回给用户信息正确或者错误，并且，因为我们是RESTfulAPI形式，所有的操作都是通过API请求与服务器联系，怎么判别用户是这个用户，并且他登陆成功他的操作都是拥有正常权限范围内的？所以这里用到了API鉴权，一个权限，一个加密码也就是token。当用户登陆验证成功之后，我们返回这个用户的ID以及整个token，这个token跟用户ID用户名都是绑定的，token就可以唯一确定用户，而这个token用户是不知道的。<br>
所以在用户创建的时候，我们就生成一个token与用户绑定。

@Consumes的意思是只有符合这个参数设置的请求才能访问到这个资源。这里我写的是MediaType.APPLICATION_JSON。

	@POST
	@Path("/register")
	@Produces(MediaType.APPLICATION_JSON)
	@Consumes(MediaType.APPLICATION_JSON)
	public ReturnUser main(Object name) {
		// 将传过来json转化成Object，然后通过强制转化成JSONObject
		JSONObject RequestUser = JSONObject.fromObject(name);
		// 读取JSON的数据
		// 防止 传过来的json信息不完全
		String userName = "";
		String password = "";
		String Email = "";
		String phone = "";
		int age = 0;
		int sex = 0;
		try {
			userName = (String) RequestUser.get("userName");
			password = (String) RequestUser.get("password");
			Email = (String) RequestUser.get("Email");
			phone = (String) RequestUser.get("phone");
			age = RequestUser.getInt("age");
			sex = RequestUser.getInt("sex");
		} catch (Exception e) {
			return new ReturnUser("body error", "body error", "body error", "body error");
		}

		// new一个user对象
		User useruser = new User(userName, password, Email, phone, age, sex);
		// 传入一个user对象给数据库校验，返回校验成果。
		ReturnUser name1 = date.validateregister(useruser);
		System.out.println(name1);
		return name1;
	}

##### JSONObject异常

在方法参数中，虽然定义规则告诉请求我们需要一个json，但是仍然不能接收一个JSONObject，也就是 <b>public ReturnUser main(Object name)</b>。当我们参数是JSONObject，会报JSONObject异常，这也是这里我要说的，即使请求发送的是一个json，但是我们不能用JSONObject来接收。显然，json是肯定发送过来，我们也接收到了。不能用JSONObject，那么我就用Object来接收，因为Object是所有对象的父类，不管请求是什么，它都可以接收。然后再将这个Object转化为JSONObject，这样我们就能正常的读取数据。

##### json缺失key异常

至于这里我们还需要做一次异常的捕捉和判断，当请求的参数中缺少Email或者phone，那么就需要告诉请求方，这个地方你没有写phone或者Email。这在后面的功能中都要需要说明，保证程序的健壮性。当请求的json中出现异常，那么我们返回这个ReturnUser的对象，Jersey会自动的把这个对象转化成json。


	ReturnUser：
	
	public class ReturnUser {
		/*
		 * 注册后返回给用户的信息 State 注册是否成功 成功为200 不成功为400 UserName 为 True 则没有重复
		 * 若为false则为有重复 Email 同上 phone 同上
		 * 
		 */
		private String State;
		private String UserName;
		private String Email;
		private String phone;
	
		public ReturnUser(String State, String UserName, String Email, String phone) {
			this.State = State;
			this.UserName = UserName;
			this.Email = Email;
			this.phone = phone;
		}
	
		public String getState() {
			return State;
		}
	
		public void setState(String state) {
			State = state;
		}
	
		public String getUserName() {
			return UserName;
		}
	
		public void setUserName(String userName) {
			UserName = userName;
		}
	
		public String getEmail() {
			return Email;
		}
	
		public void setEmail(String email) {
			Email = email;
		}
	
		public String getPhone() {
			return phone;
		}
	
		public void setPhone(String phone) {
			this.phone = phone;
		}
	
	}

如果用户登陆成功，我们就想用户返回注册正确的状态码，告诉客户端注册成功，接下来就可以做登陆或者其他操作。

数据库
	
	/*
	 * 用户注册信息验证
	 *
	 * 先做list判断，在做数据库判断
	 *
	 */
	public ReturnUser validateregister(User user) {
		String username = user.getUser();
		String password = user.getPassword();
		String Email = user.getEmail();
		String phone = user.getPhone();
		int age = user.getAge();
		int sex = user.getGender();
		boolean busername = true;
		boolean bphone = true;
		boolean bEmail = true;
		// 用户名，邮箱，电话号码是是否有重复
		for (int i = 0; i < userValidate.size(); i++) {
			if (userValidate.get(i).get("SU_phone").equals(phone)) {
				bphone = false;
			}
			if (userValidate.get(i).get("SU_Email").equals(Email)) {
				bEmail = false;
			}
			if (userValidate.get(i).get("SU_userName").equals(username)) {
				busername = false;
			}
		}

		if (!busername || !bEmail || !bphone) {
			if(!busername){
				return new ReturnUser("401", String.valueOf(busername), String.valueOf(bphone), String.valueOf(bEmail));
			}
			if(!bEmail){
				return new ReturnUser("402", String.valueOf(busername), String.valueOf(bphone), String.valueOf(bEmail));
			}
			if(!bphone){
				return new ReturnUser("403", String.valueOf(busername), String.valueOf(bphone), String.valueOf(bEmail));
			}
			
		}
		// 生成UID
		String uid = UUID.randomUUID().toString().trim().replaceAll("-", "");

		String sql = "insert into Sdnote_User(SU_userName,SU_password,SU_Token,SU_Email,SU_phone,SU_age,SU_sex) "
				+ "values ('" + username + "','" + password + "','" + uid + "','" + Email + "','" + phone + "','" + age
				+ "','" + sex + "')";

		try {
			if (busername && bEmail) {
				stst.executeUpdate(sql);
				init();
			}
		} catch (SQLException e) {
			// 未知错误
			System.out.println(e.toString());

			return new ReturnUser("400", "error", "error", "error");
		}

		return new ReturnUser("200", String.valueOf(busername), String.valueOf(bphone), String.valueOf(bEmail));
	}

接收到用户发送的json之后，我们将信息提取出来交给数据库逻辑进行判断。

##### 登陆注册信息验证以及构造验证list

在这里我发现如果每当一个用户注册的时候，服务器都要进行一次查询遍历user表，判断用户名邮箱手机号是否被注册过，没有被注册就进行插入的操作。当用户量过于大的时候频繁访问数据库，这对服务器与数据库都是很有压力，数据库的连接资源宝贵，所以这里我构建了一个list，list中的每一个节点都是一个map对象，当用户发送请求首先在list进行验证，当确认没有重复的时候，在进行插入的操作，极大的节省了服务器的资源。

	public void init() throws SQLException {
		
		userValidate = null;
		
		userValidate = new ArrayList<Map<String, String>>();

		String sql = "select  SU_userID,SU_userName,SU_password,SU_Token,SU_Email,SU_phone from Sdnote_User";
		ResultSet re = stst.executeQuery(sql);
		while (re.next()) {
			Map<String, String> map = new HashMap<String, String>();
			map.put("SU_userID", re.getString("SU_userID"));
			map.put("SU_userName", re.getString("SU_userName"));
			map.put("SU_Token", re.getString("SU_Token"));
			map.put("SU_Email", re.getString("SU_Email"));
			map.put("SU_phone", re.getString("SU_phone"));
			map.put("SU_password", re.getString("SU_password"));
			userValidate.add(map);
		}

	}

##### postman

当data对象被new出来后，我们就初始化这个list，根据对象特性可知将init()写入构造方法中即可。

因为普通的浏览器是不能做post请求的，所以我下载一个postman来构造一个post请求验证这个注册是否是正确的。

![](http://img.lunatic.wang/%E6%8D%95%E8%8E%B72.PNG)


服务器返回的结果：

![](http://img.lunatic.wang/%E6%8D%95%E8%8E%B71.PNG)



#### 登陆

在做登陆api的时候，利用我之前创建的list进行验证，当验证成功返回用户的id以及token。只有拥有token，客户端才能做这相关的操作。

为了减轻前端的压力，我们在后端也可以验证用户的输入：

	if(userName.equals("")||userName==null){
			return new LoginUser("402", "用户名为空", "", "", "");
				}

当用户输入不正确的时候，我们返回一个LoginUser给客户端状态码是402，错误信息是用户名为空。


#### 修改找回密码

在修改找回密码的api中，我统一了接口。依靠choose这个值来判断客户端请求是修改密码还是找回密码。当是1，我只获取用户名老密码新密码，至于这里的json判断只会在get中才会异常，只要在相对应的选择中发送正确的json就不会返回异常。


	@POST
	@Path("/backPw")
	@Produces("application/json")
	@Consumes(MediaType.APPLICATION_JSON)
	public Map<String, String> backPssword(Object name) {
		JSONObject RequestUser = JSONObject.fromObject(name);
		// 选择找回还是修改密码,返回值是
		// 1：修改密码
		// 2：找回密码
		try {
			int choose = RequestUser.getInt("choose");
			if (choose == 1) { // 修改密码
				String userName = RequestUser.getString("userName");
				String Oldpassword = RequestUser.getString("OldPassword");
				String newpassword = RequestUser.getString("NewPassword");
				Map<String, String> return1 = date.modifyPassword(userName, Oldpassword, newpassword);
				return return1;
			}
			if (choose == 2) {// 找回密码
				String userName = RequestUser.getString("userName");
				String newpassword = RequestUser.getString("NewPassword");
				String phone = RequestUser.getString("phone");
				Map<String, String> return1 = date.backPassword(userName, newpassword, phone);
				return return1;
			}
		} catch (Exception e) {
			Map<String, String> a1 = new HashMap<String, String>();
			
			a1.put("state", "500");
			a1.put("information", "body error");
			return a1;
		}
		Map<String, String> a1 = null;
		a1.put("state", "400");
		a1.put("information", "choose error");
		return a1;
	}


<br>
#### 笔记以及日程

笔记和日程的api方法中都有四个功能：

* 增加笔记或日程
* 更新修改笔记或日程
* 删除笔记或日程
* 获取所有的笔记或日程

在增加笔记和日程中，获取用户所发送过来的字段，验证token。当token验证成功之后我们在将笔记的内容加入到数据库中，然后基于客户端的信息更改日程或者笔记的状态。

在删除笔记中，我们不仅需要用户传递过来的token，需要修改删除的笔记或者日程的ID，我们在数据库中对每一个日程或者笔记都加入了ID，当ID正确才会进行删除该日程火车笔记。

##### 伪删除vd

当然，现在的氛围是大家都不做真删除了，都做的伪删除。毕竟现在数据的重要性不言而喻，所以这里我们也做伪删除，在笔记和日程的数据库表中加入一个删除的状态码，当是0的时候，为未删除，当为1的时候为删除状态。事实是，所有数据都未删除。

这里验证的VD字段。


	// 用户删除标签
	public StateInformation DeleteNote(JSONObject note) {
		JSONObject date;
		String noteID = note.getString("noteID");
		String Token = note.getString("Token");
		String userid = note.getString("UserID");
		// 校验 ID 以及token
		boolean T = false;
		for (int i = 0; i < userValidate.size(); i++) {
			if (userValidate.get(i).get("SU_userID").equals(userid)
					&& userValidate.get(i).get("SU_Token").equals(Token)) {
				T = true;
			}
		}
		if (T) {
			String sql = "UPDATE Sdnote_Note SET SN_vd = '1'   WHERE SN_noteID = '" + noteID + "'";

			try {
				// 成功
				int i = stst.executeUpdate(sql);
				if (i == 0) {
					return new StateInformation("400", "noteID error");
				}
				return new StateInformation("200", "success");
			} catch (SQLException e) {
				// 数据库失败
				e.printStackTrace();
				return new StateInformation("401", "error");
			}
		} else {
			// 验证失败
			return new StateInformation("400", "UserID or Token error");
		}

	}



获取所有的笔记或日程，在最开始的思路中我想直接传递一个list，前面所讲，Jersey会将这些转化成json，但是如果验证错误我无法将错误码和错误信息或者成功的状态码发送过去，所以这里用map嵌套list来完成。<br>
map是data字段的值是list，这样就解决了需要返回信息的问题。


	public Map<String, Object> getAllNote(JSONObject note) {
		Map<String, Object> a1 = new HashMap<String, Object>();
		String userid = note.getString("UserID");
		String Token = note.getString("Token");

		// 校验 ID 以及token
		boolean T = false;
		for (int i = 0; i < userValidate.size(); i++) {
			if (userValidate.get(i).get("SU_userID").equals(userid)
					&& userValidate.get(i).get("SU_Token").equals(Token)) {
				T = true;
			}
		}
		if (T) {
			String sql = "select * from Sdnote_Note where SN_userID = '" + userid + "' and SN_vd = " + 0 + "";
			System.out.println(sql);
			ArrayList<returnNote> a = new ArrayList<returnNote>();

			try {
				// 成功

				ResultSet resultSetes = stst.executeQuery(sql);
				while (resultSetes.next()) {
					a.add(new returnNote(resultSetes.getString("SN_noteID"), resultSetes.getString("SN_title"),
							resultSetes.getString("SN_time"), resultSetes.getString("SN_content"),
							resultSetes.getString("SN_tag")));
				}

				a1.put("data", a);
				a1.put("information", "success");
				a1.put("State", "200");

				return a1;
			} catch (SQLException e) {
				// 数据库失败
				e.printStackTrace();

				a1.put("information", "error");
				a1.put("State", "400");

				return a1;
			}
		} else {
			// 验证失败

			a1.put("information", "userid or Token error");
			a1.put("State", "401");

			return a1;
		}
	}


返回的结果：

![](http://img.lunatic.wang/123.PNG)


#### OCR文字识别


在最开始的时候使用java的httpclient来接入腾讯云的api一直报错，久久不能解决。然后转向用pyhton来写这一段逻辑，pyhton是成功的。pyhton的成功让我陷入了思考。

腾讯云的api规范需要head和body，head我用直接来传入，body用map传入，这里python的body是一个字典，而我在java中用map一直报错。而字典与json很想，所以我回头试着用json来传入body中,请求终于成功。


 	public String api(String url) throws Exception {
	
		CloseableHttpClient httpClient = HttpClients.createDefault();

		String entityStr = null;
		CloseableHttpResponse response = null;

		try {

			// 创建POST请求对象
			HttpPost httpPost = new HttpPost("http://recognition.image.myqcloud.com/ocr/handwriting");

			// 使用URL实体转换工具
			// 需要传递一个 json
			JSONObject param = new JSONObject();
			param.put("appid", "1252209381");
			param.put("bucket", "");
			param.put("url", url);
			StringEntity se = new StringEntity(param.toString());

			httpPost.setEntity(se);

			/*
			 * 添加请求头信息
			 */
			httpPost.addHeader("host", "recognition.image.myqcloud.com");
			httpPost.addHeader("Content-Type", "application/json");
			httpPost.addHeader("authorization", "gekuLE8u+p+J7jHadlYhyJhgCyNhPTEyNTIyMDkzODEmYj1maWxlJms9QUtJRGM4VGNhT"
					+ "nZVbGJMdUlZeVVDaTNDU2drZEtxUDJmME5kJmU9MTUyNjk2NTA1OSZ0PTE1MjQzNzMwNTkmcj0xODE1MDkzOTI2JnU9MCZmPQ==");
			// 执行请求
			response = httpClient.execute(httpPost);
			// 获得响应的实体对象
			HttpEntity entity = response.getEntity();
			// 使用Apache提供的工具类进行转换成字符串
			entityStr = EntityUtils.toString(entity, "UTF-8");

			// System.out.println(Arrays.toString(response.getAllHeaders()));

		} catch (ClientProtocolException e) {
			System.err.println("Http协议出现问题");
			e.printStackTrace();
		} catch (IOException e) {
			System.err.println("IO异常");
			e.printStackTrace();
		} finally {
			// 释放连接
			if (null != response) {
				try {
					response.close();
					httpClient.close();
				} catch (IOException e) {
					System.err.println("释放连接出错");
					e.printStackTrace();
				}
			}
		}

		// 打印响应内容
		System.out.println(entityStr);
		JSONObject result = JSONObject.fromObject(entityStr);
		JSONObject data = result.getJSONObject("data");
		JSONArray jsonArray = data.getJSONArray("items");
		String str = null;
		for (int i = 0; i < jsonArray.length(); i++) {
			JSONObject a = jsonArray.getJSONObject(i);
			str += a.get("itemstring").toString() + "\n";
			System.out.println(a.get("itemstring"));
		}
		if (str == null) {
			return "";
		}
		return str.replace("null", "");

	}




示例图片：
![](http://img.lunatic.wang/29737545_1658406064255298_3958289310792286208_n.jpg)

返回的orc识别的结果：

![](http://img.lunatic.wang/asdasd.PNG)



### 完成

至此基本的工作以及完成，整个api编写相对来说都是功能细化明确的，而且分层清晰，应该是目前的项目中完成度最高的一项。


## 后记

项目的api文档：[Sdnote API文档](https://github.com/Sdnote/Sdnote.Server/tree/master/DOC)<br>
项目的源代码:[Sdnote.Server](https://github.com/Sdnote/Sdnote.Server)




还是不得不说那句话：
		
		我们都是站在巨人的肩膀上的。
	

