---
id: ch6_then_adv
title: 深入 then 方法
sidebar_label: 深入 then 方法
---

> then 方法實際上是整個 Promise 結構的流程運作起來的主要關鍵

為了先讓你的概念能比較清楚，我列出幾個小問題的問答。這一章節是本書中內容最難理解的一章，有可能你需要多看幾次。如果你有看不懂的地方，可以翻翻前面的內容，或許你可以找到一些基礎或想法。

## 概念問與答

### promise 中會用到的回調函式，都是在執行時會以異步執行的回調函式嗎？

**當然**。

### 一個 promise 物件經過 then 後，原本的 promise 物件的內容會改變？

**不會**。then 方法執行完會另外產生一個新的 promise 物件。以下為解說範例:

```js
const promise = new Promise(function(resolve, reject) {
  resolve(1)
})

const p = promise

const p1 = promise.then(value => {
  console.log(value)
  return value + 1
})

const p2 = promise.then(value => {
  console.log(value)
  return value + 1
})

//延時執行
setTimeout(() => {
  console.log(p1)
  console.log(p2)
  console.log(p1 === p2) //false
  console.log(p === promise) //true
}, 5000)
```

## `then`方法中 onFulfilled 函式的回傳值

這個章節的內容是要討論在 onFulfilled 函式或 onRejected 函式，你可以自訂什麼回傳值。

上面已經有提到關於回傳值的差異情況，是會影響到新的 Promise 物件的狀態，你可以對照看看。

`then`方法中 onFulfilled 函式回傳值，或是 Promise 建構函式中 executor(執行者, 執行函式)的 resolve 中的參數值，這是已經進入正常的"執行 Promise 解析程序"中。相較於 onRejected 函式，它通常只會用 reason(理由)來處理，而且通常是個 Error 物件。不過，如果你的 onRejected 函式有回傳值，它的規則也會和 onFulfilled 回傳值一模一樣。

`then`方法中 onFulfilled 函式回傳值可以有三種不同類型:

- 值
- promise 物件
- thenable 物件

值的話，就一般在 JavaScript 的各種值，這沒什麼好講的，因為`then`方法最後會回傳 Promise 物件，所以這個回傳值會跟著這個新 Promise 物件到下一個連鎖方法去，這個回傳值可以用下個`then`方法的 onFulfilled 函式取到第 1 個傳入參數值。

promise 物件的話，當你不希望`then`幫你回傳 promise 物件，自己建立一個，這也是合情合理，有些開發者會乾脆把所有的的函式改寫成回傳 promise 物件，這也是可行的寫法。通常是使用 promise 建構式來 new 一個，或是直接用`Promise.resolve(value)`靜態方法，產生一個 fulfilled(已實現)狀態的 promise 物件。

thenable 物件是最特殊的，根據標準的定義為:

> `thenable` 是一個有定義 then 方法的物件或函式

按照標準上的解說，thenable 是提供給沒有符合標準的其他實作函式庫或框架，利用合理的`then`方法來進行同化的的方式。以最簡單的情況說明，thenable 物件是個純物件，然後裡面有個 then 方法的定義而已，例如以下的範例:

```js
const thenable1 = {
  then: function(onFulfill, onReject) {
    onFulfill('fulfilled!')
  },
}

const thenable2 = {
  then: function(resolve) {
    throw new TypeError('Throwing')
    resolve('Resolving')
  },
}
```

## then 中傳入參數值

這個章節的內容是要討論，你可以把什麼東西傳到`then`方法的傳入參數中，作為 onFulfilled 或 onRejected，當然正常情況應該要是個函式。

`then`方法有兩個傳入參數值，分別為 onFulfilled 與 onRejected，像下面這樣的語法:

```
promise.then(onFulfilled, onRejected)
```

`then`是 promise 物件中的方法，以 onFulfilled 與 onRejected 作為兩個傳入參數值，有幾個規則需要遵守:

- 當 onFulfilled 或 onRejected 不是函式時，忽略跳過
- 當 promise 是 fulfilled 時，執行 onFulfilled 函式，並帶有 promise 的 value 作為 onFulfilled 函式的傳入參數值
- 當 promise 是 rejected 時，執行 onRejected 函式，並帶有 promise 的 reason 作為 onRejected 函式的傳入參數值

then 方法最後還要回傳另一個 promise，也就是:

```
promise2 = promise1.then(onFulfilled, onRejected)
```

以下假設只使用 then 方法中的第 1 個 onFulfilled 傳入參數(onRejected 也是一樣)，那麼它可以有四種傳入的情況。在這個範例中，我們會把焦點放在其中有關於值的回傳情況。仔細看下面的範例中的`doSomething2`函式:

```js
function doSomething1() {
  console.log('doSomething1 start')
  return new Promise(function(resolve, reject) {
    console.log('doSomething1 end')
    resolve(1)
  })
}

function doSomething2() {
  console.log('doSomething2')
  return 2
}

function finalThing(value) {
  console.log('finalThing')
  console.log(value)
  return 0
}

// 第 1 種傳入參數
doSomething1()
  .then(doSomething2)
  .then(finalThing)

// 第 2 種傳入參數
doSomething1()
  .then(doSomething2())
  .then(finalThing)

// 第 3 種傳入參數
doSomething1()
  .then(function() {
    doSomething2()
  })
  .then(finalThing)

// 第 4 種傳入參數
doSomething1()
  .then(function() {
    return doSomething2()
  })
  .then(finalThing)
```

