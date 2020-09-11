# **JavaScript에서 문자열을 결합하는 4 가지 방법**

JavaScript에서 문자열을 결합하는 4 가지 방법이 있습니다. 내가 가장 좋아하는 방법은 템플릿 문자열을 사용하는 것입니다. 
이유는 더 읽기 쉽고, 따옴표를 이스케이프 처리하기위한 백 슬래시가없고, 어색한 공백 구분 기호가 없으며, +와 같은 지저분한 연산자가 없습니다. 👏


```
const icon = '👋';

// Template Strings
`hi ${icon}`;

// join() Method
['hi', icon].join(' ');

// Concat() Method
''.concat('hi ', icon);

// + Operator
'hi ' + icon;

// RESULT
// hi 👋
```



# 1. 템플릿 문자열
Ruby와 같은 다른 언어에서 온 경우 **string interpolation** 이라는 용어에 익숙 합니다. 
바로 템플릿 문자열이 달성하려는 것입니다. 읽기 쉽고 간결한 표현식을 문자열 생성에 포함시키는 간단한 방법입니다.
```
const name = 'samantha'; 
const country = '🇨🇦';
```
## 문자열 연결에서 공백 누락 문제
템플릿 문자열 앞에는 이것이 내 문자열의 결과 일 것입니다.
```
"Hi. I'm "+ name + "and I'm from "+ country;
```
☝️ 뭐가 잘못된지 보이십니까?
space 공백이 없습니다. 그리고 그것은 문자열을 연결할 때 매우 일반적인 문제입니다.
```
// Hi, I'm samanthaand I'm from 🇨🇦
```

## 템플릿 문자열로 해결
템플릿 문자열을 사용하면이 문제가 해결됩니다. 
문자열을 어떻게 나타낼 것인지 정확하게 쓸 표현 가능하고, 공백이 없으면 쉽게 발견 할 수 있습니다.
***Super readable now, yay!*** 👏
```
// `Hi, I'm ${name} and I'm from ${country}`;
```


# 2. join ()
이 join메소드는 배열의 요소를 결합하고 문자열을 리턴합니다. 
배열로 작업하기 때문에 문자열을 추가하려는 경우 매우 편리합니다.

```
const instagram = '@samanthaming';
const twitter = '@samantha_ming';
const array = ['My handles are ', instagram, twitter];

const tiktok = '@samantaming';

array.push(tiktok);

array.join(' ');

// My handles are @samanthaming @samantha_ming @samanthaming
```
## 구분자 사용자 정의
가장 좋은 점은 join배열 요소의 결합 방식을 사용자 정의 할 수 있다는 것입니다. 
매개 변수에 구분 기호를 전달하면됩니다.

```
const array = ['My handles are '];
const handles = [instagram, twitter, tiktok].join(', '); 
// @samanthaming, @samantha_ming, @samanthaming

array.push(handles);

array.join('');

// My handles are @samanthaming, @samantha_ming, @samanthaming
```


# 3. concat ()
concat 을 사용하면 문자열에서 메서드를 호출하여 새 문자열을 만들 수 있습니다.
```
const instagram = '@samanthaming';
const twitter = '@samantha_ming';
const tiktok = '@samanthaming';

'My handles are '.concat(instagram, ', ', twitter,', ', tiktok);

// My handles are @samanthaming, @samantha_ming, @samanthaming
```
## 문자열과 배열 결합
`concat`으로 문자열을 배열과 결합하는데 사용할 수도 있습니다.
배열 인수를 전달하면 배열 항목을 쉼표 `,`로 구분 된 문자열로 자동 변환합니다.
```
const array = [instagram, twitter, tiktok];

'My handles are '.concat(array);

// My handles are @samanthaming,@samantha_ming,@samanthaming
```
더 나은 형식을 원하면 join구분자를 사용자 정의 하는데 사용할 수 있습니다 .
```
const array = [instagram, twitter, tiktok].join(', ');

'My handles are '.concat(array);
// My handles are @samanthaming, @samantha_ming, @samanthaming
```


# 4. 플러스 연산자 (+)
문자열을 결합 할 때 `+`연산자를 사용하면 흥미로운 점이 있습니다. 
기존 문자열을 추가하여 변경 할 수 있습니다.

## Non-Mutative
여기서는 새로운 문자열을 만드는데 `+`를 사용 하고 있습니다.
```
const instagram = '@samanthaming';
const twitter = '@samantha_ming';
const tiktok = '@samanthaming';

const newString = 'My handles are ' + instagram + twitter + tiktok;
```
## Mutative

