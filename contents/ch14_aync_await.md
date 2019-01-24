---
id: ch12_other_libs
title: Async 函式
sidebar_label: Async 函式
---

## 前言

處理 JavaScript 中的異步程序總是需要面臨很多不同情況的挑戰，ES6(ES2015)中所加入的 Promise，以及 Generator，讓新版本的 JavaScript 多了一些工具和語法，可以更方便的來作這些異步的程序的流程控制。但這些新的工具或結構，或許又讓開發者多了一些新的問題，例如:

- 語法複雜，需要學習與大幅度修改原有的程式碼
- 除錯不容易
- 錯誤處理不容易
- 對於情況控制(Conditionals)與迴圈/迭代處理仍然不是很理想

Async 函式在 ECMAScript 2017(ES8) 後加入了標準之中，其目的非常的明確，就是為了要解決上述的問題。

Async 函式的基礎是 Promise 與 Generator，雖然它提供了便捷的語法，能夠很輕易的處理這些異步程序的流程控制，但是我們仍需要對 Promise 有一定的理解，才能真正靈活地應用到各種情況中。

## Async 函式

先以一個最簡單的 Promise 的範例來說明，我們使用 Fetch API 向伺服器要求待辦事項的資料，然後更新目前應用中的列表，程式碼範例如下:

```js
fetch('http://example.com/items')
  .then(response => response.json())
  .then(data => {
    updateView(data)
  })
  .catch(error => {
    console.log('Update failed', error)
  })
```

Promise 的語法結構的涵意，像是 "我想要進行這個操作，然後(then)在下一步對操作得到的資料再進行處理" 這是一種連鎖語法的結構。

使用 async/await 來修改上面的範例，會變為下面這樣的程式碼，你可以注意到 then 已經不存在這個程式碼中:

```js
const response = await fetch('http://example.com/items')
const data = await response.json()
updateView(data)
```

await 的語法涵意，會轉變為 "我想要得到這個操作的結果(值)"，這會感覺像是在撰寫寫同步的語句。

由於 await 運算子是被設計來等待 Promise 的，它只能在 async 函式 內使用。所以我們需要把它放在一個 async 函式 之中，像下面這樣的程式碼:

```js
async function updateMyView() {
  const response = await fetch('http://example.com/items')
  const data = await response.json()
  updateView(data)
}
```

至於最一開始 Promise 裡的 catch 方法，可以用來作錯誤處理，在 async/await 語法裡，要使用 try/catch 來取代它，如以下的程式碼:

```js
async function updateMyView() {
  try {
    const response = await fetch('http://example.com/items')
    const data = await response.json()
    updateView(data)
  } catch (error) {
    console.log('Update failed', error)
  }
}
```

由上面的改寫範例，你可以看到我們雖然是在撰寫異步的程序，但寫出來的程式碼卻是類似於同步的程序，有加上 await 的語句，會等待到執行的結果得到後，才會接著處理下一步的程序語句。整體的程式碼可閱讀性提高了，而且也變得更容易除錯。

> 註: 目前 await 並不能在全域(頂級作用域)直接使用，必需要在 async 函式中才能使用

## Async 函式的語法說明

async/await 函式的目的在於簡化同步操作 promise 的表現，以及對多個 Promise 物件執行某些操作。async/await 好比將 generator 與 promise 組合起來。

## 瀏覽器相容議題

## 參考資源

- [14.6Async Function Definitions - ECMAScript 2017](https://www.ecma-international.org/ecma-262/8.0/#sec-async-function-definitions)
