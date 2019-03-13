### Retrogit

##### 使用

基于网络层次的封装。
声明一个接口NetService，声明接口方法，在接口方法上添加Get/Post注解，然后构造Retrofit，添加baseurl、数据转换器Gson、调用适配器RxJava、内置OkHttp3.

NetService ns = retrofit.create(NetService.class)；ns就可以调用了。

设计模式：构建模式、代理模式。

create()：代理模式，使用Proxy代理了ns所有带有Retrofit注解的接口方法。调用ns相关方法的时候，会构造OkHttpCall(继承Call)，然后通过反射收集到方法名、参数返回值和注解等信息。最终使用这些信息去调用这个Call的request方法，也就是OkHttp执行请求了。




































