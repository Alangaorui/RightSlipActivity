# RightSlipActivity
做一个Activity的跳转，然后右滑finish跳转的界面

此处是为了自己的学习做笔记，如有侵权，请告知，必定删除


public class SlideBackLayout extends FrameLayout {

    private ViewGroup mDecorView;
    private View mRootView;
    private Activity mActivity;
    private ViewDragHelper mViewDragHelper;
    private float mSlideWidth;
    private int mScreenWidth;
    private int mScreenHeight;
    private Paint mPaint;
    private int curSlideX;


    public SlideBackLayout(Context context) {
        super(context);
        init(context);
    }

    private void init(Context context) {
        //必须是传入Activity
        mActivity = (Activity) context;
        //构造ViewDragHelper
        mViewDragHelper = ViewDragHelper.create(this, new DragCallback());
        //设置从左边缘捕捉View
        mViewDragHelper.setEdgeTrackingEnabled(ViewDragHelper.EDGE_LEFT);
        //初始化画笔
        mPaint = new Paint();
        mPaint.setStrokeWidth(2);
        mPaint.setAntiAlias(true);
        mPaint.setColor(Color.GRAY);
    }

    //绑定方法，在Activity的DecorView下插入当前ViewGroup,原来的RootView放于当前ViewGroup下
    public void bind() {
        mDecorView = (ViewGroup) mActivity.getWindow().getDecorView();
        mRootView = mDecorView.getChildAt(0);
        mDecorView.removeView(mRootView);
        this.addView(mRootView);
        mDecorView.addView(this);

        //计算屏幕宽度
        DisplayMetrics dm = new DisplayMetrics();
        mActivity.getWindowManager().getDefaultDisplay().getMetrics(dm);
        mScreenWidth = dm.widthPixels;
        mScreenHeight = dm.heightPixels;
        mSlideWidth = dm.widthPixels *0.5f;//滑动到一半的时候才消失
    }

    @Override
    public boolean onInterceptHoverEvent(MotionEvent event) {
        return mViewDragHelper.shouldInterceptTouchEvent(event);
    }

    /**
     * 触摸事件
     * @param event
     * @return
     */
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        mViewDragHelper.processTouchEvent(event);
        return true;
    }

    class DragCallback extends ViewDragHelper.Callback {

        @Override
        public boolean tryCaptureView(View child, int pointerId) {
            return false;
        }

        @Override
        public void onViewReleased(View releasedChild, float xvel, float yvel) {
            //当前回调，松开手时触发，比较触发条件和当前的滑动距离
            int left = releasedChild.getLeft();
            if (left <= mSlideWidth) {
                //缓慢滑动的方法,小于触发条件，滚回去
                mViewDragHelper.settleCapturedViewAt(0, 0);
            } else {
                //大于触发条件，滚出去...
                mViewDragHelper.settleCapturedViewAt(mScreenWidth, 0);
            }
            //需要手动调用更新界面的方法
            invalidate();

        }

        @Override
        public void onViewPositionChanged(View changedView, int left, int top, int dx, int dy) {
            curSlideX = left;
            //当滑动位置改变时，刷新View,绘制新的阴影位置
            invalidate();
            //当滚动位置到达屏幕最右边，则关掉Activity
            if (changedView == mRootView && left >= mScreenWidth) {
                mActivity.finish();
            }
        }

        @Override
        public int clampViewPositionHorizontal(View child, int left, int dx) {
            //限制左右拖拽的位移
            left = left >= 0 ? left : 0;
            return left;
        }

        @Override
        public int clampViewPositionVertical(View child, int top, int dy) {
            //上下不能移动，返回0
            return 0;
        }

        @Override
        public void onEdgeDragStarted(int edgeFlags, int pointerId) {
            //触发边缘时，主动捕捉mRootView
            mViewDragHelper.captureChildView(mRootView, pointerId);
        }
    }


    @Override
    public void computeScroll() {
        //使用settleCapturedViewAt方法是，必须重写computeScroll方法，传入true
        //持续滚动期间，不断刷新ViewGroup
        if (mViewDragHelper.continueSettling(true))
            invalidate();

    }

    @Override
    protected void dispatchDraw(Canvas canvas) {
        //进行阴影绘制,onDraw（）方法在ViewGroup中不一定会执行
        drawShadow(canvas);
        super.dispatchDraw(canvas);

    }


    private void drawShadow(Canvas canvas) {
        canvas.save();
        //构造一个渐变
        Shader mShader = new LinearGradient(curSlideX , 0, curSlideX, 0, new int[]{Color.parseColor("#eae7e8"), Color.parseColor("#eae7e8"), Color.parseColor("#eae7e8")}, null, Shader.TileMode.REPEAT);
        //设置着色器
        mPaint.setShader(mShader);
        //绘制时，注意向左边偏移
        RectF rectF = new RectF(curSlideX , 0, curSlideX, mScreenHeight);
        canvas.drawRect(rectF, mPaint);
        canvas.restore();
    }

}

主要是在里面做自定义的方法
在activity中添加
 mSlideBackLayout = new SlideBackLayout(this);
 mSlideBackLayout.bind();
 方法调用
 
 在AndroidManifest.xml中写上
 <activity android:name=".FristActivity" android:theme="@style/transparent" ></activity>
主要是做透明的效果，避免做闪屏出现



