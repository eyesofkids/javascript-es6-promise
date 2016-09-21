# 反樣式(anti-pattern)與最佳實踐

反樣式(anti-pattern)是指常發生的錯誤的用法。為避免很多濳在的問題，或是導正剛開始使用的開發者，網路上有很多整理好的反樣式提供參考，以下列出常見的幾個。

## 巢狀的(Nested) Promises

巢狀的(Nested) Promises無疑是讓錯誤處理變得更加困難，而且到底程式會怎麼執行，從程式碼中很難預期最後的結果會是什麼，而且巢狀的結構對於除錯或錯誤處理都會變得更加困難。

```js
firstThingAsync().then((result1) => {
  //巢狀Promises，不建議使用
  secondThingAsync().then((result2) => {
    // 可以存取得到result1與result2
  })
},(err) => {console.log(err.message)}    //這裡捕捉不到錯誤
).catch((err) => {console.log(err.message)}) //這裡捕捉不到錯誤
```

### 解決之道之一

如果你是要並行處理`firstThingAsync`函式與`secondThingAsync`函式，可以用`Promise.all`方法。

```js
Promise.all([firstThingAsync(), secondThingAsync()]).then(function(value) {
    console.log(value)
}).catch(function(err) {
    console.log(err.message)
})
```

### 解決之道之二

如果你希望`secondThingAsync`函式是可以獲得`firstThingAsync`函式先執行完的結果result1，可以先解決`firstThingAsync`函式，得到Promise物件與結果result1後，再用`Promise.all`方法來保証`secondThingAsync`函式與結果result1可以並行。這個結構有點複雜，而且這是因為`then`方法可以回傳一個Promise物件，所以可以這樣用。

```js
firstThingAsync().then((result1) => {
  return Promise.all([result1, secondThingAsync(result1)])
  })
  .then((result2) => { console.log(result2)})
  .catch((err) => {console.log(err.message)})
```

## 巢狀的(Nested) Promises之二

你可能開始用Promise結構之後，發現你的程式碼並沒有如預期的改善，反而因為回調函式的使用，變得更難以閱讀。

用了Promise之後的，程式碼結構如果像這樣，那實在很難一下看得懂是它的執行流程:

```js
firstThingAsync().then((result1) => {
    return secondThingAsync(result1, 'foo')
        .then((result2) => {
            return thirdThingAsync(result2, 'bar', 123)
                .then((result3) => {
                    console.log(result3)
                    return result3
                })
        })
})
```

使用Promise後，原本你希望能改善程式碼的結構是這樣:

```js
firstThingAsync().then(secondThing).then(thirdThing).then(outputThing)
```

其實只要再把原本的`secondThingAsync`與`thirdThingAsync`函式，以及輸出的函式再打包一下就行了，這種稱之為平坦化你的Promise連鎖結構。

```js
function secondThing(value){
  return secondThingAsync(value, 'foo')
}

function thirdThing(value){
  return thirdThingAsync(value, 'bar', 123)
}

function outputThing(value){
  console.log(value)
  return value
}
```

結論是儘可能保持Promise連鎖結構的簡單，你可以把原先的函式先進行包裝與加工處理。因為Promise連鎖結構是一個真正執行函式的結構，它需要更好的閱讀性與容易被除錯。


## then方法中的傳入回調函式沒有return值

在JavaScript中的函式區塊中，如果你最後沒寫上`return 值`的語句，它會照樣`return undefined`，這是一個函式預設的機制。`then`方法中的函式傳入參數回傳值，攸關這個準備要回傳的新Promise物件的狀態值，影響程度很高。像下面這樣的範例是不建議使用的反樣式，而且這是很常發生的錯誤:

```js
somePromise().then(() => {
  someOtherPromise()
}).then( () => {
  // 你覺得someOtherPromise會回傳給你嗎？ 我想不會
})
```

### 解決之道

`then`方法中的函式傳入參數，總是要有回傳值，要不然，就用`throw`拋出錯誤也行，最後有`catch`方法可以接住錯誤。

```js
somePromise().then(() => {
  return someOtherPromise()
}).then(() => {

}).catch((err) => {
    console.log(err.message)
})
```

## 理由(reason)不是一個Error物件

當然這是一直強調的，用於錯誤處理的理由(reason)最好是使用Error物件，這可以讓程式碼的錯誤處理方式統一，使用字串值或其他資料類型雖然是合法的，但並不建議這樣作。

```js
function firstThingAsync(){
  return Promise.reject(new Error('error!'))
}

firstThingAsync().then(function() {
    return secondThingAsync()
}).then(function() {
    return thirdThingAsync()
}).catch((err) => {
    console.log(err.message)
})
```

## 使用then方法中的第二傳入參數(onRejected函式)

你不可能在程式碼中一直要處理錯誤，只是針對某些可預期的、有可能會發生的錯誤或例外進行處理。

當然`then`方法的第二個傳入參數，也就是onRejected函式，它是在發生rejected(已拒絕)狀態使用的，不過我會建議在`then`使用第一個傳入參數就好，也就是onFulfilled函式，而另外使用`catch`方法。

`catch`方法雖然就是`then`方法只使用onRejected函式的語法糖，但`catch`方法的名稱上看起來就是在捕捉錯誤用的，何不只使用它來專門處理錯誤就好。這樣可以提供更好的程式碼閱讀性。

## 忘了加catch方法

如果你把所有的`then`方法都只用於fulfilled(已實現)情況，而用`catch`方法用於rejected(已拒絕)情況，這是個好主意，它可以讓你的程式碼更清楚易讀。

但至少每個連鎖的結構中，都至少要有個`catch`方法，而且它的位置是最後一個，或倒數第二個(如果最後一個是用於通知流程執行完成)，因為`catch`方法只能捕捉到在前面步驟的錯誤，這個規則你必須要記在心中。

```js
firstThingAsync().then(function() {
    return secondThingAsync()
}).then(function() {
    return thirdThingAsync()
}).catch((err) => {
    console.log(err.message)
})
```

## 回調函式沒有名稱

當然這個樣式並非絕對的反樣式，在簡單的程式碼中，使用匿名函式作為回調函式並沒有太大問題，回調函式大部份在使用時都是以匿名函式的語法。

但為何要給回調函式一個名稱？理由可能有幾個。首先，當錯誤發生時，你會很容易就知道是哪一個函式出了問題，在錯誤的堆疊上中會顯示出函式名稱。其次，你把回調函式拆出來另外再撰寫，程式碼的可閱讀性會更高，全部擠在`then`或`catch`方法的傳入參數中，如果回調函式的程式碼內容很多時，似乎是有點太擠了。

此外，使用箭頭函式在回調函式中也是一個不錯的語法，你可以學習著如何使用它，它可以讓你少打很多`function`這字詞，還有一些另外的好處，你已經在上面的內容中看到很多次，這可以讓程式碼看起來更清爽好閱讀。

```js
function firstThingAsync(){
  return Promise.resolve(1)
}

function secondThingAsync(){
  return Promise.reject(new Error('error!'))
}

function thirdThingAsync(){
  return Promise.resolve(1)
}

firstThingAsync()
.then(secondThingAsync)
.then(thirdThingAsync)
.catch((err) => {
  console.log(err.stack) //stack為非標準屬性，IE9或舊版瀏覽器不能使用
})
```
