# 靜態方法 Promise.resolve與Promise.reject

> `Promise.reject`或`Promise.resolve`只用於單純的傳入物件、值(理由)或外部的thenable物件，轉換為Promise物件的場合

`Promise.resolve`是一個靜態方法，也就是說它前面一定要加上`Promise`這樣呼叫的方法。`Promise.resolve`等於是要產生fulfilled(已實現)狀態的Promise物件，它相當於直接回傳一個以`then`方法實現的新Promise物件，帶有傳入的參數值，這個方法的傳入值有三種，相當於`then`方法中onFulfilled函式的回傳值(上一章的內容中有說)，語法如下:

```js
Promise.resolve(value);
Promise.resolve(promise);
Promise.resolve(thenable);
```

直接用`Promise.resolve`代表這個新的Promise物件的狀態是直接設定為fulfilled(已實現)狀態，這方法只是方便產生根(第一個)Promise物件使用的，也就是除了用建構式的`new Promise()`外，提供另一種方式來產生新的Promise物件。下面為範例:

```js
Promise.resolve("Success").then(function(value) {
  console.log(value) // "Success"
}, function(value) {
  // 不會被呼叫
})
```

`Promise.reject`剛好與`Promise.resolve`相反，等於是要產生rejected(已拒絕)狀態的新Promise物件。範例如下:

```js
Promise.reject(new Error("fail")).then(function(error) {
  // 不會被呼叫
}, function(error) {
  console.log(error); // Stacktrace
});
```

`Promise.reject`與`Promise.resolve`的實作的程式碼隱藏了在內部關於Promise物件產生的過程，前面有提到我們不能用沒有executor傳入參數的Promise建構式來產生新的物件實體，會有錯誤發生。這兩個靜態方法，它們在實作時的確有產生沒有executor傳入參數的Promise雛形(中介)物件，只是它還不算真正會被回傳出來的Promise物件實體。理由主要當然是不讓程式設計師自由控制Promise的狀態，要改變Promise的狀態只有靠程式碼的執行結果才行。例如下面這一段程式碼(出自es6-promise):

```js
var promise = new Constructor(noop); //noop是沒有內容的函式
_resolve(promise, object); //_resolve是內部函式，object是Promise.resolve傳入參數值
return promise;
```

`Promise.reject`與`Promise.resolve`與使用Promise建構式的方式，在使用上仍然有大的不同，executor傳入參數可以讓程式設師自訂Promise產生的過程，而且在產生的過程中是throw safety(安全)的。以下面的例子來說明:

```js
//方式一: 使用Promise建構式

function initPromise1(someObject) {

    return new Promise(function(resolve) {
        if (typeof someObject !== 'object')
            throw new Error('error occured')
        else
            resolve(someObject);
    });
}

//方式二: 使用Promise.resolve+throw

function initPromise2(someObject) {
    if (typeof someObject !== 'object')
        throw new Error('error occured')
    else
        return Promise.resolve(someObject)
}

//方式三: 使用Promise.resolve+Promise.reject

function initPromise3(someObject) {
    if (typeof someObject !== 'object')
        return Promise.reject(new Error('error occured'))
    else
        return Promise.resolve(someObject)
}

//測試用
initPromise1(1).then((value) => {
    console.log(value)
}).catch((err) => {
    console.log(err.message)
})
```

如果此時傳入的`someObject`不是物件類型，方式二會直接throw出例外，造成程式中斷，與Promise結構中對錯誤的處理方式不同，只有方式一或方式二可以正常的處理錯誤。

上面的例子是太過簡單，如果`initPromise`函式中的程式碼很複雜的時候，你在回傳`Promise.reject`或`Promise.resolve`前發生例外，是完全沒辦法控制的。

結論是`Promise.reject`或`Promise.resolve`只用於單純的傳入物件、值或外部的thenable物件，轉換為Promise物件的場合。如果你要把一整段程式碼語句或函式轉為Promise物件，不要用這兩個靜態方法，要使用Promise建構式來產生物件才是正解。

> 註: 在Promises/A+並沒有關於`Promise.reject`或`Promise.resolve`的定義，它們是ES6 Promise標準中的實作。
