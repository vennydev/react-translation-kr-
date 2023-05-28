# state에 배열 업데이트하기
자바스크립트에서 배열은 변경될 수 있습니다. 하지만 배열을 state에 저장할 때 그것들을 변할수 없는 것으로 취급해야합니다. 객체와 같이 state에 저장된 배열을 업데이트하길 원한다면 새로운 배열을 만들고(혹은 기존의 배열을 복사하거나), 그 새로운 배열을 사용하기위해 state를 세팅해야합니다.

### 배울 것
- 리액트 state 내에서 배열안의 항목을 더하고, 지우고, 변경하는 법
- 배열 안 객체를 업데이트하는 법
- immer를 이용해 덜 반복적인 코드로 배열을 복사하기

## 변경없이 배열 업데이트하기
자바스크립에서 배열은 그저 다른 종류의 객체일 뿐입니다. 여느 객체들 처럼, 리액트 state에 배열들은 읽기 전용으로 취급해야합니다. 이 말은 arr[0] = 'bird' 와 같이 배열 안에 항목을 재할당할 수 없고, push()나 pop() 과 같이 배열을 변경하는 메소드를 사용할 수 없음을 의미합니다.

대신 배열을 업데이트하고 싶을 때마다, 함수를 설정함으로써 새로운 배열에 state를 넘기고 싶을 것입니다. 그러기 위해선 filter()나 map()과 같이 변경을 일으키지않는 메소드를 호출함으로써, state에 있는 원본 배열로부터 복사한 새로운 배열을 생성해야합니다. 그럼 새로운 배열을 state로 설정할 수 있습니다.

여기 일반적인 배열 작업 참조 테이블이 있습니다. 리액트 state내에서 배열을 처리할 때, 왼쪽 컬럼의 메소드들은 피해야 합니다. 대신 오른쪽 컬럼을 선호해야합니다.

대체안으로 양쪽의 컬럼들에 있는 메소드들을 사용하게끔 도와주는 immer를 사용할 수 있습니다.

### 함정
유감스럽게도 slice와 splice는 이름은 비슷하지만 매우 다릅니다.

- Slice는 배열을 복사하거나 또는 배열의 부분을 복사합니다
- Splice는 배열을 변경합니다(항목을 삽입하거나 삭제하기 위해)

