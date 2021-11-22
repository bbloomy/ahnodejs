
# 생성자 패턴
- 객체의 생성과 관련된 디자인 패턴 


## Factory 패턴
- 새 인스턴스 생성을 감싸서 객체 생성 시 더 많은 유연성과 제어를 제공
- 객체의 생성을 분리할 수 있음 : new 연산자 또는 Object.create()를 사용하여 클래스로부터 새로운 객체를 만드는 대신 팩토리를 호출하는 것이 효율적
- 팩토리 내에서 new연산자 사용하여 객체를 생성하거나 클로저를 활용하여 상태를 기억하는 객체 리터럴 동작을 작성 또는 객체 반환 등을 하도록 가능

```javascript
  function createImage(name){
    return new Image(name)  
  }
  
  const image = createImage('photo.jpeg')
```

- 캡슐화 메커니즘으로도 사용될 수 있음 ( 외부 코드가 내부 핵심에 직접 접근을 막음) : 함수의 스코프와 클로저를 사용하는 것

```javascript
function createPerson(name){
  const privateProperties = []
  
  const person ={
    setName(name){
      if(!name){
        throw new Error('A person must have a name')
      }
      privateProperties.name = name
    },
    getName(){
      return privateProperties.name
    } 
  }

  person.setName(name)
  return person
 }
```


## Builder 패턴
- 유창한 인터페이스를 제공하여 복잡한 객체의 생성을 단순화하는 생성 디자인 패턴, 단계별로 객체를 생성할 수 있음
- 인자의 목록이 길거나, 많은 매개변수를 입력으로 사용하는 생성자가 있는 클래스에 사용하는 것이 유용함
- 복잡한 생성자를 더 읽기 쉽고 관리하기 쉬운 여러 단계로 나눔

```javascript
  class BoatBuilder{
    withMotors(count,brand,model){
      this.hasMoter = true,
      this.motorCount = count
      this.motorBrand = brand
      this.motorModel = model
      return this    
    }
    
    withSails(count, material, color){
      this.hasSails = true,
      this.sailsCount = count
      this.sailsMaterail = material
      this.sailsColor = color
      return this
    }
    
    hullColor(color)[
      this.hullColor = color
      return this    
    }
    
    withCabin(){
      this.hasCabin = true
      return this
    }
  
   build(){
    return new Boat({
      hasMotor : this.hasMoter,
      motorCount : this.motorCount, 
      motorBrand : this.motorBrand,
      motorModel : this.motorModel,'
      hasSails : this.hasSails,
      sailsCount : this.sailsCount,
      sailsMaterail : this.sailsMaterail,
      sailsColor : this.sailsColor,
      hullColor : this.hullColor,
      hasCabin : this.hasCabin    
    )}
   }
  }
  
  
  const myBoat = new BoatBuilder().withMotors(2, 'best motor', '123')
                                  .withSails(1, 'fabric', 'white')
                                  .withCabin()
                                  .hullColor('red')
                                  .build()
 ```
 
 ## 공개 생성자
 - 객체의 내부가 생성 단게에서만 조작되도록 허용하려는 경우 유용한 패턴
 - 생성시에만 수정할 수 있는 객체 생성, 생성시에만 사용자 정의 동작을 정의할 수 있는 객체 생성, 생성시 한 번만 초기화 할 수 있는 객체 생성에 사용
 
 ```javascript
  const MODIFIER_NAMES = ['swap','write','fill']
  
  export class ImmutableBuffer{
    constructor(size,executor){
      const buffer = Buffer.alloc(size) // 생성자의 인자에 지정된 크기의 버퍼를 할당
      const modifier ={}  // buffer를 변경할 수 있는 함수들을 보관하는 객체 리터럴 생성
      for (const prop in buffer){ // buffer 내부의 모든 속성들을 차례로 살펴보면서 함수가 아닌 속성은 건너뜀
        if(typeof buffer[prop] != 'function'){
          continue
        }
        if(MODIFER_NAMES.some(m=> prop.startsWith(m))){ // 속성이 함수이면서 이름이 MODIFIER_NAMES 배열에 있는 이름인지 살펴봄으로서 현재의 속성이 버퍼를 수정할 수있는지 판단
          modifier[prop] = buffer[prop].bind(buffer) // 버퍼를 수정할 수 있는 경우 buffer의 인스턴스에 바인드 한 후 modifiers 객체에 추가          
        }else{
          this[prop] = buffer[prop].bind(buffer) // modifier 함수가 아니면 현재 인스턴스에 직접 추가
        }
      }
      executor(modifier) // 생성자에서 입력으로 받은 실행함수를 호출하면서 인자로 modifier 객체를 전달하면 실행함수가 내부 buffer를 수정 가능
    }  
  } 
 ```
 
 ```javascript
  import {ImmutableBuffer} from './immutableBuffer.js'
  
  const hello ='hello'
  const immutable = new ImmutableBuffer(hello.length,
     ({write}) => {
        write(hello)     
     })
  
  console.log(String.fromCharCode(immutable.readInt8(0)))
 
  // "TypeError :immutable.write is not a funtion" 이라는 에러발생
  
  ```

