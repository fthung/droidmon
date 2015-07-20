# Droidmon - Dalvik Monitoring Framework for CuckooDroid
Contributed By Check Point Software Technologies LTD.

Background
----------
Droidmon, a key piece in CuckooDroid, monitors applications inside a virtual (guest) machine and provides insight into an application’s behavior. Droidmon is an open source Dalvik Monitoring framework based on [Xposed Framework](http://repo.xposed.info/).
 
Xposed is a framework for modules that can change the behavior of the system and apps without touching any APKs. This means that modules can work in different versions and even ROMs without any additional changes (as long as the original code did not undergo too much modification). It's also easy to reverse. As all changes are done in the memory, you can restore your original system  by deactivating the module and rebooting. Another advantage is that multiple modules can make changes to the same part of the system or app. With modified APKs, you must choose one. There is no way to combine them, unless the author builds multiple APKs with different combinations.
 
Droidmon allows you to select which functions you want to monitor and does all the monitoring for you. It works by hooking the zygote every time a package has been loaded. Droidmon opens the configuration file (location: /data/local/temp/hooks.conf), reads the list of API methods to monitor, hooks them dynamically every time a new application is opened, and reports all the monitoring information to logcat.
Features
--------
* Dynamic hooking of Dalvik methods - APIs/app specific
* Autogenerated json logs from hooked methods - when you enter the name of the app, it is automatically parsed and presented in the log

Prerequisites
-------------
* Root the device.
* Install and enable the [XposedInstaller](http://repo.xposed.info/module/de.robv.android.xposed.installer)
 
Usage
-----
1. Install `Droidmon.apk` and enable this module in XposedInstaller.
2. Push the configuration file `hooks.json` to `/data/local/tmp/` with the required hooks.
3. Reboot the Android device.
4. Verify that Droidmon module is enabled.
5. Droidmon Monitoring is ready. Open a new app. The logs will be shown in `logcat`.



Configure File Format
--------
The **hooks.json** is the configuration file written in json format. It contains a list of all the information needed to hook the required methods. Each element in the list is a dictionary which contains four key-value pairs describing the monitored method.
 
The first key-value pair contains the name of the class we want to call. The second pair is the name of the method we want to monitor. The third pair is a boolean indicating whether or not to log the information about the object which invokes the method. Finally, the fourth pair is the type of API method such as networking, sms, fingerprint, etc.
 
An example is shown below:

```
{
    "hookConfigs": [
        {
            "class_name": "android.telephony.TelephonyManager", 
            "method": "getDeviceId", 
            "thisObject": false, 
            "type": "fingerprint"
        }, 
        {
            "class_name": "android.telephony.TelephonyManager", 
            "method": "getSubscriberId", 
            "thisObject": false, 
            "type": "fingerprint"
        }
    ]
}```


Log Format
--------
Each method found in the configuration file and later invoked produces a log in the format seen below. All of the information in the configuration file appears in the log file. In addition, the log file  records the timestamp when the invocation occurred, the arguments passed to the function, and the return value. If we enabled the thisObject Boolean, it records the information of the invoking object. Most importantly, the log also includes a tag for application filtering. Example: `Droidmon-apimonitor-<Package Name>`
 
To filter the tag, use this command:
`adb logcat -d | grep Droidmon-apimonitor-com.cuckoo.test`
 
```
I/Xposed  ( 1649): 
Droidmon-apimonitor-com.cuckoo.test:
{
    "timestamp":1436953465511,
    "class":"android.telephony.SmsManager",
    "method":"sendTextMessage",
    "type":"sms",
    "args":["0735445281","000000000000000"]
}
```