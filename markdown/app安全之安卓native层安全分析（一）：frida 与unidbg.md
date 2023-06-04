## 前言

这个专题是根据白龙，龙哥的unidbg博客的案例，进行从0开始到安全分析的流程，核心部分会借鉴龙哥的unidbg，通过借鉴大佬的思路，完整的分析某个so层的加密参数

各位朋友也可以直接读龙哥的博客，我只是用我的角度进一步加工一下

原文地址： [SO入门实战教程一：OASIS\_so学习路线\_白龙\~的博客-CSDN博客](https://blog.csdn.net/qq_38851536/article/details/117418582 "SO入门实战教程一：OASIS_so学习路线_白龙~的博客-CSDN博客")

## 分析

首先拿到这个app，安装啥的就不多说了。

进入到注册界面：

![](https://img-blog.csdnimg.cn/img_convert/35ca1b0c7190e7585b95c0d3d3a764ce.png)

点击获取验证码，然后这边抓包工具抓到的包：

![](https://img-blog.csdnimg.cn/img_convert/bfe9aebaf0b238cc4b8394b52ef6d049.png)

然后，这里面的【sign】就是今天的重点了。

用神秘的工具脱壳完之后，有如下的dex：

![](https://img-blog.csdnimg.cn/img_convert/c575f30203e16746bdaf84291c17a47c.png)

把这些dex，全部选中，拖进jadx

![](https://img-blog.csdnimg.cn/img_convert/f9b8868eceef8b30dcaa6853a2a4db57.png)

## 快速定位

接下来开始找sign所在的位置，先看看抓包工具这边的参数，根据这些键名，一顿搜，总能找到一些

![](https://img-blog.csdnimg.cn/img_convert/e519284167bdd94876041aa77c55003e.png)

先搜下【sign】，虽然搜出来的结果不多，但是感觉有很多干扰项

![](https://img-blog.csdnimg.cn/img_convert/c656b8690d732b9dde64a7e3939629d2.png)

进最后那个，到这里，

![](https://img-blog.csdnimg.cn/img_convert/32321ca47dd1a97290c2257b75de05c3.png)

这些参数看着很像，用frida hook，得知，发现并没有走到这里

搜【ua】

![](https://img-blog.csdnimg.cn/img_convert/565bdbd8b84700e5b550a1123bf997d2.png)

进到这里，发现很可疑，好多参数都对上了

![](https://img-blog.csdnimg.cn/img_convert/e17bde2370dbae51d4e4d54e114ef415.png)

啥都不说，先用objection hook下，然后app端点击【重新发送】看看：

![](https://img-blog.csdnimg.cn/img_convert/964ceea08cf7c0b93c270395e7c5fcca.png)

可以，这不直接就定位逻辑了

但是我们要找【sign】，所以还得再看看，jadx查看发现，这个方法反编译效果不太好，问题不大，记住这个dex名，然后用GDA 打开这个dex，这就挺好，基本都反编译出来了

![](https://img-blog.csdnimg.cn/img_convert/6cfd48a2dc2819dcdbf4163dd0426ebf.png)

反正看着确实可疑，但是这三个方法，还不确定到底是是哪个方法里有【sign】部分，所以这里直接hook 它这个a,b，c三个方法，然后app点下重新获取

![](https://img-blog.csdnimg.cn/img_convert/b9ea879b8d944b90fd7eaaac2cf820f3.png)

![](https://img-blog.csdnimg.cn/img_convert/5c8b22363f90773e3705aad18c3079e7.png)

![](https://img-blog.csdnimg.cn/img_convert/8e415e6b78108cce7b22d727f2705723.png)

ok，发现最后的方法里基于有我们要的sign，这边再来下，用抓包工具对比下，谨慎一点，别搞半天没搞对位置：

重新hook下，然后抓包工具打开看看

![](https://img-blog.csdnimg.cn/img_convert/38da999f92498675d66807433d353464.png)

对上了，ok，就是这里了【g.a.c.g.c.a】

在jadx里，反编译失败：

![](https://img-blog.csdnimg.cn/img_convert/ff068af4f405ec3f6a3578b5992e80c8.png)

问题不大，用GDA看：

![](https://img-blog.csdnimg.cn/img_convert/9f36d57884e940bc222d5f72f1920238.png)

这不就越来越接近了吗，嘻嘻，点进去：

![](https://img-blog.csdnimg.cn/img_convert/c90e2b50081ce956f75f5aadbd03f2fa.png)

hook下这个c方法看看，ok，对上了

![](https://img-blog.csdnimg.cn/img_convert/c43ee6e9d222ffe7a0978b4777469e34.png)

接着一顿分析后再进入这里

![](https://img-blog.csdnimg.cn/img_convert/8cc73e6ef7a8f5b0774e9a60ec8e0642.png)

ok，进到了native层，那么核心的逻辑就在这里了。

像这种，常规的方法怎么解决呢？

- 如果你是为了拿数据，赶工期的话，那你可以直接主动调用这个方法
- 如果你是为了纯算还原的话，那就把这个so文件拖进ida一顿分析了
- 那么假如，这个方法的纯算很复杂，而你又不想主动调用，感觉很low的话，那你就可以尝试用unidbg了

unidbg，就是本系列文章的重点了。

## unidbg简介

### 什么是unidbg

unidbg 是一个基于 unicorn 的安全分析工具，可以实现黑盒调用安卓和 iOS 中的 so 文件

unidbg是凯神写的一个开源的java项目

### 使用场景

因为现在的大多数 app 把核心的加密算法放到了 so 文件中（比如本例的app），你要想破解签名算法，必须能够破解 so 文件。但C++ 的安全分析远比 Java 的安全分析要难得多，有各种混淆啊，ollvm啥的，所以好多时候是没法纯算还原破解的

那么你是否有过一个想法，能不能把安卓的环境模拟出来，但是又脱离了安卓真机的环境，就可以直接使用so里的方法呢？

unidbg 就是这样一个工具，它模拟好了好几种虚拟环境，他不需要直接运行 app，也无需安全分析so 文件，而是直接找到对应的 JNI 接口，然后用 unicorn 引擎（也不止这一个引擎）直接执行这个 so 文件，所以效率也比较高。

### 配置unidbg

1.首先，用git 把这个项目clone 下来：

> [zhkl0228/unidbg: Allows you to emulate an Android native library, and an experimental iOS emulation \(github.com\)](https://github.com/zhkl0228/unidbg "zhkl0228/unidbg: Allows you to emulate an Android native library, and an experimental iOS emulation (github.com)")

2.再用idea（写java项目那个编辑器）打开，

![](https://img-blog.csdnimg.cn/img_convert/dddb8173678dae7973bd8da48e0a2737.png)

首次打开，右下角会下载很多依赖环境，等待即可

![](https://img-blog.csdnimg.cn/img_convert/97ec4db71bff5d36ffeb8db651419e92.png)

然后随便找一个项目，执行下main方法，有正常输出，说明环境配置好了：

![](https://img-blog.csdnimg.cn/img_convert/fb9506b88f06d441c4cd42be4d2b9052.png)

执行结果：

![](https://img-blog.csdnimg.cn/img_convert/c3dbead1d17fd5cc8e174434edf3967e.png)

ok，这就很nice。

## frida调试

接下来，先用ida 打开那个目标so文件

![](https://img-blog.csdnimg.cn/img_convert/908174159b993b828899ec019786b8f3.png)

等左下角这个数据没有再变的时候，说明加载好了

![](https://img-blog.csdnimg.cn/img_convert/aa68d2ff48522b7e307fac8baf80585f.png)

接下来找导出表【export】，或者在左边的栏里搜【java】

![](https://img-blog.csdnimg.cn/img_convert/cea0a363100157087c965954206d998c.png)

发现并没有任何东西，那么这里就是动态注册的so方法了。

### 什么是动态注册、静态注册

静态注册（又叫静态绑定），就是，so的方法名直接export导出表里，且命名格式为【java\_app包名\_方法名】，比如 java\_com\_sina\_oasxxx\_nativeapi\_s

动态注册（又叫动态绑定）就是export到处表里没有的就是动态注册。

那么这里我们的目标方法就是动态注册的了，那咋办，看看JNI\_load方法：

![](https://img-blog.csdnimg.cn/img_convert/d9d61469a83db10db836181746a60585.png)

按下【tab】键，会由上面的汇编代码反编译为c代码：

![](https://img-blog.csdnimg.cn/img_convert/a75a62521a1c0df9c8e0916146fb566a.png)

再按下【\\】反斜杠，可读性更强点：

![](https://img-blog.csdnimg.cn/img_convert/6c151e947deee77f9cf4ae1883e43eac.png)

但是这里发现，一顿while 和if，根据龙哥的博客说的，大概率是ollvm混淆。那咋办？

用yang神的hook\_native脚本来hook出目标函数的偏移地址 ，然后加上so的基址，就可以得到目标函数的地址了（看是thumb还是arm，thumb要加1，arm不用）

> [frida\_hook\_libart/hook\_RegisterNatives.js at master · lasting-yang/frida\_hook\_libart \(github.com\)](https://github.com/lasting-yang/frida_hook_libart/blob/master/hook_RegisterNatives.js "frida_hook_libart/hook_RegisterNatives.js at master · lasting-yang/frida_hook_libart (github.com)")

用frida hook一下，结果到这就报错退出了

![](https://img-blog.csdnimg.cn/img_convert/9105e0a5f8257d796ac7199b313b0217.png)

![](https://img-blog.csdnimg.cn/img_convert/110a1e7b22e45e503e20e2657585bbce.png)

就很尴尬，看看，他报的哪个class名，用来过滤下试试：

![](https://img-blog.csdnimg.cn/img_convert/8639498911b224ee160d545f257abcec.png)

重新运行frida试试，可以，这下直接就定位到我们要的方法的位置了：

![](https://img-blog.csdnimg.cn/img_convert/4fc5344d9aef5fd4c25bb2f6b5ec4edf.png)

这里有朋友估计会说，卧槽，这不都出来了吗，这地址，拿着直接用啊，不急，我重启脚本看看：仔细看，fnptr地址变了，fnoffset和后面的jni-load + 的地址没变的

![](https://img-blog.csdnimg.cn/img_convert/4e5c138ce9d190c94e9e6021bf933906.png)

多次hook发现确实如此，因为这里就是上面说的动态注册，所以这个fnptr，也就是so的基址，app没启动一次就会变，但是目标方法的偏移值是不会变的。ok

用frida hook so看看：

相关的hoo so，有个大佬总结的很好：

> [分类: frida | 凡墙总是门 \(kevinspider.github.io\)](https://kevinspider.github.io/frida/frida-hook-so/ "分类: frida | 凡墙总是门 (kevinspider.github.io)")

ok，这里我们hook下，拿下入参和返回值看看：

```javascript
function inline_hook() {

    var so_addr = Module.findBaseAddress("liboasiscore.so");

    console.log("so_addr:", so_addr);

    if (so_addr) {

        var sub = so_addr.add(0x116cc); // 不用加1，是arm架构

        console.log("The addr_0x116cc:", sub);

        Java.perform(function () {

            Interceptor.attach(sub,

                {

                    onEnter: function (args) {

                        console.log("addr_0x116cc OnEnter :", this.context.PC,

                            this.context.x1, this.context.x5,

                            this.context.x10);

                    },

                    onLeave: function (retval) {

                        console.log("retval is :", retval)

                    },

                })

        })

    }

}





setTimeout(inline_hook, 1000)
```

运行结果：

![](https://img-blog.csdnimg.cn/img_convert/415875ac6bf7445e5445453c8ac58539.png)

发现并不可读，没事，反正至少是有了

```javascript
function stringToBytes(str) {

  return hexToBytes(stringToHex(str))

}



function stringToHex(str) {

  return str

    .split('')

    .map(function (c) {

      return ('0' + c.charCodeAt(0).toString(16)).slice(-2)

    })

    .join('')

}



function hexToBytes(hex) {

  for (let bytes = [], c = 0; c < hex.length; c += 2) bytes.push(parseInt(hex.substr(c, 2), 16))

  return bytes

}



function hexToString(hexStr) {

  let hex = hexStr.toString()

  let str = ''

  for (let i = 0; i < hex.length; i += 2) str += String.fromCharCode(parseInt(hex.substr(i, 2), 16))

  return str

}
```

```javascript
function inline_hook() {

    var so_addr = Module.findBaseAddress("liboasiscore.so");

    console.log("so_addr:", so_addr);

    if (so_addr) {

        // var sub = so_addr.add(0x116cc); // 不用加1，是arm架构

        console.log("The addr_0x116cc:", so_addr);

        var ss = "aid=01A8SBOtNRVqsR1ywgkR4tHsZEsgXkGDrgKO2OvFBeThKWZDE.&cfrom=28B5295010&cuid=0&noncestr=L83x8Z40132Wan450y736563n3kmWj&phone=138469655665&platform=ANDROID&timestamp=1681790293128&ua=Xiaomi-MI6__oasis__3.5.8__Android__Android9&version=3.5.8&vid=2010511512550&wm=20004_90024";

        var add_addr = so_addr.add(0x116cc); // 32位需要加1

        var add = new NativeFunction(add_addr, 'pointer', ['pointer', 'int']);

        console.log(add)

        var result = add(stringToByte(Memory.allocUtf8String(ss)), false)

        console.log("add2 result is ->" + result.readCString());

    }



}

function stringToByte(str) {

    var ch, st, re = [];

    for (var i = 0; i < str.length; i++) {

        ch = str.charCodeAt(i);

        st = [];

        do {

            st.push(ch & 0xFF);

            ch = ch >> 8;

        } while (ch);

        re = re.concat(st.reverse());

    }   // return an array of bytes

    return re;

}



setTimeout(inline_hook, 2000)
```

尝试主动调用，调试了很久，就是不行

![](https://img-blog.csdnimg.cn/img_convert/5b649fa5853ae97d39fe260f958a7d4d.png)

突然反应过来，这个so文件有ollvm混淆啊，虽然用yang神的代码hook到了偏移地址，但是他内部可能并不是这些参数，而我们又没法直接分析，那没法了。其实以上的代码，在其他地方是可以用的，

那接下来咋办？上unidbg吧

## unidbg调试

上面用了frida+ida调试，发现有的时候没法搞啊，那么这里，终于要用unidbg来模拟执行生成上面目标app的sign了

先创建一个文件，把该有的都放进去

![](https://img-blog.csdnimg.cn/img_convert/83dd6933b5a4e81b4f42f340ab80da6d.png)

然后再oasis里写代码，照着龙哥的搞就完了：注意文件路径，跟你实际的路径保持一致

```java
package com.sina;



import com.github.unidbg.AndroidEmulator;

import com.github.unidbg.Module;

import com.github.unidbg.linux.android.AndroidEmulatorBuilder;

import com.github.unidbg.linux.android.AndroidResolver;

import com.github.unidbg.linux.android.dvm.AbstractJni;

import com.github.unidbg.linux.android.dvm.DalvikModule;

import com.github.unidbg.linux.android.dvm.VM;

import com.github.unidbg.memory.Memory;



import java.io.File;



public class oasis extends AbstractJni {

    private final AndroidEmulator emulator;

    private final VM vm;

    private final Module module;



    oasis() {

        // 创建模拟器实例,进程名建议依照实际进程名填写，可以规避针对进程名的校验

        emulator = AndroidEmulatorBuilder.for32Bit().setProcessName("com.sina.oasis").build();

        // 获取模拟器的内存操作接口

        final Memory memory = emulator.getMemory();

        // 设置系统类库解析

        memory.setLibraryResolver(new AndroidResolver(23));

        // 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作

        vm = emulator.createDalvikVM(new File("unidbg-android\\src\\test\\java\\com\\sina\\lvzhou.apk"));

        // 加载目标SO

        DalvikModule dm = vm.loadLibrary(new File("unidbg-android\\src\\test\\java\\com\\sina\\liboasiscore.so"), true); // 加载so到虚拟内存

        //获取本SO模块的句柄,后续需要用它

        module = dm.getModule();

        vm.setJni(this); // 设置JNI

        vm.setVerbose(true); // 打印日志



        dm.callJNI_OnLoad(emulator); // 调用JNI OnLoad

    };



    public static void main(String[] args) {

        oasis test = new oasis();

    }

}
```

然后运行一下，没啥问题，说明架子是搭上了

![](https://img-blog.csdnimg.cn/img_convert/ddf6640d36f90dcad3204f70870f88d3.png)

仔细看这里，这样也把我们要的方法的地址拿到了。舒服啊

![](https://img-blog.csdnimg.cn/img_convert/4f7ed9d1f4b91a6de7380471db61117d.png)

用前面frida 的对比，好像不太一样，问题不大

![](https://img-blog.csdnimg.cn/img_convert/6cf582c339bda196e58007240f7a9f78.png)

再来看看代码：

![](https://img-blog.csdnimg.cn/img_convert/d6de83b417336db5442487c575ab0068.png)

再来仔细看看他这个main干了啥：

![](https://img-blog.csdnimg.cn/img_convert/095a1ab9dd1f72aeb95d3fa2203febf6.png)

### 调用

架子搭好了，接下来调用，调用有两种方式，一种是符号（symbol）调用，一种是地址调用，符号调用对应静态注册，地址调用对应动态注册。那么根据前面的解析，这里我们只能选用地址调用了

先看看我们要调用的方法：

![](https://img-blog.csdnimg.cn/img_convert/c27bc186f21936a349dc1ec425879d76.png)

有两个参数，一个byte数组，一个boolean，然后native层的方法，默认前面会自动加两个参数 ，一个jni env，一个jobject

还是上面的frida脚本，先hook下，看看入参和返回：

![](https://img-blog.csdnimg.cn/img_convert/a75324e856469d745aaf2c524ffb4ab8.png)

ok，直接拿着这个str参数的值去unidbg构造：

![](https://img-blog.csdnimg.cn/img_convert/205d56d25a8106668bd8ccb5c0cd6d49.png)

注意，如果你用的旧版，也就是龙哥案例的代码，直接报错：

![](https://img-blog.csdnimg.cn/img_convert/788f2eb4bf7af6b4ad1859e61bc1d1f3.png)

新版得这么用，运行结果：

![](https://img-blog.csdnimg.cn/img_convert/cbf302928ba7e68971e6411760245a73.png)

```java
package com.sina;



import com.github.unidbg.AndroidEmulator;

import com.github.unidbg.Module;

import com.github.unidbg.linux.android.AndroidEmulatorBuilder;

import com.github.unidbg.linux.android.AndroidResolver;

import com.github.unidbg.linux.android.dvm.AbstractJni;

import com.github.unidbg.linux.android.dvm.DalvikModule;

import com.github.unidbg.linux.android.dvm.VM;

import com.github.unidbg.linux.android.dvm.array.ByteArray;

import com.github.unidbg.memory.Memory;

import com.sun.jna.Pointer;



import java.io.File;

import java.nio.charset.StandardCharsets;

import java.util.ArrayList;

import java.util.List;

import java.lang.Number;



public class oasis extends AbstractJni {

    private final AndroidEmulator emulator;

    private final VM vm;

    private final Module module;



    oasis() {

        // 创建模拟器实例,进程名建议依照实际进程名填写，可以规避针对进程名的校验

        emulator = AndroidEmulatorBuilder.for32Bit().setProcessName("com.sina.oasis").build();

        // 获取模拟器的内存操作接口

        final Memory memory = emulator.getMemory();

        // 设置系统类库解析

        memory.setLibraryResolver(new AndroidResolver(23));

        // 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作

        vm = emulator.createDalvikVM(new File("unidbg-android\\src\\test\\java\\com\\sina\\lvzhou.apk"));

        // 加载目标SO

        DalvikModule dm = vm.loadLibrary(new File("unidbg-android\\src\\test\\java\\com\\sina\\liboasiscore.so"), true); // 加载so到虚拟内存

        //获取本SO模块的句柄,后续需要用它

        module = dm.getModule();

        vm.setJni(this); // 设置JNI

        vm.setVerbose(true); // 打印日志



        dm.callJNI_OnLoad(emulator); // 调用JNI OnLoad

    };



    public static void main(String[] args) {

        oasis test = new oasis();

        System.out.println(test.getSign());

    }

    public String getSign(){

        List<Object> list = new ArrayList<>(10);

        list.add(vm.getJNIEnv()); // arg1,env

        list.add(0); // arg2,jobject

        String keywords = "aid=01A8SBOtNRVqsR1ywgkR4tHsZEsgXkGDrgKO2OvFBeThKWZDE.&cfrom=28B5295010&cuid=0&noncestr=L83x8Z40132Wan450y736563n3kmWj&phone=xxxxx&platform=ANDROID&timestamp=1681790293128&ua=Xiaomi-MI6__oasis__3.5.8__Android__Android9&version=3.5.8&vid=2010511512550&wm=20004_90024";

        byte[] keyB = keywords.getBytes(StandardCharsets.UTF_8);

        ByteArray inbarr = new ByteArray(vm,keyB);

        list.add(vm.addGlobalObject(inbarr)); //arg3

        list.add(0); //arg4

//        Number number = module.callFunction(emulator,0xC365,list.toArray())[0];

        Number number = module.callFunction(emulator,0xC365,list.toArray());

        String result = vm.getObject(number.intValue()).getValue().toString();

        return result;

    }

}
```

验证下结果，对上了，舒服：

![](https://img-blog.csdnimg.cn/img_convert/09601d634c68f1251c2800c4287b5a4a.png)

## 结语

unidbg的实用之处就在这里，相信不用我多说，你已经发现了很多妙用之处