```java
/**
 * 静默安装
 *
 * 此方法需要系统签名
 * @param apkAbsolutePath
 * @param context
 */
public static void installApkSlient(final String apkAbsolutePath, Context context) {
    PackageManager packageManager = context.getPackageManager();
    Class<?> pmClz = packageManager.getClass();
    try {
        //PackageInstallObserver 系统隐藏类，android.app.PackageInstallObserver;
        if (Build.VERSION.SDK_INT >= 21) {
            // 系统隐藏api. installPackage
            packageManager.installPackage(Uri.fromFile(new File(apkAbsolutePath)), new PackageInstallObserver() {
                        @Override
                        public void onPackageInstalled(String basePackageName, int returnCode, String msg, Bundle extras) {
                            super.onPackageInstalled(basePackageName, returnCode, msg, extras);
                            KLog.i("install package:"+basePackageName);
                            File file = new File(apkAbsolutePath);  //删除安装文件
                            if (file.exists()) {
                                file.delete();
                            }
 
                        }
                    }, PackageManager.INSTALL_REPLACE_EXISTING, null);   //系统隐藏属性，PackageManager.INSTALL_REPLACE_EXISTING
             
            /* 
            Class<?> aClass = Class.forName("android.app.PackageInstallObserver");
            Constructor<?> constructor = aClass.getDeclaredConstructor();
            constructor.setAccessible(true);
            Object installObserver = constructor.newInstance();
            Method method = pmClz.getDeclaredMethod("installPackage", Uri.class, aClass, int.class, String.class);
            method.setAccessible(true);
            method.invoke(packageManager, Uri.fromFile(new File(apkAbsolutePath)), installObserver, 2, null);*/
        } else {
            Method method = pmClz.getDeclaredMethod("installPackage", Uri.class, Class.forName("android.content.pm.IPackageInstallObserver"), int.class, String.class);
            method.setAccessible(true);
            method.invoke(packageManager, Uri.fromFile(new File(apkAbsolutePath)), null, 2, null);
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
 
}
```

AndroidP 版本静默安装，在P版本中PackageManager.installPackage 方法已经不存在，参考源码中的提示，这里使用PackageInstaller进行安装，主要参考PackageInstaller的源码。此方法在android 8.1上也能使用，至于更早的就没有验证了。

