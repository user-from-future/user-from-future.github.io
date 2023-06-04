## 前言

搞安卓安全分析和正向开发，可能会有这样的需求，要保持某个或者多个app常驻手机，而不管国内还是国外的手机，我都遇到过会有所谓的节省手机资源或者省电的默认策略，会在后台杀死进程，而这样就会导致，我们的任务没了。就很尴尬

## 使用场景

- 用sekiro部署了一个主动调用方案
- 需要某个app常驻监听某个活动，比如app作为服务端
- app需要在某个时间段定时启动

## shell代码

在给代码之前，你需要知道，你要启动的app的activity名字，用以下终端命令可知

```bash
adb shell dumpsys window | findstr mCurrentFocus  # windows
adb shell dumpsys window | grep mCurrentFocus     # linux,macos
```

注意，以下的代码可能都需要root权限，我没在未root手机上试过。

代码：这段代码是保证某两个app能一直常驻在手机上，force-stop是杀死进程的意思，am start是启动app并进入指定activity界面的意思

```bash
#!/bin/bash





while true

do

    am force-stop com.geekbyte.nft18hook

    sleep 2

    am force-stop art.geekbyte.slhn

    sleep 2

    am start -n com.geekbyte.xxx/com.geekbyte.xxx.MainActivity

    sleep 2

    am start -n art.geekbyte.xxx/com.geekbyte.art.app.activity.AppMainActivity

    sleep 1800

done
```

代码2：这段代码是保证指定的两个app能在早上8点、9点的时间里，启动app，保活1800秒，其他时间则不启动， 每300间隔启动一次

```bash
#!/bin/bash



targetTime="8"

targetTime2="08"

targetTime3="9"

targetTime4="09"



while true

do

  ctime=`date +"%H"`

  echo $targetTime

  echo $targetTime2

  echo $targetTime3

  echo $targetTime4

  echo `date +"%H:%M:%S"`

  if [[ "${ctime}" == "${targetTime}" || "${ctime}" == "${targetTime2}" || "${ctime}" == "${targetTime3}" || "${ctime}" == "${targetTime4}" ]];

    then

      am force-stop com.geekbyte.xxx

      sleep 2

      am force-stop com.tencent.xxx

      sleep 2

      am start -n com.geekbyte.xxx/com.geekbyte.xxx.MainActivity

      sleep 2

      am start -n com.tencent.xxx/com.tencent.xxx.ui.LauncherUI

      sleep 1800

  else

    echo "the time is wrong, it is forbidden to run"

  fi

  sleep 300

done
```

## 注意

以上的代码，如果你启动的app界面，有可能是带参数的Intent，那么以上的代码则不适用，会报错，你需要去逆向找出参数，如果参数很复杂，比如是一个实例化的安卓对象，那需要用其他模式了

## 启动

在你手机上，装一个termux终端软件，这个app启动shell命令需要root权限，然后用termux进入到你的shell脚本目录，用 【sh  xx.sh】 启动即可

## 结语

然后你就可以愉快的专注到你的业务逻辑了