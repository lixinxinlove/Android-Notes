### ViewPager的使用总结

> viewPager是在Android的support包中提供的用来切换视图的组件，将不同的view添加到viewPager中，可以使用左右滑动切换view的效果。它也经常被用来切换广告图片。下面的列子实现两种形式的广告条，两种都是循环自动播放，但是一种是到达最后一张后，直接回到第一张，这样的一种从头循环方式；一种是到达最后一张以后，在往前，一直自动往前到达第一张，这样的往返循环模式。

实现循环的主要方法就是根据下标调用ViewPager的setCurrentItem(int)方法来实现。

##### 布局文件中的定义

> 用于ViewPager在support v4 包中，使用这个控件需要以自定义组件的形式来引用。

	<?xml version="1.0" encoding="utf-8"?>
	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <android.support.v4.view.ViewPager
        android:id="@+id/viewpager"
        android:layout_width="match_parent"
        android:layout_height="200dp" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:layout_alignBottom="@id/viewpager"
        android:background="#770D0303"
        android:gravity="center_horizontal"
        android:orientation="vertical" >

        <TextView
            android:id="@+id/tv_desc"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="5dp"
            android:text="测试文字"
            android:textColor="#fff"
            android:textSize="13sp" />

        <LinearLayout
            android:id="@+id/point_container"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal" >
        </LinearLayout>
    </LinearLayout>

	</RelativeLayout>

在布局文件中，只需要引用即可，其它不用额外定义。下面的两种实现方式使用的都是上面的这个布局。


##### 从头循环的方式

> 从头循环的方式，是将ViewPager的item设置特别大为int的最大值，然后在刚开始时，就将ViewPager的位置设置到这个最大值的一半，这样一来就可以一直向前滑动，也可以一直向后滑动，当然，由于view的数量有限，所以在下标取值的时候，是根据这个 view的数量进行了取模，否则会包数组下标越界异常。实现循环自动播放使用的是Handler中继续发相同的Handler，来达到类似与Timer的效果。

