# XUtil工具包使用
## xUtils简介
* xUtils 包含了很多实用的android工具。
* xUtils 支持大文件上传，更全面的http请求协议支持(10种谓词)，拥有更加灵活的ORM，更多的事件注解支持且不受混淆影响...
* xUitls 最低兼容android 2.2 (api level 8)

## 目前xUtils主要有四大模块：

* DbUtils模块：
  > * android中的orm框架，一行代码就可以进行增删改查；
  > * 支持事务，默认关闭；
  > * 可通过注解自定义表名，列名，外键，唯一性约束，NOT NULL约束，CHECK约束等（需要混淆的时候请注解表名和列名）；
  > * 支持绑定外键，保存实体时外键关联实体自动保存或更新；
  > * 自动加载外键关联实体，支持延时加载；
  > * 支持链式表达查询，更直观的查询语义，参考下面的介绍或sample中的例子。

* ViewUtils模块：
  > * android中的ioc框架，完全注解方式就可以进行UI，资源和事件绑定；
  > * 新的事件绑定方式，使用混淆工具混淆后仍可正常工作；
  > * 目前支持常用的20种事件绑定，参见ViewCommonEventListener类和包com.lidroid.xutils.view.annotation.event。

* HttpUtils模块：
  > * 支持同步，异步方式的请求；
  > * 支持大文件上传，上传大文件不会oom；
  > * 支持GET，POST，PUT，MOVE，COPY，DELETE，HEAD，OPTIONS，TRACE，CONNECT请求；
  > * 下载支持301/302重定向，支持设置是否根据Content-Disposition重命名下载的文件；
  > * 返回文本内容的请求(默认只启用了GET请求)支持缓存，可设置默认过期时间和针对当前请求的过期时间。

* BitmapUtils模块：
  > * 加载bitmap的时候无需考虑bitmap加载过程中出现的oom和android容器快速滑动时候出现的图片错位等现象；
  > * 支持加载网络图片和本地图片；
  > * 内存管理使用lru算法，更好的管理bitmap内存；
  > * 可配置线程加载线程数量，缓存大小，缓存路径，加载显示动画等...


----
## 使用xUtils快速开发框架需要有以下权限：

	```xml
	<uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
	```
## DbUtils使用方法：
* 构建ORM实体映射

> 可以向使用Hibernate一样配置表和实体对象的映射关系，不过不是通过.xml配置文件，而是使用工具包中提供的注解进行标识。常用的注解如下：
> 
* @Table(name="teacher") ： 标识该实体类要和DB进行映射
* @Id ： 主键注解
* @Column ： 注解一个普通列，这个注解中有两个参数：column和defaultValue
	* column ： 定义实体属性在数据库中的列名，可以指定也可以不指定，不指定是默认使用属性名
	* defaultValue ： 该属性或者该列在保存时候的默认值
* @Finder ： 一对多的注解，包含两个参数：valueColumn和targetColumn，一般和FinderLazyLoader类搭配使用，在获取一这一端后，使多的一端延迟加载。
	* valueColumn ： 与多的那端实体对象的id字段对应
	* targetColumn ： 与多的那端实体对象中定义的当前对象的外键字段对应
* @Foreign ： 外键注解，包含column和foreign两个属性
	* column ： 指定外键保存在表中的字段名
	* foreign ： 外键所对应对象的实体的主键

