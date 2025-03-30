## React Native Hot Update 
### 字段设计

minAndroidVersion

minIOSVersion

minBaseRN

forceUpdate

更新模式：`静默模式` (Silent mode) 和`活动模式` (Active mode) 。

* 静默模式：自动去下载可用的更新并且在下次 app 重启时应用他们，整个过程对于终端用户是“静默”的。
* 活动模式：当有可用的更新，弹框提醒用户有更新。在得到终端用户的授权后，开始下载更新并且立即应用更新。

### JavaScript API

sync()  

### React Native Client API

静态方法

### 灰度

### 回滚
