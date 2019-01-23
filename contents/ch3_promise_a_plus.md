---
id: ch3_promise_a_plus
title: Promises/A+標準定義
sidebar_label: Promises/A+標準定義
---

> 原生的 ES6 Promise 是符合 Promises/A+標準的

所謂的 [Promises/A+](https://promisesaplus.com/) 標準，其實就是個幾千字的一頁網頁而已，裡面的說明與用語並不會太難理解。雖然 ES6 標準中也有自己的 Promise 物件標準章節，但因為裡面涉及很多實作技術說明，明顯地用字遣詞艱澀許多，所以在這裡就不多加討論。以下使用 Promises/A+ 標準作為一個開始，來解說 Promise 的標準裡有什麼內容。這一章僅有定義部份，之後的解說也是會依照 Promises/A+ 標準中的規則來說明。

## 專門用語

- `promise` (承諾)是一個帶有遵照這個規格的 then 方法的物件
- `thenable` 是一個有定義 then 方法的物件
- `value` 合法的 JavaScript 值(包含 undefined、thenable 與 promise)
- `exception` (例外)使用 throw 語句丟出來的值
- `reason` (理由)是表明為什麼 promise 被拒絕(rejected)的值

> 註: 另外有個常見的專有名詞 `settled`(固定的) ，指的是一個 promise 最後的狀態，也就是 fulfilled(已實現) 或 rejected(已拒絕) 其中一種。

> 註: `reason`(理由)通常是一個 Error 物件，用於錯誤處理。

> 註: `promise`/帕咪死/ 的中文翻譯是"承諾"、"約定"，本書中並不會用它的中文翻譯字詞，都是直接用英文。

## Promise 狀態

promise 物件必定是以下三種狀態中的其中一種: pending(等待中)、fulfilled(已實現)或 rejected(已拒絕)。

2.1.1 當處在 pending(等待中)時，一個 promise:

    2.1.1.1 可能會轉變到不是 fulfilled(已實現) 就是 rejected(已拒絕) 狀態

2.2.1 當處在 fulfilled(已實現) 時，一個 promise:

    2.2.1.1 必定不會再轉變到其他任何狀態
    2.2.1.2 必定有不能再更動的值

2.3.1 當處在 rejected(已拒絕) 時，一個 promise:

    2.3.1.1 必定不會再轉變到其他任何狀態
    2.3.1.2 必定有不能再更動的值 reason(理由)

這個用下面的圖解說明，應該可以很清楚的理解:

![Promise狀態](https://raw.githubusercontent.com/eyesofkids/javascript-entry-level-es6/master/assets/promise_1.png)

狀態是在 Promise 結構很重要的一個屬性，因為 promise 物件一開始都是代表懸而未決的值，所以一開始在 promise 物件在建立時，狀態都是 pending(等待中)，之後可以轉變到 fulfilled(已實現)就是 rejected(已拒絕)其中一個，然後就固定不變了。如果有產生 value(值)的情況就是轉變到 fulfilled(已實現)狀態，而如果是有 reason(理由)時，代表要轉變到 rejected(已拒絕)狀態。

不過，講是這樣講，在實作時 promise 物件一旦實體化完成回傳出來，就已經決定好是那一種狀態了，要不就是 fulfilled(已實現)，要不就是 rejected(已拒絕)。只是在實體化過程中，這個還不存在的(還沒回傳出來的)的 promise 物件的確是 pending(等待中)狀態。看到後面的章節內容你大概能體會，其實這是一種保護措施，它的設計就是這樣子。
