### ExpandableListView的使用

> ExpandableListView是ListView的扩展，它可以实现分组显示ListView，像QQ的好友列表界面就属于ExpandableListView。它的使用和ListView很像，就是在ListView的基础上增加了外层的分组，然后按组来显示ListView。它对应的Adapter需要继承自BaseExpandableListAdapter。

#### 为Adapter提供数据对应的Bean

> 以下定义两个Bean，用来为Adapter提供数据，需要说明的是，这种数据结构嵌套了一层。

ExternalSmsBean是外层的Bean对象其中包含了一个集合保存的是组内的ListView对应的数据Bean.

	/**
	 * 外层ListView中的每个Item封装
	 *
	 */
	public class ExternalSmsBean {
	
		/**
		 * 外层item的名称
		 */
		private String itemName;
		/**
		 * 外层item的图标
		 */
		private int icon;
		/**
		 * 外层item对应的计数
		 */
		private int count;
		/**
		 * 包含的子ListView中的item
		 */
		private List<SmsBean> childList;
		
		private boolean isOpen = false;
	}

内层数据Bean:SmsBean

	/**
	 * 短信对象封装类
	 *
	 */
	public class SmsBean {
	
		private long _id;
		/**
		 * 短信的发送或者接收方
		 */
		private String name;
		/**
		 * 发送或者接收时间
		 */
		private long date;
		/**
		 * 短信内容
		 */
		private String msg;
		private String number;
	}

#### Adapter的实现

有了以上两个Bean以后，来看继承的Adapter中的实现：

	/**
	 * 继承ExpandableListView需要的Adapter类型:BaseExpandableListAdapter
	 *
	 */
	public class ExpendAdapter extends BaseExpandableListAdapter {
		
		private List<ExternalSmsBean> data;
		private Context context;
		
		public ExpendAdapter(Context context,List<ExternalSmsBean> data) {
			this.data = data;
			this.context = context;
		}
	
		@Override
		/**
		 * 最外层ListView的item的个数
		 */
		public int getGroupCount() {
			
			return data.size();
		}
	
		@Override
		/**
		 * 指定位置的外层ListView内包含的子ListView的item的个数
		 */
		public int getChildrenCount(int groupPosition) {
			return data.get(groupPosition).getChildList().size();
		}
	
		@Override
		/**
		 * 返回外层ListView中的一个对象
		 */
		public Object getGroup(int groupPosition) {
			return data.get(groupPosition);
		}
	
		@Override
		/**
		 * 指定位置的外层ListView的Item内包含的某个指定的Item
		 */
		public Object getChild(int groupPosition, int childPosition) {
			return data.get(groupPosition).getChildList().get(childPosition);
		}
	
		@Override
		/**
		 * 返回指定位置外层ListView中item的id，这个方法在用到的时候返回正确的id即可，不用时可不做处理
		 */
		public long getGroupId(int groupPosition) {
			return 0;
		}
	
		@Override
		/**
		 * 同上，对应的是内层的item的id
		 */
		public long getChildId(int groupPosition, int childPosition) {
			return 0;
		}
	
		@Override
		public boolean hasStableIds() {
			return false;
		}
		
		@Override
		/* 很重要：实现ChildView点击事件，必须返回true */  
		public boolean isChildSelectable(int groupPosition, int childPosition) {
			return true;
		}
	

		@Override
		public View getGroupView(int groupPosition, boolean isExpanded,
				View convertView, ViewGroup parent) {
			ExternalHolder holder = null;
			if(convertView==null)
			{
				convertView = View.inflate(context, R.layout.external_item_layout, null);
				holder = new ExternalHolder();
				holder.iv_icon = (ImageView) convertView.findViewById(R.id.iv_icon);
				holder.iv_open = (ImageView) convertView.findViewById(R.id.iv_open);
				holder.tv_name = (TextView) convertView.findViewById(R.id.tv_name);
				convertView.setTag(holder);
			}
			else
				holder = (ExternalHolder) convertView.getTag();
		
		ExternalSmsBean item = data.get(groupPosition);
		
		holder.iv_icon.setBackgroundResource(item.getIcon());
		holder.tv_name.setText(item.getItemName()+"("+item.getCount()+")");
		
		if(item.isOpen())
			holder.iv_open.setImageResource(R.drawable.close);
		else
			holder.iv_open.setImageResource(R.drawable.open);
			
		
		return convertView;
		}

		@Override
		public View getChildView(int groupPosition, int childPosition,
				boolean isLastChild, View convertView, ViewGroup parent) {
			ViewHolder holder = null;
			if(convertView==null)
			{			
				convertView = View.inflate(context, R.layout.message_item_layout, null);
				holder = new ViewHolder();
				holder.iv_icon = (ImageView) convertView.findViewById(R.id.iv_icon);
				holder.tv_address = (TextView) convertView.findViewById(R.id.tv_address);
				holder.tv_message = (TextView) convertView.findViewById(R.id.tv_message);
				holder.tv_time = (TextView) convertView.findViewById(R.id.tv_time);
				holder.cb_ischeck = (ImageView) convertView.findViewById(R.id.cb_ischeck);
				holder.cb_ischeck.setVisibility(View.GONE);
				
				convertView.setTag(holder);
			}
			else
				holder = (ViewHolder) convertView.getTag();
		
		SmsBean bean = data.get(groupPosition).getChildList().get(childPosition);
		
		String address = bean.getNumber();
		System.out.println("address:"+address);
		ContactInfo ci = ContactUtils.findContactInfoByNumber(context, address);
		
		if(ci!=null)
		{
			holder.tv_address.setText(ci.getName());
			InputStream is = ci.getIconIS();
			if(is!=null)
			{
				holder.iv_icon.setImageBitmap(BitmapFactory.decodeStream(is));
			}
			else
				holder.iv_icon.setImageResource(R.drawable.qq_contact_list_friend_entry_icon);
		}
		else			
		{
			holder.tv_address.setText(address);
			holder.iv_icon.setImageResource(R.drawable.qq_contact_list_friend_entry_icon);
		}
		
		holder.tv_message.setText(bean.getMsg());
		String time  = "";
		if(DateUtils.isToday(bean.getDate()))
		{
			time = DateFormat.getTimeFormat(context).format(bean.getDate());
		}
		else
			time = DateFormat.getDateFormat(context).format(bean.getDate());
		
		holder.tv_time.setText(time);
		
		return convertView;
		}
		class ExternalHolder{
			ImageView iv_icon,iv_open;
			TextView tv_name;
		}
		
		class ViewHolder{
			ImageView iv_icon,cb_ischeck;
			TextView tv_address,tv_message,tv_time;
		}	
		}


