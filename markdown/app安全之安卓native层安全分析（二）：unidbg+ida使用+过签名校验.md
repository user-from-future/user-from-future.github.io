## 前言

继续跟着龙哥的unidbg学习： [SO入门实战教程二：calculateS\_so \_白龙\~的博客-CSDN博客](https://blog.csdn.net/qq_38851536/article/details/117419041 "SO入门实战教程二：calculateS_so _白龙~的博客-CSDN博客")

还是那句，我会借鉴龙哥的文章，以一个初学者的角度，加上自己的理解，把内容丰富一下，尽量做到不在龙哥的基础上画蛇添足，哈哈。感谢观看的朋友

## 分析

首先抓包分析：

![](https://img-blog.csdnimg.cn/img_convert/44fd71dbf0823c407060837a6d7a0cd2.png)

其中，里面的s就是今天的需要分析的加密参数了。

## 调试

老样子，打开jadx，发现没壳，可以的，直接看吧，拿着这几个参数一顿搜，直接搜【p】

![](https://img-blog.csdnimg.cn/img_convert/89961751354610ce7761f0063bea5543.png)

感觉有两个地方很可疑，进去一看：

![](https://img-blog.csdnimg.cn/img_convert/f339906dec44f804760ab360b4596d04.png)

跟下调用栈，很快就找到这里：

![](https://img-blog.csdnimg.cn/img_convert/024f228fa13bad998a576bacd32bce09.png)

ok，用objection hook下，发现确实调用了这里

![](https://img-blog.csdnimg.cn/img_convert/76eb70b2af4df9d6053597797fdc9279.png)

再仔细看看这里，明显这里很奇怪了

![](https://img-blog.csdnimg.cn/img_convert/9dfdbf1499179f7c91ba93b70ac4eb3d.png)

![](https://img-blog.csdnimg.cn/img_convert/0427a2d08df424b9605f2204a3095414.png)

ok，终于到这里了，这里就跟龙哥给的位置一致了

![](https://img-blog.csdnimg.cn/img_convert/a44237fd9b4f704c8356191473bcfb73.png)

先不急着用unidbg，先调试下，hook下这个方法，哎哟，我擦，这直接就对上了

![](https://img-blog.csdnimg.cn/img_convert/962a23d2b9822a1960482996d57b25dc.png)

看看这三个参数 ，一个是context，上下文，又叫寄存器，第二个是账号密码加起来，第三个就是一个特殊的值，大概率是加的盐，刺激。

## unidbg调试

### 1.ida分析

先用ida打开看看：发现是静态注册的，可以的

![](https://img-blog.csdnimg.cn/img_convert/fd0cf19d8454924b04c47843f05ffdc0.png)

选中a1

![](https://img-blog.csdnimg.cn/img_convert/299e3912e09e7230e210b254b41e76ee.png)

然后按键盘【y】 ，把第一个参数的类型改成JNIEnv \*，这样就可以更好的反编译c代码：

![](https://img-blog.csdnimg.cn/img_convert/ef991328508d6c80fccdcda06a61a898.png)

点ok，瞬间这段代码的可读性就更强了

![](https://img-blog.csdnimg.cn/img_convert/859e9b0a734030dec30c652576e02393.png)

大概看了一眼，反正前面有个if判断，然后就进入主逻辑，然后返回

选到这个函数

![](https://img-blog.csdnimg.cn/img_convert/2509b23829b3de25659357311f0012b8.png)

然后按【tab】 键：这个0x1E7C就是这个函数的地址了。记一下，后面会用到

![](https://img-blog.csdnimg.cn/img_convert/dcf9963af495dff9629cd72f03e51bbe.png)

### 2.搭架子

开始搭建unidbg的架子，新建一个文件，然后把apk和目标so文件放进去

![](https://img-blog.csdnimg.cn/img_convert/fc40459ec109cba742d2cd5bb8c8b304.png)

先把架子搭起来，这里我们直接复制前面oasis的：

```java
package com.weibo;



import com.github.unidbg.AndroidEmulator;

import com.github.unidbg.Module;

import com.github.unidbg.linux.android.AndroidEmulatorBuilder;

import com.github.unidbg.linux.android.AndroidResolver;

import com.github.unidbg.linux.android.dvm.*;

import com.github.unidbg.memory.Memory;

import com.github.unidbg.pointer.UnidbgPointer;

import com.github.unidbg.utils.Inspector;

import com.sun.jna.Pointer;

import keystone.Keystone;

import keystone.KeystoneArchitecture;

import keystone.KeystoneEncoded;

import keystone.KeystoneMode;



import java.io.File;

import java.util.ArrayList;

import java.util.Arrays;

import java.util.List;



public class international extends AbstractJni {

    private final AndroidEmulator emulator;

    private final VM vm;

    private final Module module;



    international() {

        // 创建模拟器实例,进程名建议依照实际进程名填写，可以规避针对进程名的校验

        emulator = AndroidEmulatorBuilder.for32Bit().setProcessName("com.weibo.international").build();

        // 获取模拟器的内存操作接口

        final Memory memory = emulator.getMemory();

        // 设置系统类库解析

        memory.setLibraryResolver(new AndroidResolver(23));

        // 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作

        vm = emulator.createDalvikVM(new File("unidbg-android\\src\\test\\java\\com\\weibo\\sinaInternational.apk"));

        // 加载目标SO

        DalvikModule dm = vm.loadLibrary(new File("unidbg-android\\src\\test\\java\\com\\weibo\\libutility.so"), true); // 加载so到虚拟内存

        //获取本SO模块的句柄,后续需要用它

        module = dm.getModule();

        vm.setJni(this); // 设置JNI

        vm.setVerbose(true); // 打印日志

//        dm.callJNI_OnLoad(emulator); // 调用JNI OnLoad

    }



    public static void main(String[] args) {

        international test = new international();

        System.out.println(test);

    }

     



}
```

![](https://img-blog.csdnimg.cn/img_convert/c5c57201b8f1393a72cc0d7518360952.png)

运行下，没啥问题

![](https://img-blog.csdnimg.cn/img_convert/35de8ebd886a35377054580a3d504c2f.png)

### 2.地址调用

方法先写好，参数还是用list，然后用vm.addLocalObject包装一下添加进去，第一个参数是context，直接给个空就行，剩下的参数直接复制hook到的放进去

![](https://img-blog.csdnimg.cn/img_convert/b617e753041d431487bcf03f71401c28.png)

好的，现在运行一下

![](https://img-blog.csdnimg.cn/img_convert/a7ff7fead1eacf229289651b5109a4c3.png)

### 3.找异常执行原因——签名校验，以及绕过

可以的，跟龙哥的博客一样，报错了，然后找找日志，看看上面的一个

![](https://img-blog.csdnimg.cn/img_convert/414a33790c8c702cfb4ea30dcffe5d02.png)

复制这个地址，ida里，按键盘【g】输入地址跳转过去

![](https://img-blog.csdnimg.cn/img_convert/bba3af724ec2e200bf20d4c7f87964c0.png)

跳转到这里：

![](https://img-blog.csdnimg.cn/img_convert/f225682528bf72d97cb358dc7ff3bd27.png)

改下JNIEnv

![](https://img-blog.csdnimg.cn/img_convert/b37a92b154a04da74d5f273b3830b1d2.png)

好的，根据龙哥说的，有packagemanagger之类的，大概率是签名检测

![](https://img-blog.csdnimg.cn/img_convert/fccda9b22d39c592e41c9a4c98d92078.png)

选中这个方法，按【x】找交叉引用

![](https://img-blog.csdnimg.cn/img_convert/bbf6ac2f8ceb8eab9d24f217567f2ddf.png)

很有缘分的又看到了这个sub\_1c60

![](https://img-blog.csdnimg.cn/img_convert/5263d90f2045c78d925799e6e177ce48.png)

进去看：

![](https://img-blog.csdnimg.cn/img_convert/954bea68912febb71a5910b83f21adb6.png)

再回到这里，应该就是这个判断了

![](https://img-blog.csdnimg.cn/img_convert/1dfc94efccce6f86a9da6ac08fbf2d67.png)

按下tab键，记住地址，就是这个0xFFF7EBFE了

![](https://img-blog.csdnimg.cn/img_convert/8f7323427c56d4961de07c30cba03a91.png)

要能返回是true才给过，ok，直接把这里hook下，也就是直接这么改下，

![](https://img-blog.csdnimg.cn/img_convert/d4e5eaeb9066252f0acd38cdb9f22807.png)

在java层里，我们直接hook这个方法修改返回值为1就行，但是在这里，是在so里面，怎么搞呢？根据龙哥说的，用arm指令修改就行

> [Online ARM to HEX Converter \(armconverter.com\)](<https://armconverter.com/?code=mov R0,1> "Online ARM to HEX Converter (armconverter.com)")

![](https://img-blog.csdnimg.cn/img_convert/690447d7a949ee85e79ab7b3011d0acb.png)

用上面的地址，写个过验证的：

![](https://img-blog.csdnimg.cn/img_convert/e35965d34a229ba47f21c22a6c2e9188.png)

龙哥还给了另一种patch的方法：

```java
public void patchverfify(){

        int patchCode = 0x4FF00100;

        emulator.getMemory().pointer(module.base+0x1E86).setInt(0,patchCode);

    }



    public void patchVerify1(){

        Pointer pointer = UnidbgPointer.pointer(emulator, module.base + 0x1E86);

        assert pointer != null;

        byte[] code = pointer.getByteArray(0, 4);

        if (!Arrays.equals(code, new byte[]{ (byte)0xFF, (byte) 0xF7, (byte) 0xEB, (byte) 0xFE })) { // BL sub_1C60

            throw new IllegalStateException(Inspector.inspectString(code, "patch32 code=" + Arrays.toString(code)));

        }

        try (Keystone keystone = new Keystone(KeystoneArchitecture.Arm, KeystoneMode.ArmThumb)) {

            KeystoneEncoded encoded = keystone.assemble("mov r0,1");

            byte[] patch = encoded.getMachineCode();

            if (patch.length != code.length) {

                throw new IllegalStateException(Inspector.inspectString(patch, "patch32 length=" + patch.length));

            }

            pointer.write(0, patch, 0, patch.length);

        }

    }
```

然后执行下，ok，这就出来了，舒服：

![](https://img-blog.csdnimg.cn/img_convert/972b4fef300dd65927890e2a5398ef57.png)

但是好像跟hook到的返回不一样：

![](https://img-blog.csdnimg.cn/img_convert/f4bd8acd80a52c13486d03c4e5213370.png)

不急，再来看看，擦，复制的时候，激动了，把逗号复制进去了

![](https://img-blog.csdnimg.cn/img_convert/62ac520f3a72407a0fd350d564a0c326.png)

ok，这下对上了，跟hook的结果一致

![](https://img-blog.csdnimg.cn/img_convert/4416ce09d73921a65eb6ef7f1ca40da5.png)

代码

```java
package com.weibo;



import com.github.unidbg.AndroidEmulator;

import com.github.unidbg.Module;

import com.github.unidbg.linux.android.AndroidEmulatorBuilder;

import com.github.unidbg.linux.android.AndroidResolver;

import com.github.unidbg.linux.android.dvm.*;

import com.github.unidbg.memory.Memory;

import com.github.unidbg.pointer.UnidbgPointer;

import com.github.unidbg.utils.Inspector;

import com.sun.jna.Pointer;

import keystone.Keystone;

import keystone.KeystoneArchitecture;

import keystone.KeystoneEncoded;

import keystone.KeystoneMode;



import java.io.File;

import java.util.ArrayList;

import java.util.Arrays;

import java.util.List;



public class international extends AbstractJni {

    private final AndroidEmulator emulator;

    private final VM vm;

    private final Module module;



    international() {

        // 创建模拟器实例,进程名建议依照实际进程名填写，可以规避针对进程名的校验

        emulator = AndroidEmulatorBuilder.for32Bit().setProcessName("com.weibo.international").build();

        // 获取模拟器的内存操作接口

        final Memory memory = emulator.getMemory();

        // 设置系统类库解析

        memory.setLibraryResolver(new AndroidResolver(23));

        // 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作

        vm = emulator.createDalvikVM(new File("unidbg-android\\src\\test\\java\\com\\weibo\\sinaInternational.apk"));

        // 加载目标SO

        DalvikModule dm = vm.loadLibrary(new File("unidbg-android\\src\\test\\java\\com\\weibo\\libutility.so"), true); // 加载so到虚拟内存

        //获取本SO模块的句柄,后续需要用它

        module = dm.getModule();

        vm.setJni(this); // 设置JNI

        vm.setVerbose(true); // 打印日志

//        dm.callJNI_OnLoad(emulator); // 调用JNI OnLoad

    }



    public static void main(String[] args) {

        international test = new international();

        test.patchverfify();

        System.out.println(test.calculateS());

    }

    public void patchverfify(){

        int patchCode = 0x4FF00100;

        emulator.getMemory().pointer(module.base+0x1E86).setInt(0,patchCode);

    }



    public void patchVerify1(){

        Pointer pointer = UnidbgPointer.pointer(emulator, module.base + 0x1E86);

        assert pointer != null;

        byte[] code = pointer.getByteArray(0, 4);

        if (!Arrays.equals(code, new byte[]{ (byte)0xFF, (byte) 0xF7, (byte) 0xEB, (byte) 0xFE })) { // BL sub_1C60

            throw new IllegalStateException(Inspector.inspectString(code, "patch32 code=" + Arrays.toString(code)));

        }

        try (Keystone keystone = new Keystone(KeystoneArchitecture.Arm, KeystoneMode.ArmThumb)) {

            KeystoneEncoded encoded = keystone.assemble("mov r0,1");

            byte[] patch = encoded.getMachineCode();

            if (patch.length != code.length) {

                throw new IllegalStateException(Inspector.inspectString(patch, "patch32 length=" + patch.length));

            }

            pointer.write(0, patch, 0, patch.length);

        }

    }

    public String calculateS() {

        List<Object> list = new ArrayList<>(10);

        list.add(vm.getJNIEnv()); // arg1,env

        list.add(0); // arg2,jobject

        DvmObject<?> context = vm.resolveClass("android/content/Context").newObject(null);

        list.add(vm.addGlobalObject(context));

        list.add(vm.addLocalObject(new StringObject(vm, "135691695686123456789")));

        list.add(vm.addLocalObject(new StringObject(vm, "CypCHG2kSlRkdvr2RG1QF8b2lCWXl7k7")));

        Number number = module.callFunction(emulator, 0x1E7C + 1, list.toArray());



        String result = vm.getObject(number.intValue()).getValue().toString();

        return result;

    }





}
```

### 4.符号调用

前面都是地址调用，而龙哥自己也说过，他喜欢用地址调用。不过这里，总要学习下，怎么符号调用

怎么找符号，首选选中这个方法：

![](https://img-blog.csdnimg.cn/img_convert/a3e9a949c5936a101e47d7ff9ebee8a5.png)

然后按tab键，进入汇编页面：

![](https://img-blog.csdnimg.cn/img_convert/28b1bf69b419e84e87d5e868aa8eed95.png)

然后再按空格，然后这个export就是要用的符号表了，注意了，这是只是刚好都一样，很多时候，这三个，不一样的，找准export的才行

![](https://img-blog.csdnimg.cn/img_convert/3db5bf4e77c42eeaf20c11e9d6552ec2.png)

开始调用，就换了下名字，其他都没变，对比两个不同的调用方式，结果一样，没毛病

![](https://img-blog.csdnimg.cn/img_convert/eed011a180e404517dc522543cebda50.png)

代码：

```java
package com.weibo;



import com.github.unidbg.AndroidEmulator;

import com.github.unidbg.Module;

import com.github.unidbg.linux.android.AndroidEmulatorBuilder;

import com.github.unidbg.linux.android.AndroidResolver;

import com.github.unidbg.linux.android.dvm.*;

import com.github.unidbg.memory.Memory;

import com.github.unidbg.pointer.UnidbgPointer;

import com.github.unidbg.utils.Inspector;

import com.sun.jna.Pointer;

import keystone.Keystone;

import keystone.KeystoneArchitecture;

import keystone.KeystoneEncoded;

import keystone.KeystoneMode;



import java.io.File;

import java.util.ArrayList;

import java.util.Arrays;

import java.util.List;



public class international extends AbstractJni {

    private final AndroidEmulator emulator;

    private final VM vm;

    private final Module module;



    international() {

        // 创建模拟器实例,进程名建议依照实际进程名填写，可以规避针对进程名的校验

        emulator = AndroidEmulatorBuilder.for32Bit().setProcessName("com.weibo.international").build();

        // 获取模拟器的内存操作接口

        final Memory memory = emulator.getMemory();

        // 设置系统类库解析

        memory.setLibraryResolver(new AndroidResolver(23));

        // 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作

        vm = emulator.createDalvikVM(new File("unidbg-android\\src\\test\\java\\com\\weibo\\sinaInternational.apk"));

        // 加载目标SO

        DalvikModule dm = vm.loadLibrary(new File("unidbg-android\\src\\test\\java\\com\\weibo\\libutility.so"), true); // 加载so到虚拟内存

        //获取本SO模块的句柄,后续需要用它

        module = dm.getModule();

        vm.setJni(this); // 设置JNI

        vm.setVerbose(true); // 打印日志

//        dm.callJNI_OnLoad(emulator); // 调用JNI OnLoad

    }



    public static void main(String[] args) {

        international test = new international();

        test.patchverfify();

        System.out.println("offset===>:" + test.calculateS());

        System.out.println("symbol===>:" + test.calculateS1());

    }



     

     

    public String calculateS1() {

        List<Object> list = new ArrayList<>(10);

        list.add(vm.getJNIEnv()); // arg1,env

        list.add(0); // arg2,jobject

        DvmObject<?> context = vm.resolveClass("android/content/Context").newObject(null);

        list.add(vm.addGlobalObject(context));

        list.add(vm.addLocalObject(new StringObject(vm, "135691695686123456789")));

        list.add(vm.addLocalObject(new StringObject(vm, "CypCHG2kSlRkdvr2RG1QF8b2lCWXl7k7")));

        Number number = module.callFunction(emulator, "Java_com_sina_weibo_security_WeiboSecurityUtils_calculateS", list.toArray());

        String result = vm.getObject(number.intValue()).getValue().toString();

        return result;

    }



    public void patchverfify() {

        int patchCode = 0x4FF00100;

        emulator.getMemory().pointer(module.base + 0x1E86).setInt(0, patchCode);

    }



    public void patchVerify1() {

        Pointer pointer = UnidbgPointer.pointer(emulator, module.base + 0x1E86);

        assert pointer != null;

        byte[] code = pointer.getByteArray(0, 4);

        if (!Arrays.equals(code, new byte[]{(byte) 0xFF, (byte) 0xF7, (byte) 0xEB, (byte) 0xFE})) { // BL sub_1C60

            throw new IllegalStateException(Inspector.inspectString(code, "patch32 code=" + Arrays.toString(code)));

        }

        try (Keystone keystone = new Keystone(KeystoneArchitecture.Arm, KeystoneMode.ArmThumb)) {

            KeystoneEncoded encoded = keystone.assemble("mov r0,1");

            byte[] patch = encoded.getMachineCode();

            if (patch.length != code.length) {

                throw new IllegalStateException(Inspector.inspectString(patch, "patch32 length=" + patch.length));

            }

            pointer.write(0, patch, 0, patch.length);

        }

    }



    public String calculateS() {

        List<Object> list = new ArrayList<>(10);

        list.add(vm.getJNIEnv()); // arg1,env

        list.add(0); // arg2,jobject

        DvmObject<?> context = vm.resolveClass("android/content/Context").newObject(null);

        list.add(vm.addGlobalObject(context));

        list.add(vm.addLocalObject(new StringObject(vm, "135691695686123456789")));

        list.add(vm.addLocalObject(new StringObject(vm, "CypCHG2kSlRkdvr2RG1QF8b2lCWXl7k7")));

        Number number = module.callFunction(emulator, 0x1E7C + 1, list.toArray());



        String result = vm.getObject(number.intValue()).getValue().toString();

        return result;

    }





}
```

## 部署

有没有想过，搞的这个，如果想部署成一个web服务，以供后续的业务代码调用呢？不然这么个项目，部署在服务器上，然后 每次启动都调用unidbg，说实话不是太现实，所以这里就需要把unidbg打包成jar包：

最实用且简单的方法:

1.

![](https://img-blog.csdnimg.cn/img_convert/90c0d249a9338850c0150a9371dbbac4.png)

2.

![](https://img-blog.csdnimg.cn/img_convert/dc617b1c88a02790198503a417b95e01.png)

3.

![](https://img-blog.csdnimg.cn/img_convert/add128ca644683f231c2100e6c7dd61e.png)

![](https://img-blog.csdnimg.cn/img_convert/af2a69030b2fd60596d61ffe6f712e73.png)

4.

![](https://img-blog.csdnimg.cn/img_convert/b0fb337480e907bb5a7868bdb496a472.png)

![](https://img-blog.csdnimg.cn/img_convert/f9c9870c4366d667c8b217f0d39a6f6f.png)

5.

![](https://img-blog.csdnimg.cn/img_convert/42e333994a080f1c34e5df87757fc26c.png)

6.

![](https://img-blog.csdnimg.cn/img_convert/49ef2879aaab4be27556381aafbea2f8.png)

等待结果：

![](https://img-blog.csdnimg.cn/img_convert/9964fd3a8f839bc7ee62a408fb9a832d.png)

然后根目录就会多一个out目录：

![](https://img-blog.csdnimg.cn/img_convert/b8bc258e148899960298a3bee323ab7c.png)

7.这个文件就是最后能直接通过java执行的：

![](https://img-blog.csdnimg.cn/img_convert/90b020dab4c88051911a4919982f0857.png)

但是有个问题，我们写的apk和so文件，给定的地址是unidgb-android/src里的

![](https://img-blog.csdnimg.cn/img_convert/40f264566acd4c3b9c9e4c9688a120a3.png)

要打包的话，就得改下路径，改成相对路径

![](https://img-blog.csdnimg.cn/img_convert/2cd7bd9cc7c26b11410214abd3e4c3b3.png)

然后删除out文件夹，重新上面的打包操作，然后把apk和so放到跟unidbg-jar同级的目录：

![](https://img-blog.csdnimg.cn/img_convert/e6beff6cf23035866c97d063ca437a44.png)

这样就好了， 终端执行看看，很好，结果也有的，这样就可以把这个out包整个打包 ，然后部署到服务器上就行了。

![](https://img-blog.csdnimg.cn/img_convert/80c0daf9c9ec808c85f7bde5d0c751cd.png)

还有更多的打包方式：

> [Unidbg打Jar包方式\_unidbg打包成jar\_Forgo7ten的博客-CSDN博客](https://blog.csdn.net/Palmer9/article/details/125220599 "Unidbg打Jar包方式_unidbg打包成jar_Forgo7ten的博客-CSDN博客")

## 知识点总结

1.ida，按y修改JNIEnv，IDA 7.5之前，JNIEnv需要导入jni.h，7.5之后不需要导入jni.h文件

2.ida，按g 输入地址跳转

3.ida，按x，查看交叉引用

4.ida，optional->generel，把这个改成4，可以查看架构模式，arm架构是固定4位，thumb架构是混合的

![](https://img-blog.csdnimg.cn/img_convert/838c4ae23436b697d8e1a493105c634d.png)

5\. 有签名校验的需要去patch，patch的时候不能+1，只有运行和hook的时候才+1

6.setJNIload方法只有动态注册方法的时候才执行，静态注册的不用执行

7.keystone，是将汇编语言转成地址的。capstone是将地址转成汇编语言的

8.在线汇编和地址互转的网站：https://armconverter.com/

9.参数的基本类型，比如int，long等，其他的对象类型一律要手动 addLocalObject，其中context对象用：

```java
DvmObject<?> context = vm.resolveClass("android/content/Context").newObject(null);// context

list.add(vm.addLocalObject(context))
```

10.找准一行汇编，Alt+G快捷键，查看架构类型，值为1则是thumb，为0则是arm，如果ida解析有误，可以手动修改这个值

11.RM调用约定，入参前四个分别通过R0-R3调用，返回值通过R0返回

12.unidbg的项目可以打包成java包执行

## 结语

说实话，目前来说，不难，得后续的大厂项目才会很难