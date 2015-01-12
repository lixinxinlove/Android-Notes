## 常用的系统级 BroadcastReceiver 监听及相关权限

# 拨打电话

	<action  android:name="android.intent.action.NEW_OUTGOING_CALL"/>

	需要添加相应的权限
	<!-- 监听拨打电话的权限 -->
    <uses-permission android:name="android.permission.PROCESS_OUTGOING_CALLS"/>

	//电话号码是存在resultData中
	String number = getResultData();

# 接收短信

	//由于短信的接受广播属于有序广播，所以需要设置意图过滤器的优先级来指定先进入的receiver
	<intent-filter android:priority="1000">
                <action android:name="android.provider.Telephony.SMS_RECEIVED"/>
    </intent-filter>

	需要添加接收短信的权限
	<!-- 监听短息权限 -->
	<uses-permission android:name="android.permission.RECEIVE_SMS"/>

	//在自定义的广播接收器中拦截指定号码的短信
	@Override
	public void onReceive(Context context, Intent intent) {
		
		//从Bundle中取出短信数据
		Bundle data = intent.getExtras();
		//从key:puds中获取短信信息
		Object[] objs = (Object[])data.get("pdus");
		
		for (Object object : objs) {
			//从byte[]中构建短信SmsMessage
			SmsMessage smsMessage = SmsMessage.createFromPdu((byte[])object);
			//获取发送的人
			String address = smsMessage.getOriginatingAddress();
			//获取短信消息
			String body = smsMessage.getMessageBody();
			if("138123".equals(address))
			{
				Toast.makeText(context, "已拦截【138123】发送的短息："+body, Toast.LENGTH_LONG).show();
				//阻断广播
				abortBroadcast();
			}
		}
	}

# 开机完成

	//需要添加开机完成的动作监听
	 <action android:name="android.intent.action.BOOT_COMPLETED"/>

	//添加接收开机广播的权限
	<!-- 接收开机完成权限 -->
	<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

# 发送SD卡就绪广播，让系统去检测SD卡中的内容

	
	Intent intent = new Intent();
	//广播SD卡就绪动作
	intent.setAction(Intent.ACTION_MEDIA_MOUNTED);
	//并指定data为SD卡路径
	intent.setData(Uri.fromFile(Environment.getExternalStorageDirectory()));
	//发送广播
	sendBroadcast(intent);

# 读取手机SIM状态权限


	<uses-permission android:name="android.permission.READ_PHONE_STATE"/>

# 获取联系人权限
	
	 <uses-permission android:name="android.permission.READ_CONTACTS"/>
    <uses-permission android:name="android.permission.WRITE_CONTACTS"/>

# 获取地理位置的权限


	<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_MOCK_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>

使用LocationManager获取位置信息时，需要添加以上权限


