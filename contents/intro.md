# 前言

> 一個promise代表一個異步運算的最終結果 ~譯自[Promises/A+](https://promisesaplus.com/)

Promise語法結構提供了更多的程式設計上的可能性，它是一個經過長時間實戰的結構，在許多知名的函式庫或框架中很早就有見到Promise物件的身影，例如Dojo、jQuery、YUI、 Ember、Angular、WinJS、Q等等，之後Promises/A+社區則提供了統一的標準。在最近新一代的ES6標準中將會包含了Promise的實作，提供原生的語言內建支援，這將是個開始，往後會有愈來愈多API以此為基礎架構在其上。

本文的範例是介紹原生的ES6 Promise為主，從基本概念出發，以Promises/A+作為主要的規則說明，另外集合了很多的實例與深入設計原理的解說，其目的是希望剛入門Promise的學習者，能有得到扎實的基礎知識。最後提供了幾種應用實例與程式碼片段，希望能以最接近實際應用的範例，真正學會與活用Promise結構。這份文件大約陸陸續續寫了一週左右，相信你如果願意多花點時間多看幾遍內容，以及測試其中的範例，很快就可以學會Promise的基礎。

Promise這種異步執行結構的需求，在伺服器端(Node.js)遠遠大於瀏覽器端，原因是瀏覽器通常只有會有一個使用者在操作，而且除了網路連線要求之類的I/O(例如AJAX)、DOM事件處理、動畫流程處理(計時器)外，並沒有太多需要異步執行I/O處理的情況。伺服器端(Node.js)所面臨到情況就很嚴峻，除了與外部資源I/O的處理情況到處都有之外，而且伺服器端(Node.js)是需要同時服務多人使用的情況。

Promise是一個強大的異步執行流程語法結構，在ES6 Promise標準中，實作內容只有一個建構函式與一個`then`方法、一個`catch`方法，再加上四個必需以`Promise`關鍵字呼叫的靜態函式，`Promise.resolve`、`Promise.reject`、`Promise.all`、`Promise.race`。語法介紹頁面只有7頁，內容少之又少。但為何Promise結構不容易被理解？原因在於同步與異步回調函式執行的概念，以及其中很多流程運作需要用另一種方式來思考要如何進行。當然，需要理解的規則也很多。

ES6 Promise特性截至2016/7月為止，已有約市佔率70%以上的瀏覽器品牌支援，幾乎在所有新式的瀏覽器中都已經內建支援(參考[Can I use...](http://caniuse.com/#feat=promises))。對於尚未支援的舊版瀏覽器、IE系列的瀏覽器、或是某些行動裝置上的瀏覽器，都可以使用[es6-promise](https://github.com/stefanpenner/es6-promise)函式庫進行補充(Polyfill)，一樣可以使用相同的API。
