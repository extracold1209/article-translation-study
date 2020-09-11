> 제목 : Understanding Javascript Memory Management using Garbage Collection (GC 를 통해 JS 의 메모리 관리 이해하기)  
> 글쓴이 : Kewal Kothari  
> https://medium.com/front-end-weekly/understanding-javascript-memory-management-using-garbage-collection-35ed4954a67f  

> 요약

>
> 페이스북은 리액트 hooks 를 선호한다. 그 이유는 아래와 같다.
> 기존의 클래스 컴포넌트가 프로젝트가 커지면서 비대해지고,  
> 라이프사이클 단위로 쪼개지기 때문에 중복된 로직이 생길 수 있고,  
> render props 체인과 HOC 로 인한 패턴의 복잡화를 꼽았다.   

> hooks 는 로직의 관심사를 한 군데에 몰아둘 수 있고, 함수 단위이기 때문에 custom hooks 와 같은 결합이나 로직의 분리, 재활용이 쉽다.    
> 본문 내용중에서는 아래와 같이 hooks 의 장점을 설명한다.  
> 'hooks 는 class 로서 할수있는 모든 use-case 를 커버할 수 있으며 나아가 추출, 테스트, 코드재사용에 있어 더욱 유연함을 제공한다.  
> 이것이 hooks 가 리액트의 미래라고 말하는 우리들의 비전인 이유이다.'

---

