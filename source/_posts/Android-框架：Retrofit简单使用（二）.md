---
title: Android-框架：Retrofit简单使用（二）
date: 2016-09-26 13:01:30
tags:
---
啦啦啦，本次换个角度，从获取Json来进行Retrofit学习，以便以后很很很小白的人看了，也能够快速上手。。
　　两个Json：
　　`https://api.github.com/repos/square/retrofit/contributors`
　　`https://api.github.com/search/users?q=retrofit`
　　
　　点进去后，自己对比下，一个可以说是JsonArray，另一个是JsonObject。两者区别，详见[这里][1]。
## 1.创建接收Json数据的Java类
这里介绍一个插件，Android studio中，在Settings——Plugins中，下载GsonFormat。
使用方法如下：
　　
Generate
![此处输入图片的描述][2]
　　
把Json复制进去
![此处输入图片的描述][3]
　　
这里可以看到详细类型数值
![此处输入图片的描述][4]
　　因为是两个不同类型的Json，所以要创建两个类。<br/>
## 2.使用Retrofit获取Json值
怎么简单使用Retrofit上篇讲过啦，不懂回去温习下  [Android-框架：Retrofit简单使用（一）][5]。
首先是JsonArray怎么获取：

    public interface GitHubRequestAPI {
        @GET("repos/{user}/{repo}/contributors")
        Call<List<GsonArrayExample>> getGitHubSearch(@Path("user") String userName, @Path("repo") String repoName);
    }


    Retrofit retrofit = new Retrofit.Builder()
            .baseUrl("https://api.github.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build();
        
    GitHubRequestAPI gitHubRequestAPI = retrofit.create(GitHubRequestAPI.class);


　　

    Call<List<GsonArrayExample>> call = gitHubRequestAPI.getGitHubSearch("square","retrofit");
    call.enqueue(new Callback<List<GsonArrayExample>>() {
            @Override
            public void onResponse(Call<List<GsonArrayExample>> call, Response<List<GsonArrayExample>> response) {
                List<GsonArrayExample> list = response.body();
                for(GsonArrayExample gsonArrayExample : list){
                    Log.i("获得JsonArray中的login：",gsonArrayExample.getlogin);
                }
            }
    
            @Override
            public void onFailure(Call<List<GsonArrayExample>> call, Throwable t) {
    
                }
            }
        );   

<br/>
接下来是JsonObject：

    public interface GitHubRequestAPI {
        @GET("search/users")
        Call<GsonObjectExample> getGitHubSearch(@Query("q") String repoName);
    }


    Retrofit retrofit = new Retrofit.Builder()
            .baseUrl("https://api.github.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build();
        
    GitHubRequestAPI gitHubRequestAPI = retrofit.create(GitHubRequestAPI.class);


    Call<GsonObjectExample> call = gitHubRequestAPI.getGitHubSearch("retrofit");
    call.enqueue(new Callback<GsonObjectExample>() {
            @Override
            public void onResponse(Call<GsonObjectExample> call, Response<GsonObjectExample> response) {
                GsonObjectExample gsonObjectExample = response.body();
                List<GsonObjectExample.ItemsBean> list = gsonObjectExample.getItems();
                for (int i = 0; i < list.size(); i++) {
                    Log.i("获得JsonObject中的login",":"+list.get(i).getLogin());
                    }
            }
    
            @Override
            public void onFailure(Call<GsonObjectExample> call, Throwable t) {
    
                }
            }
        );  
<br/>
## 总结
通过两个Json例子，明白了`Call<List<Repo>>`中参数是`List<Repo>`的用法了吧。
另：如果在URL中需要传入多个参数的话，比如`https://api.github.com/search/repositories?q=retrofit&since=2016-03-29&page=1&per_page=3`，可以使用`@QueryMap`：

        @GET("search/users")
        Call<GsonObjectExample> getGitHubSearch(@QueryMap Map<String,String> map);
        
        Map<String,String> map = new HashMap<String, String>();
        map.put("q","retrofit");
        map.put("since","2016-03-29");
        ...
        Call<GsonObjectExample> call = gitHubRequestAPI.getGitHubSearch(map);

好啦，到此本次学习就到此结束啦，下次再见。


参考：[Retrofit2.0使用详解][6]
　　　[Retrofit官方网][7]


[1]: http://stackoverflow.com/questions/12289844/difference-between-jsonobject-and-jsonarray
[2]: http://lixin.piaozu.com.cn/RetrofitDemo4.jpg
[3]: http://lixin.piaozu.com.cn/RetrofitDemo5.jpg
[4]: http://lixin.piaozu.com.cn/RetrofitDemo6.jpg
[5]: https://leevsee.github.io/2016/09/25/Android-%E6%A1%86%E6%9E%B6%EF%BC%9ARetrofit%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8%EF%BC%88%E4%B8%80%EF%BC%89/
[6]: http://blog.csdn.net/ljd2038/article/details/51046512
[7]: http://square.github.io/retrofit/