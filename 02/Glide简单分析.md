### Glide

##### 使用
```
从多个源中获取数据，如网络、本地、Uri。
Glide不仅是一个图片缓存，它支持 Gif、WebP、缩略图、Video。
默认格式RGB565。Picass是AGRGB8888。
Glide缓存当前尺寸大小图片，Picass缓存原尺寸，二次使用Glide稍微快一点。
支持图形转换，圆角，高斯模块等，
Glide支持gif及，Picass不支持。
支持不同的网络框架去获取突破，如okhttp
支持优先级处理

Glide支持不同的生命周期策略。如Application、Activity、Fragment、FragmentActivity。
如果是Activity则Glide请求会同步Activity的生命周期一起销毁，暂停，继续下载等。
Glide如果是Activity，则生成一个空白Fragment绑定到Activity，然后接收Fragment生命周期回调，去控制Glide的请求生命周期。
如果是Application，则不会有这种效果。
```






































