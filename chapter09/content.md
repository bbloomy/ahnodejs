
## 상태
- 컨텍스트의 상태에 따라 전략이 변경되는 전략 패턴
- 전략 ( 상태 ) 는 동적이며 컨텍스트의 생존 주기 동안 변경 가능 / 내부 상태에 따라 조정 가능
- 객체가 특정 상태에 따라 행위를 달리하는 상황에서 자신이 직접 상태를 체크하여 상태에 따라 행위를 호출하지 않고 상태를 객체화하여 행동할 수 있도록 위임하는 패턴
- 
### 상태 패턴 예제
- 노트북 -> 상태에 따라 버튼 행동 다름

```java
public class Laptop {
    public static String ON = "on";
    public static String OFF = "off";
    private String powerState = "";

    public Laptop(){
        setPowerState(Laptop.OFF);
    }

    public void setPowerState(String powerState){
        this.powerState = powerState;
    }

    public void powerPush(){
        if ("on".equals(this.powerState)) {
            System.out.println("전원 off");
        }
        else {
            System.out.println("전원 on");
        }
    }
}
```

```java
public class Client {
    public static void main(String args[]){
        Laptop laptop = new Laptop();
        laptop.powerPush();
        laptop.setPowerState(Laptop.ON);
        laptop.powerPush();
    }
}
```


## 템플릿
- 어떤 작업을 처리하는 일부분을 서브 클래스로 캡슐화해 전체 일을 수행하는 구조는 바꾸지 않으면서 특정 단계에서 수행하는 내역을 바꾸는 패턴
- 전체적으로는 동일하면서 부분적으로 다른 구문으로 구성된 메서드의 코드 중복을 최소화 할 수 있다.

```javascript
import {promises as fsPromises} from 'fs'
import objectPath from 'object-path'

export class ConfigTemplate{
  async load(file){
    console.log('Deserializing from ${file}')
    this.data = this._deserialize(
      await fsPromises.readFile(file,'utf-8'))
  }
  
  async save(file){
    console.log(`Serializing to ${file}`)
    await fsPromises.writeFile(file,this._serialize(this.data))
  }
  get(path){
    return objectPath.get(this.data,path)
  }
  set(path,value){
    return objectPath.set(this.dtaa,path,value)
  }
  _serialize(){
    throw new Error('_serialize() must be implemented')  
  }
  _deserialize(){
    throw new Error('_deserialize() must be implemented')
  }
    
 }
}

```

```javascript
import {ConfigTemplate} from './configTemplate.js'

export class JsonConfig extends ConfigTemplate{
  _deserialize(data){
    return JSON.parse(data)
  }
  
  _serialize(data){
    return JSON.stringify(data,null, ' ')
  }
}
```


## 반복자
- 접근 기능과 자료구조를 분리시켜 객체화하고 서로 다른 구조를 가지고 있는 저장 객체에 대해 접근하기 위해 interface를 통일시키고 싶을 때 사용하는 패턴
- 배열 또는 트리 데이터 구조와 같은 컨테이너의 요소들을 반복하기 위한 공통 인터페이스 또는 프로토콜을 정의


### 반복자 프로토콜
- 반복자 패턴은 상속과 같은 형식적 구조보다는 프로토콜을 통해 구현
- 반복자 패턴을 구현하는 시작점은 값들의 시퀀스를 생성하기 위한 인터페이스를 정의하는 '반복자 프로토콜' 
- 반복의 다음 요소를 객체에 담아 반환하며 이를 '반복자 결과' 라고 함

```javascript
  const A_CHAR_CODE = 65
  const Z_CHAR_CODE = 90
  
  function createAlphabetIterator(){
    let currCode = A_CHAR_CODE
    
    return{
      next (){
        const currChar = String.fromCodePoint(currCode)
        if(currCode > Z_CHAR_CODE){
          return {done:true}
        }
        currCode++
        return {value: currChar, done:false}      
      }
     }
   
  }

```
- 반복자의 현재 위치를 어떤 방식으로든 추적해야하기 때문에 많은 경우 반복자는 상태 저장 객체
- 반복가능자 프로토콜 : 객체가 반복자를 반환하는 표준화된 방법 정의 => iteratpor
- javasciprt에서는 @@iterator 함수 , Symbol.iterator 함수를 통해 접근 가능한 함수를 구현

## 명령 패턴
- 실행에 필요한 모든 정보들을 캡슐화하고 이렇게 만들어진 모든 객체를 명령이라고 본다.
- 함수나 기능을 직접적으로 호출하는 대신 이러한 호출을 수행하려는 의도를 나타내는 객체를 만든다.

 명령 : 함수 또는 함수를 호출하는데 필요한 정보를 캡슐화하는 객체
 클라이언트 : 명령을 생성하고 호출자에게 제공하는 컴포넌트
 호출자 : 대상에서 명령을 실행하는 컴포넌트
 대상 : 호출의 주체, 단일한 함수이거나 객체의 멤버일 수 있음
 
[ 예제 ] 
- 명령은 나중에 실행하도록 예약 가능
- 명령을 쉽게 직렬화하여 네트워크를 통해 전송할 수 있음
- 명령을 사용하면 시스템에서 실행된 모든 작업의 기록을 쉽게 유지
- 중복제거, 결합 및 분할 같은 일련의 명령에 대한 다양한 종류의 변환 수행, 텍스트 편집 협업과 같이 오늘날 대부분의 실시간 협업 소프트웨어 기반인 ot와 같은 더 복잡한 알고리즘 적용 가능
