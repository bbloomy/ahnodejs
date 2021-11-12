# **CommonJS모듈과 ECMAScript모듈**
## CommonJS모듈
```javascript
const logger = require('./logger');
```
동적 성질로 인해 종속성 그래프가 탐색되기도 전에 모든 파일들을 실행시켜,    
**순환 종속성 문제**가 발생한다.

## ECMAScript모듈
```javascript
import * as loggerModule from './logger.js'
import {log} from './logger.js'
```
정적이다. 종속성 그래프가 완전해지기까지는 어떠한 코드도 실행되지 않는다.  
