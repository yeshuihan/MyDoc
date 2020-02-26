应用的flags标志在，手机启动时packagemanageservice初始化时，扫描本地文件时给予。
以下文件夹中的Apk，android系统会给予ApplicationInfo.FLAG_SYSTEM标志。

custom/overlay
custom/framework
framework
system/priv-app/
system/app/
system/plugin/
vendor/overlay
vendor/framework/
vendor/priv-app/
vendor/app/
vendor/plugin/
Environment.getOemDirectory(), “app”

mtk :
“1”.equals(SystemProperties.get(“ro.mtk_carrierexpress_inst_sup”));
true
String opStr = SystemProperties.get(“persist.operator.optr”);
String opFileName = “usp-apks-path” + “-” + opStr + “.txt”;
/custom/usp或/system/usp中的opFileName文件中路径没有removable的

false:
/custom/app
/custom/plugin