具体实现代码如下：

	public class ViewPageActivity extends Activity {
	
	private ViewPager viewPager;
    private TextView tv_desc;
    private LinearLayout point_container;
    
	 	// 图片资源ID
	 	private final int[] imageIds = { R.drawable.a, R.drawable.b, R.drawable.c,
	 			R.drawable.d, R.drawable.e };
	
	 	// 图片标题集合
	 	private final String[] imageDescriptions = { "巩俐不低俗，我就不能低俗",
	 			"扑树又回来啦！再唱经典老歌引万人大合唱", "揭秘北京电影如何升级", "乐视网TV版大派送", "热血屌丝的反杀" };
	
	 	private List<ImageView> imageList;
	 	
	 	private int lastPosition = 0;
	 	
	 	private int centerPosition = 0;
	 	
	 	// 标记是否循环发送Handler消息，在Activity销毁时，将其设置为false，取消消息的发送
	 	private boolean flag = true;
	 	
	 	private Handler handler = new Handler(){
	 		public void handleMessage(android.os.Message msg) {
	 			
	 			if(flag)
	 			{
	 				// 使用延时的Handler循环发送消息，达到一个计时的效果
	 				viewPager.setCurrentItem(viewPager.getCurrentItem()+1); 	 			
	 	 			handler.sendEmptyMessageDelayed(11, 2000);
	 	 			
	 			}
	 			
	 		};
	 		
	 	};
    
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		
		setContentView(R.layout.view_page_layout);
		
		viewPager = (ViewPager)findViewById(R.id.viewpager);
		
		tv_desc = (TextView) findViewById(R.id.tv_desc);
		
		point_container = (LinearLayout) findViewById(R.id.point_container);
		// 设置默认的文字描述
		tv_desc.setText(imageDescriptions[0]);
		
		imageList = new ArrayList<ImageView>();		
		for(int i = 0 ;i <imageIds.length ; i++ )
		{
			ImageView iv = new ImageView(this);
			iv.setBackgroundResource(imageIds[i]);
			
			// 添加下面显示的小圆点，在切换时其显示状态也会相应变化
			ImageView point = new ImageView(this);
			point.setBackgroundResource(R.drawable.point_bg_selector);
			
			if(i==0)
			{
				point.setEnabled(true);
			}
			else
				point.setEnabled(false);
			
			LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(-2,-2);
			params.leftMargin = 10;
			point_container.addView(point,params);
			
			imageList.add(iv);
		}
		
		adapter = new MyPagerAdapter();
		viewPager.setAdapter(adapter);
		
		centerPosition = Integer.MAX_VALUE/2 - (Integer.MAX_VALUE/2 % imageList.size());
		// 设置初始化位置为 整数最大值的中间，并且要对view个数取模，使其默认从第一个开始，这样就可以将ViewPager两边都能滑动
		viewPager.setCurrentItem(centerPosition);
		
		viewPager.setOnPageChangeListener(new OnPageChangeListener() {
			
			@Override
			/**
			 * 当ViewPager滑动到下一个界面时调用
			 */
			public void onPageSelected(int position) {		
				
				position = position % imageList.size();
				
				tv_desc.setText(imageDescriptions[position]);
				// 将上一个点设置为未选中，使其为灰色
				point_container.getChildAt(lastPosition).setEnabled(false);
				// 当前显示的点设置为白色，为选中状态
				point_container.getChildAt(position).setEnabled(true);
				
				// 当前位置下标在赋值个上一个下标做记录
				lastPosition = position;
			}
			
			@Override
			/**
			 * 当ViewPager正在滑动的时候调用
			 */
			public void onPageScrolled(int position, float positionOffset,
					int positionOffsetPixels) {
				
			}
			
			@Override
			public void onPageScrollStateChanged(int state) {
				
			}
		});
		
		handler.sendEmptyMessageDelayed(11, 2000);
	}
	
	@Override
	protected void onDestroy() {
		// 取消handler消息的发送
		flag = false;
		
		super.onDestroy();
	}
	
	private MyPagerAdapter adapter;
	
	class MyPagerAdapter extends PagerAdapter{

		
		@Override
		public Object instantiateItem(ViewGroup container, int position) {
			// 下面的这句覆盖时要删除，否则在运行时会直接抛异常
	//			super.instantiateItem(container, position);
			System.out.println("instantiateItem:==========="+position);
			position = position % imageList.size();
			
			View iv = imageList.get(position);
			container.addView(iv);
			return iv;
		}

		@Override
		public void destroyItem(ViewGroup container, int position, Object object) {
			System.out.println("destroyItem:==========="+position);
			// 删除以前出现过的条目
			container.removeView((View)object);
		}

		@Override
		public int getCount() {
			return Integer.MAX_VALUE;
		}

		@Override
		public boolean isViewFromObject(View view, Object object) {
			
			return view == object;
		}
		
	}
	}

其中的小圆点是使用selector选择器定义的一个xml文件。

PS: 之所以可以将ViewPager的item设置的那么大，和ViewPager创建和销毁 item view有关，向上面的代码中，instantiateItem()和destroyItem()方法加上了log日志输入，ViewPager在创建和销毁item view的时候，最多维护3个view，即：当前view以及其前一个view，和后一个view，其他不管getCount()方法中返回了多大的值，都不会去直接都创建出来，也就是说，它的创建和销毁是动态的。所以设置为一个特别大的值，丝毫不会对它造成影响。


##### 往返循环的方式

