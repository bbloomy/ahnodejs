# 안티패턴
## **[1] return vs return await**
호출자에 거부하는 프라미스를 반환하고 프라미스를 반환하는 함수의 로컬 try...catch 블록에서 에러가 잡히는 것을 예상하는 실수
```javascript
function delayError (milliseconds) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject(new Error(`Error after ${milliseconds}ms`))
    }, milliseconds)
  })
}
```
* ### 😊 **return await**
delayError가 반환한 프로미스의 결과값을 기다렸다가 return된다.
```javascript
async function errorCaught () {
  try {
    return await delayError(1000)
  } catch (err) {
    console.error('Error caught by the async function: ' +
      err.message)
  }
}

errorCaught()
  .catch(err => console.error('Error caught by the caller: ' +
    err.message))
```
* ### 😱 **return** 
delayError의 결과와 상관없이 return된다.
```javascript
async function errorNotCaught () {
  try {
    return delayError(1000)
  } catch (err) {
    console.error('Error caught by the async function: ' +
      err.message)
  }
}

errorNotCaught()
  .catch(err => console.error('Error caught by the caller: ' +
    err.message))
```



## 😱 **[2] 순차 실행을 위한 Array.forEach와 async/await 사용**
iteration 함수는 links의 각 요소에 대해 한번씩 호출된다. iteration 함수 내에서 spider()가 반환하는 promise에 await 표현식을 사용한다. 하지만, iteration함수에 의해 반환된 함수는 forEach()에 의해 무시된다. 모든 spider()함수는 **이벤트 루프의 동일한 라운드에서 호출**되므로 **동시에 시작**되며, 모든 **spider()작업이 완료되기를 기다리지 않고 forEach()호출 즉시 연속적**으로 실행한다.  

```javascript
arr.forEach(callback(currentvalue[, index[, array]])[, thisArg])
```
```javascript
links.forEach(async function iteration(link){
    await spider(link,nesting-1)
})
```
for(const content of contents)와 .forEach(callback())의 차이점 : forEach callback함수이다.


>### **'순차 실행 + async/await' 은 .forEach()대신에 for(const i of ilist)를 사용하자** 


## 😊 **[2+] 동시 실행을 위한 반복문과 async/await 사용**
### [권장] **Promise.all()과 .map() 사용**
```javascript
async function printFiles () {
  const files = await getFilePaths();

  await Promise.all(files.map(async (file) => {
    const contents = await fs.readFile(file, 'utf8')
    console.log(contents)
  }));
}
```
### **async / await과 for(const a of A) 사용** 
```javascript
async function spiderLinks(currentUrl, content, nesting){
     if(nesting===0){
         return
     }
     const links=getPageLinks(currentUrl,content)
     const promises= links.map(link=>spider(link,nesting-1))
     for (const promise of promises){
         await promise
     }
}
```
## 😱 **[3] 무한 재귀 프라미스 해결 체인**
```javascript
funtion leakingLoop(){
    return delay(1)
    .then(()=>{
        console.log(`Tick ${Date.now()}`)
        return leakingLoop()
    })
}
```

------------------
**참고자료**
* [forEach와 async/await을 사용하면 안되요](https://stackoverflow.com/questions/37576685/using-async-await-with-a-foreach-loop)