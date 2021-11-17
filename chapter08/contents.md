# **구조적 설계 패턴**
- 프록시, 데코레이터, 어댑터

## **[1] 프록시**
    다른 객체에 대한 엑세스를 제어할 수 있는 패턴
* ### **프록시가 하는 일**
    - 데이터 검증 : 프록시가 입력을 Subject에 전달하기 전에 **입력의 유효성을 검사**한다.

    - 보안 : 프록시는 클라이언트가 작업을 수행할 **권한이 있는 경우**만 요청을 Subject에게 전달한다.

    - 캐싱 : 데이터가 아직 캐시에 없는 경우에만 프록시 작업이 Subject에서 실행되도록 프록시는 **내부에 캐시를 유지**한다.

    - 느린 초기화 : Subject를 생성하는데 많은 비용이 드는 경우, 프록시는 **실제로 필요할 때까지 이를 지연**시킬 수 있다.

    - 기록 : 프록시는 메서드 호출과 관련 매개 변수를 가로채서 발생시 이를 기록한다.

    - 원격 객체 : 프록시는 원격 개체를 가져와서 로컬로 표시할 수 있다.

* ### **프록시 구현 기술 (2가지)**
#### **[1] 객체 컴포지션** 
: 기능을 확장해서 사용하기 위해 객체를 다른 객체와 결합하는 기술   
Subject와 동일한 인터페이스를 가진 객체가 생성    
- 예 : **StackCalculator를 변환한 SafeCalculator 만들기**  
divide()에 나누는 수가 0인지 판단하는 로직을 추가한다. 즉, 기능을 변경하려는 divide()는 가로채고, 그 외의 것들은 Subject에 위임한다.    
```javascript
class SafeCalculator {
  constructor (calculator) {
    this.calculator = calculator
  }

  // 프록시 된 함수
  divide () {
    // 추가한 휴효성 검사로직
    const divisor = this.calculator.peekValue()
    if (divisor === 0) {
      throw Error('Division by 0')
    }
    return this.calculator.divide()
  }

  // 위임된 메서드
  putValue (value) {
    return this.calculator.putValue(value)
  }
  getValue () {
    return this.calculator.getValue()
  }
  peekValue () {
    return this.calculator.peekValue()
  }
  clear () {
    return this.calculator.clear()
  }
  multiply () {
    return this.calculator.multiply()
  }
}
```
#### **[1]+ 객체리터럴과 팩토리 함수를 사용할 수도 있다.**
```javascript
function createSafeCalculator (calculator) {// 팩토리 함수를 만들었다.
  return {
    divide () { //프록시 함수
      const divisor = calculator.peekValue()
      if (divisor === 0) {
        throw Error('Division by 0')
      }
      return calculator.divide()
    },
    putValue (value) {return calculator.putValue(value)},
    getValue () {return calculator.getValue()},
    peekValue () {return calculator.peekValue()},
    clear () {return calculator.clear()},
    multiply () {return calculator.multiply()}
  }
}

const calculator = new StackCalculator()
const safeCalculator = createSafeCalculator(calculator) // 팩토리함수로 ..구현!

calculator.putValue(3)
calculator.putValue(2)
console.log(calculator.multiply()) // 3*2 = 6

safeCalculator.putValue(2)
console.log(safeCalculator.multiply()) // 6*2 = 12

calculator.putValue(0)
console.log(calculator.divide()) // 12/0 = Infinity

safeCalculator.clear()
safeCalculator.putValue(4)
safeCalculator.putValue(0)
console.log(safeCalculator.divide()) // 4/0 -> Error('Division by 0')
```
#### **[2] 객체 확장(Object augmentation)**
함수를 프록시 구현으로 대체하여 Subject를 직접 수정하는 작업을 하고 있다.
다른 위임된 함수를 구현할 필요가 없다.  
```javascript
function patchToSafeCalculator (calculator) {
  const divideOrig = calculator.divide
  calculator.divide = () => {
    const divisor = calculator.peekValue()
    if (divisor === 0) {
      throw Error('Division by 0')
    }
    return divideOrig.apply(calculator)
  }

  return calculator
}

const calculator = new StackCalculator()
const safeCalculator = patchToSafeCalculator(calculator)
```
## **[2] 데코레이터**
    기존 객체의 동작을 동적으로 증강시키는 패턴