![images](https://miro.medium.com/max/700/0*US4yPDoZAq8_44_H)

메모리 관리의 주요 목표는 요청 시 시스템에 동적으로 할당된 메모리를 제공하고 더 이상 사용하지 않는 개체를 포함하는 메모리를 나중에 해제하는 것이다.  
C, C++ 같은 언어에서는 `malloc()` 과 같은 메모리 할당을 위한 원시 함수가 있으며, 일부 고급 언어의 컴퓨터 아키텍처 (JS 같은) 는 이 작업을 수행하기 위해 GC 를 포함한다.  
이것은 메모리 할당을 추적하고 할당된 메모리가 더이상 사용되지 않는지 판단한다. 그리고나서는 자동으로 할당해제한다. 하지만 몇 알고리즘은 메모리가 필요한지 아닌지 확실히 결정하지 못한다.  
그러므로, 어떤 코드 조각에서 메모리가 필요할지 아닐지를 결정해야 하는 상황에 놓인 프로그래머에게 있어 이것은 굉장히 중요하다. 
그럼 GC 가 자바스크립트에서 어떻게 동작하는지 알아보자:

# 가비지 콜렉션

자바스크립트 엔진의 가비지 콜렉터는 기본적으로 메모리에서 지워진 닿지않는 오브젝트를 찾는다. 여기엔 두가지 가비지 콜렉션 알고리즘이 있으며 나는 하기 알고리즘을 설명하고자 한다.

- Reference-counting garbage collection
- Mark-and-sweep algorithm

# Reference-counting garbage collection

이것은 나이브한 가비지 콜렉션 알고리즘이다. 이 알고리즘은 참조가 남아있지 않은 오브젝트들을 찾는다. 만약 어떤 오브젝트에 어떤 참조도 남아있지 않다면 이 오브젝트는  
가비지 콜렉션의 대상이 되기 충분하다.

```javascript
var obj1 = {
    property1: {
         subproperty1: 20
     }
};
```

이 알고리즘을 위해 위의 예제에서 보여주는 객체를 만들어보자. 여기 `obj1` 은 `property1` 이 하나의 객체를 추가로 가지고 있는 객체를 가지고 있다.  
`obj1` 은 객체에 대한 참조를 가지고 있기 때문에, 어떤 객제도 가비지 콜렉션의 대상이 되지 않는다.

```javascript
var obj2 = obj1;
obj1 = "some random text"
```

이제, `obj2` 또한 `obj1` 이 가리키고있는 객체와 같은 객체에 대한 참조를 가지게 되었고, 하지만 그 이후 `obj1` 은 `"some random text"` 로 업데이트 되었고,  
`obj2` 가 그 객체에 대한 유일한 참조를 가지게 되도록 만들었다.

```javascript
var obj_property1 = obj2.property1;
```

이제 `obj_property1` 가 `obj2.property1` 을 가리키고 있다. 그러므로 `obj2.property1` 은 아래 표기에 따라 두개의 참조를 가지게 되었다.

1. `obj2` 의 프로퍼티로서
2. `obj_property1` 변수의 할당

```javascript
obj2 = "some random text"
```

`obj2` 는 `"some random text"` 로 업데이트 됨으로서 (객체에 대한)참조가 해제되었다. 그러므로 아마 이전에 가지고 있떤 객체는 이에 대한 참조가 없어졌고, GC 될 수 있다.  
하지만 이렇게 말하기엔 아마 틀렸을 것이다. 왜냐하면 `obj_property1` 이 `obj2.property1` 에 대한 참조를 가지고 있기 때문이다. 그러므로 객체는 아마 GC 되지 않을 것이다.

```javascript
obj_property1 = null;
``` 

이제 원래 `obj1` 에 있던 객체는 우리가 `obj_property1` 에서부터 참조를 없애는 것으로 어떠한 참조도 남아있지 않게 되었다. 그러므로 이제 이 객체는 GC 될 수 있다.

# 어디서 이 알고리즘이 실패한걸까?

```javascript
function example() {
     var obj1 = {
         property1 : {
              subproperty1: 20
         }
     };
     var obj2 = obj1.property1;
     obj2.property1 = obj1;
     return 'some random text'
}
example();
```

여기의 reference counting algorithm 은 함수 호출 이후, 두 오브젝트가 서로를 참조하면서 메모리에서부터 `obj1`, `obj2` 를 지우지 못한다.

---

# Mark-and-Sweep Algorithm

이 알고리즘은 자바스크립트의 global 객체 루트에서부터 닿지 않는 객체들을 탐지한다.  
이 알고리즘은 reference-counting algorithm 의 한계를 극복한다. 참조가 없는 객체는 닿지는 않지만, 그 반대가 될 수는 없다.(다시 찾아 연결되는 경우는 없다.)

```javascript
var obj1 = {
     property1: 35
}

```

![image](https://miro.medium.com/max/221/1*d-1V74jWR6gqkBxHhlom4A.png)

위에서 보여주는 바와 같이, 우리는 생성된 객체 `obj1` 가 어떻게 루트에서 부터 닿지 않게 되는지 볼 수 있다.

```javascript
obj1 = null
```

![image](https://miro.medium.com/max/520/1*Qc2ts7uiKU69rxLF5mYWcw.png)

우리가 `obj1` 을 `null` 로 설정할 때 객체는 더이상 루트에 닿지않고 이런 이유로 GC 될 수 있다.

이 알고리즘은 루트에서부터 시작되어 그 아래로 다른 모든 오브젝트들을 탐색하고 이들을 마킹한다.  
이는 계속 나아가 탐색된 물체를 타고 계속하여 마킹한다. 이 프로세스는 자식 노드가 없거나 어떤 경로도 마킹할 노드가 남아있지 않을때까지 계속한다.  
이제 가비지 콜렉터는 이 탐색을 통해 마킹된 연결가능한 객체들을 전부 무시한다. 그리곤 마킹되지않은, 루트로부터 확실하게 닿지않는 모든 객체들은 가비지 콜렉트 될 수 있고,  
이후 이 객체들을 삭제하는 것을 통해 메모리의 할당을 해제한다. 아래의 객체를 보고 한번 이해해보자.

![image](https://miro.medium.com/max/700/1*xndeuwtgCays2lrx2OKoMQ.png)

위에서 보여주는 바와 같이, 이것은 객체 구조가 어떻게 생겨먹었는지를 보여준다.  
우리는 루트로부터 닿지않는 객체가 어떤 모양새인지 알 수 있다. 이제 Mark-and-sweep 알고리즘이 위 예시에서는 어떻게 동작하는지 알아보자.

![image](https://miro.medium.com/max/700/1*TRr31SbiGWjPHnOwC1oB3w.png)

알고리즘은 루트에서부터 탐색한 객체들을 마킹하는것으로부터 시작한다. 위 이미지에서 초록 동그라미가 마킹된 객체라는 것을 알 수 있다. 이는 루트로부터 닿는 객체라는 것으로 입증되었다.

![image](https://miro.medium.com/max/700/1*oRCgCwBeCTfS457p43_hPg.png)

저 객체들은 루트로부터 닿지않아 마킹되지 않은 객체들이다. 그러므로, 이들은 GC 된다.

# 한계

오브젝트들은 명시적으로 닿지 않아야 한다.

> 2012 년 이후부터, 자바스크립트 엔진은 이 알고리즘을 reference-counting 가비지 콜렉션을 통해 이 알고리즘을 적용했다.

읽어주셔서 감사합니다.
