# state 구조 정하기

State 구조를 잘 짜는 것은 수정하기도 버그를 잡기에도 좋은 컴포넌트와 끊임없이 버그가 나오는 컴포넌트 간에 차이를 만듭니다. State 구조를 짤때 알아야할 팁들을 공부해봅시다.

> 무엇을 배우나요
- 단일 state를 사용할 때 vs 여러 개의 state를 사용할 때
- state를 구성할 때 피해야하는 것
- state구조에서 나오는 흔한 이슈들 해결하기

## state 구조를 짜기 위한 원칙들
몇 개의 state를 가지고 있는 컴포넌트를 작성할 때, 얼마나 많은 state 변수를 사용할 것인지 또는 데이터 구조가 어떻게 될 것인지를 정해야 합니다. 최적의 state 구조가 아니어도 프로그램을 작성할 수 있지만, 더 나은 선택을 위한 몇가지 원칙들이 있습니다:

1. **연관된 state를 묶으세요.** 만약 동시에 2개 혹은 그 이상의 state를 업데이트 한다면 하나의 state 변수에 합치는걸 고려해보세요.
2. **state안에서 모순을 피하세요.** state가 어떤 방식으로 만들어졌다면, 몇몇 state들은 모순될 수도 서로 간에 “동의하지 않는” 방식으로 구조화될 수 있습니다. 이때 실수할 여지가 생깁니다.
3. **불필요한 state를 피하세요.** 렌더링하는 동안 컴포넌트의 props나 기존 state 변수에서 어떤 정보를 계산할 수 있는 경우, 해당 정보를 해당 컴포넌트의 state에 입력해서는 안 됩니다.
4. **state 내에서 중복을 피하세요.** 여러개의 state변수들 간에나 중첩 객체 안에서 같은 데이터가 중복될 때, 그것들을 동기화하기 어렵습니다. 가능하면 중복을 줄이세요. 
5.  **깊게 중첩된 state를 피하세요.** 계층 구조가 깊은 state는 업데이트하기 불편합니다. 가능하면 평평한 방식으로 state를 구성하는 것이 좋습니다.
이러한 원칙들의 목적은 실수만들지 않고 state 업데이트를 쉽게하는 것입니다. state에 불필요하고 중복된 데이터를 지우는 것이 state의 모든 부분을 동기적으로 유지시키는데 도움을 줍니다. 이것은 버그의 가능성을 줄이기 위해 데이터베이스 엔지니어가 데이터베이스 구조를 “정규화”하는 것과 비슷합니다. 앨버트 아인슈타인의 말을 빌리자면 **“당신의 상태(state)를 최대한 단순하게 만드세요. 더 단순하게가 아니라.”**(의역: 목표는 가장 단순하게 되어야지 이전보다 더 단순하게로 만족하면 안된다는 의미)

이제 이러한 원칙들이 실제로 어떻게 적용되는지 보죠.

## 연관된 데이터끼리 묶기
State 변수를 하나로 쓸지 여러개로 쓸지 헷갈릴 때가 있습니다.
이렇게 해볼까요?

```javascript
const [x, setX] = useState(0);
const [y, setY] = useState(0);
```

아니면 이렇게 해볼까요?

```javascript
const [position, setPosition] = useState({ x: 0, y: 0 });
```

정확히는 둘다 써도 무방합니다. **하지만 두개의 state변수가 항상 동시에 바뀌어야 한다면, 하나의 state에 묶는 것이 좋을 것 입니다.** 그러면 그 state들을 동기화시키는 것을 까먹지 않을 것입니다. 이 예제에서처럼 빨간 점의 두 좌표가 커서가 움직이는대로 업데이트됩니다:

<https://codesandbox.io/s/k3wul4?file=%2FApp.js&utm_medium=sandpack>

객체나 배열 안에서 데이터를 그룹지을 수 있는 또 다른 케이스는 우리에게 필요한 state 조각들이 얼마나되지 모를 때 입니다. 예를들어 사용자가 커스텀한 필드들을 추가하는 폼을 가지고 있을 때 유용합니다.

