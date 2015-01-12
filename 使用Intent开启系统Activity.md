###从google搜索内容 
	Intent intent = new Intent(); 
	intent.setAction(Intent.ACTION_WEB_SEARCH); 
	intent.putExtra(SearchManager.QUERY,"searchString") 
	startActivity(intent); 

###浏览网页 
	Uri uri = Uri.parse("http://www.google.com"); 
	Intent it = new Intent(Intent.ACTION_VIEW,uri); 
	startActivity(it); 

###显示地图 
	Uri uri = Uri.parse("geo:38.899533,-77.036476"); 
	Intent it = new Intent(Intent.Action_VIEW,uri); 
	startActivity(it); 

###路径规划 
	Uri uri = Uri.parse("http://maps.google.com/maps?f=dsaddr=startLat%20startLng&daddr=endLat%20endLng&hl=en"); 
	Intent it = new Intent(Intent.ACTION_VIEW,URI); 
	startActivity(it); 

###拨打电话 
	Uri uri = Uri.parse("tel:xxxxxx"); 
	Intent it = new Intent(Intent.ACTION_DIAL, uri); 
	startActivity(it); 

###调用发短信的程序 
	Intent it = new Intent(Intent.ACTION_VIEW); 
	it.putExtra("sms_body", "The SMS text"); 
	it.setType("vnd.android-dir/mms-sms"); 
	startActivity(it); 

###发送短信 
	Uri uri = Uri.parse("smsto:0800000123"); 
	Intent it = new Intent(Intent.ACTION_SENDTO, uri); 
	it.putExtra("sms_body", "The SMS text"); 
	startActivity(it); 

	String body="this is sms demo"; 
	Intent mmsintent = new Intent(Intent.ACTION_SENDTO, Uri.fromParts("smsto", number, null)); 
	mmsintent.putExtra(Messaging.KEY_ACTION_SENDTO_MESSAGE_BODY, body); 
	mmsintent.putExtra(Messaging.KEY_ACTION_SENDTO_COMPOSE_MODE, true); 
	mmsintent.putExtra(Messaging.KEY_ACTION_SENDTO_EXIT_ON_SENT, true); 
	startActivity(mmsintent); 

###发送彩信 
	Uri uri = Uri.parse("content://media/external/images/media/23"); 
	Intent it = new Intent(Intent.ACTION_SEND); 
	it.putExtra("sms_body", "some text"); 
	it.putExtra(Intent.EXTRA_STREAM, uri); 
	it.setType("image/png"); 
	startActivity(it); 

	StringBuilder sb = new StringBuilder(); 
	sb.append("file://"); 
	sb.append(fd.getAbsoluteFile()); 
	Intent intent = new Intent(Intent.ACTION_SENDTO, Uri.fromParts("mmsto", number, null)); 
	// Below extra datas are all optional. 
	intent.putExtra(Messaging.KEY_ACTION_SENDTO_MESSAGE_SUBJECT, subject); 
	intent.putExtra(Messaging.KEY_ACTION_SENDTO_MESSAGE_BODY, body); 
	intent.putExtra(Messaging.KEY_ACTION_SENDTO_CONTENT_URI, sb.toString()); 
	intent.putExtra(Messaging.KEY_ACTION_SENDTO_COMPOSE_MODE, composeMode); 
	intent.putExtra(Messaging.KEY_ACTION_SENDTO_EXIT_ON_SENT, exitOnSent); 
	startActivity(intent); 

###发送Email 
	//方式一
	Uri uri = Uri.parse("mailto:xxx@abc.com"); 
	Intent it = new Intent(Intent.ACTION_SENDTO, uri); 
	startActivity(it); 
	
	//方式二
	Intent it = new Intent(Intent.ACTION_SEND); 
	it.putExtra(Intent.EXTRA_EMAIL, "me@abc.com"); 
	it.putExtra(Intent.EXTRA_TEXT, "The email body text"); 
	it.setType("text/plain"); 
	startActivity(Intent.createChooser(it, "Choose Email Client")); 
	
	//方式三
	Intent it=new Intent(Intent.ACTION_SEND); 
	String[] tos={"me@abc.com"}; 
	String[] ccs={"you@abc.com"}; 
	it.putExtra(Intent.EXTRA_EMAIL, tos); 
	it.putExtra(Intent.EXTRA_CC, ccs); 
	it.putExtra(Intent.EXTRA_TEXT, "The email body text"); 
	it.putExtra(Intent.EXTRA_SUBJECT, "The email subject text"); 
	it.setType("message/rfc822"); 
	startActivity(Intent.createChooser(it, "Choose Email Client")); 

	//方式四
	Intent it = new Intent(Intent.ACTION_SEND); 
	it.putExtra(Intent.EXTRA_SUBJECT, "The email subject text"); 
	it.putExtra(Intent.EXTRA_STREAM, "file:///sdcard/mysong.mp3"); 
	sendIntent.setType("audio/mp3"); 
	startActivity(Intent.createChooser(it, "Choose Email Client")); 

