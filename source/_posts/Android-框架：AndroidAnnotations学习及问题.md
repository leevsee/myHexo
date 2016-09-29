---
title: Android-框架：AndroidAnnotations学习及出现问题
date: 2016-09-28 23:40:14
tags:
---
这个AndroidAnnotations很多人说好用，不过个人认为上手比较难，尤其是在一些注释上，本次介绍AndroidAnnotations在AndroidStudio配置，以及简单使用。


## 1.gradle配置
在全局的build.gradle上添加：

    buildscript {
        repositories {
            mavenCentral()
        }
            
        dependencies {
            // 当前自己使用的gradle版本
            classpath 'com.android.tools.build:gradle:2.1.3'
            // 当前android-apt plugin的版本
            classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
        }
    }
    
    repositories {
        mavenCentral()
        mavenLocal()
    }
    
<br/>
　在局部gradle，也就是Module:app的gradle，添加：

    apply plugin: 'com.android.application'
    apply plugin: 'android-apt'
    def AAVersion = '4.1.0'     //本次使用的AndroidAnnotations为4.1.0
    
    dependencies {
        apt "org.androidannotations:androidannotations:$AAVersion"
    compile "org.androidannotations:androidannotations-api:$AAVersion"
    }
    
    apt {
        arguments {
            //查找解析androidManifestFile文件
            // 在gradle版本2.21以前写这个：androidManifestFile variant.processResources.manifestFile
            androidManifestFile variant.outputs[0]?.processResources?.manifestFile
            //你自己的包名
            resourcePackageName 'com.leeves.myandroidannotations'
        }
    }

　这样子就配置好了，点`Sync Now`,同步gradle即可。

## 2.使用方法
直接上demo吧，在demo里面容易看懂：

    //省去写onCreate方法中的 setContentView
    @EActivity(R.layout.activity_annotations)
    public class AndroidAnnotationsDemo extends Activity {

        //如果变量名字和EditText控件名字相同，不用写R.id.mEditText
        @ViewById
        EditText mEditText;
        
        //这里不同，所以要写
        @ViewById(R.id.iv)
        ImageView mImageView;
        
        //button直接写点击事件
        @Click(R.id.bn1)
        void setPictureUrl() {
            getPictureUrl(mEditText.getText().toString());
        }

        //后台线程处理网络请求
        @Background
        void getPictureUrl(String pUrl) {
            try {
                URL url = new URL(pUrl);
                InputStream is = url.openStream();
                Bitmap bitmap = BitmapFactory.decodeStream(is);
                is.close();
                showPicture(bitmap);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        
        //后台处理完后，在UI线程上显示
        @UiThread
        void showPicture(Bitmap bitmap){
            mImageView.setImageBitmap(bitmap);
        }
    }
<br/>
　　简单的使用呢，就是`@EActivity`绑定layout、`@ViewById`绑定控件、`@Click`绑定点击事件、`@Background`后台处理、`@UiThread`UI主线程处理，本来想写个ListView的demo，下次吧，下次再学习深入一些AndroidAnnotations，还是很强大啊，就是上手难。如果单单是为了不用写findViewById的话，那就用Butterknife或者LayoutCreator就行了。
　　

 - 出现问题及注意事项
 
　　1.如果AS出现这个错误：`Only the original thread that created a view hierarchy can touch its views`。
　　答：这个其实是你`@UiThread`导入错误，不是导入：`android.support.annotation.UiThread`，是导入这个：`import org.androidannotations.annotations.UiThread`。
　　2.上面忘了说，在`AndroidManifest.xml`声明这个Activity的时候，要加下划线：`<activity android:name=".AndroidAnnotationsDemo_"/>`，这时候AS会显示红色，编译一下就没事了。
　　3.AS出现这个错误：`Unable to find explicit activity class{...}have you declared this activity in your AndroidManifest.xml?`
　　答：跳转到该activity时，也要加下划线：`startActivity(new Intent(MainActivity.this,AndroidAnnotationsDemo_.class))`