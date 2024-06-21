---
layout: post
title:  "Android-UnCrackable-L1~L3"
date:   2024-06-22 19:39:02 +0800
categories: [android] 
---



> Android\CrackeMe\OWASP-Mobile-Application-Security\Android-UnCrackable

GitHub官方源代码 https://github.com/OWASP/mas-crackmes

相关apk: https://mas.owasp.org/crackmes/Android/

related files 

链接：https://pan.baidu.com/s/1Haim_T_3eWq_xxE_a7b7SQ?pwd=droz 

提取码：droz

# 分析环境

root-Android-10.0

jadx

apktool

IDA 7.2 (有arm64-v8a相关so的分析环境,其它IDA貌似没有)

...

小白一枚,刚入门安卓逆向, 曾学过一些Windows逆向



# 涉及的知识点







## root 检测



### 环境变量中的su

在 Android-UnCrackable-L1,  Android-UnCrackable-L2, Android-UnCrackable-L3 有应用

检测一些环境变量中有没有su

一个root的设备是可以直接输入 su 进行提权的. 同时su存在于环境变量中. 以此作为root的检测手段



```java
    public static boolean checkRoot1(){
        for(String pathDir : System.getenv("PATH").split(":")){
            if(new File(pathDir, "su").exists()) {
                return true;
            }
        }
        return false;
    }
```



### 和root有关的apk或者可执行程序

在 Android-UnCrackable-L1,  Android-UnCrackable-L2, Android-UnCrackable-L3 有应用



有些软件可以把手机root掉,

有些软件是利用root干其它事情

如果手机安装了这些软件,大概说明当前手机已经被root了



```java
    public static boolean checkRoot3() {
        String[] paths = { "/system/app/Superuser.apk", "/system/xbin/daemonsu",  "/system/etc/init.d/99SuperSUDaemon", "/system/bin/.ext/.su", "/system/etc/.has_su_daemon", "/system/etc/.installed_su_daemon", "/dev/com.koushikdutta.superuser.daemon/" };

        for (String path : paths) {
            if (new File(path).exists()) return true;
        }
        return false;
    }
```



```
"/system/app/Superuser.apk",
"/system/xbin/daemonsu", 
"/system/etc/init.d/99SuperSUDaemon", 
"/system/bin/.ext/.su",
"/system/etc/.has_su_daemon",
"/system/etc/.installed_su_daemon", 
"/dev/com.koushikdutta.superuser.daemon/"
```





## 检测当前设备是不是 开发者模式

在 Android-UnCrackable-L1,  Android-UnCrackable-L2, Android-UnCrackable-L3 有应用



```java
   public static boolean checkRoot2() {
        String buildTags = android.os.Build.TAGS;
        return buildTags != null && buildTags.contains("test-keys");
    }
```









## 反调试



### java: FLAG_DEBUGGABLE

在 Android-UnCrackable-L1,  Android-UnCrackable-L2, Android-UnCrackable-L3 有应用

检测标志位,这个貌似在AndroidManifest.xml中有

检测当前apk是否具备可调式的属性, 作为一个release的apk是不可调试的.



```java
    public static boolean isDebuggable(Context context){

        return ((context.getApplicationContext().getApplicationInfo().flags & ApplicationInfo.FLAG_DEBUGGABLE) != 0);

    }
```



### java: 检测是否被调试器连接

在Android-UnCrackable-L2, Android-UnCrackable-L3,有应用



开一后台线程,一直检测是否有调试器连接当前apk进程,

```java
        // Debugger detection
        new AsyncTask<Void, String, String>() {

            @Override
            protected String doInBackground(Void... params) {
                while (!Debug.isDebuggerConnected()) {
                    SystemClock.sleep(100);
                }
                return null;
            }

            @Override
            protected void onPostExecute(String msg) {
                showDialogAndExit("Debugger detected!");
            }
        }.execute(null, null, null);
```





### so: 子进程attach父进程

大概的效果, apk变为了2个进程

父进程被子进程调试

子进程一直处于debug循环,又没啥好调试的.



#### 案例1

在 Android-UnCrackable-L2, 有应用

