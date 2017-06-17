title: shoppingSite
date: 2015-10-31 14:26:14
tags: project
categories: note
---

# shoppingSite

本文只要记录在开发工程中遇到的问题以及解决方法以及待改进

- 技术

  - struts2
  - spring
  - mybatis


做项目时候，先将各种包，接口等做好
然后依次配置xml,先从mybatis，spring，struts2


<!--more-->

一些还能改进的地方

- 待改进

  - 商品修改时，如果修改了类别，那么新类别数目+1，旧类别数目-1，删除也是
  - 购物车添加商品后，显示慢一步
  - 商品显示分页
  - 当前位置的显示，一直是手机
  - 清空购物车

- 未完成

  - 我的账户，订单
  - 管理员查询管理订单
  - 评价功能


---
## 配置spring的`xml`文件中的`bean`

应该全部配置实例的？！！
```xml

<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
	<!--接口-->
	<property name="mapperInterface" value="com.dy.shoppingSite.dao.UserDao" />
	<property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>

<!--实例-->
<bean id="userService" class="com.dy.shoppingSite.service.impl.UserServiceImpl">
	<property name="userDao" ref="userMapper"></property>
</bean>	
```
---

## spring 与 struts2 联合使用加入`struts2-spring-plugin`

>Exception starting filter struts2 Unable to load configuration.
[unknown location] at org.apache.struts2.dispatcher.Dispatcher.init

报错原因是没有`struts2-spring-plugin`

---

## 得到id随机数串

使用`UUID.randomUUID()`
```java
public static String getID(){
	UUID uuid = UUID.randomUUID();
	return uuid.toString().replace("-", "");
}
```
---

## ajax和jquery需要服务器端返回一个无参数的函数
使用ajax和jquery来实现，需要服务器端返回一个无参数的函数,参数传递使用response
```java
public void isExist() {
	boolean b = userService.isExist(user.getName());
	try {
		PrintWriter writer = ServletActionContext.getResponse().getWriter();
		writer.print(b);
		writer.flush();
		writer.close();
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
}
```
---

## 分页操作

- 建立pageBean的实体类，使用泛型
```
public class PageBean<T> {
	//数据
	private List<T> data;
	//设置每页显示多少条，固定的
	private int pageSize;
	//当前页的记录的标号，mysql中的参数
	private int pageNo;
	//总页数
	private int totalPage;
	//当前页数
	private int page;
	//总的条数，即一共多少条记录
	private int totalNum;
}
```
- 在dao层进行查询得到data的值，比如`List<Goods>`
- 在service层实现pageBean的封装,比如`totalPage`
- 在action中传入所需要的参数,比如`page`

---
## 实现搜索条件保存和翻页的结合
在项目中搜索栏中会有各种搜索条件，搜索完成后，需要分页浏览，
但是进入下一页时，要保证搜索条件仍然显示出来
这是需要用form表单将他们放在一起，并用jquery控制分页及整体form提交

- jquery检查到page发生变化，就向后台提交

page是向后台传送的属性
```
<script type="text/javascript">
	function goPage(p){
		$("#page").val(p);
		$("#form1").submit();
	}
</script>
```

- 使用form把搜索条件和页码一起控制

```
<!-- //为了解决 将搜索内容和page都传到后台-->
<form action="goods_listGoodsByPage" method="post" id="form1"'>
	<!-- 因此添加hidden属性将page和goods一起传到后台,在jquery中goPage实现的 -->
	<!-- name是传到后台的属性名，id是jquery中使用的属性名 -->
	<input type="hidden" name="page" id="page" /> 
	<select class="auto" name="goods.categoryId" id="category">
		<option value="">选择分类</option>
		<c:forEach items="${categories}" var="category">
			<option value="${category.id}">${category.name}</option>
		</c:forEach>
	</select> 商品名： <input class="small" name="goods.name" id="name" type="text"
		value="">
	<button class="btn" type="submit">
		<span class="sel">筛 选</span>
	</button>
</form>
```

