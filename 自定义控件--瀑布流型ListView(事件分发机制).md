### 瀑布流型的ListView（事件分发机制）

> 实现了一个ViewGroup中包含3个ListView，这三个ListView均分该ViewGroup，里面填充不同的图片，同时实现滑动左边ListView所在的区域，只有左边的ListView响应滑动事件；滑动右边的ListView时，只有右边的ListView响应滑动事件；滑动中间的ListView时，当滑动的是中间ListView的上半部分则只有中间的ListView响应滑动事件，当滑动的是中间ListView的下半部分时，三个ListView一起响应滑动事件，同时上下滑动。

为了上三个ListView均分该ViewGroup，没有直接继承ViewGroup类，而是继承Linearlayout类，利用其layout_weight属性来实现均分。

#### 自定义的ViewGroup:


	public class WaterFallView extends LinearLayout {

	public WaterFallView(Context context, AttributeSet attrs) {
		super(context, attrs);
	}

	public WaterFallView(Context context) {
		super(context);
	}
	
	@Override
	/**
	 * 让该ViewGroup直接返回true，拦截所有的事件，如何分派，由下面的onTouchEvent方法中动态指定
	 */
	public boolean onInterceptTouchEvent(MotionEvent ev) {
		return true;
	}
	
	/**
	 * 记录Down事件时的点的坐标
	 */
	int pointX = 0 ;
	int pointY = 0;
	@Override
	public boolean onTouchEvent(MotionEvent ev) {
		int count = getChildCount();
		int width = getWidth() /count;
		int height = getHeight();
		
		/**
		 * 在Down事件发生的时候，就记录下触摸点的X和Y,根据值判断如何分发事件
		 */
		if(ev.getAction() == MotionEvent.ACTION_DOWN)
		{
			pointX = (int) ev.getX();
			pointY = (int) ev.getY();
		}
		if(pointX < width)	// 左边的ListView
		{
			// 让第一个ListView分派事件，其他不分派也就不会接收到触摸事件
			getChildAt(0).dispatchTouchEvent(ev);
			return true;
		}
		else if(pointX > width *2)	// 右边的ListView
		{
			getChildAt(2).dispatchTouchEvent(ev);			
			return true;
		}
		else if(pointX > width && pointX < width *2)		//中间的ListView
		{
			// 当前滑动的Y的坐标如果小于高度的一半，即滑动的是上半部分时，只让中间的ListView得到触摸事件
			if(pointY <= height / 2)		
			{
				getChildAt(1).dispatchTouchEvent(ev);
				return true;
			}
			else{// 滑动的是中间的ListVie的下半部分，此时，让3个ListView都得到触摸事件，达到一起滑动的效果
				for(int i = 0 ; i< count ; i++)
				{
					View view = getChildAt(i);
					try {
						view.dispatchTouchEvent(ev);
					} catch (Exception e) {
						e.printStackTrace();
					}
				}
				return true;
			}
		}
		
		return true;
	}
	}

上面需要注意的地方就是，onInterceptTouchEvent方法必须返回true，让该ViewGroup完全拦截事件的传播，如何分发放在该ViewGroup的onTouchEvent()中。在onTouchEvent()方法中，需要在Down事件时，就记录X和Y的值，然后根据按下这个点的X、Y来确定由哪个ListView去分发该事件。

#### 布局文件中使用

	<?xml version="1.0" encoding="utf-8"?>
	<cn.qing.selfview.view.WaterFallView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal" >
    
    <!-- 定义3个ListView使用权重均分宽度 -->
    <ListView 
        android:id="@+id/lv1"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:scrollbars="none"
        />
    <ListView 
        android:id="@+id/lv2"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:scrollbars="none"
        />
    <ListView 
        android:id="@+id/lv3"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:scrollbars="none"
        />

	</cn.qing.selfview.view.WaterFallView>

#### 在Activity中的使用

> 在Activity中使用就很简单了，为3个ListView填充图片数据

	public class WaterFallActivity extends Activity {

	private int[] images = {R.drawable.pre1,R.drawable.pre2,R.drawable.pre3,R.drawable.pre4,R.drawable.pre5,
							R.drawable.pre6,R.drawable.pre7,R.drawable.pre8,R.drawable.pre9,R.drawable.pre10,
							R.drawable.pre11,R.drawable.pre12,R.drawable.pre13,R.drawable.pre14,R.drawable.pre15,};	
	private ListView lv1;
	private ListView lv2;
	private ListView lv3;
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		
		setContentView(R.layout.wall_fall_layout);
		
		lv1 = (ListView) findViewById(R.id.lv1);
		lv2 = (ListView) findViewById(R.id.lv2);
		lv3 = (ListView) findViewById(R.id.lv3);		
		
		lv1.setAdapter(new MyAdapter());
		lv2.setAdapter(new MyAdapter());
		lv3.setAdapter(new MyAdapter());		
	}
	
	class MyAdapter extends BaseAdapter {

		@Override
		public int getCount() {
			return 100;
		}

		@Override
		public Object getItem(int position) {
			return position;
		}

		@Override
		public long getItemId(int position) {
			return 0;
		}

		@Override
		public View getView(int position, View convertView, ViewGroup parent) {

			ImageView iv = (ImageView) View.inflate(getApplicationContext(),
					R.layout.fall_item, null);
			int resId = (int) (Math.random() * images.length);
			iv.setImageResource(images[resId]);
			return iv;
		}
	}
	}