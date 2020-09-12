> 제목 : Why do I need Keys in React Lists?  
> 글쓴이 : Adhithi Ravichandran    
> 출처 : https://medium.com/@adhithiravi/why-do-i-need-keys-in-react-lists-dbb522188bbb  

> 요약

> 

---

# 리액트 리스트와 Keys

리스트는 앱 내에서 중요한 요소이다. 모든 어플리케이션은 어떠한 형태로든 리스트를 사용하기 마련이다. 당신은 캘린더 앱이나 인스타그램 처럼 사진목록을 소유한 앱, 쇼핑카트의 상품목록 등등 리스트로된 업무를 할 수 있다.

이런 유즈케이스는 셀 수 없이 많다. 어플리케이션 내의 리스트는 성능상으로 무거울 수 있다. 엄청난 양의 비디오나 사진리스트, 그리고 당신이 스크롤 할때 1천개 이상을 가져와야 하는 경우의 앱을 상상해보라. 이런 경우는 당신의 어플리케이션 퍼포먼스에 상당한 영향을 줄 수 있다.

성능이란 중요한 측면이기 때문에 리스트를 사용할 때는 최적의 성능을 위해 설계되었는지 확인해야 한다.

리액트에서, 당신이 리스트를 사용할 때 각 리스트 아이템이 고유의 키를 필요로 한다는 것을 알고있는가? 한번 리액트에서의 리스트와 키에 대해서 좀 더 알아보고, 어떻게 해야 정확한 방법으로 구현할 수 있는지 알아보자.

# 간단한 리스트 컴포넌트 렌더링

```javascript
function ListComponent(props) {
  const listItems = myList.map((item) =>
    <li>{item}</li>
  );
  return (
    <ul>{listItems}</ul>
  );
}
const myList = ["apple", "orange", "strawberry", "blueberry", "avocado"];
ReactDOM.render(
  <ListComponent myList={myList} />,
  document.getElementById('root')
);
```

위 코드는 ListComponent 가 props 를 받아 리스트 아이템들을 렌더하는 코드이다. ListComponent 내에서 우리가 호출한 render() 메소드 호출하여 props 를 통해 myList 를 전달한다. 이 코드는 아래의 아웃풋을 생성한다.

- apple
- orange
- strawberry
- avocado

하지만 이 코드를 실행시킬 때, 당신은 리액트가 워닝을 던진다는 것을 알 수 있다.

>
> “Warning: Each child in an array or iterator should have a unique ‘key’ prop.%s%s See https://fb.me/react-warning-keys for more information.%s”
> 

여기 알림은 `**고유 키**를 사용하라` 에 대한 경고문이다. 키는 당신의 리액트 앱 성능을 높이는데 필요하다. 이제 어떻게 그렇게 되는지 보자.

# 리스트에서 키를 어떻게 사용해야 하나?

키는 리액트가 아이템이 변경되었을때 각 개체를 식별하는데 도움을 준다(추가/삭제/재정렬). 키는 고유식별자를 배열 안의 모든 엘리먼트에 부여하기 위해 필요하다.

이것을 좀 더 잘 이해하기 위해서, 우리가 이전에 본 코드스니펫을 리팩토링 해보도록 하자.


```javascript
function ListComponent(props) {
  const listItems = myList.map((item) =>
    <li key={item.id}>
       {item.value}
     </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}
const myList = [{id: 'a', value: 'apple'},
                {id: 'b', value: 'orange'}, 
                {id: 'c', value: 'strawberry'},
                {id: 'd', value: 'blueberry'}, 
                {id: 'e', value: 'avocado'}];
ReactDOM.render(
  <ListComponent myList={myList} />,
  document.getElementById('root')
);
```

위 코드 스니펫에서 당신은 각 리스트 아이템에 키가 포함된 것을 알 수 있다. 내가 내 원래 리스트를 값, ID 페어로 업데이트 한 것을 확인해보자. 배열의 각 아이템은 그와 연관된 id 를 가진다. 그러므로, 이것은 그 id 가 각 아이템에 키로서 할당되었다는 것이다. 이는 리스트의 아이템에 고유 키를 부여하는 최고의 접근법이다.

이 메소드에서, 키는 각 아이템을 식별하는 고유문자열이 된다.

# 그냥 인덱스를 키로서 사용하면 안되나여? - 일부 예외에서만.

