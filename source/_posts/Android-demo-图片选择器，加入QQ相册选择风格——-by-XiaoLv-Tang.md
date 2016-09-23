---
title: 'Android demo:图片选择器，加入QQ相册选择风格——(by XiaoLv Tang)'
date: 2016-09-23 16:50:10
tags:
---
本demo源码来着于[XiaoLv Tang][1]，使用软件android studio

以下是这个demo的效果
![选择图片界面][2] ![选择后界面][3]



<br/><br/><br/>

下面是使用方法
--

<br/>build.gradle加入

    dependencies {
    compile 'com.library.tangxiaolv:telegramgallery:1.0.1'
    }

<br/>AndroidManifest.xml配置

    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    
    <activity android:name="com.tangxiaolv.telegramgallery.GalleryActivity"/>

<br/>Activit

    public class TelegramGalleryAcitity extends Activity {

    private List<String> phones;
    private BaseAdapter mBaseAdapter;
    private LayoutInflater mLayoutInflater;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test_gallery);
        mLayoutInflater = (LayoutInflater) getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        Button bn1 = (Button) findViewById(R.id.btn1);
        GridView gv = (GridView) findViewById(R.id.gv);
        gv.setAdapter(mBaseAdapter = new BaseAdapter() {
            @Override
            public int getCount() {
                return phones == null ? 0 : phones.size();
            }

            @Override
            public Object getItem(int position) {
                return phones == null ? null : phones.get(position);
            }

            @Override
            public long getItemId(int position) {
                return position;
            }

            @Override
            public View getView(int position, View convertView, ViewGroup parent) {
                ViewHolder viewHolder;
                if (convertView == null) {
                    convertView = mLayoutInflater.inflate(R.layout.item_image, null);
                    viewHolder = new ViewHolder();
                    viewHolder.mImageView = (ImageView) convertView.findViewById(R.id.iv);
                    convertView.setTag(viewHolder);
                } else {
                    viewHolder = (ViewHolder) convertView.getTag();
                }
                String photoPath = (String) getItem(position);
                BitmapFactory.Options options = new BitmapFactory.Options();
                options.inPreferredConfig = Bitmap.Config.ARGB_4444;//压缩图片
                options.inSampleSize = 4;
                viewHolder.mImageView.setImageBitmap(BitmapFactory.decodeFile(photoPath, options));
                return convertView;
            }
        });

        bn1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                GalleryActivity.openActivity(TelegramGalleryAcitity.this, false, 9, 10);
            }
        });
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (10 == requestCode && resultCode == Activity.RESULT_OK) {
            phones = (List<String>) data.getSerializableExtra(GalleryActivity.PHOTOS); //获取图片资源路径集合返回值(返回List<String>
            mBaseAdapter.notifyDataSetChanged();
        }
    }

    class ViewHolder {
        ImageView mImageView;
      }
    }

<br/>layout

        <GridView
            android:id="@+id/gv"
            android:numColumns="3"
            android:verticalSpacing="4dp"
            android:horizontalSpacing="4dp"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>
        
        <ImageView
            android:id="@+id/iv"
            android:scaleType = "centerCrop"
            android:layout_width="110dp"
            android:layout_height="110dp"
        />
        

<br/>好啦，自此结束，注释得少，如果有误请留言指出，3Q


  [1]: https://github.com/TangXiaoLv/TelegramGallery
  [2]: http://7xz8pr.com1.z1.glb.clouddn.com/picture_selector1.jpg
  [3]: http://7xz8pr.com1.z1.glb.clouddn.com/picture_selector2.jpg