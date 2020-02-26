```java
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)

LOCAL_PACKAGE_NAME := APKNAME
LOCAL_CERTIFICATE := platform
LOCAL_MODULE_TAGS := optional
LOCAL_DEX_PREOPT := false
#可以使用系统hide api 与
LOCAL_PRIVATE_PLATFORM_APIS := true
#引入系统资源文件
LOCAL_USE_AAPT2 := true
#是否为系统应用会编译进入system/priv-app目录
LOCAL_PRIVILEGED_MODULE := true
#java文件
LOCAL_SRC_FILES := $(call all-java-files-under, src)
LOCAL_SRC_FILES += $(call all-Iaidl-files-under, aidl)
#aidl
LOCAL_AIDL_INCLUDES += $(LOCAL_PATH)/aidl
#资源文件
LOCAL_RESOURCE_DIR := $(LOCAL_PATH)/res
#导入系统依赖
LOCAL_STATIC_ANDROID_LIBRARIES := android-support-design \
android-support-v4 \
android-support-v7-appcompat \
android-support-v7-recyclerview \
android-support-annotations \
android-support-compat \
android-support-core-utils \
android-support-core-ui \
android-support-fragment
#使用系统包
LOCAL_JAVA_LIBRARIES := framework

#导入jar
LOCAL_STATIC_JAVA_LIBRARIES := kka_glide-4.7.0

#导入aar
LOCAL_STATIC_JAVA_AAR_LIBRARIES += kka_ucrop-2.2.2

LOCAL_PROGUARD_ENABLED := disabled
LOCAL_PROGUARD_FLAG_FILES := proguard.cfg

LOCAL_JNI_SHARED_LIBRARIES := libkkaccount_api

LOCAL_AAPT_FLAGS := –auto-add-overlay \
–extra-packages com.yalantis.ucrop

include $(BUILD_PACKAGE)

##################################################
include $(CLEAR_VARS)

LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES := \
kka_glide-4.7.0:libs/glide-4.7.0.jar \
kka_ucrop-2.2.2:libs/ucrop-2.2.2.aar

include $(BUILD_MULTI_PREBUILT)
```



