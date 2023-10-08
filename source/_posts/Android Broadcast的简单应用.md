---
title: Android Broadcast的简单应用
date: 2015-12-6 10:05:56
update: 2018-2-13 10:05:56
categories: Android  
tags: 
    - brodcast
cover: https://i.loli.net/2020/10/27/ogxvylKmiRqLtPQ.jpg
---


## broadcast静态使用
### 在AndroidManifest.xml中注册

```xml
<receiver 
       android:name="com.example.testBR">
       <intent-filter>
            <action android:name="com.example.test.ACTION_TEST" />
            </intent-filter>
       </receiver>
```
### 重写onReceive方法

```java
@Override
	public void onReceive(Context context, Intent intent) {
		// TODO Auto-generated method stub	
	}
```
## broadcast动态注册
```java
private BroadcastReceiver mReceiver = new BroadcastReceiver(){

    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO Auto-generated method stub    
    }
};

@Override
protected void onPause() {
    super.onPause();
    unregisterReceiver(mReceiver);
}


@Override
protected void onResume() {
    super.onResume();
    IntentFilter filter = new IntentFilter("com.example.test.ACTION_TEST");
    registerReceiver(mReceiver, filter);

}
```

## 发送广播
```java
sendBroadcast(new Intent("com.example.test.ACTION_TEST"));
```

### Intent中附加信息

#### 发送带有附加消息的Intent
```java
Intent i = new Intent(this, TestService.class);
     i.putExtra(name, value);
     sendBroadcast(new Intent("com.example.test.ACTION_TEST"));
```
#### 接收Intent中的附加消息
```java
@Override
public void onReceive(Context context, Intent intent) {
	intent.getIntExtra("action", 0));
}
```

## broadcast权限
如果reeceiver声明在`AndroidManifest.xml`中，且仅限应用内部使用，则可在标签上添加`android:exported="false"`属性
这样系统的其他应用就再也无法接触到该receiver。

另外也可以通过自己创建使用权限，在`AndroidManifest.xml`中添加标签来完成。
```xml
<permission
    android:name="com.example.test.TestService.PRIVATE"
    android:protectionLevel="signature"
    ></permission>

<uses-permission android:name="com.example.test.TestService.PRIVATE"/>
```
### 发送带权限的broadcast
```java
String PREM_PRIVATE = "com.example.test.TestService.PRIVATE";
sendBroadcast(new Intent("com.example.test.ACTION_TEST"),PREM_PRIVATE);
```

## 通过ADB shell模拟发送broadcast
```shell
adb shell am broadcast <option> <INTENT>：

option:
[-a <ACTION>]
[-d <DATA_URI>]
[-t <MIME_TYPE>] 
[-c <CATEGORY> [-c <CATEGORY>] ...] 
[-e|--es <EXTRA_KEY> <EXTRA_STRING_VALUE> ...] 
[--ez <EXTRA_KEY> <EXTRA_BOOLEAN_VALUE> ...] 
[-e|--ei <EXTRA_KEY> <EXTRA_INT_VALUE> ...] 
[-n <COMPONENT>]
[-f <FLAGS>] [<URI>]
```


例如：
```shell
adb shell am broadcast -a com.android.test --es test_string "this is test string" --ei test_int 100 --ez test_boolean true
```