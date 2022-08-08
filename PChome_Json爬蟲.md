## 對PChone的Json爬蟲
搜尋網頁中的商品名稱，將其Json資料給爬取下來   
找到其JSON的程式碼即可，通常在F12/網路/Fetch/XHR/慢慢搜尋(打開預覽看看)
![image](https://user-images.githubusercontent.com/102600962/183321226-7e8012a1-7e5a-4fc6-adb3-0dbb27258d84.png)

<br>

### 一、找到其Json格式的url後，先驗證第一頁資料看看是否抓到

```Python
import pandas as pd
import json
import requests
import time

# 先用第一頁資料驗證自己程式碼及爬取資料皆正確
url = 'https://ecshweb.pchome.com.tw/search/v3.3/all/results?q=switch%20lite%20%E4%B8%BB%E6%A9%9F&page=1&sort=sale/dc'
req = requests.get(url)
# 網站的json程式碼爬下來
data = json.loads(req.content)
switch = pd.DataFrame(data['prods'])
# 只要id/name/price/origin price
switch_data = switch[['Id','name','price','originPrice']]
```
![image](https://user-images.githubusercontent.com/102600962/183323742-4ebe52de-78aa-48ac-974a-7f38260a5941.png)

<br>

### 二、成功後，即可用迴圈爬資料(範例先爬1~5頁)

```Python
keyword = "Switch lite 主機"
all_switch_data = pd.DataFrame() #準備一個空的容器
for i in range(1,6): #i為int 要轉str
    url = 'https://ecshweb.pchome.com.tw/search/v3.3/all/results?q='+keyword+'&page='+str(i)+'&sort=sale/dc'
    req = requests.get(url)
    data = json.loads(req.content)
    switch = pd.DataFrame(data['prods'])
    switch_data = switch[['Id','name','price','originPrice']]
    all_switch_data = pd.concat([all_switch_data, switch_data], axis=0) #垂直合併，所以axis=0
    
    time.sleep(8) #每次執行以8秒為間隔
```
    為什麼這裡要使用time.sleep而爬foodPandas的html不用
    原因就是迴圈裡有沒有包網址，如果沒有包網址代表我們是將網頁的格式碼存在Python裡面，資料抓取也是從Python裡面抓
    但是迴圈裡有包網址，代表我們必須重複向網頁請求程式碼，如果不加時間間隔，對方伺服器會認為我們正在持續不斷地攻擊他
    可能會造成對方伺服器癱瘓，甚至有可能觸法。

<br>

### 三、不要index，並將其儲存csv

```Python
all_switch_data.to_csv('PChome的Json爬蟲.csv', encoding='utf-8-sig', index=False)
```

<br>

## 完整程式碼

```Python
import pandas as pd
import json
import requests
import time

# 先用第一頁資料驗證自己程式碼及爬取資料皆正確
url = 'https://ecshweb.pchome.com.tw/search/v3.3/all/results?q=switch%20lite%20%E4%B8%BB%E6%A9%9F&page=1&sort=sale/dc'
req = requests.get(url)
# 網站的json程式碼爬下來
data = json.loads(req.content)
switch = pd.DataFrame(data['prods'])
# 只要id/name/price/origin price
switch_data = switch[['Id','name','price','originPrice']]



# 驗證成功後，即可開始用迴圈方式爬個5頁的資料
keyword = "Switch lite 主機"
all_switch_data = pd.DataFrame() #準備一個空的容器
for i in range(1,6): #i為int 要轉str
    url = 'https://ecshweb.pchome.com.tw/search/v3.3/all/results?q='+keyword+'&page='+str(i)+'&sort=sale/dc'
    req = requests.get(url)
    data = json.loads(req.content)
    switch = pd.DataFrame(data['prods'])
    switch_data = switch[['Id','name','price','originPrice']]
    all_switch_data = pd.concat([all_switch_data, switch_data], axis=0) #垂直合併，所以axis=0
    
    time.sleep(8)

  
all_switch_data.to_csv('PChome的Json爬蟲.csv', encoding='utf-8-sig', index=False)
```


