### Notification的使用总结

> Notification是显示在通知栏里的通知，经常用于消息推送通知，用户点击以后即可进入通知的详细信息界面，可以设置Notification点击以后就从通知栏里消失，也可以使其一直显示在通知栏中不被清除掉。Notification的设置跟Android的版本也有关，API 11或者说Android 3.0之前，只能使用一些简单的通知用于显示内容，不能自定义通知的显示界面，3.0之后可以为其指定一个显示界面，这个界面就由用户去自定义了。


##### 创建Notification

> 在API 16之前，可以使用带三个参数的构造方法来new一个Notification，但是这种方法现在已经是过时的方法。在API 16的版本之后，在Notification中增加了一个Builder的内部类，可以使用这个类来创建通知，在创建时可以使用builder类中提供的方法，来设置通知的一下属性信息。个人感觉使用Builder构建只是根据规范了一些，但是构建起来也并非简单，该设置的属性还是需要设置，而且API 版本要求高，最小SDK必须是16或以上才能满足，不能满足低版本的手机。还可以使用support v4包中定义的NotificationCompat类来创建兼容低版本的Notification。


* 直接newNotification,使用三个参照的构造方法

		Notification notif = new Notification(icon, tickerText, when);

	* icon : 指定通知要显示的图标
	* tickerText : 通知在出现时，在通知栏显示的一句通知的提示信息
	* when ： 何时显示通知，一般取值为：System.currentTimeMillis();当前时间立即显示。


* 使用Notification中定义的Builder构建：

	> 这种方式创建Notification最低SDK版本为16，所以在兼容低版本时不能使用

		Notification.Builder builder = new Notification.Builder(this);
		Notification noti = builder.build();

* 使用NotificationCompat类中的Builder来创建Notification

		NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
		builder.setLargeIcon(BitmapFactory.decodeResource(getResources(), R.drawable.ic_launcher))	// 设置最前面显示的大图标
				.setSmallIcon(R.drawable.img1)		// 设置后面也就是右边显示的小图标
				.setContentTitle("通知标题")
				;
		
		builder.setContentText("通知栏信息..."+number)
				.setNumber(++number);
		
		Intent intent = new Intent(this,NotificationInfoAcitivity.class);
		
		// 使用任务栈来返回一个响应通知栏点击事件的动作，在此为打开一个Activity，如果这个Activity已经打开过，则直接从栈里返回，不会重新创建
		TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
		
		// 设置ParentStack是想在点击通知进入某个Activity以后，用户按返回键可以回到的ParentActivity，这个父Activity实在清单文件中指定的
		/*<activity android:name=".MainActivity" >
        </activity>
        <activity
            android:name=".NotificationInfoAcitivity"
            android:parentActivityName=".MainActivity" >
            <meta-data
                android:name="android.support.PARENT_ACTIVITY"
                android:value=".MainActivity" />
        </activity>*/
		
		stackBuilder.addParentStack(NotificationInfoAcitivity.class);
		stackBuilder.addNextIntent(intent);
		
		PendingIntent pendingIntent = stackBuilder.getPendingIntent(1, PendingIntent.FLAG_UPDATE_CURRENT);
		// 设置点击响应动作
		builder.setContentIntent(pendingIntent);
		
		builder.setDefaults(Notification.DEFAULT_VIBRATE);
		// 设置点击后，通知自动取消
		builder.setAutoCancel(true);		
		
		
		nm.notify(1, builder.build());
	

* Notification中的flags属性
	> 这个属性有很多种，用来设置通知在通知栏里面的状态行为等，常用的有以下几个：
	
	* Notification.FLAG_ONGOING_EVENT：设置通知不能在通知栏中取消，一直显示，而且会一直在状态栏中显示图标，像QQ一样
	* Notification.FLAG_AUTO_CANCEL : 当用户点击了通知以后，通知将会自动在通知栏中清除
	* Notification.FLAG_INSISTENT ： 重复发出声音，直到用户响应此通知
	* Notification.FLAG_ONLY_ALERT_ONCE : 标记声音或者震动一次
	* Notification.FLAG_SHOW_LIGHTS : 设置通知呼吸灯
	* Notification.FLAG_NO_CLEAR : 设置通知不能在被通知栏的清除按钮清除
	* Notification.FLAG_FOREGROUND_SERVICE  : 标记前台服务



##### 有两种方式管理Notification：

* 使用NotificationManager管理通知

	> NotificationManager是底层系统服务，需要使用Context.getSystemService(Context.NOTIFICATION_SERVICE);来创建。创建出NotificationManager对象以后再创建Notification对象，将创建出来的Notification对象作为notify()的参数来开启一个通知，也可以使用NotificationManager的cancel()方法来取消一个通知。

* 在自定义的服务中使用startForeground()方法

	> 在service中可以调用startForeground(id, notification)方法来发送一个通知，id当前通知的标识，多个通知时，要确保唯一，后面就是定义出来的notification对象。在service中可以通过使用：stopForeground(true);来取消一个通知。

########使用Notification的简单Demo

