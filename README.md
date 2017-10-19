#这篇文章是关于Activity的一些使用的内容

##知识点概况：
* Activity的生命周期
* Activity的状态保存
* 两个Activity跳转是的生命周期问题
* 横竖屏切换的生命周期
* Activity设置成窗口的样式
* Activity退出，以及多个Activity的退出问题
* Activity的启动模式
* Activity中的Context，Activity，Application有什么区别
* Activity之间数据传递
* Context是什么

###1.Activity的生命周期
* Activity的生命周期包括：onCreate onStart onResume onPause onStop onDestroy
其实这些方法都是两两对应的
* onCreate 表示Activity正在被创建，这是生命周期的第一个方法。在这个方法中做一些初始化操作；
* onRestart 表示Activity从不可见重新便会可见状态时调用；
* onStart 表示Activity可见但是没有显示到前台时回调；
* onResume 表示Activity可见并且处于前台运行；
* onPause 表示Activity在前台不可见
* onStop 表示Activity已经停止
* onDestroy 销毁

几种特殊情况的生命周期变化
* 当启动的Activity采用了透明主题的时候，那么当前Activity不会回调onStop
* 启动另一个Activity的时候时先执行B的onResume还是A的onPause A的onPause->B的onCreate->B的onStart
->B的onResume->A的onStop  

如果对于上面的问题有异议可以去看看生命周期的那张图片，挺经典的！

###2.Activity的状态保存
* 正常情况下（这里指系统内存充足的时候）

  > Activity的状态会自动保存的，一般来说, 调用 onPause()和 onStop()方法后的 activity 实例仍然存在于内存中, activity 的所有
      信息和状态数据不会消失, 当 activity 重新回到前台之后, 所有的改变都会得到保留

* 非正常情况下（系统内存不足的时候）

  > 调用 onPause()和 onStop()方法后的 activity 可能会被系统摧毁, 此时
   内存中就不会存有该 activity 的实例对象了。如果之后这个 activity 重新回到前台, 之前所作的改变
   就 会 消 失 。 为 了 避 免 此 种 情 况 的 发 生 , 我 们 可 以 覆 写 onSaveInstanceState() 方 法 。
   onSaveInstanceState()方法接受一个 Bundle 类型的参数, 开发者可以将状态数据存储到这个
   Bundle 对象中, 这样即使 activity 被系统摧毁, 当用户重新启动这个 activity 而调用它的 onCreate()
   方法时, 上述的 Bundle 对象会作为实参传递给 onCreate()方法, 开发者可以从 Bundle 对象中取出
   保存的数据, 然后利用这些数据将 activity 恢复到被摧毁之前的状态。
   
* 需要注意的

  > onSaveInstanceState()方法并不是一定会被调用的, 因为有些场景是不需要保存
    状态数据的. 比如用户按下 BACK 键退出 activity 时, 用户显然想要关闭这个 activity, 此时是没有必
    要 保 存 数 据 以 供 下 次 恢 复 的 , 也 就 是 onSaveInstanceState() 方 法 不 会 被 调 用 . 如 果 调 用
    onSaveInstanceState()方法, 调用将发生在 onPause()或 onStop()方法之前。
     
  > onRestoreInstanceState和onSaveInstanceState这两个方法都是用来保存和获取数据的，首先要明确它们不一定时成对出现的
  当页面数据不确定重新创建的时候不会调用onRestoreInstanceState但是当确定页面数据发生变化的时候就会调用这个方法，
  举个例子就是当页面横竖屏切换的时候一定会调用这个方法的。
  
###3.两个Activity跳转时候的生命周期变化
> 一般情况下比如说有两个activity,分别叫A,B,当在A里面激活B组件的时候, A会调用 onPause()
方法,然后 B 调用 onCreate() ,onStart(), onResume()。
这个时候 B 覆盖了窗体, A 会调用 onStop()方法. 如果 B 是个透明的,或者是对话框的样式, 就
不会调用 A 的 onStop()方法
> 这里注意一点就是跳转新页面的时候只有在onPause()执行完成之后新的页面才会执行onResume(),所以在onPause()中尽量少执行耗时操作

###4.横竖屏切换的生命周期
此时的生命周期和清单文件里的配置有关系；
* 不设置 Activity 的 android:configChanges 时，切屏会重新调用各个生命周期默认首先销毁
  当前 activity,然后重新加载。
  
