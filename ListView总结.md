### ListView使用总结

####配置文件中相关属性

* android:fastScrollEnabled="true"
 
	> 这个属性是用来设置ListView是否可以快速滚动，一旦设置为true，ListView的滚动条会出现一个可以往下拖动，来实现快速向下滚动的效果。
* android:transcriptMode="alwaysScroll"

	> 设置ListView的滚动模式，有三个值可以设置：
	* disabled  默认值，关闭滚动模式
	* normal 当ListView收到数据改变通知时，如果最后一个item可见，那么ListView就滚动到最下面。
	* alwaysScroll  当ListView收到数据变化通知时，不管ListView是否位于最后一个Item，即：不管当前ListView显示到了哪个Item,它都会立刻滚动到最底部。
	
	>

* android:divider="#1D364C"

	> 设置ListView的分割线，可以是一个颜色值，也可以是一个Drawable资源。
	> 注意：当不想让ListView显示出分割线时，只需要将这个属性的值设置为：@null  即可达到效果。

* android:dividerHeight="3dp"

	> 设置分割线的高度


####代码使用中相关设置

> 在使用ListView时为了提高ListView的效率，通常会使用缓存Item，即复用getView()方法中的convertView对象，同时也会定义一个包含显示组件的包装类：ViewHolder。这种是使用ListView时最基本也是最常用的做法。但是这些不能满足一些复杂的界面显示需求，当ListView显示不同的Item时，就需要在以上说的基础上，再进行升级使用。

##### 自定义缓冲区，缓存Item，并根据数据类型不同显示不同的View作为List的Item

> 目前的需求是想在ListView中，加载不同样式的Item，显示哪种样式会在数据Bean中使用一个Flag来标识出来。正是由于加载的item都不一样，在复用缓存Item时就会出现问题，出现了item乱跳，甚至下面的覆盖上面的item的情况。主要就是由于在复用getView方法的convertView对象时存在问题，没能正确的为后面即将显示的item加载正确的View，所以在下面的实现中，定义了一个简单的Map缓存集合，专门用来缓存convertView，并在后面使用缓存的item。

以下是实现的主要代码：

	
	class ViewHolder {
		TextView tv_left_msg, tv_rgiht_msg, tv_item3_info;
		// 标记不同item的类型，和数据bean中的flag一致
		int flag;
	}

	/**
	 * 定义一个缓冲区，来存放ListView的缓存Item,
	 * 这里需要说明的是，key：作为不同数据类型的flag存在，value：作为一个List集合，存放对应的那种flag值在ListView滚动时所产生的所有缓存Item的集合
	 * 这样在后续使用缓存时，就可以从这个List集合中取一个View作为要显示的item,在从List中获取缓存的View时，必须判断该item是否还在ListView中显示，
	 * 如果还显示则不能使用其作为缓存View，因为这样会出现item乱跳的情况，就是因为在下面使用了上面还在显示的item View
	 */
	private Map<Integer,List<View>> bufferViewMap = new HashMap<Integer, List<View>>();

	@Override
		public View getView(int position, View convertView, ViewGroup parent) {

			DifferentItemInfo di = dList.get(position);
			ViewHolder holder = null;
			View view = null;
			// 获取标记，后面的Item显示什么样的View，就由flag来进行区分
			int flag = di.getFlag();

			//如果convertView为null，就根据不同的flag创建出该显示的View
			if (convertView == null) {
				view = createNewView(flag);				
				
			} else {	//如果convertView不为null，存在缓存Item时
				
				// 获取缓存convertView中已经存放的ViewHolder
				ViewHolder vh  = (ViewHolder) convertView.getTag();
				// 从ViewHolder中获取视图flag
				int tagFlag = vh.flag;
				// 判断自定义的缓存Map中是否已经存放过key = tagFlag的数据
				if(!bufferViewMap.containsKey(tagFlag))
				{
					// 如果不存在，则根据tagFlag作为key ,创建一个集合存放缓存convertView
					List<View> list = new ArrayList<View>();
					list.add(convertView);
					bufferViewMap.put(tagFlag, list);					
				}
				else
				{
					// Map中包含tagFlag作为key创建的数据，就直接将当前这个缓存item convertView加入到已存在的List集合中
					bufferViewMap.get(tagFlag).add(convertView);
				}
				
				View v = null;
				
				// 根据目前的数据bean中的flag值，从map中取对应的缓存View集合
				List<View> list = bufferViewMap.get(flag);
				if(list!=null && list.size()>0)
				{					
					// 如果缓存集合不为null,就遍历这个集合，找出一个没有在当前ListView中显示出来的Item，作为要使用的View
					for (View bufView : list) {
						if(!bufView.isShown())
						{
							v = bufView;
							break;
						}
					}
				}
				if(v != null)
				{
					System.out.println("使用的是buffer中的view");
					// 从缓存中找到可用的View以后，直接赋值给getView方法要返回的View
					view = v;
				}
				else
				{
					System.out.println("新创建的view");
					// 如果缓存中没有找到，则需要重新创建一个view
					view = createNewView(flag);
				}
			}
			
			
			holder = (ViewHolder) view.getTag();
			switch (holder.flag) {
			case 1:				
				holder.tv_left_msg.setText(di.getMessage());
				break;
			case 2:				
				holder.tv_rgiht_msg.setText(di.getMessage());
				break;
			case 3:				
				holder.tv_item3_info.setText(di.getMessage());
				break;
			}

			return view;
		}
		
		/**
		 * 当没用缓存item能使用时，根据flag来创建出不同样式的view
		 * @param flag
		 * @return
		 */
		public View createNewView(int flag)
		{
			ViewHolder holder = new ViewHolder();
			View view = null;
			// 需要加载Item1
			if (flag == 1) {
				view = getLayoutInflater().inflate(
						R.layout.different_item1, null);

				holder.tv_left_msg = (TextView) view.findViewById(R.id.tv_left_msg);
				
			}
			// 需要加载Item2
			else if(flag == 2)
			{
				view = getLayoutInflater().inflate(
						R.layout.different_item2, null);
				holder.tv_rgiht_msg = (TextView) view.findViewById(R.id.tv_right_msg);
			}
			// 需要加载Item3
			else if(flag == 3)
			{
				view = getLayoutInflater().inflate(
						R.layout.different_item3, null);
				holder.tv_item3_info = (TextView) view.findViewById(R.id.tv_item3_info);
			}
			
			holder.flag = flag;
			view.setTag(holder);
			return view;
		}

以上使用一个简单的Map集合来缓存可复用的convertView,就解决了ListView中出现的乱跳或者下面item覆盖上面item的情况，这样的思想可以借鉴，代码是否完美有待以后完善。

