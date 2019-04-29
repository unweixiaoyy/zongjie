### ListView


### RecyclerView

### 局部Item更新和Item布局局部更新

```
局部Item更新:
notifyItemChanged(int position) & notifyItemRangeChanged(int positionStart, int itemCount)
结果：
position为2的ItemView重新执行了onCreateViewHolder -> onBindViewHolder(_,_,List<Obj> payload) -> onBindViewHolder(_,_)
插入、删除操作，更新操作还是更新指定位置Item等整个ViewHolder，但是比notifyAll效率高。

Item布局局部更新：
notifyItemChanged(int position, Object payload) & notifyItemRangeChanged(int positionStart, int itemCount, Object payload)
结果：onBindViewHolder(_,_,List<Obj> payload)，判断payload参数不为空更新Holder中itemView局部视图。
在局部更新Item基础上，只更新这个Holder的ItemView的某个控件，常见场景是Holder中有图片文字，仅更新文字不会使图片更新防止图片闪烁，也提升性能。
```

#### RecyclerView如何实现局部刷新Item？大致原理？

```
需要调用Adapter具有payload参数的方法notifyItemChanged(int position, Object payload)
需要实现RecyclerView.Adapter中具有参数payload的onBindViewHolder()
在布局流程中，会去调用具有payload参数的onBinderViewHolder
```


#### RecyclerView和ListView的缓存层级不同

```
ListView是两层、RecyclerView是四层
RecyclerView支持多个离屏ItemView缓存
RecyclerView支持开发者自定义缓存处理逻辑
RecyclerView支持所有RV公用同一个RecycklerViewPool(缓存池)
```

#### RecyclerView和AbsListView的区别和联系

```
缓存机制不同：RV四级缓存; AbsListView两级缓存。
定向刷新、局部刷新、刷新动画：只有RV支持
分割线：RV自定义样式; AbsListView样式单一
布局方式：RV自定义样式；列表/网格，切换麻烦
头尾添加：RV不支持；AbsListView支持
Item点击：RV不支持；AbsListView支持
```

#### RV的4级缓存
```
 // 1、连接着RV的所有的Scraped View 
 final ArrayList<RecyclerView.ViewHolder> mAttachedScrap = new ArrayList(); 
 // 2、数据源更改过的Attached View(连接着RV的View) 
 ArrayList<RecyclerView.ViewHolder> mChangedScrap = null; 
 // 3、缓存的所有View(包括可见和不可见的ViewHolder) 
 final ArrayList<RecyclerView.ViewHolder> mCachedViews = new ArrayList(); 
 // 4、RecycledViewPool：ViewHolder的缓存池，如果多个RV之间用setRecycledViewPool设置同一个RecycledViewPool，他们就可以共享Item。 RecyclerView.RecycledViewPool mRecyclerPool;
```

#### RecyclerView的最多缓存多少个？

```
在共享缓存池的情况下： N + 2 + 5*ViewType
屏幕内最多可以显示的Item数：N
屏幕外缓存：2个
缓存池1-被多个RecyclerView共享：5 * ViewType数量
缓存池2-没有被共享：线性布局-1个 * ViewType数量；网格布局-1个 * 行数
```

#### 四级缓存

```
RecyclerView具有四级缓存
	屏幕内缓存： 在屏幕中显示的ViewHolder。缓存到mChangedScrap和mAttachedScrap中。
		首次Layout时会生成Holder并保存这里，然后清空Layout并从这里取出来绘制，处理Item动画等。
	屏幕外缓存：列表滑动出屏幕时，ViewHolder会被缓存。缓存到mCachedViews中。
		刚刚被划出屏幕外等Holder，默认数量为2，可以设置数量。
	自定义缓存：自己实现ViewCacheExtension类来实现自定义缓存。
		系统预留的Holder缓存，默认空实现
	缓存池：屏幕外缓存的mCachedViews已满时，会将ViewHolder缓存到RecycledViewPool中。
		当mCachedViews存不下时会存到RVPool中，但它的大小是5，可以设置数量。
		不同的RV可以共同使用一个RVPool，RVPool存储的是key为ViewType的Map结构的缓存。
```

#### 屏幕内缓存
```
在屏幕中显示的ViewHolder，会进行缓存。
mChangedScrap: 缓存数据已经改变的ViewHolder。
mAttachedScrap: 缓存所有-Attached-Scrapped-View，连接着的-废弃的-View。
```

#### 屏幕外缓存
```
列表滑动出屏幕时，ViewHolder会被缓存。
mCachedViews: 进行屏幕外缓存，默认大小为2。大小由mViewCacheMax决定。DEFAULT_CACHE_SIZE = 2。
Recyclerview.setItemViewCacheSize(), 可以设置屏幕外缓存的大小。
```

#### RV自定义缓存
```
可以自己实现ViewCacheExtension类实现自定义缓存
可以通过Recyclerview.setViewCacheExtension()设置。
```


#### RV缓存
```
RecyclerView 滑动场景下的回收复用涉及到的结构体两个：
mCachedViews 和 RecyclerViewPool
mCachedViews 优先级高于 RecyclerViewPool，回收时，最新的 ViewHolder 都是往 mCachedViews 里放，如果它满了，那就移出一个扔到 ViewPool 里好空出位置来缓存最新的 ViewHolder。
复用时，也是先到 mCachedViews 里找 ViewHolder，但需要各种匹配条件，概括一下就是只有原来位置的卡位可以复用存在 mCachedViews 里的 ViewHolder，如果 mCachedViews 里没有，那么才去 ViewPool 里找。
在 ViewPool 里的 ViewHolder 都是跟全新的 ViewHolder 一样，只要 type 一样，有找到，就可以拿出来复用，重新绑定下数据即可。
```