- 分页实现


```
<div class='pages_bar'>
	<a href='javascript:goPage(1)' id="first">首页</a>
	<c:forEach begin="1" end="${pageBean.totalPage}" var="p">
		<a href="javascript:goPage('${p}')">${p}</a>
	</c:forEach>
	<a href='javascript:goPage(${pageBean.totalPage})' id="last">尾页</a><span>当前第${pageBean.page}页/共${pageBean.totalPage}页</span>
</div>
```

---
## 先得到旧数据，再更新
在实际中，比如要更新商品，要先得到旧数据，然后对旧数据进行修改，输入到数据库中
这样做的原因是，修改界面不一定所有的属性都要修改，所以直接拿新数据进行修改可能会产生空值
```
public void updateGoods(Goods goods) {
	// TODO Auto-generated method stub
	//这里更新要先将beforeGoods拿来，然后把新的goods中属性付给beforeGoods
	//因为goods中不一定所有属性都进行更新，会产生空值，导致出错
	Goods beforeGoods = goodsDao.getGoodsById(goods.getId());
	beforeGoods.setCategory(goods.getCategory());
	beforeGoods.setCategoryId(goods.getCategoryId());	
	goodsDao.updateGoods(beforeGoods);

}
```

---


## mybatis相关

### 传入多个同类型的参数，使用#{序号}取出
```xml
<select id="getUserByNameAndPwd" resultType="User">
<!-- 传入多个参数可以使用map，通过名字来取出，对于同一类型的多个传入参数，可以使用列号来取出 -->
	select * from shop_user where name = #{0} and password = #{1}
</select>
```

### jdbcType = VARCHAR可以传入空值，且不报错

```xml
<!-- #{avatar,jdbcType = VARCHAR}当传入值为空的时候，使用varchar可以传入空值 -->
<insert id="addUser" parameterType="User">
	insert into shop_user
	values(#{id}, #{name}, #{password}, #{phoneNum},
	#{money},
	#{avatar,jdbcType = VARCHAR}, #{regTime}, #{role} )
</insert>
```
### 得到多个相同类型的数据
```xml
<!-- While this approach is simple, it will not perform well for large data 
	sets or lists. This problem is known as the "N+1 Selects Problem". In a nutshell, 
	the N+1 selects problem is caused. We can Nest Results for Association -->
<select id="getAddress" parameterType="string" resultType="Address">
	select * from shop_address where userid = #{0}
</select>
```
java可以直接用list取得
```java
public List<Address> getAddress(String userid) {
	// TODO Auto-generated method stub
	return addressDao.getAddress(userid);
}
```

### column不一定是数据库中的名字，property是在java中get，set方法的那些属性
`association`查询一个属性，该属性和另一张表关联，从而查出另一站标的所有属性
```
<!--property是在java中get，set方法的那些属性 -->
<resultMap type="Goods" id="GoodsResultMap">
	<id column="id" property="id" />
	<result column="goodsNo" property="goodsNo" />
	<association property="category" javaType="Category">
		<id column="id" property="id" />
		<!-- 因为goods中也有name属性，因此查询出来的结果会重复。 故把category中的name重新命名为cname，这样就不会重复。 
			由此可以看出column属性不一定是数据库中的列名， 而是查询后显示的列名 -->
		<result column="cname" property="name" />
	</association>
</resultMap>
```