```
UnCrackable-Level2
--lib
----arm64-v8a
------libfoo.so
--------init()
------------anti_debug();

# 对于L2的反调试,好像没怎么起到作用, 同时因为高内聚，低耦合的原因,我们可以nop掉 init()的anti_debug()调用
```





```c
//#include <jni.h>
#include <stdio.h>
#include <string.h>
#include <sys/ptrace.h>
#include <pthread.h>
#include <unistd.h>
#include <sys/wait.h>

static int child_pid;

void* monitor_pid(void *) {
    //only works when target-sDK below 28, now removed
//    int status;
//
//    waitpid(child_pid, &status, 0);
//
//    if (status != 11) {
//
//        // If this is a release build, the child will segfault (status 11). Otherwise, waitpid() should never return.
//
//        goodbye(); // Commit seppuku
//    }

    pthread_exit(NULL);

}

void   anti_debug() {

    child_pid = fork();

    if (child_pid == 0)
    {
        int ppid = getppid();

        int status;

        if (ptrace(PTRACE_ATTACH, ppid, NULL, NULL) == 0)
        {
            waitpid(ppid, &status, 0);

            ptrace(PTRACE_CONT, ppid, NULL, NULL);

            while (waitpid(ppid, &status, 0)) {

                if (WIFSTOPPED(status)) {
                    // If parent stops, tell it to resume.
                    ptrace(PTRACE_CONT, ppid, NULL, NULL);
                } else {
                    // Parent has exited for some reason.
                    _exit(0);
                }
            }
        }

    } else {
        pthread_t t;

        // Start the monitoring thread

        pthread_create(&t, NULL, monitor_pid, (void *)NULL);
    }
}


int main(){
	int i;
	printf("hi, i am %d \n", getppid());
	anti_debug();
	printf("now, i am %d \n", getppid());
	while(1){
		i=i;
		sleep(3);
	}
	return 0;
}

```



#### 案例2

在 Android-UnCrackable-L3 有应用

在案例1的基础上,monitor_pid函数发生了改变



```c
 

void *monitor_pid(void *) {

    int status;

    waitpid(child_pid, &status, 0);

    if (status != 11) {

        // If this is a release build, the child will segfault (status 11). Otherwise, waitpid() should never return.

        _exit(0); // Commit seppuku
    }

    pthread_exit(NULL);

}
```





## 防篡改



在 Android-UnCrackable-L3, 有应用



java层, 也就是检测 class.dex , xxx.so 是否被修改过

```java
private void verifyLibs() {

        crc = new HashMap<String, Long>();
        //mips, mips64 and armeabi are no longer supported with the new buildtools
//        crc.put("armeabi", Long.parseLong(getResources().getString(R.string.armeabi))); //"1054637268"
//        crc.put("mips", Long.parseLong(getResources().getString(R.string.mips))); //"3104746423"
        crc.put("armeabi-v7a", Long.parseLong(getResources().getString(R.string.armeabi_v7a))); //"881998371"
        crc.put("arm64-v8a", Long.parseLong(getResources().getString(R.string.arm64_v8a))); //"1608485481"
//        crc.put("mips64", Long.parseLong(getResources().getString(R.string.mips64))); //"1319538057"
        crc.put("x86", Long.parseLong(getResources().getString(R.string.x86))); //"1618896864"
        crc.put("x86_64", Long.parseLong(getResources().getString(R.string.x86_64)));  //"2856060114"

        try {
            ZipFile zf = new ZipFile(getPackageCodePath());

            for (Map.Entry<String, Long> entry : crc.entrySet()) {

                String filename = "lib/" + entry.getKey() + "/libfoo.so";

                ZipEntry ze = zf.getEntry(filename);

                Log.v(TAG, "CRC[" + filename + "] = " + ze.getCrc());

                if (ze.getCrc() != entry.getValue()) {
                    //tampered = 31337;
                    Log.v(TAG, filename + ": Invalid checksum = " + ze.getCrc() + ", supposed to be " + entry.getValue());

                }

            }

            String filename = "classes.dex";
            ZipEntry ze = zf.getEntry(filename);

            Log.v(TAG, "CRC[" + filename + "] = " + ze.getCrc());

            if (ze.getCrc() != baz()) { // baz()=25235683LL
                //tampered = 31337;
                Log.v(TAG, filename + ": crc = " + ze.getCrc() + ", supposed to be " + baz());

            }

        } catch (IOException e) {
            Log.v(TAG, "Exception");
            System.exit(0);
        }
    }
```







