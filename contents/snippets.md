# 實例&程式碼片段

## Delay(延時)

https://www.w3.org/2001/tag/doc/promises-guide#example-delay

## 包裝XMLHttpRequest

## forEach/for/while

## Promisify

```js
function promiseCall(f, ...args) {
    try {
        return Promise.resolve(f(...args));
    } catch (e) {
        return Promise.reject(e);
    }
}
```

## 回調地獄(Callback hell)的改寫

你或許有聽過Promise是可以解決一種名稱為"回調地獄(Callback hell)"或是"金字塔(pyramid of doom)"的結構，

```js
function(err, response) { }
```

```js
.then(function(response) { }).catch(function(err) { })
```

https://thomashunter.name/blog/the-long-road-to-asyncawait-in-javascript/
http://stackoverflow.com/questions/32038961/avoiding-promise-anti-patterns-with-node-asynchronous-functions
https://medium.com/@rdsubhas/es6-from-callbacks-to-promises-to-generators-87f1c0cd8f2e#.9fqj27hm1
http://jamesknelson.com/grokking-es6-promises-the-four-functions-you-need-to-avoid-callback-hell/
