https://developer.android.com/training/animation/vp2-migration?hl=zh-cn

https://juejin.cn/post/6844904050140381191

https://www.jianshu.com/p/924046eae137

https://juejin.cn/post/6844904020553760782

# viewpager

通过创建adapter填充多个view，左右滑动时，切换不同的view。（官方建议使用Fragment填充，方便page的生命周期管理）

```java
<android.support.v4.view.ViewPager
    android:id="@+id/vp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
</android.support.v4.view.ViewPager>


public class MyPagerAdapter extends PagerAdapter {
    private Context mContext;
    private List<String> mData; 
    public MyPagerAdapter(Context context ,List<String> list) {
        mContext = context;
        mData = list;
    } 
    @Override
    public int getCount() {
        return mData.size();
    } 
    ////对显示的资源进行初始化
    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        View view = View.inflate(mContext, R.layout.item_base,null);
        TextView tv = (TextView) view.findViewById(R.id.tv);
        tv.setText(mData.get(position));
        container.addView(view);
        return view;
    }
	//对超出范围的资源进行销毁
    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        // super.destroyItem(container,position,object); 这一句要删除，否则报错
        container.removeView((View)object);
    }

    @Override
    public boolean isViewFromObject(View view, Object object) {
        return view == object;
    }
}


private void setVp() {
    List<String> list = new ArrayList<>();
    for (int i = 0; i < 3; i++) {
       list.add("第"+i+"个View");
    } 
    ViewPager vp = (ViewPager) findViewById(R.id.vp);
    vp.setAdapter(new MyPagerAdapter(this,list));
    //翻页动画    ZoomOutPageTransformer
    vp.setPageTransformer(false,new DepthPageTransformer());
    //翻页监听
    //SCROLL_STATE_IDLE：空闲状态
	//SCROLL_STATE_DRAGGING：滑动状态
	//SCROLL_STATE_SETTLING：滑动后滑翔的状态
    vp.OnPageChangeListener(...)
}

//viewpager+fragment
vp.setAdapter(new FragmentStatePagerAdapter(getSupportFragmentManager(), BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT) {
        @Override
        public Fragment getItem(int position) {
            return list.get(position);
        }

        @Override
        public int getCount() {
            return list.size();
});

```

给Viewpager设置标题栏有一下几种方式：

- PagerTabStrip： 带有下划线
- PagerTitleStrip： 不带下划线
- TabLayout：5.0后推出



FragmentPagerAdapter 与FragmentStatePagerAdapter区别（都继承PagerAdapter）

FragmentStatePagerAdapter（分页浏览大量或未知数量的 Fragment 时）：**页面实例会被移除。**会销毁不需要的fragment（remove()）。“state”表明：在销毁fragment时，可在onSaveInstanceState(Bundle)方法中保存fragment的Bundle信息。用户切换回来时，保存的实例状态可用来恢复生成新的fragment

FragmentPagerAdapter（分页浏览固定数量的较少 Fragment 时）：**页面实例一直保存。**对于不再需要的fragment， 调用事务的detach(Fragment)方法来处理它。只是销毁了fragment的视图， fragment实例还保留在FragmentManager中。

懒加载的三个判断：

1. 是否为当前页面（是否可见）
2. 是否已经加载过了
3. 视图是否初始化完成（setUserVisibleHint()的调用在onCreateView之前！）

```kotlin
    var hasLoad = false
    var isViewInitiated = false
    fun loadData() {
        if (userVisibleHint && !hasLoad && isViewInitiated) {
            // load
            hasLoad = true
        }
    }

    override fun setUserVisibleHint(isVisibleToUser: Boolean) {
        super.setUserVisibleHint(isVisibleToUser)
        loadData()
    }

    override fun onActivityCreated(savedInstanceState: Bundle?) {
        super.onActivityCreated(savedInstanceState)
        isViewInitiated = true
        loadData()
   }
    override fun onDestroyView() {
        super.onDestroyView()
        // 注意考虑View被销毁，而Fragment对象还在
        isViewInitiated = false 
    }



/**
* 
* Fragment: setMaxLifecycle 淘汰了 setUserVisibleHint.  直接设置生命周期
**/
var hasLoad = false

fun loadData() {
    if (hasLoad) {
        // load
        hasLoad = true
    }
}

override fun onResume() {
    super.onResume()
    loadData()
}
```







# viewpager2

`ViewPager2` 在 RecyclerView 的基础上构建而成。

对之前的 [ViewPager](https://developer.android.com/reference/androidx/viewpager/widget/ViewPager) 实现的改进：

- RTL（从右向左）布局支持

  ```xml
   android:layoutDirection="rtl"
  ```

- 垂直方向支持

  ```xml
  android:orientation="vertical"
  ```

- 可靠的 `Fragment` 支持（包括处理底层 `Fragment` 集合的更改）

  调用 notifyDatasetChanged() 来更新界面。

- 数据集更改动画（包括 `DiffUtil` 支持）

  利用 `RecyclerView` 类中的数据集更改动画

**API 变更**

- `FragmentStateAdapter` 取代了 `FragmentStatePagerAdapter`

  用于分页浏览 Fragment。

- `RecyclerView.Adapter` 取代了 `PagerAdapter`

  用于分页浏览视图。

- `registerOnPageChangeCallback` 取代了 `addPageChangeListener`

  监听页面变化。

嵌套的可滚动 ： 对 ViewPager2 对象调用 requestDisallowInterceptTouchEvent()



```kotlin
dependencies {
    implementation "androidx.viewpager2:viewpager2:1.0.0"
}

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tab_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/pager"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

</LinearLayout>


viewPager.setPageTransformer(ZoomOutPageTransformer())
viewPager.adapter = demoCollectionAdapter
//tabLayout与viewPager关联
TabLayoutMediator(tabLayout, viewPager) { tab, position ->
            tab.text = "OBJECT ${(position + 1)}"
        }.attach()

class DemoCollectionAdapter(fragment: Fragment) : FragmentStateAdapter(fragment) { 
    override fun getItemCount(): Int = 100

    override fun createFragment(position: Int): Fragment {
        // Return a NEW fragment instance in createFragment(int)
        val fragment = DemoObjectFragment()
        fragment.arguments = Bundle().apply {
            // Our object is just an integer :-P
            putInt(ARG_OBJECT, position + 1)
        }
        return fragment
    }
}
//返回
override fun onBackPressed() {
    if (viewPager.currentItem == 0) {
        super.onBackPressed()
    } else { 
        viewPager.currentItem = viewPager.currentItem - 1
    }
}
```

