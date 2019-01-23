---
id: ch4_basic_usage
title: Promise物件建立與基本使用
sidebar_label: Promise物件建立與基本使用
---

## Promise 物件的建立

> ES6 Promise 的實作中，會確保 Promise 物件一經實體化後就會固定住狀態，要不就是"已實現"，要不就是"已拒絕"。

一個簡單的 Promise 語法結構如下:

```js
const promise = new Promise(function(resolve, reject) {
  // 成功時
  resolve(value)
  // 失敗時
  reject(reason)
})

promise.then(
  function(value) {
    // on fulfillment(已實現時)
  },
  function(reason) {
    // on rejection(已拒絕時)
  }
)
```

首先先看 Promise 的建構函式，它的語法如下(來自[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)):

```
new Promise( function(resolve, reject) { ... } )
```

用箭頭函式可以簡化一下:

```
new Promise( (resolve, reject) => { ... } )
```

建構函式的傳入參數值需要一個函式。

這個函式中又有兩個傳入參數值，`resolve`(解決)與`reject`(拒絕)都是要求一定是函式類型。成功的話，也就是有合法值的情況下執行`resolve(value)`，promise 物件的狀態會跑到 fulfilled(已實現)固定住。失敗或是發生錯誤時用執行`reject(reason)`，reason(理由)通常是用 Error 物件，然後 promise 物件的狀態會跑到 rejected(已拒絕)狀態固定住。

