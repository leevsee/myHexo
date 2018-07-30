---
title: 'Android demo:下载进度条闪光效果（自定义View）—(by LineChen)'
date: 2016-10-03 23:44:49
tags:
---
十一感冒了，好伤啊。
这是带来的是Android 仿应用宝下载进度条（[by LineChen][1]）。

![此处输入图片的描述][2]


<br/>好啦，现在头晕，我在代码里面写的很详细了，就不多说了：<br/>


    public class DownBarView extends View implements Runnable{
        //SRC_ATOP :在源图像和目标图像相交的地方绘制源图像，在不相交的地方绘制目标图像：取下层绘制非交集部分。
        private PorterDuffXfermode xfermode = new PorterDuffXfermode(PorterDuff.Mode.SRC_ATOP);
    
        private float MAX_PROGRESS = 100f;
    
        private Paint mBgPaint;
    
        private Paint mTextPaint;
    
        /**
         * 文字矩形
         */
        private Rect mTextBouds;
    
        private Canvas mPgCanvas;
    
        private int mTextSize;
    
        /**
         * 初始下载中颜色
         */
        private int mLoadingColor;
    
        /**
         * 暂停颜色
         */
        private int mStopColor;
    
        /**
         * 进度中变换颜色
         */
        private int mProgressColor;
    
        /**
         * 闪光Bitmap
         */
        private Bitmap mFlikerBitmap;
    
        /**
         * 背景Bitmap
         */
        private Bitmap mPgBitmap;
    
        private float mFlikerLeft;
    
        private Thread mThread;
    
    
        /**
         * 当前进度条
         */
        private float progress;
    
        private boolean isStop;
    
        private boolean isFinish;
    
        private String mProgressText;
    
        public DownBarView(Context context) {
            this(context, null, 0);
        }
    
        public DownBarView(Context context, AttributeSet attrs) {
            this(context, attrs, 0);
        }
    
        public DownBarView(Context context, AttributeSet attrs, int defStyleAttr) {
            super(context, attrs, defStyleAttr);
            initAttrs(attrs);
        }
    
        /**
         * 设置arrs.xml的值
         *
         * @param attrs
         */
        private void initAttrs(AttributeSet attrs) {
            if (attrs != null) {
                TypedArray ta = getContext().obtainStyledAttributes(attrs, R.styleable.DownBarView);
                mTextSize = (int) ta.getDimension(R.styleable.DownBarView_textSize, 12);
                mLoadingColor = ta.getColor(R.styleable.DownBarView_loadingColor, Color.parseColor("#40c4ff"));
                mStopColor = ta.getColor(R.styleable.DownBarView_stopColor, Color.parseColor("#ff9800"));
                ta.recycle();//回收TypedArray
            }
        }
    
        /**
         * 初始化
         */
        private void inti() {
            //用于绘制时抗锯齿
            mBgPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
            mTextPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
            //设置字体大小
            mTextPaint.setTextSize(mTextSize);
            mTextBouds = new Rect();
            mProgressColor = mLoadingColor;
            //获得发光图片及宽度
            mFlikerBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.flicker);
            mFlikerLeft = -mFlikerBitmap.getWidth();
            //获取view的宽，高度
            mPgBitmap = Bitmap.createBitmap(getMeasuredWidth(), getMeasuredHeight(), Bitmap.Config.ARGB_8888);
            mPgCanvas = new Canvas(mPgBitmap);
            mThread = new Thread(this);
            mThread.start();
            Log.d("mFlikerLeft========================", "inti: " + mFlikerLeft);
        }
    
        
        @Override
        protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
            super.onMeasure(widthMeasureSpec, heightMeasureSpec);
            // 父容器传过来的高度方向上的模式
            int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
            // 父容器传过来的高度的值
            int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);
            // 父容器传过来的宽度的值
            int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
            int height = 0;
            switch (heightSpecMode) {
                //最大模式：就是父组件，能够给出的最大的空间，当前组件的长或宽最大只能为这么大，当然也可以比这个小。
                case MeasureSpec.AT_MOST:
                    height = (int) dp2px(35);
                    break;
                //精确模式：在这种模式下，尺寸的值是多少，那么这个组件的长或宽就是多少。
                case MeasureSpec.EXACTLY:
                //未指定：当前组件，可以随便用空间，不受限制
                case MeasureSpec.UNSPECIFIED:
                    height = heightSpecSize;
                    break;
            }
            setMeasuredDimension(widthSpecSize, height);
            inti();
        }
    
        
        @Override
        protected void onDraw(Canvas canvas) {
            super.onDraw(canvas);
            //进度条边框
            drawBorder(canvas);
    
            //进度条
            drawProgress();
    
            //背景图片
            canvas.drawBitmap(mPgBitmap, 0, 0, null);
    
            //进度条中的文字
            drawProgressText(canvas);
    
            //进度条文字变色处理
            drawColorProgressText(canvas);
        }
    
        /**
         * 绘制进度条的边框
         *
         * @param canvas
         */
        private void drawBorder(Canvas canvas) {
            //设置画出的图形是空心的
            mBgPaint.setStyle(Paint.Style.STROKE);
            mBgPaint.setColor(mProgressColor);
            mBgPaint.setStrokeWidth(dp2px(1));
            //左上右下，left和top是矩形的左上角的坐标，right和bottom是矩形的右下角的坐标，而paint则是画笔对象。而不是矩形的上下左右宽度等错误理解。
            canvas.drawRect(0, 0, getWidth(), getHeight(), mBgPaint);
    
        }
    
        /**
         * 绘制进度条
         */
        private void drawProgress() {
            //设置画出的图形是实心
            mBgPaint.setStyle(Paint.Style.FILL);
            mBgPaint.setStrokeWidth(0);
            mBgPaint.setColor(mProgressColor);
            //标志位没有保存相关的位移信息，restore的时候不能恢复
            mPgCanvas.save(Canvas.CLIP_SAVE_FLAG);
            //获得进度条在矩形的位置
            float right = (progress / MAX_PROGRESS) * getMeasuredWidth();
            //在当前进度条的背景
            mPgCanvas.clipRect(0, 0, right, getMeasuredHeight());
            mPgCanvas.drawColor(mProgressColor);
            //读取把上一次save();的状态，能够不断刷新进度条
            mPgCanvas.restore();
    
            //如果不停止的话，一直绘制闪光图片
            if (!isStop) {
                mBgPaint.setXfermode(xfermode);
                mPgCanvas.drawBitmap(mFlikerBitmap, mFlikerLeft, 0, mBgPaint);
                //清除混合模式
                mBgPaint.setXfermode(null);
            }
        }
    
        /**
         * 进度条文本
         *
         * @param canvas
         */
        private void drawProgressText(Canvas canvas) {
            mTextPaint.setColor(mProgressColor);
            //获得不同状态的文字
            mProgressText = getProgressText();
            //调用了getTextBounds()方法来获取到整个文字框的宽度和高度，并返回在Rect mTextBouds中
            mTextPaint.getTextBounds(mProgressText, 0, mProgressText.length(), mTextBouds);
            int tWidth = mTextBouds.width();
            int tHeight = mTextBouds.height();
            //设置文字框在整个进度条的中间，x,y坐标
            float xCoordinate = (getMeasuredWidth() - tWidth) / 2;
            float yCoordinate = (getMeasuredHeight() + tHeight) / 2;
            canvas.drawText(mProgressText, xCoordinate, yCoordinate, mTextPaint);
        }
    
        /**
         * 文字的变色处理
         *
         * @param canvas
         */
        private void drawColorProgressText(Canvas canvas) {
            mTextPaint.setColor(Color.WHITE);
            int tWidth = mTextBouds.width();
            int tHeight = mTextBouds.height();
            float xCoordinate = (getMeasuredWidth() - tWidth) / 2;
            float yCoordinate = (getMeasuredHeight() + tHeight) / 2;
            float progressWidth = (progress / MAX_PROGRESS) * getMeasuredWidth();
    
            //判断进度条是否大于文字
            if (progressWidth > xCoordinate) {
                canvas.save(Canvas.CLIP_SAVE_FLAG);
                float right = Math.min(progressWidth, xCoordinate + tWidth * 1.1f);
                canvas.clipRect(xCoordinate, 0, right, getMeasuredHeight());
                canvas.drawText(mProgressText, xCoordinate, yCoordinate, mTextPaint);
                canvas.restore();
            }
        }
    
        public void setProgress(float progress) {
            if (!isStop) {
                this.progress = progress;
                //请求重新onDraw()，但只会绘制调用者本身
                invalidate();
            }
        }
    
        /**
         * 是否停止
         * @param stop
         */
        public void setStop(boolean stop) {
            isStop = stop;
            if (isStop) {
                mProgressColor = mStopColor;
            } else {
                mProgressColor = mLoadingColor;
                mThread = new Thread(this);
                mThread.start();
            }
            invalidate();
        }
    
        /**
         * 切换状态 暂停/下载
         */
        public void toggle() {
            if (!isFinish) {
                if (isStop) {
                    setStop(false);
                } else {
                    setStop(true);
                }
            }
        }
    
        /**
         *
         */
        public void finishLoad() {
            isFinish = true;
            setStop(true);
        }
    
        /**
         * 判断是否完成
         */
        public boolean isFinish() {
            return isFinish;
        }
    
        /**
         * 判断是否停止
         */
        public boolean isStop(){
            return isStop;
        }
    
        @Override
        public void run() {
            int width = mFlikerBitmap.getWidth();
            while (!isStop) {
                //闪光图片每次刷新前进5dp
                mFlikerLeft += dp2px(5);
                float progressWidth = (progress / MAX_PROGRESS) * getMeasuredWidth();
                //一开始的时候，把闪光图片放在view的左边，负宽度
                if (mFlikerLeft >= progressWidth) {
                    mFlikerLeft = -width;
                }
                //因为run不在主线程，所以用postInvalidate();
                postInvalidate();
                try {
                    Thread.sleep(20);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    
    
        private String getProgressText() {
            String text = "";
            if (!isFinish) {
                if (!isStop) {
                    text = "下载中" + progress + "%";
                } else {
                    text = "继续下载";
                }
            } else {
                text = "下载完成";
            }
            return text;
        }
    
    
        //Paint中都是是px，所以要把dp转成px
        private float dp2px(int dp) {
            float density = getContext().getResources().getDisplayMetrics().density;
            return dp * density;
        }
    }


<br/>下面是Activity：<br/>

    public class DownBarActivity extends Activity {
    
        DownBarView mDownBarView;
        Button mButton;
        private Handler mHandler = new DownLoadHandler(this);
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_down_bar);
            mDownBarView = (DownBarView) findViewById(R.id.downbar);
            mButton = (Button) findViewById(R.id.bn1);
    
            mButton.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    //异步操作
                    new Thread(new Runnable() {
                        @Override
                        public void run() {
                            downLoad("http://imtt.dd.qq.com/16891/55D056A6FCC3E049B252DA1FE189677F.apk");
                        }
                    }).start();
                }
            });
        }

    
        private void downLoad(String apkUrl) {
            try {
                URL url = new URL(apkUrl);
                URLConnection urlConnection = url.openConnection();
                InputStream inputStream = urlConnection.getInputStream();
                //故获得要下载文件的大小
                int contentLength = urlConnection.getContentLength(); 
                String downloadFoldersName = Environment.getExternalStorageDirectory() + File.separator + "MyTestDemo" + File.separator;
                
                File file = new File(downloadFoldersName);
                if (!file.exists()) {
                    file.mkdir();
                }
    
                String fileName = downloadFoldersName + "test.apk";
                File apkFile = new File(fileName);
                if (apkFile.exists()) {
                    apkFile.delete();
                }
                
                int downloadSize = 0;
                byte[] bytes = new byte[1024];
                int length = 0;
    
                OutputStream outputStream = new FileOutputStream(fileName);
                //把进度值放入Message中
                while((length = inputStream.read(bytes)) != -1){
                    outputStream.write(bytes,0,length);
                    downloadSize += length;
                    int progress = downloadSize * 100 / contentLength;
                    Message message = mHandler.obtainMessage();
                    message.what = 0;
                    message.obj = progress;
                    mHandler.sendMessage(message);
                }
                inputStream.close();
                outputStream.close();
            } catch (MalformedURLException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
    
        }
    
        /**
        * 通过Handler获取Message中进度条的进度，并设置进度值
        */
        public static class DownLoadHandler extends Handler{
            //弱引用
            public final WeakReference<DownBarActivity> mDownBarActivityWeakReference;
    
            public DownLoadHandler(DownBarActivity downBarActivityWeakReference) {
                mDownBarActivityWeakReference = new WeakReference<>(downBarActivityWeakReference);
            }
    
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                DownBarActivity downBarActivity = mDownBarActivityWeakReference.get();
                switch (msg.what){
                    case 0:
                        int progress = (int) msg.obj;
                        downBarActivity.getDownBarView().setProgress(progress);
                        if (msg.arg1 == 100) {
                            downBarActivity.getDownBarView().finishLoad();
                        }
                }
            }
        }
        
    
        public DownBarView getDownBarView() {
            return mDownBarView;
        }
    }


