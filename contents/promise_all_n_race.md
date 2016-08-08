# 靜態方法 Promise.all與Promise.race

> Promise.all與Promise.race的參數值，通常使用陣列結構作為傳入參數，而陣列中要不是就一般的值，要不就是Promise物件

`Promise.all`是"**並行運算**"使用的靜態方法，它的語法如下(出自[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)):

```
Promise.all(iterable);
```

`iterable`代表可傳入陣列(Array)之類的物件，JavaScript內建的有實作`iterable`協定的有String、Array、TypedArray、Map與Set這幾個物件。一般使用上都只用到陣列而已。`Promise.all`方法會將陣列中的值並行運算執行，全部完成後才會接著下個`then`方法。在執行時有幾種情況:

- 陣列中的索引值與執行順序無關
- 陣列中的值如果不是Promise物件，會自動使用`Promise.resolve`方法來轉換
- 執行過程中只要有"**其中一個(any)**"陣列中的Promise物件執行發生錯誤例外，或是有Promise的reject情況，會立即回傳一個rejected狀態promise物件
- 實現完成後，接下來的then方法會獲取到的值是陣列值

```js
const p1 = Promise.resolve(3)
const p2 = 1337
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => resolve('foo'), 1000)
});

Promise.all([p1, p2, p3]).then((value) => {
    console.log(value)
}).catch((err) => {
    console.log(err.message)
})

//結果: [3, 1337, "foo"]
```

你可能會好奇，String(字串)類型是可以傳到`Promise.all`中的參數，傳入後會變成什麼樣子，當然實際上應該沒人這樣在用的。以下為一個簡單的範例:

```js
Promise.all('i am a string').then((value) => {
    console.log(value)
}).catch((err) => {
    console.log(err.message)
})

//結果: ["i", " ", "a", "m", " ", "a", " ", "s", "t", "r", "i", "n", "g"]
```

`Promise.race`它的真正名稱應該是對比`Promise.all`的"any"，`Promise.all`指的是"**所有的**"陣列傳入參數的Promise物件都要解決(resolve)完了才進行下一步，`Promise.race`則是"**任何一個**"陣列傳入參數的Promise物件有解決，就會到下一步去。用"race(競賽)"這個字詞是比喻就像在賽跑一樣，只要有一個參賽者到達終點就行了，當然它的回傳值也只會是那個優勝者而已。

`Promise.race`的規則與`Promise.all`相同，只不過實現的話，下一步的`then`方法只會獲取跑最快的(最快實現的)的那個值，一個簡單的範例如下:

```js
const p1 = Promise.resolve(3)
const p2 = 1337
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => resolve('foo'), 1000)
});

Promise.race([p1, p2, p3]).then((value) => {
    console.log(value)
}).catch((err) => {
    console.log(err.message)
})
```

上面這個`Promise.race`範例，你把p1與p2的位置對調，就會發現最後的結果正好會對調，也就是說正好是只使用`Promise.resolve`方法的轉換情況，是和陣列中的前後順序有關的。不過因為`Promise.race`只能選出一個優腃者，p1與p2應該算同時，所以也只能以陣列的順序為順序。

> 註: `Promise.race`應該要多一個規則，如果陣列中有同時實現的promise值，以陣列中的順序優先者為回傳值

下面的例子是有加上每個陣列中的Promise物件產生時間的不同，這當然就只會回傳最快實現的那個Promise物件，也就是p3。

```js
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => resolve('p1'), 2000, )
});
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => resolve('p2'), 1000, 'p2')
});
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => resolve('p3'), 500, 'p3')
});

Promise.race([p1, p2, p3]).then((value) => {
    console.log(value)
}).catch((err) => {
    console.log(err.message)
})
```

> 註: 在Promises/A+並沒有關於`Promise.all`或`Promise.race`的定義，它們是ES6 Promise標準的實作
