## ids.xml概述

`ids.xml`：为应用的相关资源提供唯一的资源`id`。`id`是为了获得`xml`中的对象需要的参数，也就是 `Object = findViewById(R.id.id_name);` 中的`id_name`。

这些值可以在代码中用`android.R.id`引用到。
若在`ids.xml`中定义了**ID**，则在`layout`中可如下定义`@id/price_edit`，否则`@+id/price_edit`。

> 优点

1. 命名方便，我们可以把一些特定的控件先命好名，在使用的时候直接引用`id`即可，省去了一个命名环节。
2. 优化编译效率:
   - 添加`id`后会在`R.java`中生成;
   - 使用`ids.xml`统一管理,一次性编译即可多次使用.
     但使用`"@+id/btn_next"`的形式,每次文件保存`(Ctrl+s`)`后R.java`都会重新检测,如果存在该`id`则不生成,如果不存在就需要添加该`id`。故编译效率降低。


`ids.xml`文件内容：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <item name="forecast_list" type="id"/>
<!--    <item name="app_name" type="string" />-->
</resources>
```

也许有人很好奇上面有一行被注释的代码，打开注释会发现编译会报一下错误：

```java
Execution failed for task ':app:mergeDebugResources'.
> [string/app_name] /Users/tanzx/AndroidStudioProjects/AaptDemo/app/src/main/res/values/strings.xml	[string/app_name] /Users/tanzx/AndroidStudioProjects/AaptDemo/app/src/main/res/values/ids.xml: Error: Duplicate resources
```

因为`app_name`对于的资源已经在`value`中被声明了。

## public.xml概述

这个没有找到官方的具体说明，现有网络上的解释为：文件**RES/value/public.xml**用于将固定资源ID分配给Android资源。

[stackoverfloew:What is the use of the res/values/public.xml file on Android?](https://stackoverflow.com/questions/9348614/what-is-the-use-of-the-res-values-public-xml-file-on-android%E3%80%82)

[官网：选择要设为公开的资源](https://developer.android.com/studio/projects/android-library#PrivateResources)

`public.xml`文件内容：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <public name="forecast_list" id="0x7f040001" type="id" />
    <public name="app_name" id="0x7f070002" type="string" />
    <public name="string3" id="0x7f070003" type="string" />
</resources>
```

## `aapt`进行`id`的固定

> 项目环境配置（PS：吐槽一下aapt已经被aapt2代替了，aapt相关资料几乎没有，环境搭建太费劲了~！）
>
> `com.android.tools.build:gradle:2.2.0`
>
> `distributionUrl=https\://services.gradle.org/distributions/gradle-3.4.1-all.zip`
>
> `compileSdkVersion 24`
>
> `buildToolsVersion '24.0.0'`

先在`value`文件下按照上面的`ids.xml`和`public.xml`的内容以及文件名，生成对应的文件。

> 什么都不修改的直接编译

<img src="png/changed.png" style="zoom:50%;" />

通过直接编译之后的`R文件`的内容，可以看到我们想要的设置的资源`id`并没有按照我们预期的生成。

> 将`public.xml`文件拷贝到`build/intermediates/res/merged`对应的目录

```groovy
afterEvaluate {
    for (variant in android.applicationVariants) {
        def scope = variant.getVariantData().getScope()
        String mergeTaskName = scope.getMergeResourcesTask().name
        def mergeTask = tasks.getByName(mergeTaskName)
        mergeTask.doLast {
            copy {
                int i=0
                from(android.sourceSets.main.res.srcDirs) {
                    include 'values/public.xml'
                    rename 'public.xml', (i++ == 0? "public.xml": "public_${i}.xml")
                }
                into(mergeTask.outputDir)
            }
        }
    }
}
```

<img src="png/source.png" style="zoom:50%;" />

这次我们可以直接看到资源`id`按照我们需要生成了。

> 这是为什么呢？

1. `android gradle`插件`1.3一下`版本可以直接将`public.xml`放在源码`res`目录参与编译;

2. `android gradle`插件`1.3+`版本在执行`mergeResource`任务时忽略了`public.xml`，所以`merge`完成后的`build`目录下的`res`目录下没有`public.xml`相关的内容。所以需要在编译时通过脚本将`public.xml`插入到`merge`完成后的`build`目录下的`res`目录下。之所以这样做可行，是因为`aapt`本身是支持`public.xml`的，只是`gradle`插件在对资源做预处`(merge)`时对`public.xml`做了过滤。

参考文章：

[android public.xml 用法](https://www.cnblogs.com/linghu-java/p/9548039.html)

[Android-Gradle笔记](https://ljd1996.github.io/2019/08/21/Android-Gradle%E7%AC%94%E8%AE%B0/)