우리는 slice(splice가 아니라!)를 더 많이 사용하게 될 것입니다. 왜냐하면 리액트에선 state에 객체나 배열들을 변경하길 원하지 않기 때문이죠. [Updating Objects](https://react.dev/learn/updating-objects-in-state) 장에선 state에서의 변경이란 무엇이고 왜 추천하지 않는지 설명합니다.

### 배열을 추가하기

push()는 배열에 우리가 원치않는 변경을 일으킬 것입니다.

대신에 기존의 항목들과 끝에 새로운 아이템을 포함하는 새로운 배열을 만듭니다. 이를 위한 여러가지 방법이 있지만 가장 쉬운건 … [array spread](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#spread_in_array_literals) 구문을 사용하는 것 입니다.

```javascript
setArtists( // Replace the state
  [ // with a new array
    ...artists, // that contains all the old items
    { id: nextId++, name: name } // and one new item at the end
  ]
);
```

이제 잘 작동하네요:
<https://codesandbox.io/s/1vkb9e?file=%2FApp.js&utm_medium=sandpack>

이 Array spread 구문을 …artists 앞에 쓰면 아이템을 앞 쪽에 붙일 수도 있습니다.
<https://codesandbox.io/s/m1jpw3?file=%2FApp.js&utm_medium=sandpack>

```javascript
setArtists([
  { id: nextId++, name: name },
  ...artists // Put old items at the end
]);
```

이 방법으로 spread는 배열의 끝과 처음에 항목을 추가하는 push()와 unshift() 두 가지 작업을 모두 할 수 있습니다. 위에 sandbox에서 해보세요.

### 배열에서 삭제하기
배열에 항목을 삭제하는 가장 쉬운 방법은 필터링 하는 것 입니다. 즉 해당 항목을 포함하지 않는 새로운 배열을 만들어 볼겁니다. filter 메소드를 사용한 예시입니다:
<https://codesandbox.io/s/2q2hmx?file=%2FApp.js&utm_medium=sandpack>

“Delete”버튼을 몇번 클릭해보고 클릭 핸들러를 확인해보세요.

```
setArtists(
  artists.filter(a => a.id !== artist.id)
);
```

artists.filter(a => a.id !== artist.id) 은 “artist.id와는 다른 ID를 가진 artists를 포함한 배열을 생성해줘” 를 의미합니다. 즉, 각각의 아티스트들의 “Delete”버튼은 해당 아티스트를 배열에서 필터링합니다. 그리고 그 결과로 나온 배열로 다시 렌더링을 요청합니다.

### 배열 변형하기
만약 배열의 특정 부분이나 모든 항목들을 바꾸고 싶다면, 새로운 배열을 생성하기 위해 `map()`을 사용할 수 있습니다. `map`에 넘길 함수는 배열의 데이터나 인덱스(혹은 둘다)를 기반으로하여 각각의 항목에 무엇을 할지 결정할 수 있습니다.

이 예시에서 배열은 두 개의 원과 하나의 사각형의 좌표를 가지고 있습니다. 버트을 누르면 원들만 50픽셀 아래로 옮깁니다. 이것은 `map()`을 사용한 새로운 데이터의 배열을 생성함으로써 가능합니다:
<https://codesandbox.io/s/crvphm?file=%2FApp.js&utm_medium=sandpack>

### 항목 교체하기
특히나 배열에서 하나 또는 그 이상의 항목들을 교체해야하는 작업은 흔히 쓰이곤 합니다. `arr[0] = 'bird’`와 같은 할당은 원본 배열을 바꿉니다. 그래서 이것 대신 `map`을 사용하고 싶을 것 입니다.

항목을 교체하기 위해서 `map`으로 새로운 배열을 생성해세요. `map`호출 내부에서 두 번째 인자로 항목의 인덱스를 받을 수 있습니다. 기존 항목(첫번 째 인자)을 반환할지 아니면 다른 것을 반환할지 정하려면 인덱스를 사용해보세요:
<https://codesandbox.io/s/5qhfff?file=%2FApp.js&utm_medium=sandpack>

### 배열에 삽입하기
또 어떨 땐 배열의 처음도 끝도 아닌 특정 위치에 항목을 삽입하고 싶을 수도 있습니다. 이를 위해선 `…` array spread구문과 함께 `slice()` 메소드를 사용합니다. `slice()` 메소드는 배열의 “슬라이스(조각)”를 잘라 낼 수 있습니다. 항목을 삽입하려면 삽입 지점 *앞의* 슬라이스, 새 항목, 원래 배열의 나머지 부분을 펼치는 배열을 만듭니다.

아래 예시에선, 삽입 버튼은 새 항목을 항상 인덱스 **1**에 삽입합니다:
<https://react.dev/learn/updating-arrays-in-state#write-concise-update-logic-with-immer>

### 배열에서 그 외에 다른 변경하기
spread 구문이나 `map()`이나 `fliter()` 같이 변경을 일으키지 않는 메소드들로 할 수 없는 것들이 있습니다. 예를 들면 배열의 순서를 뒤집거나 정렬하고 싶을 수도 있지요. 자바스크립트의 `reverse()`나 `sort()` 메소드는 원본 배열을 변경시킵니다. 그러니 그것들을 직접 쓸 수는 없겠지요.

**하지만 먼저 그 배열들을 복사한 다음엔 배열에 변경을 일으킬 수 있습니다.**

예시입니다.
<https://codesandbox.io/s/e3b91r?file=%2FApp.js&utm_medium=sandpack>

위 코드에서 원본 배열의 복사본을 만들기위해 `[…list]` spread 구문을 사용합니다. 이제 우리는 복사본을 가지고 있고, `nextList.reverse()` 나 `nextList.sort()` 같이 변경을 일으키는 메소드들을 사용할 수 있게 되었네요. 심지어 `nextList[0] = "something"` 처럼 독립적인 항목들을 할당 할 수도 있겠네요.

**그러나 배열을 복사한다고 해도 배열 안에 기존 항목들을 직접 변경할 순 없습니다.** 그 이유는 복사가 얕게 되었기 때문입니다 - 새로운 배열은 원본 배열과 같은 항목들을 가지고 있습니다. 그래서 복사된 배열 안의 객체를 변경한다면, 기존 state도 변경합니다. 예를 들어 이 코드에는 문제가 있습니다. 

```javascript
const nextList = [...list];
nextList[0].seen = true; // Problem: mutates list[0]
setList(nextList);
```

`nextList`와 `list`가 다른 2개의 배열일지라도, `nextList[0]` 와 `list[0]`는 같은 오브젝트를 가리킵니다. 그래서 `nextList[0].seen`이 바뀜에 따라 `list[0].seen`도 바뀝니다. 이것이 우리가 피해야할 state 변경입니다! [updating nested JavaScript objects](https://react.dev/learn/updating-objects-in-state#updating-a-nested-object) 에서 비슷한 방법으로 이러한 문제를 해결할 수 있습니다. 이 페이지에는 항목들을 변경하는 것 대신 우리가 변경하고 싶은 각각의 항목들을 복사하는 방법이 있습니다.

## 배열안에 객체 업데이트하기

객체들은 실제로 배열 “안에” 있지 않습니다. 코드 상에선 객체들이 “안에” 있는 것처럼 보일 지도 모릅니다. 하지만 배열 안의 각 객체들은 해당 배열이 가리키고 있는 분리된 값입니다. 그것이 `list[0]`같이 중첩된 필드들을 바꿀 때 주의를 기울여야 하는 이유입니다. 다른 사람의 작품 목록이 그 배열에 같은 항목을 가리킬 수도 있습니다!

중첩된 state를 업데이트 할 때, 우리가 업데이트 하기를 원하는 지점부터 최상위 수준까지의 복사본을 만들어야 합니다. 이것이 어떻게 작동하는지 봐봅시다.

두 개의 작품 리스트들이 같은 초기 state값을 갖고 있는 예시가 있습니다. 그것들은 분리되어 있어야 하지만 변경으로 인해서 그들의 state는 의도치 않게 공유될 것 입니다. 결과적으로 한 리스트가 다른 리스트에 영향을 주기때문에 박스에 체크가 될 것 입니다:
<https://codesandbox.io/s/env1fs?file=%2FApp.js&utm_medium=sandpack>

문제는 이 코드입니다.

```javascript
const myNextList = [...myList];
const artwork = myNextList.find(a => a.id === artworkId);
artwork.seen = nextSeen; // Problem: mutates an existing item
setMyList(myNextList);
```

`myNextList` 그 자체로는 새로운 배열이지만, 그 안에 항목들은 원본 `myList `배열 안에있는 항목들과 같습니다. 그래서 `artwork.seen`을 바꾸는 것은 _원본_ 작품(= `myList`) 항목을 바꾸는 것과 같습니다. 역시나 원본 작품의 항목들은 `yourList` 안에도 있겠지요. 이것이 버그를 일으킵니다. 이러한 버그들은 생각해내기 힘들지만, 다행스럽게도 state 변경을 피하면 사라집니다.

**변경(mutation) 없이 기존 항목을 업데이트된 배열으로 대체하기 위해선 `map`을 사용할 수 있습니다.**

```javascript
setMyList(myList.map(artwork => {
  if (artwork.id === artworkId) {
    // Create a *new* object with changes
    return { ...artwork, seen: nextSeen };
  } else {
    // No changes
    return artwork;
  }
}));
```

여기서 `…`는 [create a copy of an object](https://react.dev/learn/updating-objects-in-state#copying-objects-with-the-spread-syntax) 파트에서 쓰인 객체 spread 구문입니다.
이렇게 접근하면, 기존 state에 있는 항목들의 어떤 것도 변경되지 않습니다. 그리고 버그도 고쳐지지요:

일반적으로는 **우리가 만들어낸 객체들만을 변경해야만 합니다**. 만약 새로운 작품을 삽입해야 할 경우엔 변경할 수 있습니다. 하지만 이미 state에 있는 무언가를 다루어야 한다면, 복사본을 만들어야만 합니다.

### immer로 간결한 업데이트 로직 작성하기

변경 없이 중첩 배열을 업데이트 하는 작업엔 반복적인 작업이 있습니다. [객체를 업데이트 ](https://react.dev/learn/updating-objects-in-state#write-concise-update-logic-with-immer)했던 것 처럼요:

- 일반적으로 몇 수준 이상의 state를 업데이트 할 필요가 없습니다. 만약 state 객체가 매우 깊은 경우엔 평평하게 되도록 [다르게 재구성](https://react.dev/learn/choosing-the-state-structure#avoid-deeply-nested-state)할 수 있습니다.
- State 구조를 바꾸고 싶지않다면, immer 사용을 선호 할 수도 있습니다. immer는  편리하지만 구문을 변경시켜주고 복사본 생성도 처리해줍니다.  

여기 immer를 사용한 Art Bucket List 예시가 있습니다.

immer를 사용하여 **`artwork.seen = nextSeen`같은 변경이 어떻게 가능해진건지** 주목 해보세요:

```
updateMyTodos(draft => {
  const artwork = draft.find(a => a.id === artworkId);
  artwork.seen = nextSeen;
});
```

이것이 가능한 이유는 원본 배열을 변경하지 않고 immer에 의해 제공된 특별한 `draft`객체를 변경하기 때문입니다. 비슷하게 `draft`의 컨텐츠에 `push()`나 `pop()`같이 변경을 일으키는 메소드들도 적용할 수 있습니다.

immer는 항상 draft에 수행한 변경 사항에 따라 항상 처음부터 다음 state를 만듭니다.  이것이 state를 계속 변경할 필요없이 매우 간결하게 이벤트 핸들러를 유지시켜줍니다.

## 복습
- State에 배열을 넣을 수 있습니다. 하지만 그것들을 변경하는 것은 불가능합니다.
- 배열을 변경하는 대신에 새로운 배열을 만들고나서 업데이트합니다.
- `[...arr, newItem]`같이 새로운 항목들이 있는 배열을 생성해주는 배열 spread 구문을 사용합니다.
- 필터링 또는 변화된 항목들이 있는 새로운 배열을 만드려면 `filter()`와 `map()`를 사용하세요.
- 코드를 간결하게 해주는 immer를 사용하세요.
