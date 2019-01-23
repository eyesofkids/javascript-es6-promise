---
id: ch11_snippets
title: 實例&程式碼片段
sidebar_label: 實例&程式碼片段
---

## Delay(延時)

```js
function delay(ms) {
  ms = Number(ms);
  ms = Number.isNaN(ms) ? +0 : Math.max(ms, +0);

  return new Promise(resolve => setTimeout(resolve, ms));
}
```

> 出自[Writing Promise-Using Specifications](https://www.w3.org/2001/tag/doc/promises-guide#example-delay)

如果要讓`delay`可以有回傳值，可以用以下的範例:

```js
function delay(ms) {
  return function(result) {
    return new Promise(function(resolve, reject) {
      setTimeout(function() {
        resolve(result);
      }, ms);
    });
  };
}

//使用範例
delay(1000)("hello").then(function(result) {
  console.log(result);
});
```

> 出自[Fun with promises in JavaScript](https://www.stephanboyer.com/post/107/fun-with-promises-in-javascript)

## 包裝 XMLHttpRequest(AJAX)

```js
function ajax(url, method, data) {
  return new Promise(function(resolve, reject) {
    var request = new XMLHttpRequest();

    request.responseType = "text";

    request.onreadystatechange = function() {
      if (request.readyState === XMLHttpRequest.DONE) {
        if (request.status === 200) {
          resolve(request.responseText);
        } else {
          reject(new Error(request.statusText));
        }
      }
    };

    request.onerror = function() {
      reject(new Error("Network Error"));
    };

    request.open(method, url, true);
    request.send(data);
  });
}

//使用範例
ajax("/", "GET").then(function(result) {
  console.log(result);
});
```

> 出自[Fun with promises in JavaScript](https://www.stephanboyer.com/post/107/fun-with-promises-in-javascript)

## 包裝 jQuery 的\$.ajax

```js
function ajax(options) {
  return new Promise(function(resolve, reject) {
    $.ajax(options)
      .done(resolve)
      .fail(reject);
  });
}

//使用範例
ajax({ url: "/" }).then(function(result) {
  console.log(result);
});
```

> 出自[Fun with promises in JavaScript](https://www.stephanboyer.com/post/107/fun-with-promises-in-javascript)

## forEach/for/while

在 forEach/for/while 等情況時，應該要使用`Promise.all`，因為它最後可以直接回傳一個陣列值，這提供了一些使用上的便利。

以下的的原本的使用範例，是希望能刪除所有的文件再到下一步:

```js
// 錯誤示範
// 我想要移除所有的文件資料
db.allDocs({ include_docs: true })
  .then(function(result) {
    result.rows.forEach(function(row) {
      db.remove(row.doc);
    });
  })
  .then(function() {
    // 到這裡所有的文件都已經被刪除
  });
```

### 正確用法

使用`Promise.all`，並以`map`取代`forEach`。

```js
db.allDocs({ include_docs: true })
  .then(function(result) {
    return Promise.all(
      result.rows.map(function(row) {
        return db.remove(row.doc);
      })
    );
  })
  .then(function(arrayOfResults) {
    // 到這裡所有的文件都已經被刪除
  });
```

> 出自[We have a problem with promises](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)

## 呼叫函式將回傳結果轉成 Promise 物件

```js
function promiseCall(f, ...args) {
  try {
    return Promise.resolve(f(...args));
  } catch (e) {
    return Promise.reject(e);
  }
}
```

說明如下:

- 當呼叫函式回傳一個值`v`時，會回傳一個以`v`實現的 promise 物件。
- 當呼叫函式會 throw 出例外`e`時，會回傳一個以`e`拒絕一個 promise 物件。

> 出自[Writing Promise-Using Specifications](https://www.w3.org/2001/tag/doc/promises-guide#shorthand-promise-calling)

## 改寫回調函式

基本上這是為了改用回調地獄用的方式，把原本的回調函式程式碼:

```js
//原本的回調函式
function(err, response) { ... }

//-------------------------------

//改用Promise的`then`方法:
.then(function(response) { ... }).catch(function(err) { ... })
```

另一個改寫的對照簡單範例:

```js
// 原本的回調結構
async1(function(){
    async2(function(){
        async3(function(){
            //....
        });
    });
});

//-------------------------------

// 改為Promise的結構
var task1 = async1();
var task2 = task1.then(async2);
var task3 = task2.then(async3);

task3.catch(function(){
    // 處理task1, task2, task3的例外
})

//-------------------------------

// 使用Promise的連鎖語法
async1(function(){..})
    .then(async2)
    .then(async3)
    .catch(function(){
        // 處理例外
    })
```

> 出自[Staying Sane With Asynchronous Programming: Promises and Generators](http://colintoh.com/blog/staying-sane-with-asynchronous-programming-promises-and-generators)
