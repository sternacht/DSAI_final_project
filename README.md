# DSAI_final_project

## 目錄

- [成員](#成員)
- [目標](#目標)
- [資料分析](#資料分析)
- [訓練模型](#訓練模型)
- [預測](#預測)
- [成績](#成績)
- [執行備註](#執行備註)
## 成員
E94076194 黃彤洋
E94076186 許中瑋

## 比賽目標
**[Foursquare - Location Matching](https://www.kaggle.com/competitions/foursquare-location-matching/overview)**
比賽要求根據給予的地點資訊(包括店名、地址、國家、類別等12個訊息)，分辨是否與其他地點具有相同的 `POI`，正確的分辨越多組地點，則分數越高。
`POI` (point-of-interest) : 電子地圖上的標記點，`POI` 可以是一個景點、商店、設施等，讓使用者對此地點感到有興趣的。


## 資料分析
根據 kaggle 提供的 train.csv 及 pairs.csv 兩份資料，我們作了以下的觀察

1. pairs.csv 中提供的配對在距離上大多很接近，我們推斷具有相同 `POI` 的地點不應該距離過遠，並且在 train.csv 尋找、計算並排序具有相同 `POI` 之兩點的距離，我們發現有 95% 的配對距離在 12 公里以內，因此以 12 公里為第一個判斷的依據，假設超過的配對不具有相同 `POI`。
![](https://i.imgur.com/fXvkElD.png)
> 超過 12 公里的比例約佔 0.049 = 4.9%
2. pairs.csv 中的 `categories` 類別，會有多個類別同時存在的狀況，並且這些類別可能並不是相似的，因此在製作 feature 的時候，我們選擇不以布林值來做，而是透過比較相似的程度，做成一個 0~1 的值，來保留多個不同類別存在的特徵。
3. 從 `name` 及 `address` 兩個類別去觀察，有些店(地)名其實是相同的，但其中一個會有額外的注釋或資訊等，直接透過相等去比較兩者並不是好方法，因此我們選擇用計算字串相似度的方式來處理，而其他類別如 `city`, `state`, `country`, `url`, `phone`等，則不適用，而是以相等與否來分別。

## 訓練模型
模型使用的是 light gbm 模型，將 pairs.csv 中的 feature 進行處理
- `latitude`與`longitude`轉換成距離
- `address`與`name`計算出 Levenshtein ratio 與 LCS
- `city`、`state`、`country`、`url`、`phone`轉成小寫並去除空格後判斷是否相同
- `categories`計算兩者中同數量後再除以(兩者的 categories 數量相加/2)

並且因為 pairs.csv 中 True/False 的比例大約為 2:1，為了平衡 True/False 的比例，將 True 的資料去除一半後再開始進行訓練


## 預測
由於 test 的資料大約會是 50、60 萬筆的等級，因此若是直接進行每一筆的比較將會有相當大的計算量，因此我們由以下的步驟還篩選要放入 model 的資料:
1. 設定一個限定的距離 d
2. 將 test 以緯度排序
3. 由第一筆開始往後比對，若是緯度超過 d 的距離則停止此筆資料的比對，開始由下一筆資料往後比對
4. 比對的資料的距離經由計算後小於 d 
5. 確認`name_ratio`、 `address_ratio`、`city_match`、`state_match` 有任何一項大於 0.5 則將此對資料處理加入要預測的列表
6. 單筆資料若超過 500 次比對則直接由下一筆資料開始比對

經由model預測後，我們就可以得到預測為 match 的 id 對
一開始所有的id都擁有自己的label,經由 model 判斷為 match 的 id 則會被合併為相同的 label，最後再依照 label 產生 submission.csv

## 成績

![](https://i.imgur.com/42yDD9J.png)

我們測試了 kaggle 上別人的 code 有得到 0.834 的成績

![](https://i.imgur.com/dOeL5td.png)

但是我們自己的 code 目前還是在 0.63x 徘徊，沒有得到一個太好的成績


## 執行備註
在 kaggle 上 train 需要將 Foursquare - Location Matching 的資料加到 notebook 中

執行 predict 需要把 train 產生的 lbg_model.txt 載下後上傳至 kaggle 的 dataset
或是利用 api 使用已經上傳的 model
```
kaggle datasets download -d duncan881220/foursquarelgb
```
若是自行上傳則要注意dataset的名稱