这样就可以实现基本效果了，但是还没有暂停再次下载的效果，下次demo弄。~~ (>_<) ~~感冒好难受了<br/><br/>

PS：开发自定义View，通常可以被用户重写的方法如下：<br/>

>**构造器**：重写构造器是定制View的最基本方式，当Java代码创建一个View事例，或根于XML布局文件加载并构建界面时将需要调用构造器。

>**onFnishInflate()：**这是一个回调方法，当应用从XML布局文件加载该组件并利用它来构建界面之后，该方法将会被回调。

>**onMeasure(int,int)：**调用该方法来检测View组件及它所包含的所有子组件的大小。

>**onLayout(boolean,int,int,int,int)：**当该组件需要分配其自组件的位置、大小时，该方法就会被回调。

>**onSizeChanged(int,int,int,int)：**当该组件的大小被改变时回调该方法。

>**onDraw(Canvas)：**当该组件需要绘制它的内容时回调该方法进行绘制。

>**onKeyDown(int,KeyEvent)：**当某个键被按下时触发该方法。

>**onKeyUp(int,KeyEvent)：**当松开某个按键时触发该方法。

>**onTrackballEvent(MotionEvent)：**当发生轨迹球事件时触发该方法

>**onTouchEvent(MotionEvent)：**当发生触摸屏事件时触发该方法

>**onFocusChanged(boolean gainFocus,int direction,Rect previouslyFocusedRect):**当该组件焦点发生改变时触发该方法

>**onWindowFocusChanged(boolean)：**当该组件得到、失去焦点时触发该方法。

>**onAttachedToWindow()：**当把组件放入某个窗口时触发该方法

>**onDetachedFrowWindow()：**当把组件从某个窗口上分离时触发该方法。

>**onWindowVisibilityChanged(int)：**当包含该组件的窗口的可见性发生改变时触发该方法。<br/>


  [1]: https://github.com/LineChen/FlickerProgressBar/blob/master/README.md
  [2]: https://github.com/LineChen/FlikerProgressBar/raw/master/screenshot/screenshot.gif