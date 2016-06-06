title: Android 网络请求OKHttp＋Retrofit
date: 2016-04-07 22:47:57
tags:
---
#简介
## Retrofit是Square开发的网络服务库, 简化Get/Post网络服务的使用, 配合Rx和Dagger有更好的效果. Retrofit已经更新到第2.0版, 本文介绍一些常见的使用方式.
https://github.com/guosen/Volley
#引入（gradle）
## 
```bash
compile 'com.android.support:appcompat-v7:23.2.1'
    compile 'com.squareup.retrofit:retrofit:2.0.0-beta1'
    compile 'com.squareup.okhttp:okhttp:2.4.0'
    compile 'com.squareup.retrofit:converter-gson:2.0.0-beta2'
```
---
```bash
/**
 * Created by shouwang on 16/4/7.
 */
public interface ApiService {
    String END_POINT="http://api.meiyaapp.cn/";

    @Headers("Cache-Control: public, max-age=3600")
    @GET("v1/users/{id}?signature=123456")
    Call<HttpResponse<UserInfo>> getUserInfo(
          @Path("id") int id,
          @Query("with_recipients") int with_recipients,
          @Query("with_talent") int with_talent
    );
}
//其中的@Headers 是在这边增加头部 实现缓存功能
```
注：在retrofit2.0里去掉了CallBack接口参数。

#定义接口服务类，用来管理和创建ApiService
## 
```bash
public class ApiManager {
    public static final String HEADER_ACCEPT_JSON = "application/json";
    public static final String HEADER_ACCEPT      = "Accept";
    private        ApiService mApiService;
    private static ApiManager sInstance;
    private Context mContext;
    public ApiManager(Context context) {
         mContext=context;
    }

    private ApiService createService() {
         OkHttpClient client=new OkHttpClient();
        //----------
        //设置头部 例如返回类型是Json/text
        //-----------
         client.interceptors().add(new Interceptor() {
             @Override
             public Response intercept(Chain chain) throws IOException {
                 final Request originalRequest = chain.request();
                 final Request requestWithUserAgent = originalRequest.newBuilder()
                         .addHeader(HEADER_ACCEPT, HEADER_ACCEPT_JSON)
                         .build();
                 Response response = chain.proceed(requestWithUserAgent);
                 return response;
             }
         });
        //日志拦截 https://github.com/square/okhttp/wiki/Interceptors
        client.interceptors().add(new LoggingInterceptor());
        client.interceptors().add(new CacheInterceptor(mContext));

        File cacheFile = new File(mContext.getCacheDir(), "[缓存目录]");
        Cache cache = new Cache(cacheFile, 1024 * 1024 * 100); //100Mb
        client.setCache(cache);




        //设置请求  并设置HttpClient (OKHttp)
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl(ApiService.END_POINT)
                .addConverterFactory(GsonConverterFactory.create())
                .client(client)
                .build();


        return retrofit.create(ApiService.class);

    }

    public ApiService getService() {
        if (mApiService == null) {
            synchronized (this) {
                if (mApiService == null)
                    mApiService = createService();
            }
        }
        return mApiService;
    }

    public static ApiManager getInstance(Context context) {
        if (sInstance == null) {
            sInstance = new ApiManager(context.getApplicationContext());
        }
        return sInstance;
    }

```
#Interceptor
##Interceptor是拦截器, 在发送之前, 添加一些参数, 或者获取一些信息. loggingInterceptor是打印参数.
```bash
public class CacheInterceptor implements Interceptor {
    private Context mContext;
    public CacheInterceptor(Context context){
        mContext=context;
    }
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        if(!NetworkUtil.isNetworkAvailable(mContext)){
            request = request.newBuilder()
                    .cacheControl(CacheControl.FORCE_CACHE)
                    .build();
//            Logger.t(TAG).w("no network");
        }
        Response originalResponse = chain.proceed(request);
        if(NetworkUtil.isNetworkAvailable(mContext)){
            //有网的时候读接口上的@Headers里的配置，你可以在这里进行统一的设置
            String cacheControl = request.cacheControl().toString();
            return originalResponse.newBuilder()
                    .header("Cache-Control", cacheControl)
                    .removeHeader("Pragma")
                    .build();
        }else{
            return originalResponse.newBuilder()
                    .header("Cache-Control", "public, only-if-cached, max-stale=2419200")
                    .removeHeader("Pragma")
                    .build();
        }
    }
}


```

#Activity里的调用
##
```bash

        Call<HttpResponse<UserInfo>> call=ApiManager.getInstance(this).getService().getUserInfo(91625, 0, 0);
        call.enqueue(new Callback<HttpResponse<UserInfo>>() {
            @Override
            public void onFailure(Throwable t) {
                Log.i("MainActivity",t.getMessage());
            }

            @Override
            public void onResponse(Response<HttpResponse<UserInfo>> response, Retrofit retrofit) {
                Log.i("MainActivity",response.body().data.username+"    ++++++++++");
            }
        });
```