## 混淆



### java层的混淆:

- Android-UnCrackable-L1 :轻度混淆, 只是修改一些变量,函数,类的名字.
- Android-UnCrackable-L2: 在L1的程度上, L2增添了很多无用的,  乱七八糟的类. 或者并没有增加,把已有的类修改的乱七八糟
- Android-UnCrackable-L3: 在L1的程度上, 3增添了很多无用的,  乱七八糟的类. 或者并没有增加,把已有的类修改的乱七八糟





解决办法:

理解相关代码, 分析并重命名



### so层混淆:llvm

见Android-UnCrackable-L4-0.9了



### so层混淆: 函数复杂多样化

函数复杂多样化就是把 一个很简单的东西写得很复杂复杂, 但因为本质很简单, 只要看破就很好理解.





>  Android-UnCrackable-L2

CodeCheck.bar中, out数组赋值代码, 是一个看上去简单, 实际上也很简单的一个数据赋值

但因为高内聚,低耦合, 复杂的函数功能却比较单一, 又因为编译器的原因

导致out数组的赋值变得很简单,甚至找不到复杂的痕迹

```
UnCrackable-Level2
--lib
----arm64-v8a
------libfoo.so
--------CodeCheck.bar(JNIEnv *env, jobject, jbyteArray in)
```







>  Android-UnCrackable-L3



```
UnCrackable-Level3
--lib
----arm64-v8a
------libfoo.so
--------CodeCheck.bar(JNIEnv *env, jobject, jbyteArray in)
----------- sub_doit((char*)out);
```



sub_doit(char* barr)是一个看上去很复杂的函数,

但是因为高内聚,低耦合的原因,功能又比较单一. 也就是一个赋值函数







## so层: init_array 手段

在 Android-UnCrackable-L3, 有应用

也就是在elf加载的时, 先于main函数的的一种东西

类似于exe的tls.

该函数的执行需要被注册. 注册后一般存在于`.init_array`节区

```
UnCrackable-Level3
--lib
----arm64-v8a
------libfoo.so
--------_start()

# 检测xposed,frida ???
# 数据初始化
```







```c
__attribute__((constructor)) static void _start(void) {
    pthread_t t;

    pthread_create( &t, NULL, __somonitor_loop, NULL );
	
    //__somonitor_loop 用于检测xposed,frida ???
    memset(x, 0, 25);

    initialized += 1;

    return;
}

void  __attribute__ ((visibility ("hidden"))) *__somonitor_loop(void *) {

    char line[512];

    FILE* fp;

    while (1) {
        fp = fopen("/proc/self/maps", "r");

        if (fp)
        {

            while (fgets(line, 512, fp))
            {
                if (strstr(line, "frida") || strstr(line, "xposed")) {

                    __android_log_print(ANDROID_LOG_VERBOSE, APPNAME, "Tampering detected! Terminating...");

                    goodbye();

                }

            }
            fclose(fp);
        } else {
            __android_log_print(ANDROID_LOG_VERBOSE, APPNAME, "Error opening /proc/self/maps! Terminating...");

            goodbye();
        }

        usleep(500);
    }

}
```











## 其它

通过源码,其实发现还有很多东西,很多知识点,很多反破解手法是我所不了解的, 所以此刻并未提出

日后有缘再来分析分析.





# 面对众多检测手段的解决办法



在java层, 

比如可以log注入,打印关键数据

比如删除对应的检测. 删除smali代码



so层

可以ida调试,查看关键数据

比如删除对应的检测. nop掉



之所以删除了部分代码,apk的功能不受影响,执行流程也没有受到影响

大概是因为 检测手段 如果涉及java层的检查, 同时检测函数 **高内聚，低耦合**, 

 

以上见解和手法 均来自自己学过的一些Windows逆向知识,所以很局限约束

 