* 设置 Activity android:configChanges="orientation|keyboardHidden|screenSize"时，切
  屏不会重新调用各个生命周期，只会执行 onConfigurationChanged 方法。
  
###5.Activity设置成窗口模式
* 只需要在Activity得清单文件中设置
```
android:theme="@android:style/Theme.Dialog"
```
###6.Activity退出，以及多个Activity的退出问题
####这里又几种方法：

* 退出一个Activity只需要按返回键，代码是直接调用finish();方法就可以了
  
* 记录每一个打开得Activity(这个一般是在BaseActivity中进行编写的)
```
//一段伪代码
List<Activity> lists ;// 在 application 全局的变量里面
lists = new ArrayList<Activity>();
lists.add(this);
for(Activity activity: lists){
    activity.finish();
}
lists.remove(this);
```
* 发送特定的广播(这个一般是在BaseActivity中进行编写的)

###7.Activity的四种启动模式
设置启动模式的方式是在清单文件中设置**android:launchMode=""**
四种启动模式分别为：
* standard 系统默认的启动模式：每次启动Activity的时候都会重新创建一个实例，不管这个实例是否存在
* singleTop 栈顶复用：就是说当Activity已经在栈顶的话就不会重新创建实例了，会走相应Activity的onNewIntent(Intent intent);方法
* singleTask 栈复用：就是任务栈里面只会又一个实例，当重新调用的时候会把上面的实例全部销毁，并且会走相应Activity的onNewIntent(Intent intent);方法
* singleInstance 新栈打开：就是在新的任务栈打开一个实例，相当于新的返回栈实例

###9.Activity之间的数据传递
启动Activity分为两种一种是显式调用，一种是隐式调用显式或者隐式都会涉及到intentFilter所以这里面主要记录一下intentFilter的使用

####intentFilter的匹配规则

* action的匹配规则：action是一个字符串，系统预定义了一些action同时也可以自定义一些action，一个过滤规则中可以又多个action，还有就是action是区分大小写的，当大小也不同的时候是不会匹配成功的！
        
        ```
        //在Manifest中相应的Activity中配置相应的action
        <action android:name="XXX"
        ```
* category的匹配规则：category是一个字符串，系统预定义了一些categor同时也可以自定义category，category和action的区别在于匹配的时候可以没有category但是至少又一个action
        ```
        //在Manifest中相应的Activity中配置相应的category
        <category android:name="XXX";
        ```
        
* data的匹配规则：data的匹配规则和action类似，如果过滤规则中定义了data，那么Intent必须要定义可配置的data
    #####data的结构
    ```
        <data android:scheme="string"
                  android:host="string"
                  android:port="string"
                  android:path="string"
                  android:pathPattern="string"
                  android:pathPrefix="string"
                  android:mimeType="string"/>
         //data的组成由两部分组成，mimeType和URI。mimeType指媒体类型，比如image/jpeg、audio/mpeg4-generic、vodio/*可以表示图片、文本、视频等不同的媒体格式
         //URI的结构：<scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]
         //URI例子：1》content://com.example.project:200/folder/subfolder/etc
                              2》http://www.baidu.com:80/search/info
     ```
        
   * scheme： URI的模式，比如http、file、content如果没有制定scheme的话整个URI的其他参数无效，这就意味这URI无效
   * Host：URI的主机名，比如www.baidu.com 如果host未制定，那么整个URI中的其他参数无效
   * Port：URI的端口号，比如80，仅当URI中制定了scheme和host参数的是偶port参数才有意义
   * path、pathPattern、pathPrefix：这三个参数表示路径信息，但是她里面可以包含通配符“*”、“*”表示0个或者多个任意字符，需要注意的是由于正则表达式的需要，“*”要写成“\\*”、"*"要写成"\\*"、"\"要写成"\\\\"pathPrefix
       表示路径的前缀信息
       
    #####data的匹配原则
    * 过滤规则一：
    
        ```
        <intent-filter>
            <data android:mimeType="image/*"/>
        </intent-filter>
        //这种过滤规则制定了媒体类型为所有类型的图片，那么Intent中的mimeType属性必须为"image/*"才能匹配，这种情况下虽然过滤规则没有制定URI但是却有默认值，URI的默认值为content和file。也就是说最然没有指定URI，但是Intent中的URI部分的schema必须为content或者file才能皮匹配，这点尤其注意
        //intent.setDataAndType(Uri.parse("file://abc"),"iamge/png")
        ```
    * 