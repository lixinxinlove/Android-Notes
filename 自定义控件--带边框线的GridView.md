### 拥有边框线的GridView

> 有时在GridView中填充item，希望每个item像表格一样，有边框线，而且可能会要求最左边和最右边的item无左右边框线，但是GridView本身不提供边框线设置，所以在这里需要自定义view，并继承至GridView，在其dispatchDraw()方法中加上绘制边框线的逻辑.

已有的第三方实现图如下：想实现的就是类似与这样的效果

![](http://www.jcodecraeer.com/uploads/20131227/13881505449997.png)

重写的GridView如下：

	public class LineGridView extends GridView {
	public LineGridView(Context context) {
		super(context);
	}

	public LineGridView(Context context, AttributeSet attrs) {
		super(context, attrs);
	}

	public LineGridView(Context context, AttributeSet attrs, int defStyle) {
		super(context, attrs, defStyle);
	}

	@Override
	/**
	 * 重写dispatchDraw
	 * 在dispatchDraw方法中，我们对每一个子view的边界按照一定的方式绘上了边框，一般一个格子只需绘制其中两条边，
	 * 需要注意的是最边上的格子需要特殊处理。super.dispatchDraw(canvas);
	 * 一定要调用，不然格子中的内容（子view）就得不到绘制的机会
	 */
	protected void dispatchDraw(Canvas canvas) {
		// 必须调用super
		super.dispatchDraw(canvas);

		int childCount = getChildCount();
		if(childCount>0)
		{
			// 获取第一个item
			View itemView1 = getChildAt(0);
			// 用总的宽度 / 一个的宽度  得到总列数
			int column = getWidth() / itemView1.getWidth();

			// 创建一个Paint对象，用它画线
			Paint localPaint = new Paint();
			localPaint.setStyle(Paint.Style.STROKE);
			localPaint.setColor(Color.parseColor("#eeeeee"));
			// 遍历所有子view，对其进行画边框操作
			for (int i = 0; i < childCount; i++) {
				View cellView = getChildAt(i);
				// 如果view是最后一列上的item，则对其进行对其进行底边的绘制
				if ((i + 1) % column == 0) {
					canvas.drawLine(cellView.getLeft(), cellView.getBottom(),
							cellView.getRight(), cellView.getBottom(),
							localPaint);
				}
				// 其他view则需要绘制右边框和低边框
				else {
					canvas.drawLine(cellView.getRight(), cellView.getTop(),
							cellView.getRight(), cellView.getBottom(),
							localPaint);
					canvas.drawLine(cellView.getLeft(), cellView.getBottom(),
							cellView.getRight(), cellView.getBottom(),
							localPaint);
				}
			}
			// 如果最后一行的item不等于column，也就是说有空着的
			if (childCount % column != 0) {
				// 遍历后几个空的item，为其绘制右边框
				for (int j = 0; j < (column - childCount % column); j++) {
					View lastView = getChildAt(childCount - 1);
					canvas.drawLine(lastView.getRight() + lastView.getWidth()
							* j, lastView.getTop(), lastView.getRight()
							+ lastView.getWidth() * j, lastView.getBottom(),
							localPaint);
				}
			}
			
			// 在最后一行绘制一个一行的底部边框
			View lastView = getChildAt(childCount - 1);
			canvas.drawLine(0, lastView.getBottom(), lastView.getRight()
					* column, lastView.getBottom(), localPaint);
		}
	}
	}

这里需要说的是，如果想让最下面一行的边框线出现，需要在gridView对应的xml文件中，为其设置一个底部padding值，否则底部边框线不会出现，被遮挡了。出现的就是下图的效果：

![](http://www.jcodecraeer.com/uploads/20131227/13881504312676.png)

