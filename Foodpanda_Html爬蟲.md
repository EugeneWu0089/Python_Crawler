## 對FoodPandas的Html爬蟲
將針對Foodpanda網頁裡的店家，透過爬蟲抓取「店家名稱」、「評級」、「標籤」、「運費」，最後匯出成CSV檔。

    初步了解爬蟲，都會需要對網頁進行Requests.get動作，但只作這個步驟有時候會出現403請求失敗的結果    
    因為使用爬蟲可能會造成對方伺服器負擔，所以我們要加個headers去讓對方認為我們是個瀏覽器再執行
```Python
# 先匯入需要的套件
import pandas as pd
import requests
from bs4 import BeautifulSoup
```

### 一、如何尋找Headers
![image](https://user-images.githubusercontent.com/102600962/183034591-9dbd8f4e-b170-4a5d-8dd5-fb726be9a0cb.png)
```Python
headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36"}
url = 'https://www.foodpanda.com.tw/city/taipei-city'
res = requests.get(url, headers=headers)
# 測試結果 回傳[200]成功    
```

### 二，使用BeautifulSoup套件解析res裡的html格式
```Python
soup = BeautifulSoup(res.content, 'html.parser')
soup
```

### 三、查找網頁裡html的格式所對應到的地方
![image](https://user-images.githubusercontent.com/102600962/183037665-3a9ab946-c0fb-49af-b890-96f85e4d3977.png)     
    
    發現figcaption裡vendor-info可能就是包含我們需要的店家資訊
```Python
# 用soup裡的Find_all試試看
getall = soup.find_all('figcaption',{'class':'vendor-info'}
i = getall[0] # 第一個店面
i
```
![image](https://user-images.githubusercontent.com/102600962/183038572-a024ccd6-16ac-4f8b-977d-4decb7d77271.png)

### 四、查找html所對應到的名稱
    
    店家名稱 <span class="name fn">臻蜜定食舖</span>
    評級     <span class="rating"><strong>4.3</strong>/5</span>
    特色     <li class="vendor-characteristic">
             <span>&lt;店內價&gt;</span>
             <span>台式</span>
    運費     <li class="delivery-fee">
             <strong>免費</strong> 外送
```Python
print(i.find('span',{'class':"name fn"}).text)
print(i.find('strong').text)
print(i.find('li',{'class':"vendor-characteristic"}).text)
#運費拆成stepbystep 是因為如果直接strong去搜尋會搜不到，所以我們一層一層剖開
step1 = i.find('ul',{'class':"extra-info mov-df-extra-info"})
step2 = step1.find('strong')
print(step2.text)
```
![image](https://user-images.githubusercontent.com/102600962/183040524-524fd839-71b4-487f-abc3-fb8b898bbebe.png)

### 五、第一家全部都搜尋到之後，我們以迴圈的方式，讓Python自動把每一家的相關資訊給列出

```Python
# 分別給個list，存取資料
ShopName = []
Rating = []
Tag = []
DeliveryFee = []
for i in getall:
    ShopName.append(i.find('span',{'class':"name fn"}).text)
    Rating.append(i.find('strong').text)
    Tag.append(i.find('li',{'class':"vendor-characteristic"}).text)
    i.find('li',{'class':"vendor-characteristic"}).text
    step1 = i.find('ul',{'class':"extra-info mov-df-extra-info"})
    step2 = step1.find('strong')
    DeliveryFee.append(step2.text)
    
# 變成DataFrame，然後再匯出成csv檔
FoodPanda =  pd.DataFrame({'店家名稱':ShopName,
                           '評級':Rating,
                           '標籤':Tag,
                           '運費':DeliveryFee
                           })
                           
```

##完整程式碼
```Python
import pandas as pd
import requests
from bs4 import BeautifulSoup

headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36"}
url = 'https://www.foodpanda.com.tw/city/taipei-city'
res = requests.get(url, headers=headers)

soup = BeautifulSoup(res.content, 'html.parser')

getall = soup.find_all('figcaption',{'class':'vendor-info'})
i = getall[0] #看第一個店面

print(i.find('span',{'class':"name fn"}).text)
print(i.find('strong').text)
print(i.find('li',{'class':"vendor-characteristic"}).text)
step1 = i.find('ul',{'class':"extra-info mov-df-extra-info"})
step2 = step1.find('strong')
print(step2.text)

#製作dataframe
ShopName = []
Rating = []
Tag = []
DeliveryFee = []
for i in getall:
    ShopName.append(i.find('span',{'class':"name fn"}).text)
    Rating.append(i.find('strong').text)
    Tag.append(i.find('li',{'class':"vendor-characteristic"}).text)
    i.find('li',{'class':"vendor-characteristic"}).text
    step1 = i.find('ul',{'class':"extra-info mov-df-extra-info"})
    step2 = step1.find('strong')
    DeliveryFee.append(step2.text)
    

FoodPanda =  pd.DataFrame({'店家名稱':ShopName,
                           '評級':Rating,
                           '標籤':Tag,
                           '運費':DeliveryFee
                           })
FoodPanda.to_csv('FoodPanda爬蟲初嘗試.csv', encoding='utf-8-sig', index=False)
```
