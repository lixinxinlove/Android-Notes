### Fragment设置主题样式

> 有时希望为Fragment或者一个普通的view设置样式主题，这个时候可能会出现单独设置xml文件中的style属性并不是生效，可以使用下面的方式在填充view的时候，在代码中为其指定主题样式。

如下：

	@Override 

    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) { 

        // create ContextThemeWrapper from the original Activity Context with the custom theme 

        Context context = new ContextThemeWrapper(getActivity(),R.style.My_Custom_Theme); 

        // clone the inflater using the ContextThemeWrapper 

        LayoutInflater localInflater = inflater.cloneInContext(context); 

        // inflate using the cloned inflater, not the passed in default    

        return localInflater.inflate(R.layout.my_layout, container, false); 

    }