以下通过一对多的映射关系来描述上面注解的使用：

	//Student实体的映射配置
	@Table(name="student")
	public class Student {

	@Id
	private int _id;
	
	@Column(column="name")
	private String name;
	
	@Column(column="age")
	private int age;
	
	@Column
	private String address;
	
	@Column
	private Date brithday;
	
	@Foreign(column="teacherId",foreign="_id")
	private Teacher teacher;

	//Teacher实体的映射配置
	@Table(name="teacher")
	public class Teacher {

	@Id
	private int _id;
	@Column(column="teacherName",defaultValue="111")
	private String teacherName;
	@Column
	private String course;
	
	@Finder(valueColumn = "_id", targetColumn = "teacherId")
	public FinderLazyLoader<Student> students;

* 创建和更新数据库
	> DbUtils db = DbUtils.create(getContext(), "student.db");
	使用create方法如果只传入上下文和数据库名称这两个参数，则只会创建出一个数据库，数据库中并无任何表结构，只有在保存某个实体对象时，才会出现实体的表结构。

	> 更新数据库时也需要使用create方法：
	DbUtils db = DbUtils.create(getContext(), "student.db", 2, dbUpgradeListener);需要传递一个新的版本号，以及一个升级数据库的监听器类DbUpgradeListener，这个类中有一个onUpgrade方法用来升级数据库。

	具体代码如下：

		//创建一个数据库升级监听器对象
		DbUpgradeListener dbUpgradeListener  = new DbUpgradeListener() {
			
			//在此方法中进行数据库结构的升级更新
			@Override
			public void onUpgrade(DbUtils dbUtil, int oldVersion, int newVersion) {
				String updateSql = "alter table student add column 'birthday' date";
				try {
					dbUtil.execNonQuery(updateSql);
				} catch (DbException e) {
					e.printStackTrace();
				}
			}
		};
		//升级数据库时，需要传入新的版本
		DbUtils db = DbUtils.create(getContext(), "student.db", 2, dbUpgradeListener);

* 进行CRUD操作
	* 增加实体对象记录
	
	代码如下：

		//创建出DbUtils对象，并使用cerate来初始化数据库，
		//create方法调用后并不会立即创建实体对象的表结构，只有在保存实体以后，其对应的表结构才会创建
		DbUtils db = DbUtils.create(getContext(), "student.db");		
		for (int i = 0; i < 10; i++) {
			Student s = new Student();
			s.setName("ding"+i);
			s.setAddress("郑州");
			s.setAge(24+i);
			s.setBrithday(new Date());
			
			try {
				//调用save是一样的结果
				//db.save(s);
				db.saveBindingId(s);
			} catch (DbException e) {
				e.printStackTrace();
			}
		}

	* 如果存在一对多或者多对一的映射关系，可以在创建两个对象以后，保存一个对象，另一个对象也会级联保存

	具体代码如下：

		DbUtils db = DbUtils.create(getContext(), "student2teacher.db");

		db.configAllowTransaction(true);
		db.configDebug(true);
		
		Teacher teacher = new Teacher();
		teacher.setTeacherName("丁老师333");
		teacher.setCourse("Android开发");
		
		for (int i = 0; i < 10; i++) {
			Student s = new Student();
			s.setName("ding"+i);
			s.setAddress("郑州");
			s.setAge(24);
			s.setBrithday(new Date());
			//设置Student对象对应的Teacher
			s.setTeacher(teacher);
			
			try {
				//执行保存后，会将Student和Teacher对象都保存起来
				db.saveBindingId(s);
			} catch (DbException e) {
				e.printStackTrace();
			}
		}

	* 更新实体对象
	
	具体代码：

		DbUtils db = DbUtils.create(getContext(), "student.db");
		try {
			
			Student s = new Student();
			s.setAge(26);
			//此方法调用，需要传入一个实体对象，并将需要update的字段值设置到实体属性中，
			//然后将实体对象作为参数传递
			//其中WhereBuilder对象是进行where条件设置，
			//最后一个参数是：String... 类型，可以接收多个String参数(即表结构中的列名)，这个参数即为你要更新的列，在此为更新age列的值
			db.update(s, WhereBuilder.b("name", ">", "ding6"), "age");
			
		} catch (DbException e) {
			e.printStackTrace();
		}

	* 删除实体对象
	
	代码如下：
	
		DbUtils db = DbUtils.create(getContext(), "student2teacher.db");
		try {
			//传入指定条件删除实体对象
			db.delete(Student.class, WhereBuilder.b("name", "=", "ding5"));
		} catch (DbException e) {
			e.printStackTrace();
		}

