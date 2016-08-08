# 執行順序 Chaining(連鎖)或Branching(分支)

Chaining(連鎖)是我們在上面一直看到的用`then`與`catch`方法串接起來的結構，這個結構會依照同步流程的原則，一步一步往下執行。如果你已經看過上面的內容，相信你已經很熟悉了。像下面的例子這樣:

```js
const p1 = new Promise((resolve, reject) => {
  resolve(1)
});


p1.then((value) => {
    console.log(value)
    return value+1
}).then((value) => {
    console.log(value)
    return value+2
}).then((value) => {
    console.log(value)
    return value+3
})

```

如果你以為相當下面這樣的程式碼，表示你對Chaining(連鎖)語法的認知是有問題的，這種叫作Branching(分支)的結構。因為經過`then`方法之後，雖然會產生一個新的Promise物件，原有的Promise物件並不會消失或被更改內容。每一次使用`p1`仍然是一樣的`p1`，每一段Promise語法由同樣的`p1`值各自發展，對程式來說它們是同步的語句:

```js
const p1 = new Promise((resolve, reject) => {
  resolve(1)
});


p1.then((value) => {
    console.log(value)
    return value+1
})

p1.then((value) => {
    console.log(value)
    return value+2
})

p1.then((value) => {
    console.log(value)
    return value+3
})
```

Chaining(連鎖)語法真正的相等語法如下:

```js
const p1 = new Promise((resolve, reject) => {
  resolve(1)
});


const p2 = p1.then((value) => {
    console.log(value)
    return value+1
})

const p3 = p2.then((value) => {
    console.log(value)
    return value+2
})

p3.then((value) => {
    console.log(value)
    return value+3
})
```

在Promise中使用連鎖的架構，就是一般稱為sequential(序列)執行的結構，而Branching(分支)並不是parallel(並行)的執行結構，真正的parallel(並行)結構應該是使用`Promise.all`方法的語法。序列執行可以保証一個Promise接著另一個執行，也就是"同步中的異步"結構。Branching(分支)的執行順序就很亂了，要看程式碼執行的順序來決定。

> 註: sequential(序列)執行結構，也有另一個名稱是waterfall(瀑布)