`+=`을 사용하여 기존 문자열에 추가 할 수도 있습니다. 
따라서 어떤 이유로 Mutative 접근 방식이 필요한 경우 선택 사항 일 수 있습니다.
```
let string = 'My handles are ';

string += instagram + twitter;

// My handles are @samanthaming@samantha_ming
```
OH darn 😱 공백을 다시 잊어 버렸습니다. 
역시! 문자열을 연결할 때 공백을 놓치기가 쉽습니다.
```
string += instagram + ', ' + twitter + ', ' + tiktok;

// My handles are @samanthaming, @samantha_ming, @samanthaming
```
아직도 지저분하게 느껴집니다. join을 써보자!
```
string += [instagram, twitter, tiktok].join(', ');

// My handles are @samanthaming, @samantha_ming, @samanthaming
```
## 문자열에서 이스케이프 문자
문자열에 특수 문자가 있으면 결합 할 때 먼저이 문자를 이스케이프해야합니다. 
몇 가지 시나리오를 살펴보고 우리가 어떻게 벗어날 수 있는지 봅시다 💪
### 작은 따옴표혹은 아포스트로피 ( ' )
문자열을 만들 때 작은 따옴표나 큰 따옴표를 사용할 수 있습니다. 
이 지식을 알면 문자열에 작은 따옴표가 있으면 매우 간단한 해결책은 반대를 사용하여 문자열을 만드는 것입니다.
```
const happy = 🙂;

["I'm ", happy].join(' ');

''.concat("I'm ", happy);

"I'm " + happy;

// RESULT
// I'm 🙂
```
물론 백 슬래시 ( \)를 사용하여 문자를 이스케이프 할 수도 있습니다. 
그러나 읽기가 조금 어려워서 이런 식으로하지 않습니다.
```
const happy = 🙂;

['I\'m ', happy].join(' ');

''.concat('I\'m ', happy);

'I\'m ' + happy;

// RESULT
// I'm 🙂
```
*템플릿 문자열이 백틱을 사용하고 있기 때문에이 시나리오에는 적용되지 않습니다.*
### 큰 따옴표 ( “ )
작은 따옴표를 이스케이프 처리하는 것과 유사하게, 우리는 반대를 사용하는 동일한 기술을 사용할 수 있습니다. 
큰 따옴표를 이스케이프 처리하려면 작은 따옴표를 사용합니다.
```
const flag = '🇨🇦';

['Canada "', flag, '"'].join(' ');

''.concat('Canada "', flag, '"');

'Canada "' + flag + '"';

// RESULT
// Canada "🇨🇦"
```
똑같이, 백 슬래시 이스케이프 문자를 사용할 수도 있습니다.
```
const flag = '🇨🇦';

['Canada "', flag, '"'].join(' ');

''.concat('Canada "', flag, '"');

'Canada "' + flag + '"';

// RESULT
// Canada "🇨🇦"
```
템플릿 문자열이 백틱을 사용하고 있기 때문에 이번 시나리오에선 다루지지 않습니다.
### 백틱 ( ` )
템플릿 문자열은 백틱을 사용하여 문자열을 생성하므로 해당 문자를 출력하려면 백 슬래시를 사용하여 이스케이프해야합니다.
```
const flag = '🇨🇦';

`Use backtick \`\` to create a template string`;

// RESULT
// Use backtick `` to create a template string
```
다른 문자열 생성은 백틱을 사용하지 않기 때문에이 이번 시나리오에서 다루지 않습니다.
## 어떤 방법을 사용합니까?
문자열을 연결하는 다른 방법을 사용하는 몇 가지 예를 보여주었습니다. 
어느 쪽이 더 나은지 상황에 따라 다릅니다. 
스타일 선호도에 관해서는 Airbnb 스타일 가이드를 따르고 싶습니다.

>프로그래밍 방식으로 문자열을 만들 때는 연결 대신 템플릿 문자열을 사용하십시오. 
>eslint : [prefer-template template-curly-spacing](https://eslint.org/docs/rules/prefer-template.html)

따라서 승리를위한 템플릿 문자열! 👑

## 다른 방법이 여전히 중요한 이유는 무엇입니까?
다른 방법도 아는 것이 여전히 중요합니다. 
모든 코드베이스 가이드 규칙을 따르지는 않으며 레거시 코드베이스를 처리 할 수도 있습니다.
개발자는 우리가 처한 환경에 적응하고 이해할 수 있어야합니다.
기술이 얼마나 오래되었는지 불평하지 말고 문제를 해결해야합니다.

## 브라우저 지원
| browser           | Template String | join       | concat   | +        |
| ----------------- | --------------- | ---------- | -------- | -------- |
| Internet Explorer | X               | O (IE 5.5) | O (IE 4) | O (IE 3) |
| Edge              | O               | O          | O        | O        |
| Chrome            | O               | O          | O        | O        |
| Firefox           | O               | O          | O        | O        |
| Safari            | O               | O          | O        | O        |

<br/><br/>

> 제목 : 4 Ways to Combine Strings in JavaScript
> 글쓴이 : Samantha Ming
> 출처 : [https://medium.com/@samanthaming/4-ways-to-combine-strings-in-javascript-b1de5302fdaa](https://medium.com/@samanthaming/4-ways-to-combine-strings-in-javascript-b1de5302fdaa)