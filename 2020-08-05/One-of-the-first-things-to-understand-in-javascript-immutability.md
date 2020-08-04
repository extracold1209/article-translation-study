> 제목 : One of the first things to understand in JavaScript — Immutability  (불변성 - 자바스크립트를 이해하는 데 중요한 것 중 하나)
> 글쓴이 : Daryll Wong  
> 출처 : https://medium.com/javascript-in-plain-english/one-of-the-first-things-to-understand-in-javascript-immutability-629fabdf4fee  

> 요약

> 자바스크립트에서의 변수, 객체의 복사/병합/사용 등 다루면서 일어날 수 있는 사소한 버그들에 대해 이유를 알아보고,
> 문제가 생길만한 곳을 미연에 방지하기 위해 기초를 잘 알아보자~ 하는 글

---

기본으로 돌아가보자. "자바스크립트에서, 변수 혹은 상수는 불변값일까?"

![image](https://miro.medium.com/max/875/1*4PrMNL-FF9Z5G5BXJliAYg.png)

대답은 "둘다 그렇지 않다" 이다. 당신이 만약 이 대답을 할때 조금이라도 망설였다면, 읽어봐라.  
모든 프로그래밍 언어는 각각의 취향과 특색이 있다. 그리고 자바스크립트에서, 이것은 가장 중요한 것 중 하나이다. 특히 우리가 파이썬, 자바등과 같은 
몇몇 언어들을 습득하는 동안 말이다.

당신은 자바스크립트 코드를 당장에 바꿔야할 필요는 없을것이다. 하지만 이를 좀 더 빨리 알았더라면, 
나중에 디버깅을 하기 어렵게 되는 더러운 상황에 빠지게 되는 것을 방지할 수 있었을 것이다.

나는 당신이 이와 같은 문제의 방지를 적용할 수 있는 몇가지 방법을 포함하였다.(얕은 & 깊은 복사 방법에 대한 다양한 방법들)

시작하기전에 빠르게 요약하자면:  
변수 (`let` 으로 초기화) - 재할당, 변경가능
상수 (`const` 로 초기화) - 재할당, 변경불가

---

우리가 자바스크립트에서 불변성을 설명하기전에, 빠르게 몇가지 기본을 훑어보자. 이 파트는 스킵해도 좋다.

자바스크립트에는 대체로 몇 가지 데이터 유형이 있다.

1. Primitive (기본형) - Boolean, Number, String
1. Non-primitive (참조형) 혹은 객체 - Object, Array, Function
1. Special - Null, Undefined

*간단한 팁으로, 당신은 `console.log(typeof unknownVar)` 로 당신이 다루는 변수의 데이터 타입을 확인할 수 있다.*

## 원시 데이터 타입은 기본적으로 불변값이다

원시 데이터타입 (boolean, number, strings 같은) 은 상수로서 선언되면 불변값이다. 왜냐면 이 데이터타입들은,
당신이 추가적으로 프로퍼티를 삽입하거나 어떤 프로퍼티를 변경할 수 없기 때문이다.

원시타입을 '변경/대체' 하기 위해서는, 그건 단순히 당신이 그것들을 재할당 해야한다는 것을 의미한다.
이건 당신이 변수로서 선언했을 경우에만 가능하다.

```javascript
let var1 = 'apple' // 'apple' 은 메모리 위치 A 에 저장된다.
var1 = 'orange' // 'orange' 는 메모리 위치 B 에 저장된다.

const var2 = 'apple'
var2 = 'orange' // ERROR: Re-assignment not allowed for constants
```

![image](https://miro.medium.com/max/625/1*xyaMxzBMpouTQbMr-O0pXg.png)

위 예제에서는, 만약 우리가 var1 문자열을 변경한다면, 자바스크립트는 단순히 다른 메모리 공간에 다른 문자열을 생성하고,
var1 은 이 새로 만들어진 메모리 위치를 가리키게 된다. 그리고 이것을 **재할당** 이라고 부른다. 이는 모든 원시 데이터 타입이 변수 혹은 상수로 
선언되었는가에 대한 여부에 관계없이 적용된다. 

그리고 모든 상수는 재할당이 불가능하다.

---

## 자바스크립트에서, 오브젝트는 참조형으로 넘겨진다

문제는 우리가 객체를 다룬것으로부터 시작된다...

### 객체는 불변하지 않다

객체는 일반적으로 원시 데이터 타입이 아닌 것을 가리킨다.(객체, 배열 그리고 함수)  
그리고 이들은 `const` 를 사용하여 상수로서 선언되었다고 하더라도 변경이 가능하다.

(이 글의 나머지 부분에 대해서는 여기서 문제가 가장 많이 발생하므로 객체 데이터 유형에 대한 예를 들겠다. 이 개념은 배열과 함수도 동일할 것이다.)

자 그럼 이것이 무엇을 의미할까?

```javascript
const profile1 = {'username':'peter'}
profile1.username = 'tom'
console.log(profile1) //{'username':'tom'}
```

![image](https://miro.medium.com/max/625/1*FluTwbCYFCQO6pW5enoLoQ.png)

이 예제에서, profile1 은 동일한 메모리 위치에 있는 개체를 가리키고 있으며, 우리가 한 일은 같은 메모리 위치에서 이 객체의 프로퍼티를 변경하기 위한 것이었다.

좋다. 충분히 간단해 보인다. 그런데 이게 왜 문제가 될까?

### 객체의 변경이 문제가 되는 경우..

```javascript
const sampleprofile = {'username':'name', 'pw': '123'}
const profile1 = sampleprofile
profile1.username = 'harry'
console.log(profile1) // {'username':'harry', 'pw': '123'}
console.log(sampleprofile) // {'username':'harry', 'pw': '123'}
```

당신이 잠재적으로, 순수하게 작성할법한 코드조각으로 보인다. 그렇지 않은가?  
근데, 문제는 벌써 생겨있다!

이건 자바스크립트에서 객체는 참조로서 전달되기 때문이다.

![image](https://miro.medium.com/max/625/1*K7JS9v4pbm1b0W4yaf-fZQ.png)

이 경우에서 '참조로서 전달' 이 의미하는 것은, 우리는 sampleprofile 값의 참조를 profile1 로 넘겼다는 것이다. 다른 말로 하자면,  
profile1 과 sampleprofile 의 상수는 동일한 메모리 위치에 있는 동일한 객체를 가리키고 있다는 것이다.

그런 이유로, 우리가 profile1 의 객체 프로퍼티를 바꾸고자 할때, 이는 sampleprofile 에도 영항을 미친다. 이 두 값은 동일한 객체를 가리키고 있기 때문이다.

```javascript
console.log(sampleprofile === profile1) // true
```

이것은 참조로서 전달되는 것이 (또한 그에따른 변조) 어떻게 잠재적으로 문제를 일으킬 수 있는가에 대한 간단한 예시였을 뿐이다.  
그러나 우린 우리의 코드가 더욱 복잡해지고 커졌을 때, 어떻게 이것이 정말 큰 문제를 야기시킬 수 있는지 상상할 수 있다.  
또한 만약 우리가 이 사실을 완벽히 잘 알지 못한다면, 몇 버그는 해결하기 어려울 것이다.

그러므로, 어떻게 방지하고 또한 이런 잠재적인 문제들을 직면하는 것을 회피하고자 할 수 있을까?

여기 객체 변조와 관련된 잠재적 문제에 효과적으로 대응하기 위해 우리가 알아야할 두 가지 개념이 있다.

- 객체 프리징으로서 변조 방지하기
- 얕은 & 깊은 복사 활용하기

나는 자바스크립트로, 바닐라 함수로서, 혹은 우리가 사용할 수 있는 유용한 라이브러리를 사용해 구현된 몇가지 예제를 당신에게 보여줄 것이다.

---

### 객체의 변조를 막기

#### 1. Object.freeze() 함수 사용

만약 당신이 객체의 프로퍼티 변경을 막고싶다면, 당신은 `Object.freeze()` 를 사용할 수 있다. 이 함수가 하는 일은 이미 존재하는 함수의 변경을 
불허하도록 만드는 것이다. 그 어떤 시도도 '조용한 실패' 를 일으킬 것이다. 이것은 성공되지 않지만 어떤 경고도 띄워주지 않는다는 것을 의미한다.

```javascript
const sampleprofile = {'username':'name', 'pw': '123'}

Object.freeze(sampleprofile)

sampleprofile.username = 'another name' // no effect

console.log(sampleprofile) // {'username':'name', 'pw': '123'}
```

그러나, 이것은 'shallow freeze' 이며, 이것은 깊게 중첩된 객체에는 동작하지 않을 것이다.

```javascript
const sampleprofile = {
  'username':'name', 
  'pw': '123', 
  'particulars':{'firstname':'name', 'lastname':'name'}
}

Object.freeze(sampleprofile)

sampleprofile.username = 'another name' // no effect
console.log(sampleprofile)

/*
{
  'username':'name', 
  'pw': '123', 
  'particulars':{'firstname':'name', 'lastname':'name'}
}
*/

sampleprofile.particulars.firstname = 'changedName' // changes
console.log(sampleprofile)

/*
{
  'username':'name', 
  'pw': '123', 
  'particulars':{'firstname':'changedName', 'lastname':'name'}
}
*/
```

위의 예제에서는, 중첩된 객체의 프로퍼티들은 아직도 변경 가능하다.

당신은 잠재적으론 중첩된 객체를 재귀적으로 열려나가는 간단한 함수를 만들수 있을것이다. (당신이 한번 시도해보고 이 글에 코멘트 달아줄 수 있는가? 😊), 
그러나 당신이 좀 귀찮다면 여기 몇가지 사용할 수 있는 라이브러리들이 있다:

#### 2. deep-freeze 사용하기

한번 진지하게, 만약 당신이 [deep-freeze](https://www.npmjs.com/package/deep-freeze) 의 [소스코드](https://github.com/substack/deep-freeze/blob/master/index.js)를 본다면, 
근본적으로 이건 단순한 재귀 함수일 뿐이다. 하지만 아무튼 당신이 쉽게 사용할순 있을 것이다..

```javascript
var deepFreeze = require('deep-freeze');

const sampleprofile = {
  'username':'name', 
  'pw': '123', 
  'particulars':{'firstname':'name', 'lastname':'name'}
}

deepFreeze(sampleprofile)
```

deep-freeze 의 또다른 대체제로서 당신이 선호할만한 것은 [ImmutableJS](https://immutable-js.github.io/immutable-js/) 이다. 
왜냐하면 이것은 당신이 이 라이브러리를 통해 만든 객체를 변경하고자 시도하면 에러를 내주기 때문이다.

---

### 참조전달에 관한 문제를 회피하기

중요한 포인트는 자바스크립트에서의 **얕은 / 깊은 복사, 복제, 병합**을 이해하는 것이다.

당신이 만든 프로그램에서의 객체들의 각 구현방식에 따라, 당신은 얕은 혹은 깊은 복사를 사용하고자 할 수 있다.  
이는 메모리와 성능에 관한 다른 고려사항들이 있을 수 있는데, 이것은 얕은 혹은 깊은 복사와 심지어 사용하고자 하는 라이브러리로도 영향을 미칠 것이다.  
하지만 이런 부분들은 추후에 다루도록 하겠다.

그럼 먼저 얕은 복사부터 시작해서, 그 다음은 깊은 복사로 넘어가겠다.

### 얕은 복사

#### 1. 전개 구문(...) 사용하기

전개구문은 ES6 에서 처음 소개되었으며 배열과 객체들을 합치는 깔끔한 방법이다.

```javascript
const firstSet = [1, 2, 3];
const secondSet= [4, 5, 6];
const firstSetCopy = [...firstset]
const resultSet = [...firstSet, ...secondSet];

console.log(firstSetCopy) // [1, 2, 3]
console.log(resultSet) // [1,2,3,4,5,6]
```

ES2018 은 또한 객체 리터철로 전개 기능을 확장시켰다. 그러므로 우린 객체에서도 이 구문을 그대로 쓸 수 있다. 
모든 객체들의 프로퍼티들은 병합될 수 있다. 하지만 충돌이 나는 프로퍼티들의 경우는, 뒤에오는 객체가 우선시된다.

```javascript
const profile1 = {'username':'name', 'pw': '123', 'age': 16}
const profile2 = {'username':'tom', 'pw': '1234'}
const profile1Copy = {...profile1}
const resultProfile = {...profile1, ...profile2}

console.log(profile1Copy) // {'username':'name', 'pw': '123', 'age': 16}
console.log(resultProfile) // {'username':'tom', 'pw': '1234', 'age': 16}
```

#### 2. Object.assign() 함수 사용하기

이것은 위에서 보여준 전개구문을 사용하는 것과 비슷하며, 이것은 배열과 객체 양쪽에서 둘다 사용할 수 있다.

```javascript
const profile1 = {'username':'name', 'pw': '123', 'age': 16}
const profile2 = {'username':'tom', 'pw': '1234'}
const profile1Copy = Object.assign({}, profile1)
const resultProfile = Object.assign({},...profile1, ...profile2)

console.log(profile1Copy) // {'username':'name', 'pw': '123', 'age': 16}
console.log(resultProfile) // {'username':'tom', 'pw': '1234', 'age': 16}
```

내가 첫번째 입력에 빈 오브젝트인 `{}` 를 사용한 것에 주목해달라. 이것은 이 함수가 첫번째 인자를 얕은 병합 결과에서부터 업데이트 하기 때문이다.

#### 3. .slice() 사용하기

이건 단순히 *배열의 얕은 복사*를 하기 위한 간편한 메소드이다!

```javascript
const firstSet = [1, 2, 3];
const firstSetCopy = firstSet.slice()

console.log(firstSetCopy) // [1, 2, 3]

//note that they are not the same objects
console.log(firstSet===firstSetCopy) // false
```

#### 4. lodash.clone() 사용하기

또한 얕은 복사도 할 수 있는 방법이 있다. 이것을 사용하는 것은 좀 지나치다고 생각하지만 (이미 lodash 가 포함되어 있지 않는 한), 하지만 예제는 남겨두겠다.

```javascript
const clone = require('lodash/clone')
const profile1 = {'username':'name', 'pw': '123', 'age': 16}

const profile1Copy = clone(profile1)
//...
```

#### 얕은 복사의 문제점:

얕은 복사의 모든 예제들에서, 문제점은 당신이 객체의 중첩이 깊어질수록 시작된다. 아래와 같은 예제에서 말이다.

```javascript
const sampleprofile = {
  'username':'name', 
  'pw': '123', 
  'particulars':{'firstname':'name', 'lastname':'name'}
}

const profile1 = {...sampleprofile}
profile1.username='tom'
profile1.particulars.firstname='Wong'

console.log(sampleprofile)
/*
{
  'username':'name', 
  'pw': '123', 
  'particulars':{'firstname':'Wong', 'lastname':'name'}
}
*/

console.log(profile1)
/*
{
  'username':'tom', 
  'pw': '123', 
  'particulars':{'firstname':'Wong', 'lastname':'name'}
}
*/

console.log(sampleprofile.particulars===profile1.particulars) //true
```

어째서 `profile1` 의 중첩된 프로퍼티 ('firstname') 의 변경기 `sampleprofile` 에도 영향을 미쳤는지 보자.

![image](https://miro.medium.com/max/700/1*7QbV9c0-yJ98rgeciFYgCg.png)

얕은 복사에서, 중첩된 객체의 참조가 카피된다. 그러므로 'particulars' 를 가진 객체들인 `sampleprofile` 과 `profile1` 둘 다 같은 메모리 위치에 있는
같은 객체를 참조하게 되는 것이다.

이런 문제가 발생하는 것을 막기 위해서, 그리고 만약 당신이 외부 참조 없이 100% 순수한 복사를 원한다면, 우린 **깊은 복사** 가 필요하다.

---

### 깊은 복사

#### 1. JSON.stringify() 와 JSON.parse() 사용하기

이것은 ES6 이전에는 불가능하였으나, 이후엔 JSON.stringify() 메소드가 중첩된 객체의 깊은 복사를 할 수 있게 되었다.  
그러나, 이 메소드는 숫자, 문자 그리고 불리언 데이터 타입에서만 잘 동작한다. 아래의 JSFiddle 예제를 보라.  
어떤 것이 카피되고 어떤 것이 아닌지 확인해보자.

[링크 대체](https://jsfiddle.net/daryllman/6rvj9u8b/)

일반적으로 당신이 원시 데이터 타입과 간단한 객체만을 사용해서 작업한다면, 이것은 짧고, 간단한 한줄짜리 코드가 될 것이다.

#### 2. lodash.deepclone() 사용하기

```javascript
const cloneDeep = require('lodash/clonedeep')
const sampleprofile = {
  'username':'name', 
  'pw': '123', 
  'particulars':{'firstname':'name', 'lastname':'name'}
}

const profile1 = cloneDeep(sampleprofile)
profile1.username='tom'
profile1.particulars.firstname='Wong'

console.log(sampleprofile)
/*
{
  'username':'name', 
  'pw': '123', 
  'particulars':{'firstname':'name', 'lastname':'name'}
}
*/

console.log(profile1)
/*
{
  'username':'tom', 
  'pw': '123', 
  'particulars':{'firstname':'Wong', 'lastname':'name'}
}
*/
```

참고로, lodash 는 create-react-app 에서 만들어주는 react app 에도 포함되어있다.

#### 3. 커스텀 재귀 함수

만약 당신이 단순히 깊은 복사를 하기위해 라이브러리를 다운받는 것을 원하지 않는다면, 간단한 재귀함수를 직접 만들어 보는것도 괜찮다!

아래의 코드 (모든 케이스를 만족하진 않는다) 는 어떻게 당신이 스스로 구현할 수 있는가에 대한 러프한 아이디어이다.

```javascript
function clone(obj) {
    if (obj === null || typeof (obj) !== 'object' || 'isActiveClone' in obj)
        return obj;

    if (obj instanceof Date)
        var temp = new obj.constructor(); //or new Date(obj);
    else
        var temp = obj.constructor();

    for (var key in obj) {
        if (Object.prototype.hasOwnProperty.call(obj, key)) {
            obj['isActiveClone'] = null;
            temp[key] = clone(obj[key]);
            delete obj['isActiveClone'];
        }
    }
    return temp;
}
// taken from https://stackoverflow.com/questions/122102/what-is-the-most-efficient-way-to-deep-clone-an-object-in-javascript
```

아마 깊은 복사를 구현하기 위해 라이브러리를 다운로드 하는 것이 더 간단하지 않을까?
깊은 복사를 해주는 소형 라이브러리인 rfdc, clone, deepmerge 와 같은 라이브러리들이 있다. 이것들은 lodash 보다 더 작은 패키지 사이즈를 가졌다.  
당신은 한가지 함수를 사용하기 위해 lodash 를 다운로드 받지 않아도 된다.

---

이 글로서 당신이 자바스크립트의 객체지향적 특성, 그리고 어떻게 객체를 조작하는것에 관련된 버그들을 핸들링 할 수 있는지에 대한 관점을 얻었으면 한다!  
이것들은 자바스크립트의 면접 질문이기도 하다. 읽어줘서 감사한다! :) 