###播放多媒体 
	Intent it = new Intent(Intent.ACTION_VIEW); 
	Uri uri = Uri.parse("file:///sdcard/song.mp3"); 
	it.setDataAndType(uri, "audio/mp3"); 
	startActivity(it); 

	Uri uri = Uri.withAppendedPath(MediaStore.Audio.Media.INTERNAL_CONTENT_URI, "1"); 
	Intent it = new Intent(Intent.ACTION_VIEW, uri); 
	startActivity(it); 

###uninstall apk 
	Uri uri = Uri.fromParts("package", strPackageName, null); 
	Intent it = new Intent(Intent.ACTION_DELETE, uri); 
	startActivity(it); 

###install apk 
	Uri installUri = Uri.fromParts("package", "xxx", null); 
	returnIt = new Intent(Intent.ACTION_PACKAGE_ADDED, installUri); 

### 打开照相机 
	<1>Intent i = new Intent(Intent.ACTION_CAMERA_BUTTON, null); 
	this.sendBroadcast(i); 
	<2>long dateTaken = System.currentTimeMillis(); 
	String name = createName(dateTaken) + ".jpg"; 
	fileName = folder + name; 
	ContentValues values = new ContentValues(); 
	values.put(Images.Media.TITLE, fileName); 
	values.put("_data", fileName); 
	values.put(Images.Media.PICASA_ID, fileName); 
	values.put(Images.Media.DISPLAY_NAME, fileName); 
	values.put(Images.Media.DESCRIPTION, fileName); 
	values.put(Images.ImageColumns.BUCKET_DISPLAY_NAME, fileName); 
	Uri photoUri = getContentResolver().insert( 
	MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values); 
	
	Intent inttPhoto = new Intent(MediaStore.ACTION_IMAGE_CAPTURE); 
	inttPhoto.putExtra(MediaStore.EXTRA_OUTPUT, photoUri); 
	startActivityForResult(inttPhoto, 10); 

###从gallery选取图片 
	Intent i = new Intent(); 
	i.setType("image/*"); 
	i.setAction(Intent.ACTION_GET_CONTENT); 
	startActivityForResult(i, 11); 

###打开录音机 
	Intent mi = new Intent(Media.RECORD_SOUND_ACTION); 
	startActivity(mi); 

###显示应用详细列表 
	Uri uri = Uri.parse("market://details?id=app_id"); 
	Intent it = new Intent(Intent.ACTION_VIEW, uri); 
	startActivity(it); 
	//where app_id is the application ID, find the ID 
	//by clicking on your application on Market home 
	//page, and notice the ID from the address bar 
	
	刚才找app id未果，结果发现用package name也可以 
	Uri uri = Uri.parse("market://details?id=<packagename>"); 
	这个简单多了 

###寻找应用 
	Uri uri = Uri.parse("market://search?q=pname:pkg_name"); 
	Intent it = new Intent(Intent.ACTION_VIEW, uri); 
	startActivity(it); 
	//where pkg_name is the full package path for an application 

###打开联系人列表 
	<1> 
	Intent i = new Intent(); 
	i.setAction(Intent.ACTION_GET_CONTENT); 
	i.setType("vnd.android.cursor.item/phone"); 
	startActivityForResult(i, REQUEST_TEXT); 
	
	<2> 
	Uri uri = Uri.parse("content://contacts/people"); 
	Intent it = new Intent(Intent.ACTION_PICK, uri); 
	startActivityForResult(it, REQUEST_TEXT); 

###打开另一程序 
	Intent i = new Intent(); 
	ComponentName cn = new ComponentName("com.yellowbook.android2", 
	"com.yellowbook.android2.AndroidSearch"); 
	i.setComponent(cn); 
	i.setAction("android.intent.action.MAIN"); 
	startActivityForResult(i, RESULT_OK);