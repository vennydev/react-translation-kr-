# State: 컴포넌트의 메모리

종종 컴포넌트들은 상호작용의 결과로 화면에 보이는 어떠한 것을 변경해야할지도 모릅니다. 폼에 글자를 입력하는 것은 입력 필드를 업데이트 할 수도 있고요.  여러개의 이미지가 있을 때 "다음"을 클릭하면 보여야할 이미지를 바꿔야 할 수도 있습니다. 또한 "구매"를 누르면 쇼핑 카트에 물건을 담아야 할 지도 모르고요. 컴포넌트는 다음과 같은 것들을 "기억"해야 할 필요가 있습니다: 현재의 입력 값, 현재 보여줘야 할 이미지, 쇼핑 카트. 리액트에서는 컴포넌트에 특정한 메모리 같은 것들을 state 라고 부릅니다.

> 무엇을 배우나요
- useState 훅을 이용하여 state변수 추가하기
- useState 훅이 반환하는 값들의 쌍
- 1개 이상의 state변수를 추가하는 법
- 왜 state가 지역적(local)이라고 불리울까요?

## 일반 변수로는 충분치 않을 때
여기 조각상 사진을 렌더링하는 컴포넌트가 있습니다. "Next" 버튼을 누르면 index를 1로 그 다음엔 2 그 다음엔 3으로 계속 바꿈으로써 다음 조각상을 보여집니다. 그러나 작동하지 않네요.(직접 해보세요!)

`handleClick`이벤트 핸들러는 지역 변수를 업데이트하고 있습니다. 하지만 다음과 같은 두 가지 이유가 업데이트를 막고있습니다.
1. **지역 변수는 렌더링 사이에 유지되지 않습니다.** 리액트가 이 컴포넌트를 두번 째 렌더링 할 떄, 맨 처음부터 컴포넌트를 렌더링합니다. 이때 어떤 지역 변수들의 변화도 알아채지 않습니다.
2. **지역변수 변경은 렌더링을 일으키지 않습니다.** 리액트는 새로운 데이터로 그 컴포넌트를 렌더링 해야한다는 것을 인식하지 못합니다.

새로운 데이터로 컴포넌트를 업데이트하기 위해서는 다음 두 가지가 일어나야합니다.
1. 렌더링 중에 데이터를 **유지**한다
2. 리액트가 새로운 데이터로 컴포넌트를 렌더링하도록 한다(re-rendering)

`useState`훅은 다음 두가지를 제공합니다.
1. 렌더링 중에 데이터를 유지하는 **state 변수**
2. 변수를 업데이트하고 리액트가 컴포넌트를 다시 렌더링하도록하는 **state setter 함수**

## state 변수 추가하기
state 변수를 추가하기위해 파일 맨 위에 리액트로 부터 가져온 `useState`를 import합니다.

```javascript
import { useState } from 'react';
```

그리고 이 라인에 대체합니다.

```javascript
let index = 0;
```

이것으로요.

```javascript
const [index, setIndex] = useState(0);
```

`index`는 state 변수고 `setIndex`는 setter함수입니다.

