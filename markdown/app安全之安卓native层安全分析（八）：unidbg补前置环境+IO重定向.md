## 前言

继续跟着龙哥的unidbg学习： [SO逆向入门实战教程七：main\_unidbg 重定向\_白龙\~的博客-CSDN博客](https://blog.csdn.net/qq_38851536/article/details/118000259?spm=1001.2014.3001.5502 "SO逆向入门实战教程七：main_unidbg 重定向_白龙~的博客-CSDN博客")

还是那句，我会借鉴龙哥的文章，以一个初学者的角度，加上自己的理解，把内容丰富一下，尽量做到不在龙哥的基础上画蛇添足。感谢观看的朋友。

## 分析

首先，抓个包

![](https://img-blog.csdnimg.cn/img_convert/10c8fdeed770adb41c8272feeaed4518.png)

里面这个mtgsig就是该app很经典的加密参数了，siua参数后续有时间就分析，没有就算了。本篇文章的重点是mtgsig.

### 1.静态分析

jadx分析，一搜：

![](https://img-blog.csdnimg.cn/img_convert/2587a099659715c19f56a2480f554db9.png)

发现其实并不多。

就拿这几个类进行hook，看看哪些方法被调用了就行了，没花多久时间，就找到这里：

![](https://img-blog.csdnimg.cn/img_convert/15d1f17ddff3e3822f5130f414ca348d.png)

> com.xxxxx.plugin.sign.core.CandyPreprocessor.makeHeader\(Uri\$Builder\) String

再跟一下：

![](https://img-blog.csdnimg.cn/img_convert/3b0ee794930cc659cda7fea4de7b76e4.png)

![](https://img-blog.csdnimg.cn/img_convert/bda79786e6d3cb813becaba65f3e1902.png)

![](https://img-blog.csdnimg.cn/img_convert/6f66b8f399079cff3ada3dc4bbbf6612.png)

这个main就是我们要的部位了。我们知道，第一个int参数，其实都写死了，就是203

![](https://img-blog.csdnimg.cn/img_convert/c7e0e2918a4f488d91c80dd5bbe2ae81.png)

第二个是一个对象数组

hook下这两个方法，main的地方参数中间有个byte\[\]没有打印出来

![](https://img-blog.csdnimg.cn/img_convert/d36d60f121086d2abed6510f7d0342a0.png)

### 2.frida动态调试

![](https://img-blog.csdnimg.cn/img_convert/7f399740b203a9f7c027b5cf0b829a17.png)

![](https://img-blog.csdnimg.cn/img_convert/1ca12414e4537aba5a500ee43f8fd485.png)

所以这个byte\[\]就是个 请求方式  url的path  子路径

好了，入参大概都知道了

## ida调试

调试之前，得找找他加载的是哪个so文件

![](https://img-blog.csdnimg.cn/img_convert/5c41a8787867c04e866321acf4849e93.png)

这静态看代码，看了半天，愣是没看出来哪里用了system.loadlibrary。

用这个grep找到的是这个so，其实并不是

![](https://img-blog.csdnimg.cn/img_convert/ebbf8c047d97b350637565890a656284.png)

这里继续用yang神的hook\_RegisterNative.js，一顿输出，直接找到这个so:

![](https://img-blog.csdnimg.cn/img_convert/8ef6f381e0ad52767eb209f5ff5ff585.png)

很nice。

卧槽，跑完直接把我手机干关机了。有点6

接下来打开ida看看，导出表都不用看了

![](https://img-blog.csdnimg.cn/img_convert/7e362857c2e775b660bc373c850751a2.png)

方法也看不到啊，上面用hook\_RegisterNative已经hook到位置了，直接定位过去就行

![](https://img-blog.csdnimg.cn/img_convert/3e3ea1a14a5c80dec77cb9ab01525079.png)

结果就是解析异常了。

## unidbg调试

那直接unidbg调试吧

### 1.搭架子

![](https://img-blog.csdnimg.cn/img_convert/999ef16bc055208987bcbae1b1ed316b.png)

跑起来，好像没啥问题 ，main也有了。

### 2.调用

那么直接调用吧，首先把之前hook到的数据，传递进去

上面需要的都有hook了，

![](https://img-blog.csdnimg.cn/img_convert/c903f95278a304be3d4c879474ad7793.png)

现在就差这个key，往上找找：

![](https://img-blog.csdnimg.cn/img_convert/05afb18a71daf8036e0759af74158ba8.png)

hook下，等会儿，还hook啥呀，前面已经有了，就是这个9b69xxxx，思路不能乱，慢慢来

![](https://img-blog.csdnimg.cn/img_convert/261195063b51f2a32ad854b65a2243c8.png)

开始调用，结果如下：

![](https://img-blog.csdnimg.cn/img_convert/03ecf5f485211e19f98bd0872fd6c0f7.png)

尴尬，报错的意思是这个指针结果是一个空指针。根据龙哥的博客，应该是上下文缺失。jnitrace也可以发现。

### 3.解决问题

再来看看，这个main的调用实例

![](https://img-blog.csdnimg.cn/img_convert/d32415a99c34127745fdf60599ffaa42.png)

有这么多

用fridahook下这些

![](https://img-blog.csdnimg.cn/img_convert/66a54cbaa02b4cd2c9f81c7a9c1451a3.png)

发现，果然是了，刚开始调用，还是null，然后再次调用，就有值了。

再来，把入参打印一下：

![](https://img-blog.csdnimg.cn/img_convert/d7420c96dc4eceba86e2f862121afceb.png)

看来第一次是111，也就是代码的这里：

![](https://img-blog.csdnimg.cn/img_convert/522e289ec83503c221ceb36343e2ca30.png)

这个好像还算简单。

补一下这里的前置条件，然后执行：

![](https://img-blog.csdnimg.cn/img_convert/53470d295d0491f4a736fb4dbcd85357.png)

补下这个环境，但是这个是啥呢，直接用firda hook可知如下：

因为他是static

![](https://img-blog.csdnimg.cn/img_convert/4d20baf2e9ac3de0f887180edebcfbdb.png)

那么直接调用

![](https://img-blog.csdnimg.cn/img_convert/870ec3c0653785442e0832df9755f7d5.png)

补完执行：

![](https://img-blog.csdnimg.cn/img_convert/f6f3c89bf300e50e576399962fc219d7.png)

又来一个新的，同样操作

![](https://img-blog.csdnimg.cn/img_convert/1bd988af43835feb24c8dda4cbfb32a1.png)

![](https://img-blog.csdnimg.cn/img_convert/7d56f415f98902ab60b2e730a1192e97.png)

还有报错，

![](https://img-blog.csdnimg.cn/img_convert/1eba3cdf41cecfbcf81b8419ac22461c.png)

继续

![](https://img-blog.csdnimg.cn/img_convert/55d29c2df85fdcbf59f6ea7cffe64a32.png)

卧槽，还有报错，而且这个报错，龙哥的博客好像没有这个，我手机是米10，安卓12，估计有啥不一样

![](https://img-blog.csdnimg.cn/img_convert/9e126a14437bbf1c0f55c2faefff6d31.png)

问题不大，他既然要返回一个i，也就是整形，那直接这么补，然后执行：

![](https://img-blog.csdnimg.cn/img_convert/4a0c583a9d69dde108908f0176ed0835.png)

说白了这个就是要获取当前的进程apk的路径，继续补

![](https://img-blog.csdnimg.cn/img_convert/5a9a82d4ab4d4872279bfd83be1ba955.png)

没啥可说的，继续补：

![](https://img-blog.csdnimg.cn/img_convert/01a6b92929a8067570510f495d843d4f.png)

![](https://img-blog.csdnimg.cn/img_convert/512c6134985da9d87d00dbce7e91f6b5.png)

继续

![](https://img-blog.csdnimg.cn/img_convert/973026324259fedddae791b674fc7e4c.png)

![](https://img-blog.csdnimg.cn/img_convert/6b5f46993e1aeab3c10252d2919ec9f9.png)

继续

![](https://img-blog.csdnimg.cn/img_convert/6dbd5dcd5ef49599cba1b51200e6bdd5.png)

好了，终于不报错了，但是结果并没有被打印出来

仔细看这里，根据龙哥的博客，意思是这里做了文件读取操作，但是-100，好像读取异常了

![](https://img-blog.csdnimg.cn/img_convert/51f77b7845814235a6e810d9535afbb6.png)

这时候有两种方案解决这个

- 到unidbg目录，把apk放进去，用这个文件src/main/java/com/github/unidbg/file/BaseFileSystem.java获取当前程序路径
- 还可以用代码的方式进行操作，设置io重定向，把访问的路径放到我们指定的路径即可

这里我用重定向的方式：

![](https://img-blog.csdnimg.cn/img_convert/86cd99e34b3306a5f4839ecf56dc6708.png)

![](https://img-blog.csdnimg.cn/img_convert/41b84dd61add728764d244ade4d39450.png)

然后执行：

![](https://img-blog.csdnimg.cn/img_convert/5afbcf082e0feb8ecf69d4b234ca9acc.png)

继续补

![](https://img-blog.csdnimg.cn/img_convert/79dfc840d6c126bf359877d726f28802.png)

但这里出现了问题，这里是一个空，尴尬，查看用例：

![](https://img-blog.csdnimg.cn/img_convert/12555699b47abf3490e5c01270cf1e8f.png)

就这里做了赋值

![](https://img-blog.csdnimg.cn/img_convert/340c39ab9362d361b005aaa08e31d992.png)

想hook一下，结果hook不到，尴尬，但是那边确实看到是空，龙哥的博客是有数据的，不重要，遵从内心，补一个空，执行看看：

![](https://img-blog.csdnimg.cn/img_convert/ed1f1eeaa979919ff4f1f262ccea8a73.png)

唉，结果有了，好像结果数据有点不对，最后是一个dvmobject，那就跟前面的文章一样了，转一下，打印下

这两个方式都可以：

![](https://img-blog.csdnimg.cn/img_convert/f99a96c17941649104c44eae7d51789d.png)

这个结果能不能用呢？

重新抓个包，然后替换下试试：

![](https://img-blog.csdnimg.cn/img_convert/7a73ac987bbf37ba4a482ad379e392e1.png)

ok，果然可以。很nice

代码

```java
package com.sankuai;

import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Emulator;
import com.github.unidbg.Module;
import com.github.unidbg.file.FileResult;
import com.github.unidbg.file.IOResolver;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.linux.android.dvm.api.Binder;
import com.github.unidbg.linux.android.dvm.api.ServiceManager;
import com.github.unidbg.linux.android.dvm.array.ArrayObject;
import com.github.unidbg.linux.android.dvm.array.ByteArray;
import com.github.unidbg.linux.android.dvm.wrapper.DvmInteger;
import com.github.unidbg.linux.file.SimpleFileIO;
import com.github.unidbg.memory.Memory;


import java.io.ByteArrayInputStream;
import java.io.File;
import java.io.UnsupportedEncodingException;
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.security.cert.CertificateException;
import java.security.cert.CertificateFactory;
import java.util.ArrayList;
import java.util.List;
import java.util.Locale;

public class meituan extends AbstractJni implements IOResolver {
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;

    meituan() {
        // 创建模拟器实例,进程名建议依照实际进程名填写，可以规避针对进程名的校验
        emulator = AndroidEmulatorBuilder.for32Bit().setProcessName("com.sankuai.meituan").build();
        // 获取模拟器的内存操作接口
        final Memory memory = emulator.getMemory();
//        emulator.getSyscallHandler().setEnableThreadDispatcher(true);
        // 设置系统类库解析
        memory.setLibraryResolver(new AndroidResolver(23));
        // 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作
        vm = emulator.createDalvikVM(new File("unidbg-android\\src\\test\\java\\com\\xxx\\xxx.apk"));
        // 加载目标SO
//        DalvikModule dm = vm.loadLibrary(new File("unidbg-android\\src\\test\\java\\com\\xxxx\\libmtguard.so"), true); // 加载so到虚拟内存

        emulator.getSyscallHandler().addIOResolver(this);
        vm.setVerbose(true); // 设置是否打印Jni调用细节
        DalvikModule dm = vm.loadLibrary(new File("E:\\work\\myproject\\unidbg\\unidbg-android\\src\\test\\java\\com\\xxxx\\libmtguard.so"), true);


        //获取本SO模块的句柄,后续需要用它
        module = dm.getModule();
        vm.setJni(this); // 设置JNI
        vm.setVerbose(true); // 打印日志
        dm.callJNI_OnLoad(emulator); // 调用JNI OnLoad
    }


    @Override
    public DvmObject<?> callStaticObjectMethodV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {
        switch (signature) {
            case "com/meituan/android/common/mtguard/NBridge->getPicName()Ljava/lang/String;":
                return new StringObject(vm, "ms_com.xxx.xxx");
            case "com/meituan/android/common/mtguard/NBridge->getSecName()Ljava/lang/String;":
                return new StringObject(vm, "ppd_com.xxx.xxx.xbt");
            case "com/xxxx/android/common/mtguard/NBridge->getAppContext()Landroid/content/Context;":
                return vm.resolveClass("android/content/Context").newObject(null);
            case "com/xxxx/android/common/mtguard/NBridge->getMtgVN()Ljava/lang/String;":
                return new StringObject(vm, "4.4.7.3");
            case "com/xxx/android/common/mtguard/NBridge->getDfpId()Ljava/lang/String;":
                return new StringObject(vm, "");


        }

        return super.callStaticObjectMethodV(vm, dvmClass, signature, vaList);
    }

    @Override
    public DvmObject<?> newObjectV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {
        switch (signature) {
            case "java/lang/Integer-><init>(I)V": {
                int input = vaList.getIntArg(0);
                return vm.resolveClass("java/lang/Integer").newObject(input);
            }
        }

        return super.newObjectV(vm, dvmClass, signature, vaList);
    }

    @Override
    public int getStaticIntField(BaseVM vm, DvmClass dvmClass, String signature) {
        switch (signature) {
            case "android/content/pm/PackageManager->GET_SIGNATURES:I":
                return 1;

        }
        return super.getStaticIntField(vm, dvmClass, signature);
    }

    @Override
    public int getIntField(BaseVM vm, DvmObject<?> dvmObject, String signature) {
        switch (signature) {
            case "android/content/pm/PackageInfo->versionCode:I":
                return 2;

        }
        return super.getIntField(vm, dvmObject, signature);
    }


    @Override
    public DvmObject<?> callObjectMethodV(BaseVM vm, DvmObject<?> dvmObject, String signature, VaList vaList) {
        switch (signature) {
            case "android/content/Context->getPackageCodePath()Ljava/lang/String;":
                return new StringObject(vm, "/data/app/com.xxx.xxx-TEfTAIBttUmUzuVbwRK1DQ==/base.apk");
        }
        return super.callObjectMethodV(vm, dvmObject, signature, vaList);
    }

    public static void main(String[] args) {
        meituan test = new meituan();
        test.callMain111();
        System.out.println(test.getSign());
    }

    public void callMain111() {
        List<Object> list = new ArrayList<>(10);
        list.add(vm.getJNIEnv()); // arg1
        list.add(0); //arg2
        list.add(111); //arg3
        DvmObject<?> obj = vm.resolveClass("java/lang/object").newObject(null);
        vm.addLocalObject(obj);
        // 完整的参数2
        list.add(vm.addLocalObject(new ArrayObject(obj)));
        Number number = module.callFunction(emulator, 0x5a38d, list.toArray());
    }

    public String getSign() {
        List<Object> list = new ArrayList<>(10);
        list.add(vm.getJNIEnv()); // arg1
        list.add(0); //arg2
        list.add(203); //arg3
        StringObject input2_1 = new StringObject(vm, "9b69f861-e054-4bc4-9daf-d36ae205ed3e");
        ByteArray input2_2 = new ByteArray(vm, "GET /api/entry/indexLayer __reqTraceID=aa14663c-9979-445b-a311-8017c596f343&ci=1&group=xxxx&msid=xxxxx&userid=-1&utm_campaign=AgroupBgroupC0E0&utm_content=xxxx&utm_medium=android&utm_source=wandoujia&utm_term=1100090405&uuid=xxxxx&version_name=11.9.405".getBytes(StandardCharsets.UTF_8));
        DvmInteger input2_3 = DvmInteger.valueOf(vm, 2);
        vm.addLocalObject(input2_1);
        vm.addLocalObject(input2_2);
        vm.addLocalObject(input2_3);
        // 完整的参数2
        list.add(vm.addLocalObject(new ArrayObject(input2_1, input2_2, input2_3)));
        Number number = module.callFunction(emulator, 0x5a38d, list.toArray());
//        String result = vm.getObject(number.intValue()).getValue().toString();
//        return result;

        DvmObject[] result1 = (DvmObject[]) vm.getObject(number.intValue()).getValue();
        String sign = result1[0].toString();
        System.out.println(sign);

        StringObject result2 = (StringObject) ((DvmObject[])((ArrayObject)vm.getObject(number.intValue())).getValue())[0];
        System.out.println(result2.getValue());
        return result2.getValue();
    }

    @Override
    public FileResult resolve(Emulator emulator, String pathname, int oflags) {
        if (("/data/app/com.sankuai.meituan-TEfTAIBttUmUzuVbwRK1DQ==/base.apk").equals(pathname)) {
            // 填入想要重定位的文件
            return FileResult.success(new SimpleFileIO(oflags, new File("unidbg-android\\src\\test\\java\\com\\xxx\\xxx.apk"), pathname));
        }
        return null;
    }
}
```

## 知识点总结

1.找不到该native方法引入的哪个so文件时，用hook\_RegisterNative脚本分析

2.执行结果发现不对返回空指针时，很大可能是逻辑问题，需要结合jnitrace分析

3.报dirfd=-100，就是读取文件错误，需要指导程序到正确的路径

4.补文件读取有两个方法

- 到unidbg目录，把apk放进去，用这个文件src/main/java/com/github/unidbg/file/BaseFileSystem.java获取当前程序路径
- 还可以用代码的方式进行操作，设置io重定向，把访问的路径放到我们指定的路径即可

5.补环境的时候getIntArg是获取方法的参数

我的代码是：

```java
int input = vaList.getIntArg(0);

return vm.resolveClass("java/lang/Integer").newObject(input);
```

龙哥的代码：

```java
int input = vaList.getInt(0);

return new DvmInteger(vm, input);
```

6.如果最后的数据是一个数组而不是字符串，有两种方式获取

```java
StringObject result = (StringObject) ((DvmObject[])((ArrayObject)vm.getObject(number.intValue())).getValue())[0];
System.out.println(result.getValue());
```

```java
DvmObject[] result1 = (DvmObject[]) vm.getObject(number.intValue()).getValue();
String sign = result1[0].toString();
System.out.println(sign);
```