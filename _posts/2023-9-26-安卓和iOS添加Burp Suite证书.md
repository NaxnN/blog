## Android
### 在user certificate store中安装证书（可以拦截Chrome浏览器的流量）
#### 老版本
1. 首先在电脑上下载burp证书，然后将该文件的后缀改成将der后缀改成crt后缀
2. 电脑启动一个http server，安卓手机访问在http server上的带crt后缀的证书文件
3. 下载完成后，点击该文件即可安装

#### 新版本
1. 对于新系统版本（安卓11以后），按照以上方式安装时，会出现：
```
Install CA certificates in Settings
This certificate from null must be installed in Settings. Only install CA certificates from organizations you trust
```
2. 解决方法：Open Device settings · Go to 'Security' · Go to 'Encryption & Credentials' · Go to 'Install from storage' or 'Install a certificate' 
用户证书位置：`/data/misc/user/0/cacerts-added/`

### 在system certificate store安装证书(安卓系统版本小于等于13。安卓版本在14之后系统证书不能修改)
1. 首先重命名证书文件：
```sh
hashed_name=`openssl x509 -subject_hash_old -in cacert.crt | head -1` && cp cacert.crt $hashed_name.0
```
2. 然后安装此步骤: https://docs.mitmproxy.org/stable/howto-install-system-trusted-ca-android/

### 在system certificate store安装证书（安卓14）
* https://httptoolkit.com/blog/android-14-install-system-ca-certificate/

### 参考链接
https://portswigger.net/burp/documentation/desktop/mobile/config-android-device
https://httptoolkit.com/blog/android-11-trust-ca-certificates/


## iOS
1. iPhone<u>使用safari浏览器</u>访问 http://burp_address/cert，下载profile
2. 访问 设置-通用-关于本机-VPN与设备管理，来信任证书
3. 访问 设置-通用-关于本机-证书信任设置，来信任证书


## 监听流量
* 以上步骤完成后，可在wifi处配置代理