Robin Pokorny 가 작성한 이 [아티클](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318)을 확인해보면 인덱스를 키로 사용하는 것은 안티패턴이라 하고 있다. 당신은 왜 배열을 반복할때 인덱스를 키로 사용하면 안되는지 궁금할 수 있다.

하지만, 많은 개발자들이 자기 코드에서 이게 반드시 이상적인 것은 아니라고 했다. 리액트는 인덱스를 키로 사용하지 않기를 추천하며, 이것이 성능에 악영향을 미칠 수 있으며 나아가 몇가지 불안정한 컴포넌트 상태로 만들 수 있기 때문이라고 전한다.

---

```javascript
const todoItems = todos.map((todo, index) =>
  // Only do this if items have no stable IDs
  <li key={index}>
    {todo.text}
  </li>
);
```

위 예제에서 당신은 우리가 todos를 돌면서 각 아이템의 인덱스를 키로서 할당하는 것을 볼 수 있다. 이것은 몇가지 우리가 아래에서 볼 예제에서만 허용된다.

리스트의 재배열, 혹은 리스트에서 아이템들을 추가하거나 제거하는 동작은 인덱스를 키로 사용하는 경우 컴포넌트의 상태의 문제를 발생시킬 수 있다. 만약 키가 인덱스라면, 아이템을 재배열 하는 행위는 이를 바꾼다. 그러므로, 컴포넌트 상태는 혼동될 수 있으며, 다른 컴포넌트의 이전 키를 사용하게 될 수 있다.

그러므로, 이런 관행을 피하고, 키로서 할당되기 위해 만들어진 고유아이디로 확실히 해야한다.

아래의 상황들은 인덱스를 키로서 할당해도 문제가 없는 경우이다.

# 인덱스를 키로 사용해도 안전한 몇가지 예외가 있는가?

1. 리스트가 고정이며 변경될일이 없을 경우
2. 리스트가 절대 재정렬될리 없을 경우
3. 리스트가 필터되지 않을 경우 (리스트에서 아이템이 추가, 삭제 되는 경우)
4. 리스트에 아이템을 위한 id 가 없는 경우

이 모든 예외가 충족되는 경우, 인덱스를 키로 사용해도 된다.

**노트**: 인덱스를 키로 사용하는 것은 잠재적으로 컴포넌트의 예상되지 않은 행동을 초래할 수 있다.

# 키는 형제 엘리먼트 간에서만 고유성이 필요하다.

각 아이템의 키는 고유해야 한다는 생각을 가지는 것은 유용하다. 이 룰은 배열에 한해 적용된다. 다시말해, 배열에 포함된 각 아이템들은 고유키를 필요로 한다. 하지만 이 것이 글로벌하게 고유해야 하는 것은 아니다.

동일한 키는 연관이 없는 다른 컴포넌트와 리스트에서는 사용해도 된다.

# 키는 자동으로 컴포넌트에 props 으로서 전달되지 않는다

아래의 예제에서, 당신은 내가 명시적으로 키로 사용되는 item.id 를 또다른 prop(id) 에 전달한것을 볼 수 있다.  
이는 리액트가 자동으로 key 를 prop 처럼 자동으로 넘겨주지 않기 때문이다. 만약 당신이 키를 몇가지 연산을 위해 사용하고자 한다면, 우리가 예제에서 그랬던 것 처럼 또 다른 prop 으로서 값을 넘겨줄 필요가 있다.

```javascript
const content = items.map((item) =>
  <MyComponent
    key={item.id}
    id={item.id}
    title={item.title} />
);
```

예제의 MyComponent 는 props.id 와 props.title 은 읽을 수 있어도 props.key 는 읽을 수 없다.

# 결론

이제 이 포스트의 하이라이트를 정리해보자.

1. 리스트는 무거운 성능을 지녔고 사용에 주의해야 한다
2. 리스트의 모든 아이템이 고유키값을 가지도록 해야한다
3. 당신이 리스트가 정적리스트(추가/재정렬/삭제 없음) 임을 확신하지 않는한 인덱스를 키로 사용하는 것은 선호되지 않는다.
4. 절대 Math.random() 같이 안정적이지 않은 키를 사용하지 마라
5. 리액트는 만약 안정적이지 않은 키를 사용한 경우 성능저하나 예상하지 못한 행동을 보인다
