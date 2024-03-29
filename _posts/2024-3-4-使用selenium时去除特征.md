### 去除webdriver变量和banner
```python
# 修改 navigator.webdriver 为 false
options.add_argument('--disable-blink-features=AutomationControlled')  

# 去掉"Chrome is being controlled by automated test software"的banner
options.add_experimental_option("excludeSwitches",["enable-automation"]) 
```

### 去除`cdc_...`的变量
#### 手动去除
1. 首先找到selenium正在使用的chromedriver。可在selenium运行时，使用ps -ef|grep chromedriver命令找到其路径。
2. 复制该chromedriver，并在vscode中使用hex editor编辑复制后的文件，替换`cdc_...`，注意保持长度一致。（不能在vscode中使用text editor，不然文件性质会变）。
3. 如果在arm Mac下运行，则须重新签名。
4. 指定修改后的chromedriver运行，发现浏览器控制台中`cdc_...`的变量消失（被替换）。
    ```python
    service = Service(executable_path='./chromedriverNew')
    ```
#### 使用脚本去除（包括去除其他特征）
1. `pip3 install undetected-chromedriver`
2. 复制现在已有的chromedriver到当前工程目录下。
3. (optional)找到patcher.py文件，修改`console.log("undetected chromdriver 1337!")`里的内容为其他
4. 跑一遍
    ```python
    from undetected_chromedriver import patcher
    pa = patcher.Patcher(executable_path="./chromedriver")  # 要修改的chromedriver
    pa.auto()
    ```
    发现chromedriver已被patch
5. 如果在arm Mac下运行，则须重新签名。

### 也可以直接使用`undetected-chromedriver`代替`selenium`

### 连接已有的chrome
```python
# 连接已有的chrome的debug port  `/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9014`
options.add_experimental_option("debuggerAddress", "127.0.0.1:9014")
```

### 完整代码（以上几点）

```python
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from selenium import webdriver
options = Options()

# 修改 navigator.webdriver 为 false
options.add_argument('--disable-blink-features=AutomationControlled')  

# 去掉"Chrome is being controlled by automated test software"的banner
# 选项与“连接已有的chrome”互斥
# options.add_experimental_option("excludeSwitches",["enable-automation"]) 


# 连接已有的chrome的debug port  `/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9014`
options.add_experimental_option("debuggerAddress", "127.0.0.1:9014")

# 新webdriver的路径
service = Service(executable_path='./chromedriverNew')

# 使用无头模式时，若不加--user-agent选项，user-agent中会有headlesschrome的特征。以下代码目的是去除该特征
# options.add_argument("--headless=new")
# options.add_argument('--user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36')


driver = webdriver.Chrome(options=optioins,service=service)
```
