---
title: Android 框架：Retrofit简单使用（一）
date: 2016-09-25 23:36:29
tags:
---



最近在学习Retrofit网络框架，首先是请求API，已经Json处理，以下是学习总结：<br/>
　　本次的demo是使用百度IP地址查询API，因为百度API需要在`header`传入`apikey`这个参数<br/>
　　
![API参数][1]
<br/>
　　请求后返回的结果：

    {
        "errNum": 0,
        "errMsg": "success",
        "retData": {
            "ip": "117.89.35.58",
            "country": "中国",
            "province": "江苏",
            "city": "南京",
            "district": "玄武",
            "carrier": "中国电信"
        }
    }

　　自己实践的话可以去看百度IP地址查询的[API][2]，下面一步一步来对比上代码！<br/>
　　


### 1.根据[Retrofit官方文档][3]首先写一个业务接口

    public interface GitHubService {   
        @GET("users/{user}/repos")   
        Call<List<Repo>> listRepos(@Path("user") String user); 
    }

　　根据自己需求写一个业务接口

    public interface IPQueryService {
        @GET("apistore/iplookupservice/iplookup")
        Call<String> getSearchIPData(@Query("ip") String iPaddress, @Header("apikey") String apikey);
    }

　　这里因为返回的Json数据不多，所以采用`String`，而不是`List<Repo>`，`@Query`表示需要传入的参数`ip`，`@Header`表示需要在`header`传入的参数a`pikey`。
<br/><br/>
### 2.使用Retrofit实现业务接口

       Retrofit retrofit = new Retrofit.Builder()     
                .baseUrl("https://api.github.com/")     
                .build();
    
       GitHubService service = retrofit.create(GitHubService.class);

本次的demo如下：

        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://apis.baidu.com/")
                .addConverterFactory(ScalarsConverterFactory.create())  //转换成String
                .build();
    
        IPQueryService service = retrofit.create(IPQueryService.class);//Java的动态代理模式
<br/><br/>

### 3.使用Call（来自于上一步create的接口），可以用来做同步或者异步操作。

        Call<List<Repo>> repos = service.listRepos("octocat");

　　 这里我传入参数，使用了异步操作，并在主UI中显示

        Call<String> call = service.getSearchIPData("117.89.35.58","38dc05aa7caf3e7d040b619c8184ce8");
    
        call.enqueue(new Callback<String>() {
                         @Override
                         public void onResponse(Call<String> call, Response<String> response) {
                             try {
                                 JSONObject dataJson = new JSONObject(response.body()); 
                                 JSONObject info = dataJson.getJSONObject("retData");
                                 mTextView.setText(info.getString("province"));//可以直接显示在UI上
                             } catch (JSONException e) {
                                 e.printStackTrace();
                             }
                         }
    
                         @Override
                         public void onFailure(Call<String> call, Throwable t) {
                         }
            })
<br/>


​    

 - 好啦，根据官方文档，对比着来写，应该都明白了吧。下面放出整个代码:<br/>
<br/>

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_api_request);
            protected void onCreate(Bundle savedInstanceState) {
            mTextView = (TextView) findViewById(R.id.response_data_textView);
    
            Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://apis.baidu.com/")
                .addConverterFactory(ScalarsConverterFactory.create())  //转换成String
                .build();
    
            IPQueryService service = retrofit.create(IPQueryService.class);//Java的动态代理模式
    
            Call<String> call = service.getSearchIPData("117.89.35.58","38dc05aa7caf3e7d040b619c8184ce8");
    
            call.enqueue(new Callback<String>() {
                         @Override
                         public void onResponse(Call<String> call, Response<String> response) {
                             try {
                                 JSONObject dataJson = new JSONObject(response.body());
                                 JSONObject info = dataJson.getJSONObject("retData");
                                 mTextView.setText(info.getString("province"));//可以直接显示在UI上
                                 Log.i("根据IP地址获取到的国家是：",info.getString("country"));
                             } catch (JSONException e) {
                                 e.printStackTrace();
                             }
                         }
    
                         @Override
                         public void onFailure(Call<String> call, Throwable t) {
                         }
                    }
                );
        }
    
        public interface IPQueryService {
            @GET("apistore/iplookupservice/iplookup")
            Call<String> getSearchIPData(@Query("ip") String iPaddress, @Header("apikey") String apikey);
        }


[1]: http://lixin.piaozu.com.cn/RetrofitDemo2.jpg
[2]: http://apistore.baidu.com/apiworks/servicedetail/2422.html
[3]: http://square.github.io/retrofit/