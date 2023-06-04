## 前言

继续跟着龙哥的unidbg学习： [SO逆向入门实战教程四：mfw\_so层逆向\_白龙\~的博客-CSDN博客](https://blog.csdn.net/qq_38851536/article/details/117550316 "SO逆向入门实战教程四：mfw_so层逆向_白龙~的博客-CSDN博客")

还是那句，我会借鉴龙哥的文章，以一个初学者的角度，加上自己的理解，把内容丰富一下，尽量做到不在龙哥的基础上画蛇添足，哈哈。感谢观看的朋友

## 分析

首先打开目标app，然后随便搜个东西，抓个包看看：  

![](https://img-blog.csdnimg.cn/img_convert/2b2d5af2ed1ba9665ec1141ef7d7f12b.png)

其中，上面的这个zzzghostsigh参数就是我们要逆向的加密参数了。

### 1.找加密位置

jadx直接搜：

![](https://img-blog.csdnimg.cn/img_convert/1062253df55ad1fed0401aa85ae80cf1.png)

![](https://img-blog.csdnimg.cn/img_convert/d0fa44d1ef800f8b84f8c34d3ddde07f.png)

然后再搜这个【HTTP\_BASE\_PARAM\_GHOSTSIGH】

![](https://img-blog.csdnimg.cn/img_convert/7b1bd4c2cbffd0811edf80d92012fc30.png)

![](https://img-blog.csdnimg.cn/img_convert/9e2e78a8b398b229a62b8cec714089f2.png)

![](https://img-blog.csdnimg.cn/img_convert/dbd4794c600a4f7f179f69fc72a26ce5.png)

![](https://img-blog.csdnimg.cn/img_convert/1c36cec34af66fa3e3c5787edc36047f.png)

![](https://img-blog.csdnimg.cn/img_convert/cf680d1b60f99928b3c2323c8f08d7d5.png)

ok，根据刚才的逻辑跟栈，就是这个xPreAuthencode方法了

objection hook确认：

![](https://img-blog.csdnimg.cn/img_convert/570001ae5575de914c47d89d860fdde0.png)

好的，就是这里了，看到第一个参数是一个context，第二个是哥字符串，感觉像是把url啥的全拼到一起了，第三个参数就是这个app的包名

### 2.ida分析so

把目标so文件拖进去，卧槽，一团乱，也是动态注册的，卧槽

![](https://img-blog.csdnimg.cn/img_convert/2aec10a9e660c56633609933a59e074b.png)

多说无益，直接用unidbg吧

## unidbg调试

### 1.搭架子

```java
package com.mfw;

import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Module;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.AbstractJni;
import com.github.unidbg.linux.android.dvm.DalvikModule;
import com.github.unidbg.linux.android.dvm.VM;
import com.github.unidbg.memory.Memory;

import java.io.File;

public class roadbook extends AbstractJni {
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;

    roadbook() {
        // 创建模拟器实例,进程名建议依照实际进程名填写，可以规避针对进程名的校验
        emulator = AndroidEmulatorBuilder.for32Bit().setProcessName("com.sina.oasis").build();
        // 获取模拟器的内存操作接口
        final Memory memory = emulator.getMemory();
        // 设置系统类库解析
        memory.setLibraryResolver(new AndroidResolver(23));
        // 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作
        vm = emulator.createDalvikVM(new File("unidbg-android\\src\\test\\java\\com\\mfw\\mafengwo.apk"));
        // 加载目标SO
        DalvikModule dm = vm.loadLibrary(new File("unidbg-android\\src\\test\\java\\com\\mfw\\libmfw.so"), true); // 加载so到虚拟内存
        //获取本SO模块的句柄,后续需要用它
        module = dm.getModule();
        vm.setJni(this); // 设置JNI
        vm.setVerbose(true); // 打印日志
        dm.callJNI_OnLoad(emulator); // 调用JNI OnLoad
    }


    public static void main(String[] args) {
        roadbook test = new roadbook();
    }
}
```

运行一下，好像没有啥问题，而且还把地址打印出来了：

![](https://img-blog.csdnimg.cn/img_convert/b016a86641ebe24ba0c6bbe900aa76a1.png)

### 2.主动调用

直接干吧，没啥好说的，把hook的参数复制过去

```java
package com.mfw;

import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Module;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.memory.Memory;

import java.io.File;
import java.util.ArrayList;
import java.util.List;

public class roadbook extends AbstractJni {
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;

    roadbook() {
        // 创建模拟器实例,进程名建议依照实际进程名填写，可以规避针对进程名的校验
        emulator = AndroidEmulatorBuilder.for32Bit().setProcessName("com.sina.oasis").build();
        // 获取模拟器的内存操作接口
        final Memory memory = emulator.getMemory();
        // 设置系统类库解析
        memory.setLibraryResolver(new AndroidResolver(23));
        // 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作
        vm = emulator.createDalvikVM(new File("unidbg-android\\src\\test\\java\\com\\mfw\\mafengwo.apk"));
        // 加载目标SO
        DalvikModule dm = vm.loadLibrary(new File("unidbg-android\\src\\test\\java\\com\\mfw\\libmfw.so"), true); // 加载so到虚拟内存
        //获取本SO模块的句柄,后续需要用它
        module = dm.getModule();
        vm.setJni(this); // 设置JNI
        vm.setVerbose(true); // 打印日志
        dm.callJNI_OnLoad(emulator); // 调用JNI OnLoad
    }


    public static void main(String[] args) {
        roadbook test = new roadbook();
        System.out.println(test.xPreAuthencode());
    }
    public String xPreAuthencode(){
        List<Object> list = new ArrayList<>(10);
        list.add(vm.getJNIEnv());
        list.add(0);
        DvmObject<?> context = vm.resolveClass("android/content/Context").newObject(null);
        list.add(vm.addGlobalObject(context));
        list.add(vm.addLocalObject(new StringObject(vm,"GET&https%3A%2F%2Fmapi.mafengwo.cn%2Frest%2Fapp%2Fv2%2Fmdd%2Fposition%2F104.068237_30.537864&app_code%3Dcom.mfw.roadbook%26app_ver%3D9.3.7%26app_version_code%3D734%26brand%3DXiaomi%26channel_id%3DMFW%26dev_ver%3DD1907.0%26device_id%3D4C%253A49%253AE3%253ACA%253A75%253A35%26device_type%3Dandroid%26hardware_model%3DMI%25206%26has_notch%3D0%26mfwsdk_ver%3D20140507%26o_lat%3D30.537864%26o_lng%3D104.068237%26oauth_consumer_key%3D5%26oauth_nonce%3D0b4bc80a-6496-4e96-82f9-b67d289759d1%26oauth_signature_method%3DHMAC-SHA1%26oauth_timestamp%3D1682412429%26oauth_token%3D0_0969044fd4edf59957f4a39bce9200c6%26oauth_version%3D1.0%26open_udid%3D4C%253A49%253AE3%253ACA%253A75%253A35%26patch_ver%3D3.0%26screen_height%3D1920%26screen_scale%3D2.88%26screen_width%3D1080%26sys_ver%3D9%26time_offset%3D480%26x_auth_mode%3Dclient_auth")));
        list.add(vm.addLocalObject(new StringObject(vm,"com.mfw.roadbook")));
        Number number =  module.callFunction(emulator,0x2e301,list.toArray());
        String result = vm.getObject(number.intValue()).getValue().toString();
        return result;
    }
}
```

然后执行，结果一致

![](https://img-blog.csdnimg.cn/img_convert/7f35eb3780963510188dc2a18ad5841c.png)

就很轻松加愉快

## 知识点总结

第四部分没啥技术知识点，都是老套路