동작이 해당 클래스의 모든 객체에 적용되지 않고 명시적으로 데코레이팅된 인스턴스에만 추가되기 때문에 클래스의 상속과는 다르다. 프록시 패턴과 유사하지만, 객체의 기존 인터페이스 동작을 개선하거나 수정하는 대신 새로운 기능으로 확장한다.   
* ### **데코레이터 구현 기술(4가지)**
#### **[1] 컴포지션**
새로운 메서드(add())를 추가하였다.
```javascript
class EnhancedCalculator {
  constructor (calculator) {
    this.calculator = calculator
  }

  // 새로운 메서드 추가
  add () {
    const addend2 = this.getValue()
    const addend1 = this.getValue()
    const result = addend1 + addend2
    this.putValue(result)
    return result
  }

  // modified method
  divide () {
    const divisor = this.calculator.peekValue()
    if (divisor === 0) {
      throw Error('Division by 0')
    }
    return this.calculator.divide()
  }

  // delegated methods 
  // ...
 
}

const calculator = new StackCalculator()
const enhancedCalculator = new EnhancedCalculator(calculator)

enhancedCalculator.putValue(4)
enhancedCalculator.putValue(3)
console.log(enhancedCalculator.add()) // 4+3 = 7
enhancedCalculator.putValue(2)
console.log(enhancedCalculator.multiply()) // 7*2 = 14
```
#### **[2] 객체 확장**
```javascript
function patchCalculator (calculator) { //원하는 객체에만 add함수를 추가할 수 있다.
  calculator.add = function () {   //객체에 함수 추가
    const addend2 = calculator.getValue()
    const addend1 = calculator.getValue()
    const result = addend1 + addend2
    calculator.putValue(result)
    return result
  }

  const divideOrig = calculator.divide
  calculator.divide = () => {
    const divisor = calculator.peekValue()
    if (divisor === 0) {
      throw Error('Division by 0')
    }
    return divideOrig.apply(calculator)
  }

  return calculator
}

const calculator = new StackCalculator()
const enhancedCalculator = patchCalculator(calculator)

enhancedCalculator.putValue(4)
enhancedCalculator.putValue(3)
console.log(enhancedCalculator.add()) // 4+3 = 7
enhancedCalculator.putValue(2)
console.log(enhancedCalculator.multiply()) // 7*2 = 14
// enhancedCalculator.putValue(0)
// console.log(enhancedCalculator.divide()) // 14/0 -> Error('Division by 0')
```

#### **[3] 프록시 객체 이용**
```javascript
const enhancedCalculatorHandler = {
  get (target, property) { //Proxy의 get 사용
    if (property === 'add') {
      // new method
      return function add () {
        const addend2 = target.getValue()
        const addend1 = target.getValue()
        const result = addend1 + addend2
        target.putValue(result)
        return result
      }
    } else if (property === 'divide') {
      return function () {
        const divisor = target.peekValue()
        if (divisor === 0) {
          throw Error('Division by 0')
        }
        return target.divide()
      }
    }
    return target[property]
  }
}

const calculator = new StackCalculator()
const enhancedCalculator = new Proxy(calculator, enhancedCalculatorHandler)
```

