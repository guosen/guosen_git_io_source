title: Android原生权限探索
date: 2016-06-22 16:42:53
tags:
- Android
- 权限
---
# AppOpsManager
## AppOps就是管理应用权限，但是API被谷歌隐藏了。
###就是我们平常说的应用程序权限管理，但是每次更新时，谷歌都会隐藏入口。（6.0y已经加入运行时权限 也就是基于该类的）
#  AppOpsManager总体概览
## 核心服务是AppOpsService，API是AppOpsManager，UI层是AppOpsSummary，AppOpsCategory，配置文件为appops.xml  appops_policy.xml。
# 相关API
## int
checkOp(String op, int uid, String packageName)
 Op对应一个权限操作，该接口来检测应用是否具有该项操作权限。
 
int
noteOp(String op, int uid, String packageName)
和checkOp基本相同，但是在检验后会做记录。
 
int
checkOpNoThrow(String op, int uid, String packageName)
 和checkOp类似，但是权限错误，不会抛出SecurityException，而是返回AppOpsManager.MODE_ERRORED.
 
int
noteOpNoThrow(String op, int uid, String packageName)
类似noteOp，但不会抛出SecurityException。

# 如何使用？
## 接口被隐藏了，有人想到直接把类打包成jar然后去调用，其实我们可以采用反射的方式代码如下
```bash
final int version = Build.VERSION.SDK_INT;
        if (version>19) {
            Object appOpsManager = getContext().getSystemService(Context.APP_OPS_SERVICE);
            Class c = appOpsManager.getClass();
            try {
                Class[] arg = new Class[3];
                arg[0] = int.class;
                arg[1] = int.class;
                arg[2] = String.class;
                Method lmethod = c.getDeclaredMethod("checkOp", arg);
                //27代表录音权限
                //op 1~47
                return (Integer) lmethod.invoke(appOpsManager, 27, Binder.getCallingUid(), getContext().getPackageName());
            } catch (NoSuchMethodException ex) {
                ex.printStackTrace();
            } catch (InvocationTargetException ex) {
                ex.printStackTrace();
            } catch (IllegalArgumentException ex) {
                ex.printStackTrace();
            } catch (IllegalAccessException ex) {
                ex.printStackTrace();
            }
        }
        return -1;
    }





```
# 参考
## http://blog.csdn.net/hyhyl1990/article/details/46842915
http://tmq.qq.com/2016/06/let-your-brief-encounter-android-permission-to-business-practices/
http://tmq.qq.com/2016/06/android-authorization-management-theory/