> [] 구문은 [array destructing](https://javascript.info/destructuring-assignment) (배열 분해)라고 부릅니다. 이것은 배열로부터 값들을 읽을 수 있게 도와줍니다. `useState`에 의해 반환된 이 배열은 매번 2개의 아이템을 갖습니다.

<https://codesandbox.io/s/0efc0f?file=%2FApp.js&utm_medium=sandpack>

## 첫 번째 훅 만나보기
리액트에서 `useState` 뿐만 아니라 "`use`"로 시작하는 다른 함수들도 모두 훅으로 불립니다.
*훅*들은 오직 리액트가 [렌더링](https://react.dev/learn/render-and-commit#step-1-trigger-a-render)하는 동안만 사용할 수 있는 특별한 함수들입니다.(다음 장에서 더 자세히 다룹니다.) 이를 통해 다양한 리액트의 기능에 "연결(hook - 걸다)" 할 수 있습니다" 

### 함정
`use`로 시작하는 훅 함수들은 오직 컴포넌트나 우리가 만든 훅들의 최상위 레벨에서 호출될 수 있습니다. 조건문, 반복문 또는 중첩된 함수들 안에서 훅을 호출할 수는 없지요. 훅은 함수입니다. 하지만 우리가 만든 컴포넌트의 필요에 맞춘 비조건적인 선언으로써 그것들을 생각해보면 도움이 될 것 입니다. 파일 최상단에서 모듈들을  "import"하는 방법처럼 리액트의 기능들을 컴포넌트의 최상단에서 "사용"할 수 있습니다.

## useState 분석하기
[useState](https://react.dev/reference/react/useState)를 호출할 때, 우리는 리액트에게 이 컴포넌트가 무언가를 기억하게 해줘라고 말합니다.

```javascript
const [index, setIndex] = useState(0);
```

이 경우는 우리는 리액트가 `index`를 기억하길 원합니다.

> Note
> `const [something, setSomething]` 과 같이 쌍으로 이름 짓는 것이 규칙입니다. 원하는대로 이름 지을 수 있지만, 규칙을 사용하면 프로젝트를 전반적으로 이해하기 쉽습니다.

`useState`의 유일한 인자는 state변수의 초깃값입니다. 이 예제에선 `index`의 초깃값은 `useState(0)`에 의해 `0`으로 설정됩니다.

컴포넌트가 렌더링 될 때마다, `useState`는 2개의 값들을 담은 배열을 줍니다:
1. 저장한 값의 state 변수(`index`).
2. state 변수를 업데이트할 수 있고 리액트가 컴포넌트를 다시 렌더링하게할 수 있는 state setter 함수(`setIndex`).

작동 방식은 다음과 같습니다.

```javascript
const [index, setIndex] = useState(0);
```
1. **컴포넌트가 처음으로 렌더링 됩니다.** `useState`를 이용해 `index`의 초깃값으로 `0`을 넘겨주었으므로 `[0, setIndex]`를 반환합니다. 리액트는 최신 state 값으로 `0`을 기억합니다.
2. **state를 업데이트 합니다.** 사용자가 버튼을 클릭하면 `setIndex(index + 1)`가 호출됩니다. `index`는 `0`이기 때문에 `setIndex(1)` 가 됩니다. 이것이 리액트가 현재 `index`를 `1`로 기억하게 하고 또 다른 렌더링을 일으킵니다.
3. **컴포넌트가 두 번째로 렌더링 됩니다.** 리액트는 여전히 `useState(0)`을 보고 있지만 `index`를 `1`로 설정했던걸 기억하기 때문에 대신 `[1, setIndex]`를 반환합니다.

## 컴포넌트에 여러개의 state변수를 주기
컴포넌트에 원하는 만큼의 많은 타입의 state변수들을 갖고싶을 수 있습니다. 이 컴포넌트는 숫자인 `ìndex`와 "Show details"를 클릭하면 토글해주는 boolean타입의 `showMore`, 이렇게 2개의 state 변수를 가지고 있습니다.

<https://react.dev/learn/state-a-components-memory>

이 예시의 `index`나 `showMore`처럼 관련없는 여러개의 state 변수들을 가진다면 좋은 생각입니다.하지만 종종 2개의 state 변수를 함께 바꾸어야 한다면, 그것들을 하나로 묶는게 더 쉬울 수 있습니다. 예를 들어 많은 필드를 가지고 잇는 폼태그가 있다면, 필드마다 state 변수를 가지고 있는것보다 객체를 가지고 있는 하나의 state 변수를 갖는것이 더 편리합니다.

#### 심화학습
**리액트는 어떤 state가 반환되야 할지 어떻게 알까?**
`useState` 호출이 참조하는 state 변수에 대한 정보를 받지 않는다는 것을 알 수 있습니다. 
