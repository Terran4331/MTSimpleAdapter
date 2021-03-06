# MTSimpleAdapter
a Simple, powerful adapter for ListView,GridView or RecyclerView!

本文中的DEMO和library已上传到github: [https://github.com/devilthrone/MTSimpleAdapter](https://github.com/devilthrone/MTSimpleAdapter)

本文相关博客地址：[http://blog.csdn.net/a253664942/article/details/51170066]

欢迎fork and star O(∩_∩)O

**需求背景**
----
我们在Android开发中经常会遇到下列场景：一个ListView中需要展示多种不同的布局。例如：聊天界面、消息盒子等场景。

![这里写图片描述](http://img.blog.csdn.net/20160416163956835)


通常的实现方式是：

重写getViewTypeCount方法返回布局的类别数量   

重写getItemViewType(int）根据position返回不同的布局


这种写法存在很多的缺点。  

 缺点如下：
 
 1.需要定义多个变量，用来标志不同的布局类型：  TYPE_RECEIVE收到的消息  TYPE_SEND发送的消息等等；
 
2.如果布局很复杂，那么定义多个不同的ViewHolder来承载不同的布局。

3.在getView方法和getItemViewType方法中，存在多个if else判断语句，结构混乱

4.如果需要扩展，比如添加一种新布局，那么就需要重新定义一个ViewHolder ,在3中的两个方法里添加一个else分支。

综上所述，上面的传统方法缺少灵活性，不易扩展，每次添加新类型都需要修改源码，不符合开闭原则——对修改关闭，对扩展开放。
所以。噔噔噔噔！今天的主角粉墨登场！

**MTSimpleAdapter介绍**
-----------------
MTSimpleAdapter是我为了简化日常开发中 ListView GridView 和 RecyclerView多itemView场景开发流程，所编写的一个library.
该库使用起来简单、方便、高效，不仅封装了convertView缓存复用操作，还封装了ViewHolder加载的操作，极大地减少了开发时候的代码量。并且使用了非常规的模式来实现多模板构造功能，具有很强的扩展性、灵活性。

Demo中实现效果如下图：
![这里写图片描述](http://img.blog.csdn.net/20160416192303042)

**集成方式**
--------
1.直接copy源码~    

2.引用jar文件  MTSimpleAdapter.jar已在github项目中给出   

3.在Android studio中用gradle添加依赖,使用最新版本

> compile 'com.devilthrone:MTSimpleAdapter:+'


**使用方法**
--------

1.编写Provider类  实现ViewProdiver接口
 
在MTSimpleAdapter中，每一个Provider类对应着一种模板类型。   
 
ViewProvider中有两个方法：

> **bindView（Context context, ViewHolder viewHolder, int position, T item）**
为ItemView中的每一项填充数据。ViewHolder中封装了很多View的常用操作。
> 
> **getLayoutId()** 返回ItemView的布局

具体例子如下：

> ConversationProvider 实现Demo中会话的布局

```
public class ConversationProvider implements ViewProvider<ConversationBean> {
    @Override
    public void bindView(Context context, ViewHolder viewHolder, int position, ConversationBean item) {
            viewHolder.setText(R.id.tv_name,item.name)
                      .setText(R.id.tv_msg,item.msg)
                      .setText(R.id.tv_time,item.time);

    }
	//返回会话模板的布局
    @Override
    public int getLayoutId() {
        return R.layout.item_conversation_info;
    }
}

```

> HeadLineProvider 实现Demo中QQ新闻的布局

```
public class HeadLineProvider implements ViewProvider<HeadLineBean> {
    @Override
    public void bindView(Context context, ViewHolder viewHolder, int position, HeadLineBean item) {
        viewHolder.setText(R.id.tv_title,item.title)
                  .setText(R.id.tv_msg,item.msg)
                  .setText(R.id.tv_time,item.time);
    }
	//返回新闻头条的模板布局
    @Override
    public int getLayoutId() {
        return R.layout.item_headline_info;
    }
}
```

2. javaBean继承IItemBean接口
3. 
数据模型需要继承IItemBean接口，并实现getViewProviderClass()方法，返回当前模型所对应的Provider

> ConversationBean 会话数据模型

```
public class ConversationBean implements IItemBean {
    public String name;
    public String msg;
    public String time;

    public ConversationBean(String name, String msg, String time) {
        this.name = name;
        this.msg = msg;
        this.time = time;
    }

    @Override
    public String toString() {
        return "ConversationBean{" +
                "name='" + name + '\'' +
                ", msg='" + msg + '\'' +
                ", time='" + time + '\'' +
                '}';
    }

    @Override
    public Class<? extends ViewProvider> getViewProviderClass() {
        return ConversationProvider.class;
    }
}

```

> HeadLineBean 新闻头条数据模型

```
public class HeadLineBean implements IItemBean {
    public String title;
    public String msg;
    public String time;

    public HeadLineBean(  String title,String time,String msg) {
        this.msg = msg;
        this.time = time;
        this.title = title;
    }

    @Override
    public String toString() {
        return "HeadLineBean{" +
                "title='" + title + '\'' +
                ", msg='" + msg + '\'' +
                ", time='" + time + '\'' +
                '}';
    }

    @Override
    public Class<? extends ViewProvider> getViewProviderClass() {
        return HeadLineProvider.class;
    }
}

```

3. 在Activity中初始化Adapter

> ListView  or  GridView

```
ListViewAdapter mAdapter = new ListViewAdapter(this,mList);
        mAdapter.addProvider(HeadLineProvider.class);
        mAdapter.addProvider(ConversationProvider.class);
        mListView.setAdapter(mAdapter);
```

> RecyclerView

```
 RecyclerAdapter mAdapter = new RecyclerAdapter(this,mList);
        mAdapter.addProvider(HeadLineProvider.class);
        mAdapter.addProvider(ConversationProvider.class);
        mRecycleView.setLayoutManager(new LinearLayoutManager(RecycleActivity.this));
        // 设置RecyclerView的点击事件
        mAdapter.setOnItemClickListener(new RecyclerAdapter.OnItemClickListener() {
            @Override
            public void onItemClick(int position) {
                Toast.makeText(RecycleActivity.this, "Click item : "
                        + mAdapter.getItem(position), Toast.LENGTH_SHORT).show();
            }
        });
        mAdapter.setOnItemLongClickListener(new RecyclerAdapter.OnItemLongClickListener(){

            @Override
            public void onItemLongClick(int position) {
                Toast.makeText(RecycleActivity.this, "Long Click item : "
                        + mAdapter.getItem(position), Toast.LENGTH_SHORT).show();
            }
        });
        mRecycleView.setAdapter(mAdapter);
```
可以看见，想要添加一种新模板类型相当简单，只需要：

1）.定义相应的数据模型 实现IItemBean接口 ;

2）.定义一个Provider绑定数据并返回布局；

3）.使用adapter.addProvider方法添加该Provider即可。

扩展性非常强，以一种可插拔式的方式扩展或删除模板！


4.常用扩展特性：emptyView、loadingView、errorView
通常，我们在使用ListView和RecyclerView的时候，经常有这三方面的需求：

 1. 当正在加载数据时，显示一个loading页面

 2. 当无数据时，显示一个空页面
 
 3. 当获取数据网络访问出错，显示一个错误页面
 
 因此，基于上述三方面的需求，MTSimpleAdapter做了一些常用扩展，简化了ListView和RecyclerView的开发流程，大大减少了代码量。

**1. LoadingProvider(加载数据时的loading页面)**

	 （1）创建LoadingProvider
	 

```
public class LoadingProvider implements ViewProvider {
    @Override
    public void bindView(Context context, ViewHolder viewHolder, int position, IItemBean item) {

    }

    @Override
    public int getLayoutId() {
        return R.layout.item_loading;
    }
}
```
其中R.layout.item_loading是loading页面的布局，这里简单起见 只要一个progress和一个textView显示正在加载。。。

（2）在adapter中设置LoadingProvider

```
mAdapter.setLoadingProvider(LoadingProvider.class);
        mListView.setAdapter(mAdapter);
        mAdapter.setLoading(true);
        initData();
        mAdapter.setLoading(false);
```
通过ListViewAdapter和RecyclerAdapter的setLoadingProvider方法添加LoadingProvider，然后在获取数据前调用mAdapter.setLoading(true)来显示loading界面，在数据获取完成后调用mAdapter.setLoading(false)来隐藏loading界面即可。

  **2. EmptyProvider(数据为空时的empty页面)**
  
  （1）创建EmptyProvider
  

```
public class EmptyProvider implements ViewProvider {
    @Override
    public void bindView(Context context, ViewHolder viewHolder, int position, IItemBean item) {

    }

    @Override
    public int getLayoutId() {
        return R.layout.item_empty;
    }
}

```

 其中R.layout.item_empty是empty页面的布局

（2）在adapter中设置ErrorProvider

```
mAdapter.setEmptyProvider(EmptyProvider.class);
```
通过ListViewAdapter和RecyclerAdapter的setEmptyProvider方法添加EmptyProvider，当adapter中的数据为空时会自己显示empty页面。


5.其他主要方法
MTSimpleAdapter中还封装了一些其他方法，用来简化针对数据以及模板类别的操作：

> **addProvider**方法：添加一种Provider模板类别

> **removeProviderAndData**方法：删除一种Provider模板类别，并且同时删除该模板所对应的数据

> **addItem**方法：添加一条数据

> **addItems**方法：添加多条数据

> **addItemToHead**方法：添加一条数据到头部。可以用来实现HeadView

> **remove**方法：移除一条数据








 



