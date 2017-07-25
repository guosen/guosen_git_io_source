title: 关于gson 解析不规范json
date: 2017-07-25 16:25:26
tags:
---
{"data":{"sign":"2113fe846f348d1b07263573eaa1c530","result":"{\"eaccountId\":41,\"sex\":\"1\",\"contactTel\":\"16585854282\",\"merchantProvince\":\"福建省\",\"city\":null,\"bankCount\":1,\"balance\":113,\"area\":null,\"is_finger_pay\":\"0\",\"token\":\"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjdXN0SWQiOiI0MSIsInVzZXJJZCI6IjQxIiwiaWF0IjoxNTAwOTY4OTA2NDU4LCJjdXN0VHlwZSI6IjAifQ.HXkuHO_OrG4LbOAILFv6eJA8T4VepKJCeqT6YtRiwbE\",\"nickName\":\"守望\",\"merchantType\":\"0\",\"province\":\"北京 北京市 西城区\",\"merchantName\":\"莆田卤面\",\"alipayAccount\":\"18060031709\",\"custId\":41,\"merchantLogo\":\"header/758f2ffa-f057-474f-b0c4-57a703a8dfa0.jpg\",\"is_open_pay_code\":\"0\",\"avatar\":\"header/e5be6d17-c27d-4db5-8fe6-d20cf89e5598.jpg\",\"isOpenEcash\":\"1\",\"merchantArea\":\"思明区\",\"isAuth\":\"1\",\"email\":\"1432967264@qq.com\",\"contactName\":\"林国森\",\"contactType\":\"00\",\"is_free_pay\":0,\"realName\":\"林国森\",\"merchantCity\":\"厦门市\",\"is_set_pay_pwd\":1,\"mobile\":18060031709}"},"description":"服务处理成功","status":"success

如上json
应该说对于对象必须是不加双引号的，如果加上双引号会识别为字符串。
那也只能按照字符串去解析
``java
package com.npay.merchant.retrofit.response;

import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;
import com.npay.merchant.bean.User;

import org.json.JSONException;
import org.json.JSONObject;

import java.lang.reflect.Type;

/**
 * Created by guosenlin on 2017/7/25.
 */
public class UserData<T> {
    String sign;
    String result;

    public T getResult(Class<T> clazz){
        try {
            JSONObject jsonObject = new JSONObject(result);
            Type fooType = new TypeToken< T>() {}.getType();
             return new Gson().fromJson(result, clazz);
        } catch (JSONException e) {
            e.printStackTrace();
        }
        return null;
    }

    //一般来说没必要在这边再继续序列化一次 但是由于 这个后端的json格式有问题多了""（嵌套对象）

}

``


``java
package com.npay.merchant.retrofit.response;

import com.google.gson.Gson;

import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;

/**
 * Created by guosenlin on 2017/6/23.
 */

public class HttpResponse<T>     {

    public UserData<T> data;

    public String description;

    public String status;


    public static HttpResponse fromJson(String json, Class clazz) {
        Gson gson = new Gson();
        Type objectType = type(HttpResponse.class, clazz);
        return gson.fromJson(json, objectType);
    }

    public String toJson(Class<T> clazz) {
        Gson gson = new Gson();
        Type objectType = type(HttpResponse.class, clazz);
        return gson.toJson(this, objectType);
    }

    static ParameterizedType type(final Class raw, final Type... args) {
        return new ParameterizedType() {
            public Type getRawType() {
                return raw;
            }

            public Type[] getActualTypeArguments() {
                return args;
            }

            public Type getOwnerType() {
                return null;
            }
        };
    }

}



    private class HttpResultFunc<T> implements Func1<T, T> {

        @Override
        public T call(T httpResult) {
            if (httpResult  == null) {
                throw new ApiException(100);
            }
            return httpResult;
        }
    }

    /***
     * 用户登录
     * @param subscriber
     * @param phone
     * @param pwd
     * @param custType
     */
    public void accountLogin(Subscriber<HttpResponse<User>> subscriber, String phone, String pwd, int custType ){
        Observable observable = mApiService.login(phone,pwd,custType)
                .map(new HttpResultFunc<HttpResponse<User>>());
        toSubscribe(observable, subscriber);
    }
``
