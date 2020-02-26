**电池相关：**

```shell
adb shell dumpsys battery //查看电池状态
adb shell dumpsys battery unplug //断开充电
adb shell dumpsys battery reset //重置
adb shell dumpsys battery set status 2 //设置电池状态，2

adb shell dumpsys procstats //用于查看手机内存的使用情况
```



**Log相关：**

```shell
adb shell cat /proc/sys/kernel/printk
adb logcat -b radio //查看radio缓冲区 日志
adb logcat -b kernel //查看kernel日志
adb logcat -s PowerManagerService //显示PowerManagerService的日志信息

adb logcat *:W //显示所有优先级大于等于“warning”的日志

adb logcat ActivityManager:I PowerManagerService:D *:S //仅输出标记为“ActivityManager”且优先级大于等于“Info”和标记为“PowerManagerService”并且优先级大于等于“Debug”的日志：
```



**monkey 测试**

```shell
adb shell monkey

-p 指定包名
adb shell monkey -p com.htc.Weather
adb shell monkey 100 100个随机事件-v 反馈信息的级别， 每个-v代表一级 -v 0,-v -v 1,-v -v -v 2
–throttle <毫秒> 延时
–ignore-timeouts 忽略ANR错误-

关闭monkey
adb shell ps|grep monkeykill pid 12345
adb shell monkey -v ?–throttle 1000 –ignore-crashes 100000

adb shell wm dismiss-keyguard //解锁屏幕
```

