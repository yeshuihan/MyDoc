使用cmd脚本：

```shell
@echo off
cd "%~dp0"
%~d0
java -jar signapk.jar platform.x509.pem platform.pk8 %1% after.apk
```



这当中有两个签名文件：platform.x509.pem ，platform.pk8 。在系统中的路径如下：
每个product的系统签名一般在alps\device\mediatek\common\security\kon6737m_r7_65_n下，与alps\build\target\product\security 可能不同
alps\build\target\product\security 下的签名一般不用，但是阿里云的好像是这个
这个signapk.jar 是更改签名的jar文件，可以去系统中找（7.0后系统生成的不能直接使用），下面是文件链接

链接：https://pan.baidu.com/s/17GGzp5Yf8xtbzNcyrwpjHg 密码：j3oy

