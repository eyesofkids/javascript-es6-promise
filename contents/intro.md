---
id: intro
title: 前言
sidebar_label: 前言
---

> 一個 promise 代表一個異步運算的最終結果 ~譯自[Promises/A+](https://promisesaplus.com/)

Promise 語法結構提供了更多的程式設計上的可能性，它是一個經過長時間實戰的結構，在許多知名的函式庫或框架中很早就有見到 Promise 物件的身影，例如 Dojo、jQuery、YUI、 Ember、Angular、WinJS、Q 等等，之後 Promises/A+社區則提供了統一的標準。在最近新一代的 ES6 標準中將會包含了 Promise 的實作，提供原生的語言內建支援，這將是個開始，往後會有愈來愈多 API 以此為基礎架構在其上。

本文的範例是介紹原生的 ES6 Promise 為主，從基本概念出發，以 Promises/A+作為主要的規則說明，另外集合了很多的實例與深入設計原理的解說，其目的是希望剛入門 Promise 的學習者，能有得到扎實的基礎知識。最後提供了幾種應用實例與程式碼片段，希望能以最接近實際應用的範例，真正學會與活用 Promise 結構。這份文件大約陸陸續續寫了一週左右，相信你如果願意多花點時間多看幾遍內容，以及測試其中的範例，很快就可以學會 Promise 的基礎。

Promise 這種異步執行結構的需求，在伺服器端(Node.js)遠遠大於瀏覽器端，原因是瀏覽器通常只有會有一個使用者在操作，而且除了網路連線要求之類的 I/O(例如 AJAX)、DOM 事件處理、動畫流程處理(計時器)外，並沒有太多需要異步執行 I/O 處理的情況。伺服器端(Node.js)所面臨到情況就很嚴峻，除了與外部資源 I/O 的處理情況到處都有之外，而且伺服器端(Node.js)是需要同時服務多人使用的情況。

Promise 是一個強大的異步執行流程語法結構，在 ES6 Promise 標準中，實作內容只有一個建構函式與一個`then`方法、一個`catch`方法，再加上四個必需以`Promise`關鍵字呼叫的靜態函式，`Promise.resolve`、`Promise.reject`、`Promise.all`、`Promise.race`。語法介紹頁面只有 7 頁，內容少之又少。但為何 Promise 結構不容易被理解？原因在於同步與異步回調函式執行的概念，以及其中很多流程運作需要用另一種方式來思考要如何進行。當然，需要理解的規則也很多。

ES6 Promise 特性截至 2016/7 月為止，已有約市佔率 70%以上的瀏覽器品牌支援，幾乎在所有新式的瀏覽器中都已經內建支援(參考[Can I use...](http://caniuse.com/#feat=promises))。對於尚未支援的舊版瀏覽器、IE 系列的瀏覽器、或是某些行動裝置上的瀏覽器，都可以使用[es6-promise](https://github.com/stefanpenner/es6-promise)函式庫進行補充(Polyfill)，一樣可以使用相同的 API。