* ### **LevelUP, LevelDB**
특정 패턴의 객체가 DB에 저장될 때마다 알림을 받을 수 있도록 하는 LevelUp용 플러그인
```javascript
export function levelSubscribe (db) {
  db.subscribe = (pattern, listener) => { // ① subscribe()라는 새로운 함수로 db객체를 데코레이트한다. 제공된 DB 인스턴스에 함수를 직접 연결하기만 하면 된다. (객체 확장)
    db.on('put', (key, val) => { // ② DB에서 수행된 모든 입력 작업을 수신한다.
      const match = Object.keys(pattern).every(
        k => (pattern[k] === val[k]) // ③  DB가 입력될 때, 제공된 패턴의 모든 속성에 대해 존재여부를 검증하는데 간단한 패턴매치 알고리즘을 수행한다.
      )
      if (match) {
        listener(key, val) // ④ 일치하는 속성이 있으면 리스너에게 알린다.
      }
    })
  }

  return db
}
```
* LevelUp 플러그인 사용하기
```javascript
import { dirname, join } from 'path'
import { fileURLToPath } from 'url'
import level from 'level'
import { levelSubscribe } from './level-subscribe.js'

const __dirname = dirname(fileURLToPath(import.meta.url))

const dbPath = join(__dirname, 'db')
const db = level(dbPath, { valueEncoding: 'json' }) // ①
levelSubscribe(db) // ②  db객체를 데코레이트 하는 플러그인을 연결한다.

db.subscribe( // ③  tweet, en을 사용하는 모든 입력 객체를 구독할 것임을 지정한다.
  { doctype: 'tweet', language: 'en' },
  (k, val) => console.log(val)
)
db.put('1', { // ④ 
  doctype: 'tweet',
  text: 'Hi',
  language: 'en'
})
db.put('2', {
  doctype: 'company',
  name: 'ACME Co.'
})
```


## **[3] 어댑터**
    다른 인터페이스를 사용하여 객체의 기능을 엑세스할 수 있는 패턴
어댑터 패턴은 객체의 인터페이스를 가져와서 주어진 클라이언트가 예상하는 다른 인터페이스와 호환되도록 하는 데 사용된다. 어댑터는 다른 인터페이스를 사용하는 컨텍스트에서 사용할 수 있도록 객체의 인터페이스를 변환시킨다.  
* ### **어뎁터 구현 방식**
#### 컴포지션
```javascript
import { dirname, join } from 'path'
import { fileURLToPath } from 'url'
import level from 'level'
import { createFSAdapter } from './fs-adapter.js'

const __dirname = dirname(fileURLToPath(import.meta.url))
const db = level(join(__dirname, 'db'), {
  valueEncoding: 'binary'
})
const fs = createFSAdapter(db)   // 어뎁터 추가

fs.writeFile('file.txt', 'Hello!', () => {
  fs.readFile('file.txt', { encoding: 'utf8' }, (err, res) => {
    if (err) {
      return console.error(err)
    }
    console.log(res)
  })
})

// try to read a missing file
fs.readFile('missing.txt', { encoding: 'utf8' }, (err, res) => {
  console.error(err)
})
 ```
 ```javascript
import { resolve } from 'path'

export function createFSAdapter (db) {
  return ({
    // 파일 읽기
    readFile (filename, options, callback) {
      if (typeof options === 'function') {
        callback = options
        options = {}
      } else if (typeof options === 'string') {
        options = { encoding: options }
      }

      db.get(resolve(filename), { // ①
        valueEncoding: options.encoding
      },
      (err, value) => {
        if (err) {
          if (err.type === 'NotFoundError') { // ②
            err = new Error(`ENOENT, open "${filename}"`)
            err.code = 'ENOENT'
            err.errno = 34
            err.path = filename
          }
          return callback && callback(err)
        }
        callback && callback(null, value) // ③
      })
    },
    // 파일 쓰기
    writeFile (filename, contents, options, callback) {
      if (typeof options === 'function') {
        callback = options
        options = {}
      } else if (typeof options === 'string') {
        options = { encoding: options }
      }

      db.put(resolve(filename), contents, {
        valueEncoding: options.encoding
      }, callback)
    }
  })
}
```