> 往返循环的方式具体是通过判断当前viewPager所显示的item view所在的位置，根据它的位置和总共的view数量进行比较，进行边间判断，根据边界来判断是该向前移动还是向后移动。自动循环切换使用的也是handler中继续发送handler。具体实现如下：

	public class ViewPager2Activity extends Activity {

	private ViewPager viewpager;
	private TextView tv_desc;

	private LinearLayout point_container;

	private MyPageAdapter adapter;

	// 图片资源ID
	private final int[] imageIds = { R.drawable.a, R.drawable.b, R.drawable.c,
			R.drawable.d, R.drawable.e };

	// 图片标题集合
	private final String[] imageDescriptions = { "巩俐不低俗，我就不能低俗",
			"扑树又回来啦！再唱经典老歌引万人大合唱", "揭秘北京电影如何升级", "乐视网TV版大派送", "热血屌丝的反杀" };

	private List<ImageView> imageList;

	private int lastPosition = 0;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);

		setContentView(R.layout.view_page_layout2);

		viewpager = (ViewPager) findViewById(R.id.viewpager);
		tv_desc = (TextView) findViewById(R.id.tv_desc);
		tv_desc.setText(imageDescriptions[0]);

		point_container = (LinearLayout) findViewById(R.id.point_container);

		imageList = new ArrayList<ImageView>();

		for (int i = 0; i < imageIds.length; i++) {
			ImageView iv = new ImageView(this);
			iv.setBackgroundResource(imageIds[i]);

			imageList.add(iv);

			ImageView point = new ImageView(this);
			point.setBackgroundResource(R.drawable.point_bg_selector);

			if (i == 0) {
				point.setEnabled(true);
			} else
				point.setEnabled(false);

			LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(
					-2, -2);
			params.leftMargin = 10;
			point_container.addView(point, params);
		}

		adapter = new MyPageAdapter();

		viewpager.setAdapter(adapter);

		viewpager.setOnPageChangeListener(new OnPageChangeListener() {

			@Override
			/**
			 * 当ViewPager滚动到下一个界面以后，调用
			 */
			public void onPageSelected(int position) {

				tv_desc.setText(imageDescriptions[position]);

				point_container.getChildAt(lastPosition).setEnabled(false);
				point_container.getChildAt(position).setEnabled(true);

				lastPosition = position;
			}

			@Override
			public void onPageScrolled(int position, float positionOffset,
					int positionOffsetPixels) {
			}

			@Override
			public void onPageScrollStateChanged(int state) {
			}
		});
		// 发送一个延时的handler，进行viewPager切换
		handler.sendEmptyMessageDelayed(22, 2000);
	}

	// 判断当前是向后移动的情况，还是向前移动的情况
	private int nextOrPre = 0;
	private int currentPosition = 0;

	// 标记是否循环发送Handler消息，在Activity销毁时，将其设置为false，取消消息的发送
	private boolean flag = true;

	private Handler handler = new Handler() {
		public void handleMessage(android.os.Message msg) {
			
			if(flag)
			{				
				currentPosition = viewpager.getCurrentItem();
				// 向后移动
				if (nextOrPre == 0) {
					// 到达最后一张，需要向前移动
					if (currentPosition == imageList.size() - 1) {
						currentPosition--;
						nextOrPre = 1;
					} else
						currentPosition++;
				}
				// 向前移动
				else if (nextOrPre == 1) {
					if (currentPosition == 0) {
						currentPosition++;
						nextOrPre = 0;
					} else
						currentPosition--;
				}
				
				viewpager.setCurrentItem(currentPosition);
				// 继续发送一个同样的handler，达到循环调用的效果
				handler.sendEmptyMessageDelayed(22, 2000);
			}
		};
	};

	class MyPageAdapter extends PagerAdapter {

		@Override
		public int getCount() {
			return imageList.size();
		}

		@Override
		public boolean isViewFromObject(View view, Object object) {
			return view == object;
		}

		@Override
		public Object instantiateItem(ViewGroup container, int position) {

			View view = imageList.get(position);
			container.addView(view);
			return view;
		}

		@Override
		public void destroyItem(ViewGroup container, int position, Object object) {

			container.removeView((View) object);
		}

	}
	
	@Override
	protected void onDestroy() {
		// 取消handler的循环发送
		flag = false;
		super.onDestroy();
	}
	}

这种往返循环切换的方式也是比较实用的。


