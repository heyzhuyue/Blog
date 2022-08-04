# RecyclerView处理鼠标滑轮滑动事件

## RecyclerView处理鼠标滑轮滑动事件

> 项目进行中，遇到了RecyclerView的Item无法根据鼠标滑轮进行滑动问题，对于RecyclerView的滑动体验较差，因此撰写此博客分享RecyclerView鼠标滑轮事件解决方案。

1.  重写RecyclerView的onGenericMotionEvent事件，针对事件中的ACTION_SCROLL进行处理 
2.  通过event.getAxisValue(MotionEvent.AXIS_VSCROLL)获取设备屏幕相对位置，通过RecyclerViewy的scrollBy方法滑动RecyclerView位置,代码如下:  

```plain
 public class CustomRecyclerView extends RecyclerView {
 		private float mVerticalScrollFactor = 20.f;

 		public CustomRecyclerView(Context context) {
 			super(context);
 		}

 		public CustomRecyclerView(Context context, @Nullable AttributeSet attrs, int defStyle) {
 			super(context, attrs, defStyle);
 		}

 		 public CustomRecyclerView(Context context, @Nullable AttributeSet attrs) {
 			super(context, attrs);
 		}

 	 	@TargetApi(Build.VERSION_CODES.HONEYCOMB_MR1)
 	 	@Override
 			public boolean onGenericMotionEvent(MotionEvent event) {//鼠标滑轮滑动事件
				 if ((event.getSource() & InputDevice.SOURCE_CLASS_POINTER) != 0) {
    				 if (event.getAction() == MotionEvent.ACTION_SCROLL &&
           			  getScrollState() == SCROLL_STATE_IDLE) {//鼠标滑轮事件执行&&RecyclerView不是真正滑动
        				 final float vscroll = event.getAxisValue(MotionEvent.AXIS_VSCROLL);//获取轴线距离
         			if (vscroll != 0) {
            			 final int delta = -1 * (int) (vscroll * mVerticalScrollFactor);
            			 if (ViewCompat.canScrollVertically(this, delta > 0 ? 1 : -1)) {
               		  scrollBy(0, delta);
               		  return true;
             }
         }
     }
 }
 		return super.onGenericMotionEvent(event);
 	}
 }
```

## ListView嵌套ScrollView滑动事件冲突

项目进行中会遇到ScrollView嵌套ListView,ScrollView不可滑动情况,解决此问题的方法主要有:

1.  重写ListView的onMeasure方法让ListView高度达到最大，实现根据Item自适应效果,代码如下:  

```plain
 public class ListViewForScrollView extends ListView {
      public ListViewForScrollView(Context context) {
          super(context);
  }

     public ListViewForScrollView(Context context, AttributeSet attrs) {
          super(context, attrs);
  }

 public ListViewForScrollView(Context context, AttributeSet attrs,
                          int defStyle) {
     super(context, attrs, defStyle);
 }

  @Override
 /**
   * 重写该方法，达到使ListView适应ScrollView的效果
 */
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
      int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2,
         MeasureSpec.AT_MOST);
     super.onMeasure(widthMeasureSpec, expandSpec);
  }
     }
```