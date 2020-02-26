1. 在frameworks\base\core\res\res\values\string.xml中增加字符串，比如：
   Some Text

2. 在frameworks\base\core\res\res\values\public.xml中增加对应的内容，比如：

3. 运行 make update-api 命令，此命令会自动更新下面文件
   frameworks\base\api\current.text
   frameworks\base\api\system-current.text
   frameworks\base\api\test-current.text
   在这之中增加如下内容：
   field public static final int some_text = 17040287; // 0x104039f 这里ID系统自动生成

4. 打开frameworks\base\api\current.text文件找到刚才添加的字段名，拷贝后面的16进制编号，比如这里的0x104039f
5. 在frameworks\base\core\res\res\values\public.xml中刚才增加的内容上增加id内容