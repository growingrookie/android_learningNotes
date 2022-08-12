Android系统存储信息的获取(RAM,ROM）
===================================

Android系统的存储设备一般分为RAM，ROM，SDCard三个部分。其中RAM是Random Access Memory的缩写，是随机存储器，在工作状态时可以随机读写数据，断电以后会丢失数据，即我们常说的内存。手机的ROM和传统的ROM（Read Only Memory）又有些不一样， 它分为两部分，一部分是用于系统，另外一部分是用作用户存储数据 。SDCard即为我们平时所说的存储卡，8G，16G等，常用的有TF卡， 用于存储用户数据。

## 内存（RAM）信息获取

### 获取总内存

android的总内存大小信息存放在系统的/proc/meminfo文件里面，可以通过读取这个文件来获取这些信息：

```kotlin
public void getMemoryInfo() {  
                String str1 = "/proc/meminfo";  
                String str2="";  
                try {  
                        FileReader fr = new FileReader(str1);  
                        BufferedReader localBufferedReader = new BufferedReader(fr, 8192);  
                        while ((str2 = localBufferedReader.readLine()) != null) {  
                                Log.d(TAG, "---" + str2);  
                        }  
                } catch (IOException e) {  
                }  
        }  
```

### 获取当前剩余内存(ram)：

通过Android系统提供的ActivityManager对象获取其可用的内存。

```java
public static long getAvailMemory(Context context) {          
        ActivityManager am = (ActivityManager)context.getSystemService(Context.ACTIVITY_SERVICE);  
        ActivityManager.MemoryInfo mi = new ActivityManager.MemoryInfo();  
        am.getMemoryInfo(mi);  
        return mi.availMem;  
    } 
```



ROM信息获取
===========

```java
/** 
    * 获取手机内部空间大小 
    * @return 
    */  
    public static long getTotalInternalStorgeSize() {  
       File path = Environment.getDataDirectory();  
       StatFs mStatFs = new StatFs(path.getPath());  
       long blockSize = mStatFs.getBlockSize();  
       long totalBlocks = mStatFs.getBlockCount();  
       return totalBlocks * blockSize;  
   }  
   /** 
    * 获取手机内部可用空间大小 
    * @return 
    */  
    public static long getAvailableInternalStorgeSize() {  
       File path = Environment.getDataDirectory();  
       StatFs mStatFs = new StatFs(path.getPath());  
       long blockSize = mStatFs.getBlockSize();  
       long availableBlocks = mStatFs.getAvailableBlocks();  
       return availableBlocks * blockSize;  
   }  
```
