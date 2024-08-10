## Local account（本地账户登录）
* 不管网络环境（network profile type）是public network还是private network，当Windows访问其他地址的smb share时，会自动以Net-NTLMv2 hash发送自身的身份信息（本地账户名和本地密码）。

## Microsoft account（微软账户登录）
* 网络环境（network profile type）是private network时，当Windows访问其他地址的smb share时，会自动以Net-NTLMv2 hash发送自身的身份信息（微软账户名和微软账户密码）。
* 网络环境（network profile type）是public network时，当Windows访问其他地址的smb share时，需手动输入账号密码。