#### ❗️함정
>만약 state 변수가 객체 형태라면, 다른 필드들을 정확히 복사하지 않고는 폼안에 오직 하나의 필드만을 바꿀 수는 없음을 기억하세요. 예를 들어, 위의 예제에서  `setPosition({ x: 100 })`은 y 프로퍼티가 없기 때문에 사용할 수 없습니다.  `x`만을 설정하고 싶다면 `setPosition({ ...position, x: 100 })`처럼 쓰거나, 그것들을 2개의 state 변수들로 나누어 `setX(100)`를 써야합니다.

 
## state안에서 모순을 피하기
여기 `isSending`과 `isSent` state 변수가 있는 호텔의 피드백을 다루는 폼이 있습니다.

<https://codesandbox.io/s/7qx53i?file=%2FApp.js&utm_medium=sandpack>

이 코드가 작동되는 동안, 실행이 “불가능한” state에 대해 가능성을 열어놓습니다.  예를 들어 `setIsSent`와 `setIsSending`을 같이 호출해야하는 걸 잊어버리면,  `isSending`과 `isSent`가 동시에 `true`인  상황에 처할 수 있습니다. 컴포넌트가 복잡해질수록 무슨 일이 일어난건지 파악하기가 힘들죠.

**`isSending`과 `isSent`가 동시에 `true`가 될리는 없기 때문에, 3개의 유효한 state 중 하나를 맡을 `status` state 변수로 그것들을 바꾸는게 더 낫습니다: 'typing' (초깃값), 'sending', 그리고 'sent':**

<https://codesandbox.io/s/2t7gxo?file=%2FApp.js&utm_medium=sandpack>

가독성을 위해 몇 개의 상수들을 선언할 수도 있습니다.

```javascript
const isSending = status === 'sending';
const isSent = status === 'sent';
```

하지만 이것들은 state변수들은 아닙니다. 그러므로 그것들이 서로 동기화되지 않을까 걱정할 필요가 없습니다.

## 불필요한 state 피하기
렌더링하는 동안 컴포넌트의 props나 기존 state 변수에서 어떤 정보를 계산할 수 있는 경우, 해당 정보를 해당 컴포넌트의 state에 입력해서는 안 됩니다.
예를 들어 이 폼을 보죠. 잘 작동하지만 불필요한 state를 찾을 수 있나요?

<https://codesandbox.io/s/dnvs2u?file=%2FApp.js&utm_medium=sandpack>

이 폼에는 3개의 state 변수들이 있습니다: `firstName`, `lastName`, `fullName`. 그런데 `fullName`은 필요하지 않습니다. 매번 렌더링할 때마다 `firstName`과 `lastName`으로 부터 `fullName`을 계산합니다. 그러니 state로 부터 지웁시다.
이렇게 말이죠.

<https://codesandbox.io/s/sc71ts?file=%2FApp.js&utm_medium=sandpack>

여기서 `fullName`은 state 변수가 아닙니다. 렌더링하는동안 계산됩니다.

```javascript
const fullName = firstName + ' ' + lastName;
```

결과적으로 변경하는 핸들러는 `fullName`을 업데이트하기위해 어떤 작업도 필요하지 않습니다. `setFirstName`이나 `setLastName`을 호출할 때, 다시 렌더링을 요청하고나면 새로운 데이터로부터 `fullName`이 계산될 것 입니다.

#### 📒심화 학습
**props를 state로 미러링하지마세요!**
흔히 일어나는 불필요한 state 코드의 예제입니다.

```javascript
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
```

prop으로 받은 `messageColor`가 `color` state변수로 초기화됩니다. 문제는 **부모 컴포넌트가 나중에 다른 `messageColor`값을 보내면(예를 들어 `red`를 `blue`로 바꾸면), `color` state변수는 업데이트되지 않을 것 입니다 ** state는 오직 최초 렌더링 동안에만 초기화 되기 때문입니다.

