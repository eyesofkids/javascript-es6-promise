---
id: ch12_other_libs
title: Promise外部函式庫議題
sidebar_label: Promise外部函式庫議題
---

在 ES6 標準正式定案將 Promise 特性加入後，再加上各家瀏覽器紛紛發佈已附有實作 Promise 特性的新版本後，約莫 2015 年中開始，有許多 Promise 相關的外部函式庫，有逐漸的發佈趨緩，或是停止再維護的現象。

目前唯一還在在更新版本的，就只有[bluebird](https://github.com/petkaantonov/bluebird/releases)而已。下面分析為何會有這個現象發生，以及到底你是要選擇用原生的 ES6 Promise 就好，還是要使用其他外部函式庫中的 Promise 相關 API 的一些建議。

根據一篇在 2015/4 月網路上的問答[為何原生的 ES6 Promise 會比 bluebird 慢而且更耗記憶體](http://programmers.stackexchange.com/questions/278778/why-are-native-es6-promises-slower-and-more-memory-intensive-than-bluebird)的內容，回答的內容是由 bluebird 函式庫的創作者所寫的。當時(2015 初)的原生 ES6 Promise，又慢又耗記憶體的主因如下:

1. 尚未最佳化，實作雖然有也是 JavaScript 程式碼
2. 原生 ES6 Promise 使用非常耗資源的`new Promise`與 executor 方式來建立根(第一個)Promise 物件

從效能評比來看，當時(2015)的效能評比資料還在[這裡](https://github.com/petkaantonov/bluebird/tree/master/benchmark)。

- 序列執行(sequential): 原生 ES6 Promise 的執行時間需要 bluebird 的 4 倍，記憶體需要 3.6 倍
- 並行執行(parallel): 原生 ES6 Promise 的執行時間需要 bluebird 的 9 倍，記憶體需要 5.7 倍

看到這個結果，當然有很多人會存疑，為何 bluebird 會快成這樣，或是原生 ES6 Promise 會慢成這樣，就算是都用 JavaScript 程式碼實作，這麼大的差距也是很誇張。

實際上會出在設計出發點與本質並不相同，bluebird 以較有效率而且節省資源的方式來設計，不只是原生的 ES6 Promise，其他的函式庫都沒有辦法這麼快又節省資源。根據[這篇 bluebird 的 issue](https://github.com/petkaantonov/bluebird/issues/381)中的開發者指出，以 Q 與 bluebird 的設計比較，Q 會慢 bluebird 幾十倍是因為 Q 針對瀏覽器提供某些安全上的設計，bluebird 則是極致的追求執行效率，假設所有程式碼都是在完全可信任的環境例如 Node.js 中執行。或許也因為如此，在網路上很多類似的問答中，通常建議只在伺服器端(Node.js)使用 bluebird，而瀏覽器端建議使用原生 ES6 Promise 或 Q。

當然，經過一年之後的原生 ES6 Promise，其效率與記憶體消耗問題已有了很大的改善，根據最近(2016/7)Chrome 瀏覽器使用的 V8 引擎在[版本 53 發佈的消息](http://v8project.blogspot.tw/2016/07/v8-release-53.html)，目前的原生 ES6 Promise 進行最佳化後，效能改善了 20-40%。也就是說未來的原生 ES6 Promise 在經過最佳化後，與外部函式庫的執行效率並不會相差太遠，這個消息無疑為原生 ES6 Promise 打了一劑強心針。

所以如果你要要使用外部函式庫，其理由可能是以下幾個:

- 為了能使用豐富的 API。效率有可能在將來並不是唯一的重點。
- 資源存取類型的函式庫或模組，現在都會提供具有 Promise 的 API，例如資料庫、檔案處理、網路資源存取的模組。例如[MongoDB Node.JS Driver](https://mongodb.github.io/node-mongodb-native/)就是雙模式的。有一些 AJAX 的函式庫也都是用 Promise 的架構，例如[axios](https://github.com/mzabriskie/axios)，或是像新式的標準 Fetch API，也是基於 Promise 的。
- 工具類型的函式庫，現在可能也有提供相容於 Promise 標準的物件，例如 jQuery(3.0 之後)。

雖然 jQuery 3.0 中實作了符合 Promises/A+標準的 Promise 物件，但是 jQuery 並非單純用於 Promise 架構的外部函式庫，而且 3.0 也才剛發佈不久，再加上長期以來，jQuery 有很多自己設計的擴充 API，API 的名稱與標準中有些不同。如果你原本就有使用 jQuery，是可以使用它新版本中的 deferred 物件與 Promise 物件的 API，但可能還是需要具備 Promise 的使用基礎知識比較好。

原生 ES6 Promise 會被認為是一個基礎，如果是在很簡單的執行流程，或許就已經夠用了。學習語法和使用方法後，以這個基礎你可以因應不同的需求，使用專屬的資源存取模組(例如資料庫)。而這些專門使用在 Promise 擴充的函式庫，雖然額外提供了更多可使用的 API，但使用它們的情況相信會愈來愈少，這也是為什麼後來這些函式庫都停止維護的主要原因。如果你現在需要在伺服器端(Node.js)中，馬上就要使用高效能或豐富 API 的外部函式庫，那麼有可能 bluebird 會是比較好的選擇。原生 ES6 Promise 仍然需要一點時間，除了它的最佳化尚未完全完成之外，它的 API 的確真的少，面對複雜的應用情況可能會不符使用。
