# Promises/A+標準定義

> 原生的ES6 Promise是符合Promises/A+標準的

所謂的[Promises/A+](https://promisesaplus.com/)標準，其實就是個幾千字的一頁網頁而已，裡面的說明與用語並不會太理解。雖然ES6標準中也有自己的Promise物件章節，但因為涉及實作技術，內容明顯用字遣詞艱澀許多，在此不多加討論。以下使用Promises/A+標準作為一個開始，來解說Promise的標準裡有什麼內容。

## 專門用語

- `promise` (承諾)是一個帶有遵照這個規格的then方法的物件
- `thenable` 是一個有定義then方法的物件
- `value` 合法的JavaScript值(包含undefined、thenable與promise)
- `exception` (例外)使用throw語句丟出來的值
- `reason` (理由)是表明為什麼promise被拒絕(rejected)的值

> 註: 另外有個常見的專有名詞 `settled`(固定的) 一個promise最後的狀態，也就是fulfilled(已實現)或rejected(已拒絕)

> 註: `reason`(理由)通常是一個Error物件，用於錯誤處理。

## Promise狀態

promise物件必定是以下三種狀態中的其中一種: pending(等待中)、fulfilled(已實現)或rejected(已拒絕)。

2.1.1 當處在pending(等待中)時，一個promise:

    2.1.1.1 可能會轉變到不是fulfilled(已實現)就是rejected(已拒絕)狀態

2.2.1 當處在fulfilled(已實現)時，一個promise:

    2.2.1.1 必定不會再轉變到其他任何狀態
    2.2.1.2 必定有不能再更動的值

2.3.1 當處在rejected(已拒絕)，一個promise:

    2.3.1.1 必定不會再轉變到其他任何狀態
    2.3.1.2 必定有不能再更動的值reason(理由)

這個用下面的圖解說明，應該可以很清楚的理解:

![Promise狀態](https://raw.githubusercontent.com/eyesofkids/javascript-entry-level-es6/master/assets/promise_1.png)

狀態是在Promise結構很重要的一個屬性，因為promise物件一開始都是代表懸而未決的值，所以一開始在promise物件在建立時，狀態都是pending(等待中)，之後可以轉變到fulfilled(已實現)就是rejected(已拒絕)其中一個，然後就固定不變了。通常有value(值)的情況是轉變到fulfilled(已實現)狀態，而如果是有reason(理由)時，代表要轉變到rejected(已拒絕)狀態。

不過，講是這樣講，在實作時promise物件一旦實體化完成回傳出來，就已經決定好是那一種狀態了，要不就是fulfilled(已實現)，要不就是rejected(已拒絕)。只是在實體化過程中，這個還不存在的(還沒回傳出來的)的promise物件的確是pending(等待中)狀態。
