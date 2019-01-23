---
id: ch1_intro
title: 前言
sidebar_label: 前言
---

> 一個 promise 代表一個異步運算的最終結果 ~譯自[Promises/A+](https://promisesaplus.com/)

Promise 語法結構提供了更多的程式設計上的可能性，它是一個經過長時間實戰的結構，在許多知名的函式庫或框架中很早就有見到 Promise 物件的身影，例如 Dojo、jQuery、YUI、 Ember、Angular、WinJS、Q 等等，之後 Promises/A+ 社區則提供了統一的標準。在最近新一代的 ES6 標準中將會包含了 Promise 的實作，提供原生的語言內建支援，這將是個開始，往後會有愈來愈多 API 以此為基礎架構在其上。

本文的範例是介紹原生的 ES6 Promise 為主，從基本概念出發，以 Promises/A+ 作為主要的規則說明，另外集合了很多的實例與深入設計原理的解說，其目的是希望剛入門 Promise 的學習者，能有得到扎實的基礎知識。最後提供了幾種應用實例與程式碼片段，希望能以最接近實際應用的範例，真正學會與活用 Promise 結構。這份文件大約陸陸續續寫了一週左右，相信你如果願意多花點時間多看幾遍內容，以及測試其中的範例，很快就可以學會 Promise 的基礎。

Promise 這種異步執行結構的需求，在伺服器端(Node.js)遠遠大於瀏覽器端，原因是瀏覽器通常只有會有一個使用者在操作，而且除了網路連線要求之類的 I/O(例如 AJAX)、DOM 事件處理、動畫流程處理(計時器)外，並沒有太多需要異步執行 I/O 處理的情況。伺服器端(Node.js)所面臨到情況就很嚴峻，除了與外部資源 I/O 的處理情況到處都有之外，而且伺服器端(Node.js)是需要同時服務多人使用的情況。

Promise 是一個強大的異步執行流程語法結構，在 ES6 Promise 標準中，實作內容只有一個建構函式與一個`then`方法、一個`catch`方法，再加上四個必需以`Promise`關鍵字呼叫的靜態函式，`Promise.resolve`、`Promise.reject`、`Promise.all`、`Promise.race`。語法介紹頁面只有 7 頁，內容少之又少。但為何 Promise 結構不容易被理解？原因在於同步與異步回調函式執行的概念，以及其中很多流程運作，需要用另一種方式來思考要如何進行。當然，需要理解的規則也很多。以初學者來說，從規則來理解它的運作方式，會是比較容易進入的學習路徑。

> 註：Promise 另一個會受到重視的原因，是因為新式的 HTML5 中的 Fetch API(用於取代舊有的 AJAX 語法結構)也是使用它。

## 相容性議題

ES6 Promise 特性截至 2019/1 為止，已有市佔率 89% 以上的瀏覽器支援，在所有新式的瀏覽器中都已經內建支援(參考[Can I use...](http://caniuse.com/#feat=promises))。對於尚未支援的舊版瀏覽器、IE 系列的瀏覽器、或是某些行動裝置上的瀏覽器，都可以使用[es6-promise](https://github.com/stefanpenner/es6-promise)函式庫進行補充(Polyfill)，一樣可以使用相同的 API。

> 註：補充(Polyfill)因為是使用舊有的例如 setTimeout 等 API 模擬出的語法，效能必定非常的差，只是作為過渡的相容性解決方案，不建議在複雜的應用中使用。

## 效能議題

Promise 最大的效益在於它改變了異步處理的程式碼結構，而不在於增加了多少效能，以原本的 callback 結構(CPS 風格)與 Promise 相比來說：

- 一個 callback: 是一種函式，傳入到另一個函式作為傳入參數值
- 一個 Promise: 是一種函式的物件(代理 Proxy)，代表未決定的狀態值

Promise 與 callback 是全然不同的概念的東西。而 Promise 的對於資源高消費是必然的，執行引擎內部的實作也更複雜，它的效能永遠比不上原本單純的 callback 結構，這一點初學者必需要很清楚的認知。

Chrome 瀏覽器的執行引擎 V8 開發團隊，非常重視關於 Promise 的執行效率，在這份 2018/11 的部落格[Faster async functions and promises](https://v8.dev/blog/fast-async)中，展示了從 Node.js v7 到 v10 中(約 2 年的時間)，在 Promise 這種異步的執行效能，幾乎執是以倍數的進步，相信未來的執行效能會更好。