這個建構函式的傳入函式稱為 executor(執行者, 執行函式)，這有強制執行的意味。這是一種名稱為[暴露的建構式樣式 Revealing Constructor Pattern](https://blog.domenic.me/the-revealing-constructor-pattern/)的樣式。executor 會在建構式回傳物件實體前立即執行，也就是說當傳入這個函式時，Promise 物件會立即決定裡面的狀態，看是要執行`resolve`來回傳值，還是要用`reject`來作錯誤處理。也因為它與一般的物件實體化的過程不太一樣，所以常會先包到一個函式中，使用時再呼叫這個函式來產生 promise 物件，例如像下面這樣的程式碼:

```js
function asyncFunction(value) {
  return new Promise(function(resolve, reject) {
    if (value) resolve(value)
    // 已實現，成功
    else reject(reason) // 有錯誤，已拒絕，失敗
  })
}
```

Promise 建構函式與 Promise.prototype 物件的設計目的，主要是要讓開發者作 Promisify 使用的，也就是要把原本異步或同步的程式碼或函式，打包成為 Promise 物件後在程式碼裡使用。經過包裝之後才能使用 Promise 的語法結構，也就是我們所說的異步函式執行流程的語法結構。

你可能會認為在`new Promise(function(resolve, reject){...})`中的`resolve`與`reject`兩個傳入參數值的名稱，就一定要是這樣。而且學到最後面，最會讓人搞混的是 Promise 物件中也有兩個靜態方法(說明在下面章節)，名稱也恰好是`Promise.resolve`與`Promise.reject`。

正確的解答是，"**並不是一定要這樣命名**"。這兩個名稱只是對照之後執行時，為了閱讀性或習慣上這樣命名而已。你可以執行下面的範例，看能不能執行:

```js
const promise = new Promise(function(resolveParam, rejectParam) {
  //resolveParam(1)
  rejectParam(new Error('error!'))
})

promise
  .then(value => {
    console.log(value) // 1
    return value + 1
  })
  .then(value => {
    console.log(value) // 2
    return value + 2
  })
  .catch(err => console.log(err.message))
```

為什麼可以換傳入參數值的名稱？要回答這個問題，要先來解說一下進入 Promise 建構函式的大概執行流程，當然下面都是簡化過的說明:

1. 先用一個內部的物件，我把它稱為雛形物件，然後狀態(state)設定為 pending(等待中)，值(value)為 undefined，理由(reason)為 undefined
2. 再來初始化這個物件的工作，用`init(promise, resolver)`函式，傳入建構式中的傳入參數，也就是`function(resolve, reject){...}`當作`resolver`(解決者，解決用函式)傳入參數。
3. 把真正實作的 Promise 中的兩個內部`_resolve`函式，與`_reject`函式，對映到`init(promise, resolver)`中執行。

下面是把步驟說明實作出來的範例，不過這個只是為了方便解說用的，並不是真正可用的 Promise 建構函式，參考自[es6-promise](https://github.com/stefanpenner/es6-promise):

```js
//內部用的雛形物件，實作上包含在建構式中用this
const pInternal = {
  state: 'pending',
  value: undefined,
  reason: undefined,
}

//這個就是稱為executor的傳入參數
function resolver(resolve, reject) {
  resolve(10)
  //reject(new Error('error occured !'))
}

//初始化內部雛形物件用的函式
function init(promise, resolver) {
  try {
    resolver(
      function resolvePromise(value) {
        _resolve(promise, value)
      },
      function rejectPromise(reason) {
        _reject(promise, reason)
      }
    )
  } catch (e) {
    _reject(promise, e)
  }

  return promise
}

//隱藏在內部的私有函式
function _resolve(promise, value) {
  console.log(value)
  promise.state = 'onFulfilled'
  promise.value = value
}

//隱藏在內部的私有函式
function _reject(promise, reason) {
  console.log(reason)
  promise.state = 'onRejected'
  promise.reason = reason
}

//最後生成回傳的promise物件
const promise = init(pInternal, resolver)
console.log(promise)
```

以上面的範例來說，`resolver`函式在`init`中被呼叫時，`resolver`的第 1 個傳入參數，它的執行程式碼內容會被`init`中的`resolver`呼叫時的第一個傳入參數`resolvePromise`所取代，然後再加上`init`函式傳入參數`promise`物件，最後呼叫執行`_resolve(promise, value)`這個內部私有方法。也就是說這只是一個函式傳入參數的代換過程。所以如果你改成這樣也是一樣的結果:

```js
//這個就是稱為executor的傳入參數
function resolver(rs, rj) {
  rs(10)
  //rj(new Error('error occured !'))
}

//初始化內部雛形物件用的函式
function init(promise, resolver) {
  //改用匿名函式
  resolver(
    function(value) {
      _resolve(promise, value)
    },
    function(reason) {
      _reject(promise, reason)
    }
  )

  return promise
}
```

或用箭頭函式來更簡化程式碼:

```js
//這個就是稱為executor的傳入參數
function resolver(rs, rj) {
  rs(10)
  //rj(new Error('error occured !'))
}

//初始化內部雛形物件用的函式
function init(promise, resolver) {
  //改用匿名函式與箭頭函式
  resolver(
    value => _resolve(promise, value),
    reason => _reject(promise, reason)
  )
  return promise
}
```

那麼 executor(執行者，執行函式)是必要的傳入參數值嗎？是的，如果你沒傳入任何的參數，會產生一個類型錯誤，錯誤訊息中有一個 resolver(解決者，解決函式)字詞，它應該算是 executor 的別名(或者是第一個函式型傳入參數的名稱?)，如果你傳入一個沒有任何內容的空白函式，雖然不會有錯誤發生，但會產生一個完全無用的 Promise 物件:

```js
const promise = new Promise()
//Uncaught TypeError: Promise resolver undefined is not a function

const promise = new Promise(function() {})
//不會有錯誤，但會產生一個完全無用的promise，無法改變狀態
//Promise {[[PromiseStatus]]: "pending", [[PromiseValue]]: undefined}
```

另外，Promise 物件中的兩個靜態方法，`Promise.resolve`與`Promise.reject`，它們的實作是雖然接近上面範例中的`_resolve`與`_reject`內部方法，但定義中都說它是與`then`回傳出來的 Promise 物件相同，請不要搞混了。下面的章節會專門來討論它們的用法。

你可能會覺得很奇怪，為何要一定要用這種方式來建構一個 promise 的物件實體？而且實際上這是一種相當高消費的語法(三個閉包)。有幾個原因:

1. 暴露的建構式樣式: Promise 只是個用來包裹現有函式或程式語句的物件，所以把建構式外露出來給開發者自行定義其中的程式碼，稱之為"暴露的建構式樣式(Revealing Constructor Pattern)"。

2. 封裝: Promise 物件不外露狀態，也無法從外部程式碼中直接修改其狀態，狀態由 executor 的執行結果決定。此外，Promise 物件一旦被固定住兩種其一的狀態，就無法再改變狀態，這也是這種樣式在致力保護的原則。

3. Throw safety(安全性): 確保在建構函式在執行過程時，如果有 throw 例外的情況也是安全的，能作異步的錯誤處理。在混用同步/異步程式之後，可能會產生的問題，稱之為 [Zalgo](http://blog.izs.me/post/59142742143/designing-apis-for-asynchrony)。

> 註: 有一些術語或形容詞的意思大概都是相同的，fulfilled(已實現)、resolve(解決)、successful(成功的)、completed(完成的)差不多意思，尤其有些外部函式庫，會 使用`fulfill`代替這裡的`resolve`。而 rejected(已拒絕)、fail(失敗)、error(錯誤)是相反的一個意思。

### then 與 catch

> then 方法是 Promise 的最核心方法，標準規則有極大一部份都是在定義 then 方法

在 Promise 的標準中，一直不斷的提到一個方法 - `then`，中文是"然後、接著、接下來"的意思，這個是一個 Promise 的重要方法。有定義 then 方法的物件被稱之為`thenable`物件，標準中花了一個章節在講`then`方法的規格，它的語法如下(出自[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then)):

```js
p.then(onFulfilled, onRejected)

p.then(
  function(value) {
    // fulfillment
  },
  function(reason) {
    // rejection
  }
)
```

`then`方法一樣用兩個函式當作傳入參數，`onFulfilled`是當 promise 物件的狀態轉變為 fulfilled(已實現) 呼叫的函式，有一個傳入參數值可用，就是 value(值)。而`onRejected`是當 promise 物件的狀態轉變為 rejected(已拒絕) 呼叫的函式，也有一個傳入參數值可以用，就是 reason(理由)。

為什麼說它是"一樣"？因為比對到 promise 物件的建構式，與那個傳入的函式參數值的樣子非常像，也是兩個函式當作傳入參數，只是名稱的定義上有點不同，但意義接近。

那麼`then`方法最後的回傳值是什麼？是另一個"**新的**"promise 物件。

> Promises/A+標準 2.2.7
>
> `then`必須回傳一個 promise。`promise2 = promise1.then(onFulfilled, onRejected);`

這樣設計的主要目的，是要能作連鎖(chained)的語法結構，也就是稱之為合成(composition)的一種運算方式，在 JavaScript 中如果回傳值相同的函式，可以使用連鎖的語法。像下面這樣的程式碼:

```js
const promise = new Promise(function(resolve, reject) {
  resolve(1)
})

promise
  .then(function(value) {
    console.log(value) // 1
    return value + 1
  })
  .then(function(value) {
    console.log(value) // 2
    return value + 2
  })
  .then(function(value) {
    console.log(value) // 4
  })
```

`then`方法中的 onFulfilled 函式，也就是第一個函式傳入參數，它是有值時使用的函式，經過連鎖的結構，如果要把值往下傳遞，可以用回傳值的方式，上面的例子可以看到用`return`語句來回傳值，這個值可以繼續的往下面的`then`方法傳送。

onRejected 函式，也就是`then`方法中第二個函式的傳入參數，也有用回傳值往下傳遞的特性，不過因為它是使用於錯誤的處理，除非你是有要用來修正錯誤之類的理由，不然說實在這樣作會有點混亂，因為不論是 onFulfilled 函式或 onRejected 函式的傳遞值，都只會記錄到新產生的那個 Promise 物件之中，對"值"來說並沒有區分是 onFulfilled 回傳的來的，還是 onRejected 回傳來的。當一直有回傳值時就可以一直傳遞回傳值，當出現錯誤時，就會因為獲取不到之前的值，會導致之前的值消失不見。

`then`方法中的兩個函式傳入參數，第 1 個 onFulfilled 函式是在 promise 物件有值的情況下才會執行，也就是進入到 fulfilled(已實現)狀態。第 2 個 onRejected 函式則是在 promise 物件發生錯誤、失敗，才會執行。這兩個函式都可以寫出來，但為了方便進行多個不同程式碼的連鎖，通常在只使用`then`方法時，都只寫第 1 個函式傳入參數，也就是只用 onFulfilled 函式這個傳入參數值。

而對於錯誤處理，通常交給另一個`catch`方法來作，`catch`只需要一個函式傳入參數，(出自[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch)):

```js
p.catch(onRejected)

p.catch(function(reason) {
  // rejection
})
```

`catch`方法相當於`then(undefined, onRejected)`，也就是`then`方法的第一個函式傳入參數沒有給定值的情況，它算是個`then`方法的語法糖。`catch`方法正如其名，它就是要取代同步`try...catch`語句用的異步例外處理方式。

> 註: 也因為`catch`方法與`try...catch`中的`catch`同名，這會造成 IE8 以下的瀏覽器產生名稱上的衝突與錯誤。有些函式庫會用`caught`這個名稱來取代它，或是乾脆用`then`方法就好。

對於值的傳遞情況，需要注意是在中途發生有 promise 物件有 rejected(已拒絕)的情況。在下面的範例中，第二個`then`方法假設在中途發生錯誤，這樣會導致下一個執行被強迫只能使用`catch`方法，`catch`方法中的回調(相當於`then`中的 onRejected 函式)是只能得到理由(錯誤)而得不到值的，所以導致最後一個`then`方法中的 onFulfilled 函式獲取不到之前的值。這一段內容如果你並不是很理解，你可以先看其他章節的內容，再回來這裡重新執行一次這個範例，程式碼如下:

```js
const p1 = new Promise((resolve, reject) => {
  resolve(4)
})

p1.then(val => {
  console.log(val) //4
  return val + 2
})
  .then(val => {
    console.log(val) //6
    throw new Error('error!')
  })
  .catch(err => {
    //catch無法抓到上個promise的回傳值
    console.log(err.message)
    //這裡如果有回傳值，下一個then可以抓得到
    //return 100
  })
  .then(val => console.log(val, 'done')) //val是undefined，回傳值消息
```

以上是有關於 then 與 catch 兩個重要方法的基本使用介紹。初次學習時，要仔細看清楚範例中的程式碼是怎麼用這兩個方法的，以及各相關的說明，這些細節和概念先務必掌握，對於未來深入的學習會非常有幫助。