prop을 state변수로 “미러링”하는 것이 우리를 혼란스럽게 하는 이유입니다. 대신 코드에서 직접 `messageColor`prop을 쓰세요. 더 짧은 이름을 원한다면 상수(const)를 사용하세요.

```javascript
function Message({ messageColor }) {
  const color = messageColor;
```

이렇게하면 부모 컴포넌트로 전달받은 prop이 동기화됩니다.

state에 “미러링하는 것”은 오직 특정 prop의 모든 업데이트를 무시하고 싶을 떄만 의미가 있습니다. 규칙에 따르면 새 값들이 무시됨을 명확히 하기 위하여 prop의 이름은 `initial`이나 `default`로 시작합니다.

```javascript
function Message({ initialColor }) {
  // The `color` state variable holds the *first* value of `initialColor`.
  // Further changes to the `initialColor` prop are ignored.
  const [color, setColor] = useState(initialColor);
```

## state에 중복 피하기

이 메뉴 리스트 컴포넌트를 사용하면 여러 여행 간식 중 하나를 선택할 수 있습니다.

<https://codesandbox.io/s/0m3ydb?file=%2FApp.js&utm_medium=sandpack>

현재 `selectedItem` state변수 안에 객체로써 선택된 아이템을 저장합니다. 그러나 썩 좋진않네요: **`selectedItem`의 컨텐츠들이 `items` 리스트 안에 있는 아이템들 중 하나와 동일한 객체이기 때문입니다.** 이는 항목 자체에 대한 정보가 두 위치에 복제되었음을 의미합니다.

이게 왜 문제일까요? 각각의 아이템을 수정 가능하게 만들어 봅시다.

<https://codesandbox.io/s/769ixj?file=%2FApp.js&utm_medium=sandpack>

아이템의 “Choose”버튼을 처음 클릭하고 아이템을 수정했을때, **인풋은 업데이트되지만, 하단의 라벨은 그 수정사항을 반영하지 않는걸 보세요.

`selectedItem`을 업데이트 할 수도 있지만 중복된걸 지우는 것이 가장 쉬운 방법입니다. 이 예제에서는 `selectedItem` 객체(`items` 내 객체로 중복을 만들어내는) 대신 `selectedId` state변수를 갖습니다. 그리고 해당 ID를 가진 아이템을 위해 `items` 배열을 검색함으로써 `selectedItem` 변수를 가져옵니다:

<https://codesandbox.io/s/ubzyo9?file=%2FApp.js&utm_medium=sandpack>

(또는 선택한 인덱스를 state로 유지할 수 있습니다.)

state는 이렇게 중복되었었습니다:

- `items = [{ id: 0, title: 'pretzels'}, …]`
- `selectedItem = {id: 0, title: 'pretzels’}`

하지만 변경한 뒤에는 이렇게 되었습니다:
- `items = [{ id: 0, title: 'pretzels'}, ...]`
- `selectedId = 0`
중복은 없어졌고 꼭 필요한 state만 가지고 있네요!
 
이제는 *선택된* 아이템을 수정하면 메시지가 즉시 업데이트 될 것 입니다. 그 이유는 `setItems`가 렌더링을 다시 일으키고 `items.find(...)`가 업데이트된 타이틀이 있는 아이템을 찾기 때문입니다. state엔 오직 *selected ID*만이 필수적으로 필요했기 때문에 더이상  *selected item*을 가지고 있을 필요가 없었습니다. 나머지는 렌더링 하는동안에 계산할 수 있습니다.

## 깊게 중첩된 state를 피하기
행성들, 대륙들, 나라들로 구성된 여행 계획이 있다고 상상해보세요. 중첩 객체나 중첩 배열을 이용한 state를 짜려고 시도해 볼 수 있습니다. 이 예제처럼요.