第 1 種: 正常的函式傳入參數。最後的`finalThing`可以得到`doSomething2`回傳值`2`。

第 2 種: 雖然說`then`方法的規則，如果 onFulfilled 不是函式時會忽略，但這裡是執行`doSomething2()`函式，onFulfilled 相當於`doSomething2()`的回傳值，JavaScript 中的函式回傳值可以是個函式，沒執行過怎麼會知道它是不是回傳一個函式？所以會執行`doSomething2()`，但最後得到 onFulfilled 不是一個函式，所以忽略它。依照連鎖規則 2.2.7.3(前面的章節中有)，當 onFulfilled 不是函式，繼續用 fulfilled 狀態與帶值回傳新的 Promise 物件到下個 then 方法，最後的`finalThing`得到的值是`doSomething1`中的`1`。

第 3 種: 正常的函式傳入參數，因為在函式中執行`doSomething2()`，這個 onFulfilled 最後的回傳值其實是 undefined，但是算有回傳值，回傳的新 Promise 物件也是 fulfilled 狀態，不過值變成`undefined`。最後的`finalThing`得到`undefined`值。

第 4 種: 正常的函式傳入參數，`then`方法執行完 onFulfilled 最後的回傳值是`doSomething2()`的執行後的值也就是`2`，回傳的新 Promise 物件也是 fulfilled 狀態，最後的`finalThing`得到`2`值。

> 註: 在 JavaScript 中函式的設計，必定有回傳值，沒寫只是回傳 undefined，相當於`return undefined`

> 註: 第 2 種與第 3 種是反樣式(anti-pattern)，是經常會發生錯誤的使用方式，實際使用時要避免使用。

## then 中傳入參數值(帶異步執行的函式)

上面的概念如果在 doSomething1、doSomething2 與 finalThing 函式中，都有異步的 callbacks(回調)時，這時除了值的情況，我們還會關心整體的執行順序。這個程式碼範例是來自[We have a problem with promises](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)，其實這已經是稍微進階的討論議題了。

```js
function doSomething1() {
  console.log('doSomething1 start')
  return new Promise(function(resolve, reject) {
    setTimeout(function() {
      console.log('doSomething1 end')
      resolve(1)
    }, 1000)
  })
}

function doSomething2() {
  console.log('doSomething2 start')
  return new Promise(function(resolve, reject) {
    setTimeout(function() {
      console.log('doSomething2 end')
      resolve(2)
    }, 1000)
  })
}

function finalThing(value) {
  console.log('finalThing start')
  return new Promise(function(resolve, reject) {
    setTimeout(function() {
      console.log('finalThing end')
      console.log(value)
      resolve(0)
    }, 1000)
  })
}

// 第 1 種傳入參數，finalThing最後的值為2
doSomething1()
  .then(doSomething2)
  .then(finalThing)

//第 2 種傳入參數，finalThing最後的值為1
doSomething1()
  .then(doSomething2())
  .then(finalThing)

// 第 3 種傳入參數，finalThing最後的值為undefined
doSomething1()
  .then(function() {
    doSomething2()
  })
  .then(finalThing)

// 第 4 種傳入參數，finalThing最後的值為2
doSomething1()
  .then(function() {
    return doSomething2()
  })
  .then(finalThing)
```

執行的結果會出乎想像，只有第 1 種與第 4 種，才是完整的 Promise 流程順序，也就是像下面的流程:

```
doSomething1 start
doSomething1 end
doSomething2 start
doSomething2 end
finalThing start
finalThing end
```

第 2 種中的`then`方法裡 onFulfilled 傳入參數的`doSomething2()`執行語句，它是一個同步的語句，所以會在`doSomething1`還沒執行完成時，就先被執行，所以流程會變為:

```
doSomething1 start
doSomething2 start
doSomething1 end
finalThing start
doSomething2 end
finalThing end
```

第 3 種在`then`方法裡裡 onFulfilled 的`function(){ doSomething2() }`裡面的`doSomething2()`執行語句，也是一個同步的語句，但外團的匿名函式卻是一個異步函式，因為這樣會在`doSomething1()`結束才開始執行，但是也是在`finalThing`開始後才會結束。不過流程也是怪異:

```
doSomething1 start
doSomething1 end
doSomething2 start
finalThing start
doSomething2 end
finalThing end
```

從這個範例中，我會認為並不需要太深究其中的順序的原因是為何。這範例的主要目題，其實是在告訴你不要亂用`then`方法中的傳入參數值，要不就是個堂堂正正的函式，要不然就寫好一個有回傳值的匿名函式。更進一步，如果你要在 Promise 結構中使用其他的異步 API，要特別注意它們的執行順序，用想的還不如直接寫出來執行看看，最後結果有可能不見得是最後是你要的。

> 註: 第 2 種與第 3 種是反樣式(anti-pattern)，是經常會發生錯誤的使用方式，實際使用時要避免使用。
