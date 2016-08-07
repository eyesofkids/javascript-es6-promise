# 執行流程與錯誤處理

> Promise中會吞掉(隱藏)throw例外的錯誤輸出，改用轉變狀態為rejected(已拒絕)來作錯誤處理

## throw與reject

在promise建構函式中，直接使用`throw`語句相當於用`reject`方法的作用，一個簡單的範例如下:

```js
// 用throw語句取代reject
const p1 = new Promise((resolve, reject) => {
    throw new Error('rejected!') // 用throw語句
    //相當於用以下的語句
    //reject(new Error('rejected!'))
})

p1.then((val) => {
        console.log(val)
        return val + 2
    }).then((val) => console.log(val))
    .catch((err) => console.log('error:', err.message))
    .then((val) => console.log('done'))

//最後結果:
//error: rejected!
//done
```

原先在錯誤處理章節中，對於`throw`語句的說明是，`throw`語句執行後會直接進到最近的`catch`方法的區塊中，執行區塊中程式碼後中斷之後程式碼。但這個理解是針對一般的同步程式結構，對異步的Promise結構並不適用。所以**並不會**中斷Promise結構的繼續執行，上面最後有個`then`中會輸出'done'字串，它依然會輸出。而且這裡的`throw`語句並不會瀏覽器的錯誤主控台出現錯誤訊息。

使用`throw`語句與用`reject`方法似乎是同樣的結果，都會導致promise物件的狀態變為rejected(已拒絕)，那麼這兩種方式有差異嗎？

有的，這兩種方式是有差異的。

首先，`throw`語句用於一般的程式碼中，它代表的意義是程式執行到某個時候發生錯誤，也就是`throw`語句會立即完成resolve(解決)，在then方法中按照規則，不論是onFulfilled函式或onRejected函式，只要丟出例外，就會導致新的promise物件的狀態直接變為Rejected(已拒絕)。

而`reject`則是一個一旦呼叫了就會讓Promise物件狀態變為Rejected(已拒絕)的方法，用起來像是一般的可呼叫的方法。

根據上面的解說，這兩種方式明顯是用在不同的場合的，實際上也無關優劣性。

其次，而在Promise中(建構函式或then)使用其他異步callback的API時會出現明顯的差異，這時候完全不能使用`throw`語句，以下的範例你可以試試:

```js
const p1 = new Promise(function(resolve, reject){
     setTimeout(function(){
         // 這裡如果用throw，是完全不會拒絕promise
         reject(new Error('error occur!'))
         //throw new Error('error occur!')
     }, 1000);
});

p1.then((val) => {
        console.log(val)
        return val + 2
    }).then((val) => console.log(val))
    .catch((err) => console.log('error:', err.message))
    .then((val) => console.log('done'))
```

## 執行流程&連鎖反應

