> 제목 : An interesting kind of javascript memory leak (자바스크립트 메모리 누수 중 흥미로운 한가지)  
> 글쓴이 : David Glasser  
> 출처 : https://blog.meteor.com/an-interesting-kind-of-javascript-memory-leak-8b47d2e7f156  

> 요약

> 자바스크립트 클로저는 상위 함수 스코프를 하위 함수 스코프가 참조할 수 있는 구조를 말한다.
> 그러나 특정 상황 (상위 함수 스코프에 사용된 클로저가 여럿 있으나, 전부 반환되진 않아 사용할 수 없는 클로저가 있는 상태) 이 되어도,
> 사용할 수 있는 / 또 없는 클로저 둘다 로컬 변수 내 큰 크기의 오브젝트를 참조하는 경우
> 이 상위 함수가 실행될 때마다 이전의 모든 클로저들이 참조되는 오브젝트를 가지고 있기 때문에 GC 가 일어나지 않는 이슈가 있음
> 그러므로 이와 같은 문제를 직면했을 경우, 함수 사용 후 클로저를 참조하는 변수들을 초기화 시켜 GC 를 유도해야 할 것.

---

최근, Avi 와 David 는 Meteor 의 실시간 HTML 템플릿 렌더링 시스템에서 놀라운 자바스크립트 메모리 누수를 탐지했다.  
해당이슈의 수정은 0.6.5 릴리즈에서 수정될 예정이다.(현재는 QA 최종단계에 있다)

나는 `자바스크립트 클로저 메모리 누수` 으로 검색해보았고, 관련있는 부분을 찾을 수 없었다.  
그래서 이건 자바스크립트 컨텍스트에서도 상대적으로 거의 알려지지 않은 이슈로 보인다. (해당 질문에서 찾은 거의 대부분은, IE 구버전에서 발생하는 가비지 콜렉션의 나쁜 알고리즘에 대해 말하고 있었다. 하지만 이 문제는 내가 최근 설치한 크롬에서도 발생하는 문제였다!)

