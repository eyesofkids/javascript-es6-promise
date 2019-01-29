---
id: ch12_other_libs
title: microtask
sidebar_label: microtask
---

## 前言

microtask 與 macrotask 的標準出自於 [HTML Living Standard - Event loops](https://html.spec.whatwg.org/multipage/webappapis.html#task-queue)，這兩者指的是 microtask queue(隊列) 與 macrotask queue(隊列) 實際上這裡面並沒有 marcrotask queue 這個字詞，而是用 task queue，指的就是一般的工作隊列而已，用 marco-這開頭字詞，應該是為了要對比 micro-開頭字詞，作為一個區分。

那麼 microtask 與 Promise 有何種關係？

Event loop 與工作隊列與瀏覽器內部實作的執行底層有關，首先會影響到的，是程式碼執行的方式與順序，Promise 在現行的瀏覽器已經是原生支援的情況下，都是使用 microtask 來實作，這也代表原本使用工作隊列的程序，和 Promise 程序的執行方式與順序會有些不同。

當然，標準並沒有訂定到那麼細部，所以與執行環境實際上執行出來的結果會有些落差，這世界上不是有很多套不同的瀏覽器引擎，執行結果可能在基本的運作上是相同，但在交互應用的執行順序，會因內部實作的機制不同造成結果有些不同。

## Promises/A+中的註解

Promises/A+在註解中有說明它的平台程式碼實作，但並沒有說明一定要使用 microtask，只是說兩種都可以，只是稍微說明一下而已。在 [註解 3.1](https://promisesaplus.com/#point-67)中文翻譯如下:

這裡所說的 “平台程式碼(platform code)” 意指引擎、環境與 promise 實作的程式碼。在實務上，在事件迴圈(event loop)轉變到剛好 then 被呼叫，以及在一個全新的堆疊(stack)時，這會需要確保 onFulfilled 與 onRejected 能異步執行。這可以使用不論是“macro-task”機制，例如 setTimeout 或 setImmediate，或是“micro-task”機制例如 MutationObserver 或 process.nextTick 其中之一來實作。既然 promise 實作會被認為是平台程式碼，在處理程式被呼叫時，它可能自己要包含一個工作排程隊列(task-scheduling queue)或“trampoline(蹦牀)”。

> 註: trampoline 是一個專有名詞，它是一種在程式語言中用來最佳化 遞歸(recursion) 與防止堆疊溢出例外的技術，主要是針對沒有支援 尾呼叫最佳化(tail call optimization) 的程式語言使用的，例如 JavaScript ES5 等等。
