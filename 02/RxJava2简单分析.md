
### RxJava

RxJava2是一个基于观察者模式，以响应流的形式处理异步请求的工具。

#### 框架解决的问题

RxJava帮助我们使各种异步请求变成链式的清晰。
创建一个Observable，完成异步任务，最终将任务结果发射给Observer处理。

#### 原理
```java
1，创建被观察者，2，创建观察者，3，被观察者和观察者建立订阅关系。
```

#### 它的好处

```java
1，简化规则。封装了一系列的操作符，提供各种各样的操作场景。
2，简化代码，简化嵌套。通常多层次嵌套，会让代码可读性变差，难以理解以及修改，RxJava通过封装实现了用链式请求代替回调嵌套。
3，方便的线程切换。它可以设置任务的每次执行所在线程，也可以设置在任务结束后回调的线程。
```


#### 核心实现

```java
Observer：观察者。
Observable：被观察者。
Subscriber：观察者。
Flowable：被观察者。
被观察者订阅观察者，当被观察者发生变化时，会通知注册了事件的观察者。
流程3步曲：1，创建被观察者，2，创建观察者，3，被观察者和观察者建立订阅关系。
Observable不支持背压，Flowable支持背压。
```

#### 线程调度

```java
subscriberOn()观察者接受事件等线程，observerOn()上一个被观察者执行事件的线程。
subscriberOn调用多次也只是第一次的为准；observerOn可以调用多次，代表上一次被观察者处理事件所在线程
```

#### 背压
```java
Observable不支持背压，Flowable支持背压。
背压：当被观察者的数据发送事件过快，而观察者处理事件速度过慢，导致被观察者堆积了一堆事件，最终缓存爆了出了问题。背压就是根据下游的速度，去通知被观察者发送事件的速度，这样不会出现缓存爆的问题。而剩下堆积的事件，可以有不同的策略处理，抛出异常、不处理、加大缓存等。

```

#### 常用操作符

##### map

场景：采用 map 操作符进行网络数据解析，把String转换成Bean。
```java
Observable.create(new ObservableOnSubscribe<String>() {
                        @Override
                        public void subscribe(ObservableEmitter<String> observableEmitter) throws Exception {
                            //异步处理数据，通常是Retrofit返回一个Observable了
                            observableEmitter.onNext("1");
                        }
                    })
                .observeOn(new IoScheduler())
                .map(new Function<String, BaseResponse>() {
                    @Override
                    public BaseResponse apply(String s) throws Exception {
                        JSONObject jsonObject = new JSONObject(s);
                        return new BaseResponse(jsonObject.getInt("code")
                                , jsonObject.getString("message"));
                    }
                })
                .observeOn(new IoScheduler())
                .subscribeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<BaseResponse>() {
                    @Override
                    public void accept(BaseResponse baseResponse) throws Exception {
                        Log.d("1", "" + baseResponse.message);
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(Throwable throwable) throws Exception {
                        Log.d("1", "" + throwable.getMessage());
                    }
                });
```

##### concat

利用 concat 的必须调用 onComplete 后才能订阅下一个 Observable 的特性。
场景：可以先读取缓存数据，若无缓存再调用 onComplete() 以执行获取网络数据的 Observable；
如果缓存数据正确，则直接调用 onNext()，不会调用另一个Observable。防止过度的网络请求，浪费用户的流量。
```java
Observable<BaseResponse> cache = Observable.create(new ObservableOnSubscribe<BaseResponse>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<BaseResponse> e) throws Exception {
                BaseResponse data = CacheManager.getInstance().getBaseResponseCache();
                // 在操作符 concat 中，只有调用 onComplete 之后才会执行下一个 Observable
                if (data != null){ // 如果缓存数据不为空，则直接读取缓存数据，而不读取网络数据
                    e.onNext(data);
                }else {
                    e.onComplete();
                }
            }
        });
         //这里假设通过Retrofit获取数据得到这个Observable
        Observable<BaseResponse> network = Retrofir.getData();
        // 两个 Observable 的泛型应当保持一致
        Observable.concat(cache,network)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<BaseResponse>() {
                    @Override
                    public void accept(@NonNull FoodList tngouBeen) throws Exception {
                        Log.d("1", "" + baseResponse.message);
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(@NonNull Throwable throwable) throws Exception {
                        Log.d("1", "" + throwable.getMessage());
                    }
                });

```

##### flatMap

flatMap 操作符可以将一个发射数据的 Observable 变换为多个 Observables ，然后将它们发射的数据合并后放到一个单独的 Observable。
场景：注册后进行登陆。
```java
 		//这里假设通过Retrofit获取数据完成注册并得到返回数据
        Observable<UserInfo> network = Retrofir.getUserInfo();
		 network.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .flatMap(new Function<UserInfo, ObservableSource<Login>>() {
                    @Override
                    public ObservableSource<Login> apply(@NonNull UserInfo u) throws Exception {
                        //这里假设通过Retrofit进行登陆
                        return Observable<Login> network = Retrofir.doLogin();
                    }
                })
                .observeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Login>() {
                    @Override
                    public void accept(@NonNull Login l) throws Exception {
                        Log.d("1", "" + baseResponse.message);
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(@NonNull Throwable throwable) throws Exception {
                        Log.d("1", "" + throwable.getMessage());
                    }
                });
```

##### zip

2个Observable处理完毕，然后合并2个Observable到一个新的Observable，再发射事件。
```java
	Observable.zip(Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> observableEmitter) throws Exception {

            }
        }), Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> observableEmitter) throws Exception {
                
            }
        }), new BiFunction<String, Integer, BaseResponse>() {
            @Override
            public BaseResponse apply(String s, Integer integer) throws Exception {

                return new BaseResponse(integer, s);
            }
        })
```

##### interval

场景：轮询操作，计时器操作。
```java
Flowable.interval(1, TimeUnit.MILLISECONDS)
```

##### 操作符太多了。。。想起来用到什么场景再说吧。













