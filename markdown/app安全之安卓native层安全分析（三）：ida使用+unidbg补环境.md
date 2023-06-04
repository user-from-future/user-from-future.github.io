## 前言

继续跟着龙哥的unidbg学习： [SO入门实战教程三：V2-Sign\_unidbg context\_白龙\~的博客-CSDN博客](https://blog.csdn.net/qq_38851536/article/details/117533396 "SO入门实战教程三：V2-Sign_unidbg context_白龙~的博客-CSDN博客")

还是那句，我会借鉴龙哥的文章，以一个初学者的角度，加上自己的理解，把内容丰富一下，尽量做到不在龙哥的基础上画蛇添足，哈哈。感谢观看的朋友

## 分析

打开app，抓包，发现有个sign

![](https://img-blog.csdnimg.cn/img_convert/4b0b2041d58935eea6e6858c7045f864.png)

这个sign就是今天的重点了，jadx打开apk，可以，没有加壳，一搜，发现很快就搜到这些了，而且也不多

![](https://img-blog.csdnimg.cn/img_convert/c08a56cd555e1e3e23b6537757309383.png)

问题不大，用objcetion 把这几个都hook了，看看是走的哪里，没搞多久，就看到这里，入参和返回值，感觉就是这里了

![](https://img-blog.csdnimg.cn/img_convert/427ef0f2902150e60674807f96772112.png)

因为这个返回值的格式，根抓包看到的格式基本一致

还有一个，我们看看请求的加密和解密，objection:

![](https://img-blog.csdnimg.cn/img_convert/3268bea3778bcd55689f02e2422ed800.png)

感觉objection不是太好看这个结果，那么这里改用frida看看吧，虽然objection 可以改源码，然后让结果可读，但是这里感觉不是有太大的必要，直接用frida就行了。

![](https://img-blog.csdnimg.cn/img_convert/85e03b4c511fbc09eac090b3fab2bb2d.png)

就是这里了，直接就解密了。

## ida查看

打开ida看看这个so:

![](https://img-blog.csdnimg.cn/img_convert/e4f750c221b87808b8e0343eb69a3a30.png)

卧槽，这个so文件有点硬啊，看看，jni\_onload:

![](https://img-blog.csdnimg.cn/img_convert/e2c02ba54b8d699a75cfb2ab9c8f4699.png)

卧槽，确实硬啊，这就尴尬了。那直接上unidbg吧

## unidbg调试

### 1.搭架子

先把main跑起来

```java
package com.xiaochuankeji;



import com.github.unidbg.AndroidEmulator;

import com.github.unidbg.Module;

import com.github.unidbg.linux.android.AndroidEmulatorBuilder;

import com.github.unidbg.linux.android.AndroidResolver;

import com.github.unidbg.linux.android.dvm.AbstractJni;

import com.github.unidbg.linux.android.dvm.DalvikModule;

import com.github.unidbg.linux.android.dvm.VM;

import com.github.unidbg.memory.Memory;

import com.weibo.international;



import java.io.File;



public class tieba extends AbstractJni {

    private final AndroidEmulator emulator;

    private final VM vm;

    private final Module module;

    tieba() {

        // 创建模拟器实例,进程名建议依照实际进程名填写，可以规避针对进程名的校验

        emulator = AndroidEmulatorBuilder.for32Bit().setProcessName("cn.xiaochuankeji.tieba").build();

        // 获取模拟器的内存操作接口

        final Memory memory = emulator.getMemory();

        // 设置系统类库解析

        memory.setLibraryResolver(new AndroidResolver(23));

        // 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作

        vm = emulator.createDalvikVM(new File("unidbg-android\\src\\test\\java\\cn\\xiaochuankeji\\right573.apk"));

        // 加载目标SO

        DalvikModule dm = vm.loadLibrary(new File("unidbg-android\\src\\test\\java\\com\\xiaochuankeji\\libnet_crypto.so"), true); // 加载so到虚拟内存

        //获取本SO模块的句柄,后续需要用它

        module = dm.getModule();

        vm.setJni(this); // 设置JNI

        vm.setVerbose(true); // 打印日志

        dm.callJNI_OnLoad(emulator); // 调用JNI OnLoad

    }



    public static void main(String[] args) {

        tieba test = new tieba();

        System.out.println(test);

    }

}
```

运行，看起来没啥问题

![](https://img-blog.csdnimg.cn/img_convert/85f71cef8a2c69d1e56bcf2de98e6295.png)

根据龙哥的文章，说如果这里设置为false:

![](https://img-blog.csdnimg.cn/img_convert/eaa066c9842f58c2857c599532c737cf.png)

结果就会乱码。原因是这个so样本，做了【字符串的混淆或加密】 ，以此来对抗分析人员，但字符串总是要解密的

这个解密一般发生在Init array节或者JNI OnLoad中，又或者是该字符串使用前的任何一个时机，而本例呢，就发生在Init array节中，Shift+F7快捷键查看节区验证这一点

![](https://img-blog.csdnimg.cn/img_convert/947fb8440f51ad149a41976f60de610a.png)

Init array节内有大量函数，解密就发生在其中。当我们使用Unidbg模拟执行时，如果加载SO时配置为不执行Init相关函数，这导致整个SO中的字符串都没有被解密，自然输出就是一团”乱码“。

由此还可以衍生出一个小话题——如果样本中的字符串被加密了，如何还原？使得分析者可以愉快的用IDA静态分析？

- 从内存中Dump出解密后的SO或者字符串（可以用Frida/IDA 脚本/ adb 等），将结果回填或者说修复本身SO。
- 使用Unicorn或基于Unicorn的模拟执行工具（Unidbg、ExAndroidNativeemu等）运行SO，dump解密后的虚拟内存，回填修复SO

### 2.执行native\_init

为啥要执行native\_init，看jadx，他注册so的时候，同时执行了这个方法，那么我们要完全的模拟安卓环境，那就也要先执行这个方法，再执行核心的方法

![](https://img-blog.csdnimg.cn/img_convert/969615ecf6c55e2b841887c42709bd06.png)

再看看，刚才的main函数执行打印的日志，刚好，已经有了native的方法地址

![](https://img-blog.csdnimg.cn/img_convert/ebc6650ea3383d6df29aff20fc6c427e.png)

用ida看看是thumb还是arm，看这个应该是thumb

![](https://img-blog.csdnimg.cn/img_convert/7d0380d386d30840974378fa3120cd4e.png)

执行下这个方法：

![](https://img-blog.csdnimg.cn/img_convert/de164ed224a848d545d9ec2f85501518.png)

卧槽，这个啥情况，意思是没有这个方法，那不急，不加1看看

![](https://img-blog.csdnimg.cn/img_convert/85c84f7d7929773930e7e4e4dc47f28c.png)

现在好像可以了，这个就尴尬了，跟咱们之前说的规则对不上啊，这就奇怪了。不过问题不大，因为前面说了，这个so有加密过，所以ida识别可能有问题。这个不纠结。

再仔细看看这个，好像是报错：

![](https://img-blog.csdnimg.cn/img_convert/1a3f86f2af6318c6f90473c9a7ab3b5b.png)

它提示，意思是不支持这个方法，可以的，终于到了补环境的环节

### 3.unidbg补环境

点进第一个报错的地方看看：

![](https://img-blog.csdnimg.cn/img_convert/ef4b45bc226832dbb465ad22cd5580fd.png)

确实有补了很多java环境，但是差我们这个，那我们就自己补了，怎么补，关于补法，有两种实践方法都很有道理

- 全部在用户类中补，防止项目迁移或者Unidbg更新带来什么问题，这样做代码的移植性比较好。
- 样本的自定义JAVA方法在用户类中补，通用的方法在AbstractJNI中补，这样做的好处是，之后运行的项目如果调用通用方法，就不用做重复的修补工作

那么这里我就直接在自己定义的地方补了，防止对unidbg源码修改过多，没啥意义。再看，这个报错，它要的是一个context，那么给他一个context就行。context怎么来，在第二篇有讲过了：

> vm.resolveClass\("android/content/Context"\).newObject\(null\)

![](https://img-blog.csdnimg.cn/img_convert/e808dc5bd6753cc9ce96335ff4a6ad37.png)

再运行看看，卧槽，内存不够了，这么猛的吗

![](https://img-blog.csdnimg.cn/img_convert/0dcf275e029ecd334f81004d5686abeb.png)

没事，查阅资料：

> [unidbg/SignalTest.java at 56c1397c639716817896766df9bbce9c3a0a15b9 · zhkl0228/unidbg \(github.com\)](https://github.com/zhkl0228/unidbg/blob/56c1397c639716817896766df9bbce9c3a0a15b9/unidbg-android/src/test/java/com/github/unidbg/android/SignalTest.java#L53 "unidbg/SignalTest.java at 56c1397c639716817896766df9bbce9c3a0a15b9 · zhkl0228/unidbg (github.com)")

加一行这个就行：

![](https://img-blog.csdnimg.cn/img_convert/3a2ac6e7634d19e05f70b8b33b8763d9.png)

再次运行，现在好像没报错了：

![](https://img-blog.csdnimg.cn/img_convert/72416fc13adf0b766cea2d431a6d1318.png)

### 4.执行sign方法

解析来开始执行加密方法sign了：

![](https://img-blog.csdnimg.cn/img_convert/386db59c4bbd2c64871f66c214974965.png)

![](https://img-blog.csdnimg.cn/img_convert/f395a534abcf7f13c8c3bb54d9e22f5f.png)

点进去，

![](https://img-blog.csdnimg.cn/img_convert/fca8354e0f47515ec4d3245b5f009e20.png)

照着补了

![](https://img-blog.csdnimg.cn/img_convert/bcb9be7c5068b165a80228083b61934e.png)

运行，又有新的环境缺失：

![](https://img-blog.csdnimg.cn/img_convert/b93fda3da999351110d4454c0c9e8dd1.png)

捋一下完整的流程，在com/izuiyou/common/base/BaseApplication中调用getAppContext方法，获得一个Context上下文，然后getClass获取它的类，最后查看它的类名。类名就是这一系列操作的最终目的，我们前面几步都只浅浅的补了一下，只能说类型给对了，别的都没给。但只要最后的类名给它返回正确的字符串，就没问题。

用objection  的wallerbreaker 看看：

![](https://img-blog.csdnimg.cn/img_convert/8abe06eeb06d745523757900a2a3fc2a.png)

![](https://img-blog.csdnimg.cn/img_convert/09007b931d50b4d73c4bc201584bf663.png)

看这个，说明class就是这个AppController了，补完接着跑：

![](https://img-blog.csdnimg.cn/img_convert/a90ae9c80c6851786e76af8851dad11d.png)

它要一个文件的地址，补完接着跑，

![](https://img-blog.csdnimg.cn/img_convert/efcc83b760382a4447bb9b1417d37b38.png)

它要绝对路径，其实还是上面那个，接着补，新的报错：

![](https://img-blog.csdnimg.cn/img_convert/e66b8a02a6c114ac4564e1bca8dddc75.png)

查看这个是否在调试，这里，按正常的逻辑，肯定是给false

![](https://img-blog.csdnimg.cn/img_convert/2d3929d512e468bc18204c84d4e812d7.png)

又有新的报错，继续补：

![](https://img-blog.csdnimg.cn/img_convert/613689ec793cce867e5ba81a1afe13ca.png)

卧槽，好像终于出结果了！！！！，不容易啊。

为了保险起见，还是验证下，跟hook到的值，是否能保持一致

### 5.验证值

先用frida 重新hook下这个方法：

![](https://img-blog.csdnimg.cn/img_convert/c714a54a62402ca69a29a5178853499e.png)

本来想把这个bytearray转成字符串看看的，试了以下几种方法，都不行：

> console.log\('sign arg2===》', ByteString.of\(arg2\).utf8\(\)\)  
> console.log\('sign arg2===》', ByteString.of\(arg2\).toString\(\)\)  
> console.log\('sign arg2===》', hex2str\(ByteString.of\(arg2\).hex\(\)\)\)  
> console.log\('sign arg2===》', jbhexdump\(arg2\)\)  
> console.log\('sign arg2===》', Java.use\("java.lang.String"\).\$new\(Java.array\('byte', arg2\)\)\)  
> console.log\(Java.use\("java.lang.String"\).\$new\(arg2\)\)

那没法了，直接复制这个byte array吧

unidbg里稍微调一下：

![](https://img-blog.csdnimg.cn/img_convert/726acd9cecea19d28705541ab5634b49.png)

牛逼，结果对上了。舒服

完整代码：

```java
package com.xiaochuankeji;



import com.github.unidbg.AndroidEmulator;

import com.github.unidbg.Module;

import com.github.unidbg.linux.android.AndroidEmulatorBuilder;

import com.github.unidbg.linux.android.AndroidResolver;

import com.github.unidbg.linux.android.dvm.*;

import com.github.unidbg.linux.android.dvm.api.AssetManager;

import com.github.unidbg.linux.android.dvm.array.ByteArray;

import com.github.unidbg.memory.Memory;

import com.weibo.international;



import java.io.File;

import java.nio.charset.StandardCharsets;

import java.util.ArrayList;

import java.util.List;



public class tieba extends AbstractJni {

    private final AndroidEmulator emulator;

    private final VM vm;

    private final Module module;



    tieba() {

        // 创建模拟器实例,进程名建议依照实际进程名填写，可以规避针对进程名的校验

        emulator = AndroidEmulatorBuilder.for32Bit().setProcessName("cn.xiaochuankeji.tieba").build();

        // 获取模拟器的内存操作接口

        final Memory memory = emulator.getMemory();

        emulator.getSyscallHandler().setEnableThreadDispatcher(true);

        // 设置系统类库解析

        memory.setLibraryResolver(new AndroidResolver(23));

        // 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作

        vm = emulator.createDalvikVM(new File("unidbg-android\\src\\test\\java\\com\\xiaochuankeji\\right573.apk"));

        // 加载目标SO

        DalvikModule dm = vm.loadLibrary(new File("unidbg-android\\src\\test\\java\\com\\xiaochuankeji\\libnet_crypto.so"), true); // 加载so到虚拟内存

        //获取本SO模块的句柄,后续需要用它

        module = dm.getModule();

        vm.setJni(this); // 设置JNI

        vm.setVerbose(true); // 打印日志

        dm.callJNI_OnLoad(emulator); // 调用JNI OnLoad

    }



    public static void main(String[] args) {

        tieba test = new tieba();

//        System.out.println(test);

        test.native_init();

        test.sign();

    }



    public void native_init() {

        List<Object> list = new ArrayList<>(10);

        list.add(vm.getJNIEnv()); // arg1,env

        list.add(0); // arg2,jobject

        module.callFunction(emulator, 0x4a069, list.toArray());

    }



    @Override

    public DvmObject<?> callStaticObjectMethodV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {

        switch (signature) {

            case "com/izuiyou/common/base/BaseApplication->getAppContext()Landroid/content/Context;":

                return vm.resolveClass("android/content/Context").newObject(null);

        }

        return super.callStaticObjectMethodV(vm, dvmClass, signature, vaList);

    }



    @Override

    public DvmObject<?> callObjectMethodV(BaseVM vm, DvmObject<?> dvmObject, String signature, VaList vaList) {

        switch (signature) {

            case "android/content/Context->getClass()Ljava/lang/Class;":

                return dvmObject.getObjectType();

            case "java/lang/Class->getSimpleName()Ljava/lang/String;":

                return new StringObject(vm, "AppController");

            case "android/content/Context->getFilesDir()Ljava/io/File;":

            case "java/lang/String->getAbsolutePath()Ljava/lang/String;":

                return new StringObject(vm, "/data/user/0/cn.xiaochuankeji.tieba/files");

        }

        return super.callObjectMethodV(vm, dvmObject, signature, vaList);

    }



    @Override

    public boolean callStaticBooleanMethodV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {

        switch (signature) {

            case "android/os/Debug->isDebuggerConnected()Z":

                return false;

        }

        return super.callStaticBooleanMethodV(vm, dvmClass, signature, vaList);

    }

    @Override

    public int callStaticIntMethodV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {

        switch (signature) {

            case "android/os/Process->myPid()I":

                return emulator.getPid();

        }

        return super.callStaticIntMethodV(vm, dvmClass, signature, vaList);

    }



    public String sign() {

        List<Object> list = new ArrayList<>(10);

        list.add(vm.getJNIEnv()); // arg1,env

        list.add(0); // arg2,jobject

        list.add(vm.addLocalObject(new StringObject(vm, "https://api.izuiyou.com/index/recommend"))); // arg3

        byte[] ss = new byte[]{-61,8,81,112,-113,62,93,124,-101,59,-104,72,103,-122,-91,69,-101,86,18,29,118,-53,-110,-34,-35,100,9,114,-24,-56,84,20,-62,117,-95,76,90,-53,66,85,-128,-104,126,114,-91,13,111,11,121,12,77,68,101,52,-60,25,61,-30,38,-59,118,31,110,9,109,-87,119,83,-12,48,-111,41,116,-114,103,98,43,-12,-84,122,-9,-91,28,-26,-79,-45,-74,-52,6,56,63,-32,108,106,3,-23,-67,56,35,-100,91,-42,-22,-68,-112,-83,-79,39,49,92,112,-111,100,103,-127,16,-112,-10,-12,83,46,-108,-121,-54,-16,26,82,-102,83,97,-115,-2,118,-103,-52,-46,-118,-120,9,63,59,77,31,9,-14,68,69,-125,-16,105,-88,103,-80,61,78,115,112,-4,10,101,-21,53,39,-78,57,92,56,-77,20,-17,79,-76,-82,-12,17,-4,2,84,115,-71,20,115,-58,-105,116,-102,-103,-43,-105,96,95,-35,-53,-35,-56,-7,104,7,72,20,-113,-109,37,117,107,64,37,41,-107,70,10,-12,-127,-37,-24,74,21,66,-82,57,-7,76,97,-37,-19,-18,4,60,-49,-15,120,-24,12,-23,3,-63,-55,-12,-70,65,-16,117,9,-90,35,-3,110,-86,49,-97,24,85,60,38,16,-36,-64,24,117,-55,106,9,67,-5,-115,-74,20,-83,32,5,18,54,17,93,83,-18,-19,56,78,87,80,57,46,-105,10,49,-109,97,4,-104,48,69,73,119,45,9,-57,74,-44,3,-97,-101,57,124,44,-17,-54,66,-94,32,-64,-46,-30,87,-104,17,-68,-85,-69,36,46,-42,23,16,0,-117,-72,117,-20,-96,3,-115,104,-11,-26,-35,-32,127,24,37,21,77,16,66,-75,124,101,42,117,118,89,26,-86,62,45,-106,-111,6,-74,-83,13,0,32,-120,-94,-56,44,0,-12,-51,58,105,1,-74,8,105,65,22,-98,112,-90,-66,-13,-126,-96,-122,-113,-97,-85,-100,-126,123,47,-89,109,14,93,15,35,-61,46,76,40,63,119,52,87,-19,-28,41,-11,68,17,69,-101,21,-60,-73,2,-72,-97,-95,-98,-20,-97,65,-8,-29,-4,78,-118,25,-31,-16,-105,90,91,38,-100,77,24,34,-28,-23,81,74,44,-78,68,-62,115,125,-109,-76,-97,88,51,-13,100,-38,110,110,-51,17,51,5,91,-24,10,-102,40,60,-27,106,71,97,-125,37,-75,40,40,-43,101,-8,41,-124,-8,93,-18,-128,8,-50,-128,-128,-99,-23,97,-25,-118,-25,-23,24,-125,-47,-91,108,-66,4,73,19,31,-34,62,-78,122,-81,5,41,11,66,-26,-116,81,107,-14,-106,-110,35,78,-9,-88,20,-41,-111,-37,39,-104,82,71,-36,-116,-47,29,24,-107,24,32,-7,-90,-52,124,-94,-107,57,-7,117,38,-99,-37,50,-126,-68,45,-32,-25,-118,-69,20,55,-24,40,17,-116,111,-36,-15,109,102,37,-7,-118,63,88,52,46,58,-91,111,33,-23,102,116,-77,-100,102,-124,8,7,92,-84,-33,-26,54,-4,75,56,100,-5,-59,-66,51,127,-126,-31,39,-20,-81,72,106,64,-71,8,88,9,-18,-50,-118,-109,31,-55,11,89,0,-49,-69,-54,-118,14,-78,109,36,95,-6,68,21,58,-86,-103,55,94,4,-23,-75,106,-60,-20,72,3,-66,54,78,12,-9,-70,-13,106,-112,-51,100,62,-101,28,-63,8,-120,-67,24,-10,50,41,114,-41,42,-108,-19,-116,-62,-45,17,112,81,91,18,-108,-113,108,108,-88,-18,57,-40,117,-64,38,65,36,-24,-90,53,-61,-37,-45,-70,-14,-41,109,3,-59,-7,-101,-87,84,-39,25,85,-91,-111,17,35,77,-57,75,-105,-97,65,-37,-103,-112,-62,51,-42,40,59,70,79,50,-83,63,-23,-6,58,29,-5,-3,-33,-29,-91,-59,-36,55,34,24,74,38,-72,99,125,84,-124,60,97,-111,47,40,-29,-85,-76,-119,-89,106,-1,38,17,-17,-123,94,-104,-85,-10,13,53,110,103,100,123,-21,-14,107,11,45,-47,-22,-40,7,-102,26,59,98,120,-28,-27,123,55,97,18,23,-111,109,-127,-14,27,88,-19,-93,59,-118,-118,114,-17,-55,-99,86,-87,-73,-80,17,71,-32,-37,-50,53,29,-112,-81,-124,89,11,83,41,-41,24,-75,6,27,54,-121,12,-19,110,-73,35,-111,51,-118,115,30,106,-56,32,99,39,-75,67,-104,-77,-124,71,7,-36,-38,75,1,-8,-96,-44,99,115,-7,28,-128,38,39,-74,-43,-24,-35};

        ByteArray inbarr = new ByteArray(vm, ss);

        list.add(vm.addGlobalObject(inbarr)); //arg4

        Number number = module.callFunction(emulator, 0x4a28d, list.toArray());

        String result = vm.getObject(number.intValue()).getValue().toString();

        return result;

    }

}
```

## python调用

有没有想过，把这个项目，打包成jar后，给py调用呢，这样不就可以直接模拟调用了？有点意思对吧，上一章是直接给java调用，好像还没有给python调用过

先改一下这两个文件的路径，然后打包

![](https://img-blog.csdnimg.cn/img_convert/f7e7e6a5a792f6f310b4cb91c4aa7e48.png)

打包流程就省略了，前面有

![](https://img-blog.csdnimg.cn/img_convert/331d33470a29b78f270be96365b62ed7.png)

用python调用，记得把apk和so也放到这个py所在的路径下，因为打包的时候写的相对路径，谁调用，谁就是主程序，就要把这两个文件放到主程序文件所在同级目录：  

![](https://img-blog.csdnimg.cn/img_convert/a8897a11276c86382755f5902292aaa6.png)

这里这个test.py就是我的文件

```python
# coding:utf-8



import os

import subprocess

import jpype

import time





def query_by_java_jar(jar_path, param=None):

    if param:

        execute = "java -jar {} '{}'".format(jar_path, param)

    else:

        execute = "java -jar {}".format(jar_path)

    print(execute)

    output = subprocess.Popen(execute, shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)

    res = output.stdout.readlines()

    print('result', res)

    return res





if __name__ == '__main__':

    query_by_java_jar(r"E:\work\myproject\unidbg\out\artifacts\unidbg_jar\unidbg.jar")
```

运行结果：

![](https://img-blog.csdnimg.cn/img_convert/fdcbbfba4552d0edc377db4f478a12ce.png)

我试了下那个jpype，感觉各种问题，还是这个subproess方便点，直接出结果

## 知识点总结

1.看so的调用逻辑，如果有loadlibrary的同时有调用某个方法，unidbg模拟执行的时候也要先调用这个方法

2.如果加载so文件的时候，给定的第二个参数是false，加上so文件有字符串加密和混淆的话就会乱码，所以这里最好给为true，ida里，shift+F7，可以看

3.补环境的时候，要根据报错，最核心的部分进行补环境，同时也要结合实际的执行逻辑

4.构造context，用vm.resolveClass\("android/content/Context"\).newObject\(null\)

5.dvmObject.getObjectType\(\)可以获取当前类的class

6.emulator.getPid\(\)，获取pid

7.6种 frida里打印byte\[\]的方法

> console.log\('sign arg2===》', ByteString.of\(arg2\).utf8\(\)\)  
> console.log\('sign arg2===》', ByteString.of\(arg2\).toString\(\)\)  
> console.log\('sign arg2===》', hex2str\(ByteString.of\(arg2\).hex\(\)\)\)  
> console.log\('sign arg2===》', jbhexdump\(arg2\)\)  
> console.log\('sign arg2===》', Java.use\("java.lang.String"\).\$new\(Java.array\('byte', arg2\)\)\)  
> console.log\(Java.use\("java.lang.String"\).\$new\(arg2\)\)