####TabPageIndicator+ViewPager+Fragment实现带tab的页面切换

> 实现类似与网易新闻客户端的tab页切换功能，每切换一个标题，下面的界面转换到对应的界面展示。主要实现是使用第三方的lib库中的TabPageIndicator类，这个第三方包中有很多这种结合ViewPager的切换效果，如：带圆点的、带线的、带标题的、带tab的、带图标的等等。TabPageIndicator只是这个第三方lib库中的一个。


具体的lib库可以在github上下载，下面是其地址：

https://github.com/JakeWharton/Android-ViewPagerIndicator

下面以一个简单的demo来描述其使用。效果图如下：

![](https://raw.githubusercontent.com/dcq123/Android-Notes/master/images/tab_view.png)


##### 引入这个library库

将下载的Android-ViewPagerIndicator项目中，导入其中的library到Eclipse中，然后在自己新创建的项目中引入这个library,然后就可以使用其中不同的Indicator，这个单词是指示器的意思，意味着使用它来指示Viewpager显示那个item view。其中包含的不同类型的Indicator如下：

![](https://raw.githubusercontent.com/dcq123/Android-Notes/master/images/indicator.png)

![](https://raw.githubusercontent.com/dcq123/Android-Notes/master/images/yinyong.png)

####布局文件中使用

	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity" >

    <!-- 直接定义TabPageIndicator不用其他的设置 -->
    <com.viewpagerindicator.TabPageIndicator
        android:id="@+id/tabpage"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    
    <!-- 下面定义一个ViewPager组件 -->
    <android.support.v4.view.ViewPager
        android:id="@+id/viewpage"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        />

	</LinearLayout>

#####在Activity中使用

> 在使用时，使用FragmentPagerAdapter作为ViewPager的Adapter，这样我们就可以在每个View item中都使用一个Fragment，这样界面显示及管理更加容易。


	public class MainActivity extends FragmentActivity {
	
	private ViewPager viewPager;
	private TabPageIndicator tabPage;
	// 标题
	public static String[] TITLE = {"推荐","互联网","手机","小米","魅族","Iphone","美女","时尚","热门","程序员"};
	// 标题前面的icon
	private static final int[] ICONS = new int[] {
        R.drawable.perm_group_calendar,
        R.drawable.perm_group_camera,
        R.drawable.perm_group_device_alarms,
        R.drawable.perm_group_location,
	};

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		FragmentPagerAdapter adapter = new MyPageAdapter(getSupportFragmentManager());
		
		tabPage = (TabPageIndicator) findViewById(R.id.tabpage);
		viewPager = (ViewPager) findViewById(R.id.viewpage);
		viewPager.setAdapter(adapter);
		
		// 设置ViewPager和TabPageIndicator之间的关联
		tabPage.setViewPager(viewPager);
		/**
		 * 设置当再次选择当前所在的这个tab标签时，执行的回调方法
		 */
		tabPage.setOnTabReselectedListener(new OnTabReselectedListener() {
			
			@Override
			public void onTabReselected(int position) {
				Toast.makeText(getApplicationContext(), "------setOnTabReselectedListener----", Toast.LENGTH_SHORT).show();
			}
		});
		
		/**
		 * 和ViewPager的setOnPageChangeListener监听一样，其内部也是实现了ViewPager的setOnPageChangeListener，
		 * 但是在此处监听页面变化，需要调用TabPageIndicator来进行监听
		 */
		tabPage.setOnPageChangeListener(new OnPageChangeListener() {
			
			@Override
			public void onPageSelected(int position) {
				Toast.makeText(getApplicationContext(), "------setOnPageChangeListener----"+TITLE[position], Toast.LENGTH_SHORT).show();
				
			}
			
			@Override
			public void onPageScrolled(int arg0, float arg1, int arg2) {				
			}
			
			@Override
			public void onPageScrollStateChanged(int arg0) {				
			}
		});
	}
	
	/**
	 * 
	 * @author qing 
	 * @date 2015-1-20
	 * 这个类是为ViewPager提供Adapter的类，使用FragmentPagerAdapter，
	 * 可以使每一个ViewPager的item都是一个Fragment，这样更有利于管理每个界面
	 * 
	 * 其中又实现了IconPagerAdapter接口，这是为每个tab标签前面添加icon图标的，
	 * 如果不想要前面的图标，去掉IconPagerAdapter接口实现即可
	 *
	 */
	class MyPageAdapter extends FragmentPagerAdapter implements IconPagerAdapter{

		public MyPageAdapter(FragmentManager fm) {
			super(fm);
		}

		@Override
		/**
		 * 返回一个Fragment作为viewPager的item
		 */
		public Fragment getItem(int position) {
			return PageFragment.newInstance(TITLE[position]);
		}
		
		@Override
		/**
		 * 设置tab的标题
		 */
		public CharSequence getPageTitle(int position) {
			return TITLE[position];
		}

		@Override
		public int getCount() {
			return TITLE.length;
		}

		@Override
		/**
		 * 设置tab前面的icon
		 */
		public int getIconResId(int index) {
			
			return ICONS[index%ICONS.length];
		}
		
	}	

	}

	
#####使用到的PageFragment

> 在下面的Fragment中，就可以根据需求加载不同的界面。

	public class PageFragment extends Fragment {
	private String content = "";
	
	/**
	 * 该方法每次都返回一个新的Fragment实例
	 * @param content
	 * @return
	 */
	public static PageFragment newInstance(String content)
	{
		PageFragment pf = new PageFragment();
		pf.content = content;
		return pf;
	}

	
	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container,
			Bundle savedInstanceState) {
		View view = inflater.inflate(R.layout.page_fragment_layout, null);
		TextView tv_msg = (TextView) view.findViewById(R.id.tv_msg);
		tv_msg.setText(content);
		
		return view;
	}
	}


##### 在清单文件中配置样式

> 在定义好布局和Activity以后，清单文件中配置Activity时需要额外的指定其theme，使用这个来定义TabPageIndicator的显示样式，样式定义在styles.xml文件中。

	<activity
            android:name="cn.qing.test.MainActivity"           
            android:theme="@style/StyledIndicators"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>


##### styles.xml指定样式

	<!--   TabPageIndicator的自定义样式 -->
     <style name="StyledIndicators" parent="@android:style/Theme.Light">
        <item name="vpiCirclePageIndicatorStyle">@style/CustomCirclePageIndicator</item>
        <item name="vpiLinePageIndicatorStyle">@style/CustomLinePageIndicator</item>
        <item name="vpiTitlePageIndicatorStyle">@style/CustomTitlePageIndicator</item>
        <item name="vpiTabPageIndicatorStyle">@style/CustomTabPageIndicator</item>
        <item name="vpiUnderlinePageIndicatorStyle">@style/CustomUnderlinePageIndicator</item>
    </style>
    
     <style name="CustomTitlePageIndicator">
        <item name="android:background">#18FF0000</item>
        <item name="footerColor">#FFAA2222</item>
        <item name="footerLineHeight">1dp</item>
        <item name="footerIndicatorHeight">3dp</item>
        <item name="footerIndicatorStyle">underline</item>
        <item name="android:textColor">#AA000000</item>
        <item name="selectedColor">#FF000000</item>
        <item name="selectedBold">true</item>
    </style>

    <style name="CustomLinePageIndicator">
        <item name="strokeWidth">4dp</item>
        <item name="lineWidth">30dp</item>
        <item name="unselectedColor">#FF888888</item>
        <item name="selectedColor">#FF880000</item>
    </style>

    <style name="CustomCirclePageIndicator">
        <item name="fillColor">#FF888888</item>
        <item name="strokeColor">#FF000000</item>
        <item name="strokeWidth">2dp</item>
        <item name="radius">10dp</item>
        <item name="centered">true</item>
    </style>

    <style name="CustomTabPageIndicator" parent="Widget.TabPageIndicator">
        <item name="android:background">@drawable/custom_tab_indicator2</item>
        <item name="android:textAppearance">@style/CustomTabPageIndicator.Text</item>
        <item name="android:textColor">#FF555555</item>
        <item name="android:textSize">16sp</item>
        <item name="android:divider">@drawable/custom_tab_indicator_divider</item>
        <!-- 下面的两个属性是用来设置两个tab直接的分割线的，需要在API 11以上才能正常显示 -->
        <item name="android:dividerPadding">10dp</item>
        <item name="android:showDividers">middle</item> 
        <item name="android:paddingLeft">8dp</item>
        <item name="android:paddingRight">8dp</item>
        <item name="android:fadingEdge">horizontal</item>
        <item name="android:fadingEdgeLength">8dp</item>
    </style>

    <style name="CustomTabPageIndicator.Text" parent="android:TextAppearance.Medium">
        <item name="android:typeface">monospace</item>
    </style>

    <style name="CustomUnderlinePageIndicator">
        <item name="selectedColor">#FFCC0000</item>
        <item name="android:background">#FFCCCCCC</item>
        <item name="fadeLength">1000</item>
        <item name="fadeDelay">1000</item>
    </style>
	<!--   TabPageIndicator的自定义样式 end -->

上面的样式，指定的是应用的类库中所有的Indicator的样式，在library中有默认样式，这里面定义的是可以根据我们需求来修改的样式属性。

样式中使用到了一些drawable资源，可以直接从Android-ViewPagerIndicator项目的example例子中复制出来。