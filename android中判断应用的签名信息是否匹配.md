```java
/**
 * 应用签名处理工具
 *
 * @author fzw
 * @version 1.0
 * @date 2018-11-30
 */
public class ApkSignatureUtil {
    /**
     * 获取apk签名信息
     * @param filePath apk文件路径
     * @param context
     * @return
     * @throws FileNotFoundException 当filePath文件不存在时
     * @throws IllegalAccessException 当filePath文件不能访问时
     */
    public static Signature[] getSignaturesByApkFile(String filePath, Context context) throws FileNotFoundException, IllegalAccessException {
        PackageManager packageManager = context.getPackageManager();
        File file = new File(filePath);
        if (!file.exists()) {
            throw new FileNotFoundException("File "+filePath+" is not exists");
        }
        if (!file.canRead()) {
            throw new IllegalAccessException("File "+filePath+" can not read");
        }
        PackageInfo packageInfo = packageManager.getPackageArchiveInfo(filePath,PackageManager.GET_SIGNATURES);
        return packageInfo == null ? null: packageInfo.signatures;
    }
 
    /**
     * 获取已安装应用的签名信息
     * @param packageName 应用包名
     * @param context
     * @return
     */
    public static Signature[] getSignaturesByPackageNameFromInstalled(String packageName, Context context){
        PackageManager packageManager = context.getPackageManager();
        try {
            PackageInfo packageInfo = packageManager.getPackageInfo(packageName,PackageManager.GET_SIGNATURES);
            return packageInfo.signatures;
        } catch (PackageManager.NameNotFoundException e) {
            e.printStackTrace();
        }
        return null;
    }
 
    /**
     * 判断签名是否匹配
     * @param s1
     * @param s2
     * @return
     */
    public static boolean isSignaturesMatch(Signature[] s1, Signature[] s2){
        return checkSignatures(s1,s2)  == PackageManager.SIGNATURE_MATCH;
    }
 
    /**
     * 判断签名是否匹配
     * @param installedPackageName 已安装的应用包名
     * @param apkPath apk文件路径
     * @param context
     * @return
     * @throws FileNotFoundException 当apkPath文件不存在时
     * @throws IllegalAccessException 当apkPath文件不能访问时
     */
    public static boolean isSignaturesMatch(String installedPackageName, String apkPath, Context context) throws FileNotFoundException, IllegalAccessException {
        return isSignaturesMatch(getSignaturesByApkFile(apkPath,context),
                getSignaturesByPackageNameFromInstalled(installedPackageName,context));
    }
 
    /**
     * 签名信息匹对，参考PackageManagerService.compareSignatures
     * @param s1
     * @param s2
     * @return
     */
    public static int checkSignatures(Signature[] s1, Signature[] s2){
        if (s1 == null) {
            return s2 == null
                    ? PackageManager.SIGNATURE_NEITHER_SIGNED
                    : PackageManager.SIGNATURE_FIRST_NOT_SIGNED;
        }
 
        if (s2 == null) {
            return PackageManager.SIGNATURE_SECOND_NOT_SIGNED;
        }
 
        if (s1.length != s2.length) {
            return PackageManager.SIGNATURE_NO_MATCH;
        }
 
        // Since both signature sets are of size 1, we can compare without HashSets.
        if (s1.length == 1) {
            return s1[0].equals(s2[0]) ?
                    PackageManager.SIGNATURE_MATCH :
                    PackageManager.SIGNATURE_NO_MATCH;
        }
 
        Set<Signature> set1 = new HashSet<>();
 
        //ArraySet<Signature> set1 = new ArraySet<Signature>();
        for (Signature sig : s1) {
            set1.add(sig);
        }
        Set<Signature> set2 = new HashSet<>();
        //ArraySet<Signature> set2 = new ArraySet<Signature>();
        for (Signature sig : s2) {
            set2.add(sig);
        }
        // Make sure s2 contains all signatures in s1.
        if (set1.equals(set2)) {
            return PackageManager.SIGNATURE_MATCH;
        }
        return PackageManager.SIGNATURE_NO_MATCH;
    }
}
```