```java
/**
 * 静默安装 P
 *
 * 此方法需要系统签名
 * @param apkAbsolutePath  安装包路径
 * @param context
 */
private static void installApkSlient_P(final String apkAbsolutePath, Context context){
    KLog.i(TAG,"installApkSlient_P:Path:"+apkAbsolutePath);
    if (!processPackage(apkAbsolutePath,context)){
        KLog.w(TAG,"apk 解析失败");
        return;
    }
    PackageInstaller.SessionParams params = new PackageInstaller.SessionParams(
            PackageInstaller.SessionParams.MODE_FULL_INSTALL);
    params.installFlags = PackageManager.INSTALL_FULL_APP|PackageManager.INSTALL_REPLACE_EXISTING;
    File file = new File(apkAbsolutePath);
    try {
        PackageParser.PackageLite pkg = PackageParser.parsePackageLite(file, 0);
        params.setAppPackageName(pkg.packageName);
        params.setInstallLocation(pkg.installLocation);
        params.setSize(
                PackageHelper.calculateInstalledSize(pkg, false, params.abiOverride));
    } catch (PackageParser.PackageParserException e) {
        KLog.e(TAG, "Cannot parse package " + file + ". Assuming defaults.");
        KLog.e(TAG,
                "Cannot calculate installed size " + file + ". Try only apk size.");
        params.setSize(file.length());
    } catch (IOException e) {
        KLog.e(TAG,
                "Cannot calculate installed size " + file + ". Try only apk size.");
        params.setSize(file.length());
    }
    int sessionId = -1;
    try {
        sessionId = context.getPackageManager().getPackageInstaller().createSession(params);
    } catch (IOException e) {
        e.printStackTrace();
    }
 
    PackageInstaller installer = context.getPackageManager().getPackageInstaller();
    PackageInstaller.SessionInfo sessionInfo = installer.getSessionInfo(sessionId);
 
    if (sessionInfo != null && !sessionInfo.isActive()) {
        InstallingAsyncTask installingTask = new InstallingAsyncTask(context,sessionId,apkAbsolutePath);
        installingTask.execute();
    } else {
        // we will receive a broadcast when the install is finished
        KLog.w(TAG,"安装失败");
    }
 
}
/**
 * 尝试解析apk,避免无效apk
 *
 * @param apkAbsolutePath  安装包路径
 * @param context
 */
private static boolean processPackage(final String apkAbsolutePath, Context context){
    File sourceFile = new File(apkAbsolutePath);
    PackageParser.Package parsed = getPackageInfo(context, sourceFile);
    // Check for parse errors
    if (parsed == null) {
        KLog.w(TAG, "Parse error when parsing manifest. Discontinuing installation");
        return false;
    } else {
        KLog.i(TAG, "Package:"+parsed);
    }
    return true;
}
private static PackageParser.Package getPackageInfo(Context context, File sourceFile) {
    final PackageParser parser = new PackageParser();
    parser.setCallback(new PackageParser.CallbackImpl(context.getPackageManager()));
    try {
        return parser.parsePackage(sourceFile, 0);
    } catch (PackageParser.PackageParserException e) {
        return null;
    }
}
 
private static final class InstallingAsyncTask extends AsyncTask<Void, Void,
            PackageInstaller.Session> {
    WeakReference<Context> mContent;
    int mSessionId;
    String mApkAbsolutePath;
 
    volatile boolean isDone;
    InstallingAsyncTask(Context context,int sessionId,String apkAbsolutePath ){
        mContent= new WeakReference<Context>(context);
        mSessionId = sessionId;
        mApkAbsolutePath = apkAbsolutePath;
    }
 
    @Override
    protected PackageInstaller.Session doInBackground(Void... params) {
        Context context = mContent.get();
        if (context == null) {
            return null;
        }
        PackageInstaller.Session session;
        try {
            session = context.getPackageManager().getPackageInstaller().openSession(mSessionId);
        } catch (IOException e) {
            return null;
        }
 
        session.setStagingProgress(0);
 
        try {
            File file = new File(mApkAbsolutePath);
 
            try (InputStream in = new FileInputStream(file)) {
                long sizeBytes = file.length();
                try (OutputStream out = session
                        .openWrite("PackageInstaller", 0, sizeBytes)) {
                    byte[] buffer = new byte[1024 * 1024];
                    while (true) {
                        int numRead = in.read(buffer);
 
                        if (numRead == -1) {
                            session.fsync(out);
                            break;
                        }
 
                        if (isCancelled()) {
                            session.close();
                            break;
                        }
 
                        out.write(buffer, 0, numRead);
                        if (sizeBytes > 0) {
                            float fraction = ((float) numRead / (float) sizeBytes);
                            session.addProgress(fraction);
                        }
                    }
                }
            }
 
            return session;
        } catch (IOException | SecurityException e) {
            KLog.e(TAG, "Could not write package "+ e);
            session.close();
 
            return null;
        } finally {
            synchronized (this) {
                isDone = true;
                notifyAll();
            }
        }
    }
 
    @Override
    protected void onPostExecute(PackageInstaller.Session session) {
        Context context = mContent.get();
        if (context == null) {
            return;
        }
        if (session != null) {//定义广播接收安装的结果
            Intent broadcastIntent = new Intent(context, PackageInstallStatusReceiver.class);
            broadcastIntent.putExtra(PackageInstallStatusReceiver.FILE_PATH_KEY, mApkAbsolutePath);
 
            PendingIntent pendingIntent = PendingIntent.getBroadcast(
                    context,
                    0,
                    broadcastIntent,
                    PendingIntent.FLAG_UPDATE_CURRENT);
            session.commit(pendingIntent.getIntentSender());
        } else {
            context.getPackageManager().getPackageInstaller().abandonSession(mSessionId);
        }
    }
}
```

广播定义如下：

```java
//PackageInstallStatusReceiver.java
public class PackageInstallStatusReceiver extends BroadcastReceiver {
    public static final String TAG = InstallApkUtil.TAG;
    public final static String FILE_PATH_KEY = "filePath";
 
    @Override
    public void onReceive(Context context, Intent intent) {
        int status = intent.getIntExtra(PackageInstaller.EXTRA_STATUS, 0);
        final String filePath = intent.getStringExtra(FILE_PATH_KEY);
        if (status == PackageInstaller.STATUS_PENDING_USER_ACTION) {
            return;
        }
        String statusMessage = intent.getStringExtra(PackageInstaller.EXTRA_STATUS_MESSAGE);
        int legacyStatus = intent.getIntExtra(PackageInstaller.EXTRA_LEGACY_STATUS, 0);
        if (status == PackageInstaller.STATUS_SUCCESS) {
            KLog.i(TAG, "install success");
        } else {
            KLog.w(TAG, "install failed legacyStatus:" + legacyStatus + " statusMessage:" + statusMessage);
        }
        //删除文件
        if (!TextUtils.isEmpty(filePath)) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    File file = new File(filePath);
                    if (file.exists()) {
                        file.delete();
                    }
                }
            }).start();
        }
    }
}
```