> 以下的demo中是常用的Notification的属性设置，而且这些都是API中提供的，如一些声音、震动、呼吸灯效果的设置，在状态栏中的显示状态flags的设置等，显示的都是一些简单的通知界面，即：标题+信息文字


	private NotificationManager nm;
	private int SIMPLE_NOTIFICATION_FLAG = 1;
	private int BUILDER_CREATE_FLAG = 2;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		nm = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
	}

	public void startSimpleNotif(View v)
	{
		int icon = R.drawable.ic_launcher;
		String tickerText = "我是通知栏出现时显示的提示文字...";
		long when = System.currentTimeMillis();
		//此方法已过时，但是还可以使用，这是低版本时用的方法
		Notification notif = new Notification(icon ,tickerText , when );
		
		
		
		// 设置通知的声音，震动，以及呼吸灯，如果指定为DEFAULT_ALL，则全部使用系统默认的通知设置
		//		notif.defaults = Notification.DEFAULT_ALL;
		
		// 设置自定义通知声音，声音资源放在raw目录中
		notif.sound = Uri.parse("android.resource://"+getPackageName()+"/"+R.raw.ylzs);
		
		// 在android 2.3版本中，使用震动需要添加震动权限
	//		<!-- 震动权限 -->
	//	    <uses-permission android:name="android.permission.VIBRATE"/>
		
		// Notification.FLAG_ONGOING_EVENT： 设置通知不能在通知栏中取消，一直显示，而且会一直在状态栏中显示图标，像QQ一样
		// Notification.FLAG_AUTO_CANCEL : 当用户点击了通知以后，通知将会自动在通知栏中清除
		// Notification.FLAG_INSISTENT ： 重复发出声音，直到用户响应此通知
		// Notification.FLAG_ONLY_ALERT_ONCE : 标记声音或者震动一次
		// Notification.FLAG_SHOW_LIGHTS : 设置通知呼吸灯
		// Notification.FLAG_NO_CLEAR : 设置通知不能在被通知栏的清除按钮清除
		// Notification.FLAG_FOREGROUND_SERVICE  : 标记前台服务
		notif.flags = Notification.FLAG_NO_CLEAR;	
		
		
		// PendingIntent中要触发的Intent动作
		Intent doIntent = new Intent(this,MainActivity.class);
		PendingIntent contentIntent = PendingIntent.getActivity(this, 1, doIntent, PendingIntent.FLAG_UPDATE_CURRENT);
		
		// 此方法也是过时的，这个方法是用来显示通知标题和内容，并在用户点击Notification时，指定点击以后的PendingIntent
		notif.setLatestEventInfo(this, "通知标题", "通知标题下面的内容...", contentIntent);
		
		// 使用NotificationManager添加通知
		nm.notify(SIMPLE_NOTIFICATION_FLAG , notif);
	}
	
	public void endSimpleNotif(View v)
	{
		// 取消指定的Notification，根据id值
		nm.cancel(SIMPLE_NOTIFICATION_FLAG);
	}
	
	
	public void startNotiByBuilder(View v)
	{
		// 使用Builder来构建Notification所需要的属性信息，需要传递Context上下文
		Notification.Builder builder = new Notification.Builder(this);
		// 设置图标
		builder.setSmallIcon(R.drawable.ic_launcher);
		// 设置Ticker
		builder.setTicker("我是通知栏出现时显示的提示文字...");
		// 设置when
		builder.setWhen(System.currentTimeMillis());
		// 设置默认的声音、震动、呼吸灯效果
		builder.setDefaults(Notification.DEFAULT_ALL);	// 以上两个属性在new Builder()的时候，就自动设置上了默认属性
		
		// 设置标题
		builder.setContentTitle("通知标题");
		 // 设置标题下面的内容
		builder.setContentText("通知标题下面的内容...");
		
		// PendingIntent中要触发的Intent动作
		Intent doIntent = new Intent(this,MainActivity.class);
		PendingIntent contentIntent = PendingIntent.getActivity(this, 1, doIntent, PendingIntent.FLAG_UPDATE_CURRENT);
		
		// 设置点击通知后要执行的动作
		builder.setContentIntent(contentIntent);	
		
		
		// 使用build()来创建出Notification
		Notification noti = builder.build();
		// 标记声音或震动只发生一次
		noti.flags = Notification.FLAG_ONLY_ALERT_ONCE;
		
		nm.notify(BUILDER_CREATE_FLAG , noti);
	}

########使用自定义View作为Notification的显示视图

> 以下是通过自定义一个view，将其设置为Notification对象的显示视图demo，通过这样的方式，我们就可以在通知栏中定义出自己想要的视图界面。

	/**
	 * 自定义通知视图
	 * @param v
	 */
	public void startSelfNoti(View v)
	{
		
		Notification noti = new Notification();
		// 设置icon,经过实验，如果不设置icon属性，自定义的视图view不会显示在通知栏中
		noti.icon = R.drawable.ic_launcher;
		noti.tickerText = "手机安全卫士，锁屏清理功能已经开启...";
		// 设置flags为一直显示在通知栏中
		noti.flags = Notification.FLAG_ONGOING_EVENT;
		// 设置通知的默认效果
		noti.defaults = Notification.DEFAULT_ALL;
		
		// 获取自定义的通知界面view ，通过RemoteViews对象来获取显示在通知栏中的View
		RemoteViews rv = new RemoteViews(getPackageName(), R.layout.notification_layout);
		// 为view中某一个id组件设置显示文本
		rv.setTextViewText(R.id.start_activity, "开启主界面");
		
		// PendingIntent中要触发的Intent动作
		Intent doIntent = new Intent(this,MainActivity.class);
		doIntent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
		PendingIntent pendingIntent = PendingIntent.getActivity(this, 1, doIntent, PendingIntent.FLAG_UPDATE_CURRENT);
		
		// 为view中某个id组件设置点击的响应事件
		rv.setOnClickPendingIntent(R.id.ll_start, pendingIntent);
		
		// 将自定义视图view赋值给Notification对象的contentView属性，这个属性专门用来显示自定义的视图
		noti.contentView = rv ;
		
		// 使用NotificationManager来发送通知
		nm.notify(SELF_NOTI_FLAG , noti);		
	}