如果你有看過其他的Promise教學，用`then`方法與`catch`來組成一個有錯誤處理的流程，例如像以下的範例程式，來自[JavaScript Promises](http://www.html5rocks.com/en/tutorials/es6/promises/):

```js
asyncThing1().then(function() {
  return asyncThing2();
}).then(function() {
  return asyncThing3();
}).catch(function(err) {
  return asyncRecovery1();
}).then(function() {
  return asyncThing4();
}, function(err) {
  return asyncRecovery2();
}).catch(function(err) {
  console.log("Don't worry about it");
}).then(function() {
  console.log("All done!");
});
```

在這篇文章中[JavaScript Promises](http://www.html5rocks.com/en/tutorials/es6/promises/)的也有一個這個範例的流程圖。

我建議不要單純用只有onFulfilled函式的`then`方法，以及`catch`方法來看整體流程，其實會有點混亂。從完整的`then`方法可以把流程看得更清楚，每個`then`方法(或`catch`方法)都會回傳一個完整的Promise物件，當新的Promise往下個`then`方法傳遞時，因原來其中程式碼執行的不同，狀態也不同。

> Promises/A+標準 2.2.7
>
> `then`必須回傳一個promise。
> `promise2 = promise1.then(onFulfilled, onRejected);`
>
> 2.2.7.1 當不論是onFulfilled或onRejected其中有一個是有回傳值`x`，執行Promise解析程序`[[Resolve]](promise2, x)`
>
> 2.2.7.2 當不論是onFulfilled或onRejected其一丟出例外`e`，promise2必須用`e`作為理由而拒絕(rejected)
>
> 2.2.7.3 當onFulfilled不是一個函式，而且promise1是fulfilled(已實現)時，promise2必須使用與promise1同樣的值被fulfilled(實現)
>
> 2.2.7.4 當onRejected不是一個函式，而且promise1是rejected(已拒絕)時，promise2必須使用與promise1同樣的理由被rejected(拒絕)

標準的2.2.7.4正好說明了，當你單純用只有onFulfilled函式的`then`方法時，如果發生在上面的Promise物件rejected(已拒絕)情況，為何會一直往下發生rejected(已拒絕)的連鎖反應，因為這個時候`then`方法並沒有onRejected函式，相當於undefined，也就是不是個函式，滿足了這條規則。所以會一直連鎖反應往下到有出現`catch`方法(或是有onRejected函式的then方法)，執行過後才會又恢復使用2.2.7.1的正常規則。

標準的2.2.7.3恰好相反，它是在一堆只有`catch`方法的連鎖結構出現，`catch`方法代表沒有onFulfilled函式的`then`方法，只要最上面的Promise物件fulfilled(已實現)時，會一直連鎖反應到某個`then`方法，才會又恢復使用2.2.7.1的正常規則。當然，這種情況很少見，因為在整個結構中，我們使用一連串的`catch`方法是不太可能的，錯誤處理頂多只會使用幾個而已。

> 註: catch方法通常會擺在整體流程的後面，這是因為擺前面會捕捉不到在它後面的then方法中的錯誤

我把上面的範例用箭頭函式簡化過，像下面這樣的範例:

```js
async1()
    .then(() => async2())
    .then(() => async3())
    .catch((err) => errorHandler1())
    .then(() => async4(), (err) => errorHandler2())
    .catch((err) => console.log('Don\'t worry about it'))
    .then(() => console.log('All done!'))
```

然後再把`then`方法中的兩個函式傳入參數都補齊，`catch`方法也改用`then`方法來改寫。這樣作只是要方便解說這個規則影響流程是怎麼跑的。實際使用你還是可以用上面的範例，當然前提是你已經很了解這些流程的規則。

```js
async1()
    .then(() => async2(), undefined)       
    .then(() => async3(), undefined)
    .then(undefined, (err) => errorHandler1())
    .then(() => async4(), (err) => errorHandler2())
    .then(undefined, (err) => console.log('Don\'t worry about it'))
    .then(() => console.log('All done!'), undefined)
```

情況: 當async1回傳的promise物件的狀態是rejected時。以下為每個步驟流程的說明:

1. 2.2.7.4規則，async2不會被執行，新的Promise物件直接是rejected(已拒絕)狀態
2. 2.2.7.4規則，async3不會被執行，新的Promise物件直接是rejected(已拒絕)狀態
3. 2.2.7.1規則，errorHandler1()被執行，新的Promise物件為fulfilled(已實現)狀態
4. 2.2.7.1規則，async4()被執行，新的Promise物件為fulfilled(已實現)狀態
5. 2.2.7.3規則，跳過輸出字串，新的Promise物件為fulfilled(已實現)狀態，回傳值繼續傳遞
6. 2.2.7.1規則，輸出字串

第3步是最不易理解的，實際上它需要根據errorHandler1()函式的回傳值來決定新的Promise是哪一種狀態。上面結果是假設當errorHandler1()只是簡單的回傳值或沒回傳值時。

根據連鎖反應，rejected狀態的Promise物件，會一直往下找到有`onRejected`函式的`then`方法或是`catch`方法，才會再進入2.2.7.1規則，也就是正常的Promise處理程序，那麼在正常的處理程序中，對於處理時發生的情況，又是怎麼決定回傳的新Promise物件？下面是大概的摘要。

2.2.7.1規則裡面的執行原則，主要是由回傳值`x`來決定的，這裡所謂的`x`是經過onFulfilled或onRejected其中一個被執行後的回傳值`x`，有幾種情況:

- `x`不是函式或物件，直接用`x`實現(fulfill)promise物件(**注意: undefined也算，所以沒回傳值算這個規則**)
- `x`是promise，看`x`最後是實現(fulfill)或拒絕狀態，就直接對應到promise物件，回傳值或理由也會一併跟著
- `x`是函式或物件，指定`then = x.then`然後執行`then`。其實它是在測試`x`是不是thenable物件，如果是thenable物件，會執行thenable物件中的then方法，一樣用類似then方法的兩選一來解析狀態，最後的狀態與值會影響到promise物件。一般情況就是如果不是thenable物件，也是直接用`x`作實現(fulfill)promise物件。

所以上面的第3步，因為errorHandler1()沒有回傳值或簡單回傳一個值，所以會讓新的Promise物件的狀態變為fulfilled(已實現)，然後繼續下一步。

實際上大部份只要有回傳值的情況，都會讓新的Promise物件的狀態變為fulfilled(已實現)。
