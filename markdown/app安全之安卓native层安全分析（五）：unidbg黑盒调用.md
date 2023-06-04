## 前言

继续跟着龙哥的unidbg学习： [SO逆向入门实战教程五：qxs\_白龙\~的博客-CSDN博客](https://blog.csdn.net/qq_38851536/article/details/117828334 "SO逆向入门实战教程五：qxs_白龙~的博客-CSDN博客")

还是那句，我会借鉴龙哥的文章，以一个初学者的角度，加上自己的理解，把内容丰富一下，尽量做到不在龙哥的基础上画蛇添足，哈哈。感谢观看的朋友

## 分析

首先，安装app，发现龙哥给的apk包已经安装不上，网上重新找了个最新版，安装后，正常打开，然后，抓个包：

![](https://img-blog.csdnimg.cn/img_convert/fca5589d28b1462bcdc3c337b0070807.png)

ok，这个sfsecurity参数就是今天的重点了。

这个app有ajm的壳，没事，脱壳手段上，结果搞半天没脱出来，卧槽了，而且还有 frida反调试

脱壳机也不在身边，那没法，直接黑盒调用，unidbg就是干这个的。

## 调试

### 1.搭架子

运行没问题

![](https://img-blog.csdnimg.cn/img_convert/c6a0db8849283722ad75d4afaab95ff4.png)

不过这里，unidbg并没有打印出来我们要的方法和地址

### 2.ida找地址

这时候就需要用ida找地址了，结果发现，这个so文件有点奇怪，搜不到，但是在导出表里看得到，这就很奇怪了，根据分析，猜测是有一定的保护，也没办法通过F5反编译

![](https://img-blog.csdnimg.cn/img_convert/d4b1fb00644e523559e021af107cfb6e.png)

很快就找到这里了，【0xAB2C】 ，注意，这里我用的是最新版，所以地址跟龙哥博客的地址不一样

### 3.黑盒调用\&补环境

这里，我选用的是32位，所以得加1，因为我把apk解包，从lib\\armeabi-v7a里找的这个so文件，所以他一定是32位的（根据安卓开发规范）

![](https://img-blog.csdnimg.cn/img_convert/bbcd83a02e0dd5135fe7fb595505b017.png)

ok，报了个环境的错，补一补：

![](https://img-blog.csdnimg.cn/img_convert/ca009678e527a25849c16c9d79ba0ac9.png)

ok，又有新的报错，继续补：

![](https://img-blog.csdnimg.cn/img_convert/580e0c28933358bdb1798b5c4d0b8284.png)

ok，搞定

这里，由于我是根据龙哥的博客，知道是哪个so方法，所以直接黑盒调用，也没有用objection hook，它有frida反调试（objection基于frida），那就尴尬了，所以这里没法验证hook值和主动调用值是否一致了。那就暂时这样，反正值是有了

完整代码：

```java
package com.qxs;

import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Module;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.linux.android.dvm.api.Binder;
import com.github.unidbg.linux.android.dvm.api.ServiceManager;
import com.github.unidbg.memory.Memory;

import java.io.File;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.security.cert.CertificateException;
import java.security.cert.CertificateFactory;
import java.util.ArrayList;
import java.util.List;
import java.util.Locale;
import java.util.UUID;

public class qxs extends AbstractJni {
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;

    qxs() {
        // 创建模拟器实例,进程名建议依照实际进程名填写，可以规避针对进程名的校验
        emulator = AndroidEmulatorBuilder.for32Bit().setProcessName("com.sina.oasis").build();
        // 获取模拟器的内存操作接口
        final Memory memory = emulator.getMemory();
        // 设置系统类库解析
        memory.setLibraryResolver(new AndroidResolver(23));
        // 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作
        vm = emulator.createDalvikVM(new File("unidbg-android\\src\\test\\java\\com\\qxs\\com.sfacg.apk"));
        // 加载目标SO
        DalvikModule dm = vm.loadLibrary(new File("unidbg-android\\src\\test\\java\\com\\qxs\\libsfdata.so"), true); // 加载so到虚拟内存
        //获取本SO模块的句柄,后续需要用它
        module = dm.getModule();
        vm.setJni(this); // 设置JNI
        vm.setVerbose(true); // 打印日志

        dm.callJNI_OnLoad(emulator); // 调用JNI OnLoad
    }
    public static void main(String[] args) {
        qxs test = new qxs();
        System.out.println(test.sfsecurity());
    }
    public String sfsecurity(){
        List<Object> list = new ArrayList<>(10);
        list.add(vm.getJNIEnv());
        list.add(0);
        DvmObject<?> context = vm.resolveClass("android/content/Context").newObject(null);
        list.add(vm.addGlobalObject(context));
        list.add(vm.addLocalObject(new StringObject(vm,"test")));
        Number number =  module.callFunction(emulator,0xAB2C+1,list.toArray());
        String result = vm.getObject(number.intValue()).getValue().toString();
        return result;
    }
    @Override
    public DvmObject<?> callStaticObjectMethodV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {
        switch (signature) {
            case "java/util/UUID->randomUUID()Ljava/util/UUID;":
                return dvmClass.newObject(UUID.randomUUID());
        }
        return super.callStaticObjectMethodV(vm, dvmClass, signature, vaList);
    }

    @Override
    public DvmObject<?> callObjectMethodV(BaseVM vm, DvmObject<?> dvmObject, String signature, VaList vaList) {
        switch (signature) {
            case "java/util/UUID->toString()Ljava/lang/String;":
                String uuid = dvmObject.getValue().toString();
                return new StringObject(vm, uuid);
        }
        return super.callObjectMethodV(vm, dvmObject, signature, vaList);
    }
}
```

## 知识点总结

本篇和上一篇感觉的挺简单的，所以没有啥总结的