나는 나중에 이 주제에 관해 서술한 V8 개발자가 작성한 [훌륭한 블로그 포스트](http://mrale.ph/blog/2012/09/23/grokking-v8-closures-for-fun.html)를 발견했다. 그러나 아직 대부분의 자바스크립트 사용자가 이런 이슈에 대해 조심해야 한다는 것을 모르는 것 같다.

`자바스크립트는 비공식적인 함수형 프로그래밍언어로, 이 함수들은 클로저이다-함수 객체는 자신을 감싼 상위 스코프에 정의된 변수에 접근할 수 있고, 이는 해당 스코프가 종료되더라도 가능하다-  
클로저에 의해 보존된 로컬 변수는 자신을 정의한 함수가 종료되는 순간 가비지콜렉터에 의해 정리되며, 그 스코프안의 모든 함수들도 GC 된다.`

자 이제, 아래의 코드를 한번 보자:
```javascript
var theThing = null;
var replaceThing=  function () {
    var originalThing = theThing;
    theThing = {
        longStr: new Array(100000).join('*')
    };
};
setInterval(replaceThing, 1000);
```

매 초마다, 우리는 `replaceThing` 함수를 실행할 것이다. 이는 `theThing` 을 엄청 긴 문자열을 새로 메모리할당 받은 새 오브젝트로 치환할것이며, `theThing` 의 원래 값을 로컬변수인 `originalThing` 에 저장할 것이다.

함수가 반환된 이후에는, `theThing` 의 긴 문자열을 포함하고 있던 원래 값은 참조되는 부분이 아무것도 없게 되어 GC 될 수 있을 것이다.  
그러므로 이 코드에서 사용하는 메모리는 거의 일정할 것이다.: 긴 문자열을 할당해서 들고있지만, 매번 이전의 긴 문자열은 지울 수 있기 떄문이다.

하지만 `replaceThing` 을 오래 지속시키는 클로저를 만들었다면 어떻게 될까?

```javascript
var theThing = null;
var replaceThing = function () {
  var originalThing = theThing;
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log(someMessage);
    }
  };
};
setInterval(replaceThing, 1000);
```

`someMethod` 의 바디는 이론적으로 `originalThing` 을 참조하고, 그러므로 `someMethod` 가 살아있는 한 `originalThing` 은 유지되어야 하지 않을까?

이는 각각 이전버전의 `theThing` 이전 버전의 포인터에 전부 연결되어있기 때문에, 무한히 메모리 사용량이 증가하는 문제로 진행될 수 있다.

다행히도, 모던 자바스크립트 구현체들(굳이 말하자면 크로과 노드에 포함된 최신 V8 엔진) 은 `originalThing` 이 실제적으로 `someMethod` 클로저 안에서 사용되지 않는 다는 것을 알아차릴 정도로 충분히 똑똑하다.

그러므로 `someMethod` 의 어휘적 환경(lexical environment) 에 들어가지 않고, `replaceThing` 이 끝나면 이전의 `theThing` 이 GC 문제 없이 진행될 것이다.

(하지만 잠깐, 당신에게 물어보겠다! 만약 누군가가 이전에 `console.log = eval` 을 실행시켰고, 겉보기에는 문제가 없는 라인인 `console.log(someMessage)` 가 실제로는 `originalThing` 을 참조하는 어떤 코드로 `eval` 되었다면?
글쎄, 자바스크립트 스탠다드는 당신보다 한발 앞서있다. 당신이 이런 비열한 방식으로 `eval` 을 사용하려고 한다면 (어떤 방식에서든 그냥 eval 을 호출하는 것 이외의 방법으로), 이는 '간접적 `eval`' 로 불리며, 이는 실제적으론 어휘적 환경에 접근할 수 없다!  
만약, 또 다른 방법으로, `someMethod` 가 해당 명칭을 `eval` 로 바로 호출하는 코드를 포함하고 있다면, 이건 확실히 `originalThing` 에 접근할 것이고, 자바스크립트 환경은 어휘적 환경에서 `originalThing` 의 유지하지 못한다. 이는 누수로 이어질 것이다.)

뭐, 좋다! 자바스크립트는 우리를 메모리 누수로부터 지켜줄 것이다. 그렇지 않은가? 흠, 위 두 예제를 합친 버전으로 하나만 더 시도해보자.

```javascript
var theThing = null;
var replaceThing = function () {
  var originalThing = theThing;
  var unused = function () {
    if (originalThing)
      console.log("hi");
  };
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log(someMessage);
    }
  };
};
setInterval(replaceThing, 1000);
```

당신의 개발자 도구의 타임라인 탭을 열어서, 메모리 뷰의 기록을 한번 보면:

![image](https://miro.medium.com/max/1400/1*cibopxRYk29S-HSNuNjtWA.png)

우리가 매 초당 1메가 이상을 쓰고있는 것으로 보인다! 심지어 쓰레기통 아이콘을 눌러서 강제로 수동 GC를 시도하는 것도 도움이 되지 않는다. 
이건 우리가 `longStr` 을 누수시키고 있는걸로 보인다.

하지만 이건 그냥 이전이랑 똑같은 상황이 아닌가? `originalThing` 은 단지 `replaceThing` 의 메인 바디안에서 참조되었고, 그 안엔 `unused` 도 있다. `unused` 는 (우린 얘를 실행시키지도 않았다!) `replaceThing` 이 끝나면 정리가 될것이다... 
`replaceThing` 에서 탈출한 유일한 함수는 두번째 클로저인 `someMethod` 뿐이다. 그리고 `someMethod` 는 `originalString` 을 참조하지조차 않았다!

따라서 어떤 코드도 `originalThing` 을 재참조할 방법이 없다 하더라도 이건 절대 GC 되지 않는다! 왜? 음, 
클로저가 선언되는 일반적인 방법은 각 함수객체가 어휘적 범위를 나타내는 사전 방식의 객체 링크를 가지는 것이다. 만약 `replaceThing` 안에 선언된 두 함수가 실제로 `originalThing` 을 사용한 경우, 
`originalThing` 이 반복해서 할당 되더라도, 둘 다 동일한 객체를 얻는 것이 중요하므로 두 함수가 동일한 어휘적 환경을 공유한다.

현재 크롬의 V8 엔진은 변수가 어휘적 환경에서 클로저가 사용되지 않는 경우 변수를 유지하기에 충분히 똑똑하다. 이것이 첫번째 예제에서 누수가 일어나지 않는 이유이다.

그러나 변수가 어떤 클로저에서라도 사용된다면, 이는 모든 스코프 내에 있는 클로저가 어휘적 환경을 공유하게 된다.  
그리고 이는 메모리 누수로 이어질 수 있다.

![image](https://miro.medium.com/max/880/1*jkpHoXjLULpVoTFngljaaw.png)

당신은 이 문제를 피할만한 어휘적 환경을 보다 영리하게 구현할 수 있다고 상상할 수 있다.  
각 클로저는 실제로 읽고 쓰는 변수만 포함하는 사전(dictionary)를 가질 수 있다;이 사전 안의 값은 다수의 클로저들이 가진 어휘적 환경을 공유하는 변경가능한 부분일 것이다.  
ECMAScript 5 표준을 가볍게 읽어보면, 이정도는 타당할 것이다.

어휘적 환경 에 대한 설명을 보면 이는 "ECMAScript 구현의 특정 요소와 일치할 필요가 없는 사양 메커니즘" 이라고 설명한다.  
하지만 이 표준은 "가비지" 라는 단어를 실제로 포함하진 않고, "메모리" 라는 단어만 한번 나온다.

일단 이 형태의 메모리 누수를 고치기 위해, ['the fix to the Meteor bug'](https://github.com/meteor/meteor/commit/49e9813) 에서 알 수 있듯,  
단순히 `replaceThing` 의 마지막에 `originalThing = null` 을 넣는 것으로 고칠 수 있다.

이 방법은, `someMethod` 의 어휘적 환경안에 `originalThing` 이라는 이름이 아직 남아있음에도 불구하고, 이전의 거대한 값을 링크하고 있지는 않을 것이다.

```javascript
var theThing = null;
var replaceThing = function () {
  var originalThing = theThing;
  // Define a closure that references originalThing but doesn't ever
  // actually get called. But because this closure exists,
  // originalThing will be in the lexical environment for all
  // closures defined in replaceThing, instead of being optimized
  // out of it. If you remove this function, there is no leak.
  var unused = function () {
    if (originalThing)
      console.log("hi");
  };
  theThing = {
    longStr: new Array(1000000).join('*'),
    // While originalThing is theoretically accessible by this
    // function, it obviously doesn't use it. But because
    // originalThing is part of the lexical environment, someMethod
    // will hold a reference to originalThing, and so even though we
    // are replacing theThing with something that has no effective
    // way to reference the old value of theThing, the old value
    // will never get cleaned up!
    someMethod: function () {}
  };
  // If you add `originalThing = null` here, there is no leak.
};
setInterval(replaceThing, 1000);
``` 

그러므로 요약하자면,

만약 당신이 몇 클로저에서 큰 오브젝트가 사용된다면, 또 하지만 그 어떤 클로저도 당신이 계속 사용할 필요는 없는 경우,
작업이 완료되면 로컬 변수가 더이상 클로저를 참조하지 않도록 하십시오.

불행하게도, 이런 버그들은 좀 미묘할 수도 있다; 자바스크립트 엔진들이 당신이 이런 것에 대해 생각하지 않게 하는 것이 더 좋을 것이다.
