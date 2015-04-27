# 用Chrome插件调用exe程序

环境

* Windows 7 64-bit
* Chrome 41

基本原理：

使用Chrome的[Native messaging API](https://developer.chrome.com/extensions/messaging#native-messaging)

1. 在本机建立一个native messaging host
2. Chrome插件可以与这个host采用消息进行对话

这样“用Chrome插件调用exe程序”这个问题就简化为：**在启动native messaging host时调用想要的exe文件**。

## 创建Chrome extension
将下面四个文件放在同一个目录下：

![files](http://sigma3.qiniudn.com/files.png)

### `manifest.json`

```JSON
{
  "manifest_version": 2,
  "name": "Dog",
  "version": "1.0",
  "description": "Love dogs or not",
  "browser_action": {
    "default_icon": "icon.jpg",
    "default_title": "Dog",
    "default_popup": "popup.html"
  },
  "permissions": ["nativeMessaging"]
}
```

### `icon.jpg`
这是在Chrome工具条上出现的图标的图片。注意文件名与`manifest.json`里指定的`default_icon`的值一致。

### `popup.html` 
注意文件名与`manifest.json`里指定的`default_popup`的值一致。

```HTML
<div style="width:200px;">
  <p id="label">You want a dog?</p>
  <button id="yes-button">Yes</button>
</div>	
<script type="text/javascript" src="script.js"></script>
```
里面的内容不重要，关键是一个按钮的ID为`yes-button`，以及下面对`script.js`的引用。`script.js`为插件增加行为。

### `script.js`
```
document.getElementById('yes-button').addEventListener('click', function(){
	chrome.runtime.connectNative('com.google.chrome.demo');
});
```
注意：`com.google.chrome.demo`是native messaging host的名字，在下面会用到。这里做的事情就是在用户点击按钮的时候连接本机上叫做`com.google.chrome.demo`的host.

## 测试Chrome extension
打开Chrome, 选择菜单 > 更多工具 > 扩展程序，勾选“开发者模式”启用它，点击“加载正在开发的扩展程序……”，选择上述四个文件所在的目录。会看到类似下面这样的界面。

![extension page](http://sigma3.qiniudn.com/extension_page.png)

记录下`ID`的值，后面会用到。

点击右上角的插件图标，应该看到一个类似下面的弹出窗口：

![extension working](http://sigma3.qiniudn.com/extension_working.png)

## 建立navtive messaging host

### 创建批处理文件
在建立host之前，需要一个可执行文件。在任意目录创建`native.bat`，内容如下：
```
notepad.exe
```
因为这里只是一个demo，所以我们让其打开Windows上的记事本程序。

### 建立host

在任意目录创建`host.json`，其内容如下：

```
{
  "name": "com.google.chrome.demo",
  "description": "Native messaging API host",
  "path": "C:\\native.bat",
  "type": "stdio",
  "allowed_origins": [
    "chrome-extension://knldjmfmopnpolahpmmgbagdohdnhkik/"
  ]
}
```
注意下面几点：

* `name`必须与`script.js`里调用时的一致
* `allowed_origins`里的内容`knldjmfmopnpolahpmmgbagdohdnhkik`替换成之前记录下的ID
* `path`为上一步建立的批处理文件的绝对路径

### 修改Windows注册表

需要让Windows知道我们刚刚建立的host的存在。在任意目录创建`item.reg`，内容如下：
```
Windows Registry Editor Version 5.00
[HKEY_CURRENT_USER\Software\Google\Chrome\NativeMessagingHosts\com.google.chrome.demo]
@="C:\\path\\to\\host.json"
```
注意第二行的最后，名称必须与之前建立的host的`name`一致，第三行的路径为`host.json`所在路径。保存后双击`item.reg`安装注册表项。

## 验证

点击Chrome插件里的Yes按钮即可打开记事本。

## Chrome插件打包

在下面的界面中点击“打包扩展程序……”按照指示操作即可。

![extension page](http://sigma3.qiniudn.com/extension_page.png)