上面Adapter中的方法虽然很多，但是根据名称就能知道方法的意思，主要要实现的是：getGroupView和getChildView，这两个方法跟ListView中的getView方法做的事情一样。

其中需要特别注意的一个是：isChildSelectable()这个方法的返回值必须的是true时，子ListView中的item才能选中，响应点击事件。

#### 布局文件中的使用

	<ExpandableListView
        android:id="@+id/elv_sms"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:groupIndicator="@null"
        android:cacheColorHint="@android:color/transparent"
        android:listSelector="@android:color/transparent" />

主要需要说的是groupIndicator属性，这个属性是设置组前面显示的图标，默认ExpandableListView这个组件的组前面会有一个圆形的图标，点击时会上下切换箭头，但是图标不好看，所以可以将groupIndicator设置为 @null 去掉前面的箭头图标。



#### 在Activity中的使用


	public class FloderActivity extends Activity {

	private ExpandableListView elv;
	private List<ExternalSmsBean> data;
	private ExpendAdapter adapter;
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.floder_main_layout);
		
		elv = (ExpandableListView) findViewById(R.id.elv_sms);
		
		initData();		
		
		/**
		 * 设置外层组Item的点击监听
		 */
		elv.setOnGroupClickListener(new OnGroupClickListener() {
			
			@Override
			public boolean onGroupClick(ExpandableListView parent, View v,
					int groupPosition, long id) {
				
				
				ExternalSmsBean esb = data.get(groupPosition);
				esb.setOpen(!esb.isOpen());
				
				adapter.notifyDataSetChanged();
				// 在这里如果返回true后，子ListView就不能打开，所以此处必须的返回false
				return false;
			}
		});
		
		/**
		 * 设置子ListView中Item的点击事件
		 */
		elv.setOnChildClickListener(new OnChildClickListener() {
			
			@Override
			public boolean onChildClick(ExpandableListView parent, View v,
					int groupPosition, int childPosition, long id) {
				
				SmsBean sb = data.get(groupPosition).getChildList().get(childPosition);
				Toast.makeText(getApplicationContext(), "你点击的是："+sb.getName(), 0).show();
				return false;
			}
		});
		
	}

	private void initData()
	{
		
		data = new ArrayList<ExternalSmsBean>();
		
		for(int i= 0 ;i < names.length ; i++)
		{			
			ExternalSmsBean esb = queryExternalSmsBean(i);
			data.add(esb);
		}
		adapter = new ExpendAdapter(this,data);
		elv.setAdapter(adapter);
	}

 从上面的代码中可以看到，在Activity中使用ExpandableListView也非常简单，主要需要注意的是它的Click监听，这个监听需要分别对待，组item的监听是：setOnGroupClickListener。子Item的监听是：setOnChildClickListener。