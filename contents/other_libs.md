# Promise外部函式庫議題

在ES6標準正式定案將Promise特性加入後，再加上各家瀏覽器紛紛發佈已附有實作Promise特性的新版本後，約莫2015年中開始，有許多Promise相關的外部函式庫，有逐漸的發佈趨緩，或是停止再維護的現象。以下列出幾套常見有遵照Promise標準，而且有進行擴充的函式庫(單純只加速或polyfill的不列入)，以及它們的發佈情況:

- [q](https://github.com/kriskowal/q/releases): 維護停止，最後發佈版本在2015/5
- [bluebird](https://github.com/petkaantonov/bluebird/releases): 維護持續(唯一)，最後發佈版本在2016/6
- [when](https://github.com/cujojs/when/releases): 維護停止，最後發佈版本在2015/12
- [then-promise](https://github.com/then/promise/releases): 維護停止，最後發佈版本在2015/12
- [rsvp.js](https://github.com/tildeio/rsvp.js/releases): 維護停止，最後發佈版本在2016/2
- [vow](https://github.com/dfilatov/vow/releases): 維護停止，最後發佈版本在2015/12

> 註: 維護停止，代表6個月左右都沒有新的發佈版本

由列表中可以看到至2016/7目前唯一還在在更新版本的，就只有[bluebird](https://github.com/petkaantonov/bluebird/releases)一套而已。下面分析為何會有這個現象發生，以及到底你要選擇用原生的ES6 Promise還是要使用外部函式庫的一些建議。

首先根據一篇在2015/4月網路上的問答[為何原生的ES6 Promise會比bluebird慢而且更耗記憶體](http://programmers.stackexchange.com/questions/278778/why-are-native-es6-promises-slower-and-more-memory-intensive-than-bluebird)中回答，這是由bluebird函式庫的作者回答的內容來看，當時的原生ES6 Promise，又慢又耗記憶體的主因如下:

1. 尚未最佳化，實作當時也只是JavaScript程式碼
2. 原生ES6 Promise使用非常耗資源的`new Promise`與executor方式來建立根(第一個)Promise物件

當時2015年的效能評比大概的資料還在[這裡](https://github.com/petkaantonov/bluebird/tree/master/benchmark)。

- 序列執行(sequential): 原生ES6 Promise的執行時間需要bluebird的4倍，記憶體需要3.6倍
- 並行執行(parallel): 原生ES6 Promise的執行時間需要bluebird的9倍，記憶體需要5.7倍

看到這個結果，當然有很多人會存疑，為何bluebird會快成這樣，或是原生ES6 Promise會慢成這樣，就算是都用JavaScript程式碼實作，這麼大的差距也是很誇張。實際上在設計的本質上並不相同，bluebird以較有效率而且節省資源的方式來設計整體的執行流程，其他的函式庫都沒有辦法這麼快又節省資源。根據[這篇bluebird的issue](https://github.com/petkaantonov/bluebird/issues/381)中的開發者指出，以Q與bluebird的設計比較，Q會慢bluebird幾十倍是因為Q針對瀏覽器提供某些安全上的設計，bluebird則是極致的追求執行效率，假設所有程式碼都是在完全可信任的環境例如Node.js中執行。或許也因為如此，在網路上很多類似的問答中，通常建議只在伺服器端(Node.js)使用bluebird，而瀏覽器端建議使用原生ES6 Promise或Q。

當然，經過一年之後的原生ES6 Promise，其效率與記憶體消耗問題已有了很大的改善，根據最近Chrome瀏覽器使用的V8引擎在[版本53發佈的消息](http://v8project.blogspot.tw/2016/07/v8-release-53.html)，目前的原生ES6 Promise進行最佳化後，效能改善了20-40%。也就是說未來的原生ES6 Promise在經過最佳化後，與外部函式庫的執行效率並不會相差太遠，這個消息無疑為原生ES6 Promise打了一劑強心針。

所以如果你要要使用外部函式庫，其理由可能是以下幾個:

- 為了能使用豐富的API，但不見得是為了效率，效率有可能在將來並不是唯一的重點。
- 資源存取類型的函式庫或模組，現在都會提供具有Promise的API，例如資料庫、檔案處理、網路資源存取的模組。例如[MongoDB Node.JS Driver](https://mongodb.github.io/node-mongodb-native/)就是雙模式的。有一些AJAX的函式庫也都是用Promise的架構，例如[axios](https://github.com/mzabriskie/axios)，或是像新式的標準Fetch API，也是基於Promise的。
- 工具類型的函式庫，現在也提供相容於Promise的物件，例如jQuery(3.0之後)。

雖然jQuery 3.0中實作了符合Promises/A+標準的Promise物件，但是jQuery並非單純用於Promise架構的外部函式庫，而且3.0也才剛發佈不久，再加上長期以來，jQuery有很多自己設計的擴充API，API的名稱與標準中有些不同。如果你原本就有使用jQuery，是可以使用它新版本中的deferred物件與Promise物件的功能，但要仔細的理解與原生ES6 Promise的差異性。

原生ES6 Promise會被認為是一個基礎，如果是在很簡單的執行流程，或許就已經夠用了。學習語法和使用方法後，以這個基礎你可以因應不同的需求，使用專屬的資源存取模組(例如資料庫)。而這些專門使用在Promise擴充的函式庫(例如bluebird)，雖然提供了豐富API，但使用它們的情況相信會愈來愈少，這也是為什麼後來這些函式庫都停止維護的主要原因。