### parameterType不是hashmap等时，不需要加所属，反之必加&hashmap注意判空&`bind`实现模糊查询
`parameterType`是`Goods`，是普通的java类型，因此不要`goods.name`,mybatis直接会用Goods中的get/set方法
```
<select id="getGoodses" resultMap="GoodsResultMap" parameterType="Goods">
	<!-- 这里给c.name使用别名cname，因为g中也name列，会重复 -->
	select g.*, c.name cname from shop_goods g, shop_category c where
	g.categoryId = c.id
	<!-- parameterType单一的情况下，就是只有goods或者只有string，(不是hashmap或者list等)， 查询时不需要加所属名字的，比如goods.name -->
	<if test="categoryId != null and categoryId!= '' ">
		and g.categoryId = #{categoryId}
	</if>
	<if test="name != null and name != '' ">
		<!-- 模糊查询，1.要用bind 将查询的东西与页面的显示联系起来 -->
		<bind name="name" value="'%' + name + '%'" />
		<!-- 模糊查询，2.like关键字模糊查询 -->
		and g.name like #{name}
	</if>
</select>
```

- `parameterType`是`hashmap`，含有多种不同类型的值，一定要加上key值，才能取出，比如`goods.categoryId`,
mybatis直接会用hashmap中的get/set得到key值，然后在用key中的get/set方法得到属性值

- hashmap中放多种key时，注意**判断key是否为空**，否则报错`source is null for getProperty(null, "categoryId")`