<https://codesandbox.io/s/1pun7l?file=%2Fplaces.js&utm_medium=sandpack>

이제 우리가 이미 다녀온 장소를 삭제하는 버튼을 추가하길 원한다고 해봅시다. 어떻게 할건가요? [중첩된 state 업데이트하려면](https://react.dev/learn/updating-objects-in-state#updating-a-nested-object) 변경된 부분부터 객체의 끝까지 복사한 복사본을 만드는 작업이 포함됩니다. 깊게 중첩된 곳을 삭제하려면 상위 사슬에 있는 모두를 복사해야합니다. 그러면 코드가 매우 길어질 수 있죠.

**state가 너무 중첩되어 쉽게 업데이트할 수 없는 경우 그 state를 “flat(평평한)”으로 만드는 것이 좋습니다.** 이 데이터를 재구성할 수 있는 한 방법이 있습니다. 각각의 `place`가 그것에 대한 *하위 place들의* 배열을 가진 트리처럼 생긴 구조 대신, 그 하위 장소들의 ID 배열을 가지고 있는 각각의 `place`를 가질 수 있습니다. 그리고 각 장소 ID에서 해당 장소로의 매핑을 저장합니다.

이 데이터 재구성은 마치 데이터베이스 테이블을 상기시킬 수 있습니다:

<https://codesandbox.io/s/pz7l98?file=%2Fplaces.js&utm_medium=sandpack>

**이제 state가 “flat”해졌으니 중첩된 아이템 업데이트가 쉬워졌네요.(“normalized(정규화)”라고도 알려진)**

장소를 삭제하기 위해서 state의 2레벨만 업데이트하면 됩니다.

- 상위 장소의 업데이트된 버전은 `childIds` 배열로부터 제거된 ID를 제외해야합니다.
- 루트 “테이블” 객체의 업데이트 버전은 상위 장소의 업데이트된 버전을 포함해야합니다.

이걸 어떻게 하는지에 대한 예제가 있습니다.

<https://codesandbox.io/s/x9kuzx?file=%2FApp.js&utm_medium=sandpack>


state를 원하는만큼 중첩시킬 수 있지만, state를 “flat”하게 만드는 것은 수많은 문제들을 해결해줍니다. State 업데이트를 쉽게해주고, 중복 객체의 다른 부분에 중복이 없는지 확인하는데 도움을 줍니다.

#### 📒심화학습
**메모리 사용량 개선시키기**
>이상적으로는 메모리 사용량을 개선시키기위해서 삭제된 아이템들(및 그 하위 아이템들도)을 “테이블” 객체에서도 삭제해야 합니다. 여기 그 버전이 있습니다. 여기서 또한 업데이트 로직을 더 간결하게 하기위해 [immer를 사용합니다.](https://react.dev/learn/updating-objects-in-state#write-concise-update-logic-with-immer)

<https://codesandbox.io/s/x9kuzx?file=%2FApp.js&utm_medium=sandpack>

때론 중첩된 state의 일부를 해당 자식 컴포넌트들로 옮기면서 state 중첩을 줄일 수 있습니다.  그 작업은 저장할 필요가 없는 일시적인 UI state에 적합합니다. 아이템이 호버될지 말지 정하는 state 변수처럼 말이죠.

## 복습
- 만약 2개의 state변수가 동시에 변한다면 하나로 합치는 것이 좋습니다.
- “불가능한” state가 생성되지 않도록 state 변수를 신중히 선택하세요.
- 실수로 업데이트 할 가능성을 줄이는 방식으로 state를 구성하세요.
- 동기화를 유지할 필요가 없게끔 불필요하고 중복된 state를 피하세요.
- 업데이트를 특정하게 막고싶은게 아니라면 state에 props를 넣지 마세요.
- selection과 같은 UI패턴을 위해, state에 객체 그 자체 보다는 ID나 index를 저장하세요. 
- 깊게 중첩된 state를 업데이트하는게 복잡하다면 “flat” 작업을 하세요.