- `bind`实现模糊查询:[模糊查询](https://mybatis.github.io/mybatis-3/dynamic-sql.html "https://mybatis.github.io/mybatis-3/dynamic-sql.html")

```
<select id="listGoodsByPage" parameterType="hashmap" resultMap="GoodsResultMap">
	select g.*, c.name cname from shop_goods g, shop_category c
	where
	g.categoryId = c.id
	<!--  一定要判断goods是否为空，否则会报错
	source is null for getProperty(null, "categoryId") -->
	<if test="goods != null"> 
		<!-- parameterType为hashmap的情况下，查询时一定要加key的名字，比如goods.name -->
		<if test="goods.categoryId != null and goods.categoryId!= '' ">
			and g.categoryId = #{goods.categoryId}
		</if>
		<!-- 这里要指出key值，因为hashmap中放了多个值，比如goods.name -->
		<if test="goods.name != null and goods.name != '' ">
			<!-- 模糊查询，1.要用bind 将查询的东西与页面的显示联系起来 -->
			<bind name="goods.name" value="'%' + goods.name + '%'" />
			<!-- 模糊查询，2.like关键字模糊查询 -->
			and g.name like #{goods.name}
		</if>
	</if> 
	ORDER BY id LIMIT #{pageNo}, #{pageSize}

</select>
```

---

## struts2 相关

### struts2 文件上传

[struts upload](https://struts.apache.org/docs/file-upload.html "https://struts.apache.org/docs/file-upload.html")

```java
// 上传头像
// struts2对于文件上传已经规定好了属性
// setX(File file),setXFileName(String fileName),setXContentType(String contentType)
private File avatar;
private String avatarFileName;
// 上传用户头像
public String uploadAvatar() {
	String userId = ((User) ActionContext.getContext().getSession()
			.get("user")).getId();
	// 得到文件存放的绝对地址
	String path = ServletActionContext.getServletContext().getRealPath(
			"/userAvatars");
	System.out.println(path);
	if (avatar != null) {
		// 得到才上传文件的后缀名
		String suffix = avatarFileName.substring(avatarFileName
				.lastIndexOf("."));
		System.out.println(userId + suffix);
		// 得到新的文件，与上传文件相同，文件名是id+后缀名
		File saveFile = new File(new File(path), userId + suffix);
		try {
			if (!saveFile.getParentFile().exists()) {
				saveFile.getParentFile().mkdir();
			}
			// 把上传文件拷贝到服务器端/avatar
			FileUtils.copyFile(avatar, saveFile);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		// 存放到数据库中
		userService.upadteAvatar(userId, "userAvatars/" + userId + suffix);
		// 更新session
		((User) ActionContext.getContext().getSession().get("user"))
				.setAvatar("userAvatars/" + userId + suffix);
		return "usercenter";

	}
	return path;

}
```

### struts得到界面的值

jsp中有captcha，要去后台验证是否正确
```
<td><input style="width:85px" type='text' class='normal' name='captcha' />
<label>填写下图所示字符</label></td>
```
在action中可以直接定义该属性，要保持名字一致，并且使用set/get方法
```
private String captcha;
public String getCaptcha() {
	return captcha;
}

public void setCaptcha(String captcha) {
	this.captcha = captcha;
}
```

### strut2 验证码
注意将`BufferedImage`转化为`ByteArrayInputStream`，才能在前端显示出来
`RandomAction.java`后台验证
```
public class RandomAction {

	private ByteArrayInputStream image;   
	
	public String execute(){
		RandomUtils randomUtils = RandomUtils.Instance();
		//验证码的图片
		image = randomUtils.getImage();
		//把验证码放入session中
		ActionContext.getContext().getSession().put("vcode", randomUtils.getString());
		System.out.println("vcode"+randomUtils.getString());
		System.out.println("session"+ActionContext.getContext().getSession().get("vcode")
				.toString());
		return "success";
		
	}

	public ByteArrayInputStream getImage() {
		return image;
	}

	public void setImage(ByteArrayInputStream image) {
		this.image = image;
	}

	
}
```
`RandomUtils.java`产生验证码
```
public class RandomUtils {

	// public static final char[] CHARS = { '2', '3', '4', '5', '6', '7', '8',
	// '9', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'J', 'K', 'L', 'M',
	// 'N', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z' };

	public static final char[] CHARS = { '0', '1', '2', '3', '4', '5', '6',
			'7', '8', '9', };
	private ByteArrayInputStream image;// 图像
	private String str;// 验证码

	private RandomUtils() {
		init();// 初始化属性
	}

	public static RandomUtils Instance() {
		return new RandomUtils();
	}

	/*
	 * 取得验证码图片
	 */
	public ByteArrayInputStream getImage() {
		return this.image;
	}

	/*
	 * 取得图片的验证码
	 */
	public String getString() {
		return this.str;
	}

	private void init() {
		// 在内存中创建图象
		int width = 85, height = 20;
		BufferedImage image = new BufferedImage(width, height,
				BufferedImage.TYPE_INT_RGB);
		// 获取图形上下文
		Graphics g = image.getGraphics();
		// 生成随机类
		Random random = new Random();
		// 设定背景色
		g.setColor(getRandColor(200, 250));
		g.fillRect(0, 0, width, height);
		// 设定字体
		g.setFont(new Font("Times New Roman", Font.PLAIN, 18));
		// 随机产生155条干扰线，使图象中的认证码不易被其它程序探测到
		g.setColor(getRandColor(160, 200));
		for (int i = 0; i < 155; i++) {
			int x = random.nextInt(width);
			int y = random.nextInt(height);
			int xl = random.nextInt(12);
			int yl = random.nextInt(12);
			g.drawLine(x, y, x + xl, y + yl);
		}
		// 取随机产生的认证码(6位数字)
		StringBuffer sRand = new StringBuffer();
		for (int i = 0; i < 6; i++) {
			String rand = String.valueOf(CHARS[random.nextInt(CHARS.length)]);
			sRand.append(rand);

			// 将认证码显示到图象中
			g.setColor(new Color(20 + random.nextInt(110), 20 + random
					.nextInt(110), 20 + random.nextInt(110)));
			// 调用函数出来的颜色相同，可能是因为种子太接近，所以只能直接生成
			g.drawString(rand, 13 * i + 6, 16);
		}
		// 赋值验证码
		this.str = sRand.toString();

		// 图象生效
		g.dispose();
		ByteArrayInputStream input = null;
		ByteArrayOutputStream output = new ByteArrayOutputStream();
		try {
			ImageOutputStream imageOut = ImageIO
					.createImageOutputStream(output);
			ImageIO.write(image, "JPEG", imageOut);
			imageOut.close();
			input = new ByteArrayInputStream(output.toByteArray());
		} catch (Exception e) {
			System.out.println("验证码图片产生出现错误：" + e.toString());
		}

		this.image = input;/* 赋值图像 */
	}

	/*
	 * 给定范围获得随机颜色
	 */
	private Color getRandColor(int fc, int bc) {
		Random random = new Random();
		if (fc > 255)
			fc = 255;
		if (bc > 255)
			bc = 255;
		int r = fc + random.nextInt(bc - fc);
		int g = fc + random.nextInt(bc - fc);
		int b = fc + random.nextInt(bc - fc);
		return new Color(r, g, b);
	}
}
```
前端显示
```
<tr>
	<th valign="middle">验证码：</th>
	<td><input style="width:85px" type='text' class='normal' name='captcha' />
	<label>填写下图所示字符</label></td>
</tr>
```
### 跳转
`redirectAction`直接跳转到action中
```
<action name="address_*" class="addressAction" method="{1}">
	<result name="oprsuc" type="redirectAction">address_list</result>
	<result name="list">/usercenter/address_list.jsp</result>
</action>
```
---

## spring相关

### spring事务
一般在service层可以操作多个dao，因此要设置事务
service层，每个dao是一个对数据库的操作，下面这种2个dao一起添加的操作一定是事务
```
public void addGoods(Goods goods) {
	goods.setId(MyUntil.getID());
	goodsDao.addGoods(goods);
	//这是类别的更新，与商品添加一起完成
	categoryDao.updateGoodsNum(goods.getCategoryId(), 1);
}
```
添加`c3p0`，并在spring的xml文件中配置
不用添加任何其余代码就完成了事务的配置，基于AOP
```
<!-- 添加事务的dataSource -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
	<property name="driverClass" value="com.mysql.jdbc.Driver"></property>
	<property name="jdbcUrl" value="jdbc:mysql://localhost:3306/shoppingsite"></property>
	<property name="user" value="root"></property>
	<property name="password" value="mysql"></property>
</bean>

<bean id="transactionManager"
	class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"></property>
</bean>
<tx:advice id="txAdvice" transaction-manager="transactionManager">
	<tx:attributes>
		<!-- REQUIRED表示不是事务就建立事务 -->
		<tx:method name="*" propagation="REQUIRED" />
		<!--SUPPORTS不是事务就不建立 -->
		<tx:method name="get*" propagation="SUPPORTS" read-only="true" />
	</tx:attributes>
</tx:advice>
<aop:config>
<!-- 包名一定要写清楚，对应好 com.dy.shoppingSite.*.*(..)这个就不能对应，因为shoppingSite.*是包，也就没有方法-->
	<aop:pointcut id="pointcut" expression="execution(* com.dy.shoppingSite.service.impl.*.*(..))" />
	<aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut" />
</aop:config>
```

---

## jsp

### `c:forEach`中的`varStatus`&el表达式计算比较等要放在括号内
- `c:forEach`中的`varStatus`
`varStatus`有很多属性，`index`是序号，从0开始，也有`first`等
more:[varStatus](http://www.opencms-wiki.org/wiki/C:forEach "http://www.opencms-wiki.org/wiki/C:forEach")
- el表达式计算比较等要放在括号内
比如`${string1 eq string 2}`而不是`${string1} eq ${string 2}`
```
<c:set var="totalMoney" value="0"></c:set>
	<c:forEach items="${orderDetails}" var="orderDetail" varStatus="s">
		<tr><td>
			<!-- //传递orderDetail要使用index，直接传是不成功的 -->
			<!-- 下标用s.index表示，http://www.opencms-wiki.org/wiki/C:forEach -->
			<input type="hidden" name="orderDetails[${s.index }].goods.id" value="${orderDetail.goods.id}"/>
			<input type="hidden" name="orderDetails[${s.index }].nums" value="${orderDetail.nums}" />
			<b class="red2">${orderDetail.nums*orderDetail.goods.price2}</b>
		</td></tr>
	<c:set var="totalMoney" value="${totalMoney+orderDetail.nums*orderDetail.goods.price2}"/>
</c:forEach>

```


