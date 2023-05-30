# State 관리하기

어플리케이션의 규모가 커질 수록, 우리가 만든 state가 어떻게 이루어지고 컴포넌트 간 데이터의 흐름이 어떻게 흘러가는지 의식하는 것이 좋습니다. 중복되고 불필요한 state는 흔한 버그의 주범입니다. 이번 챕터에서는 어떻게 state를 잘 짤 수 있는지, state를 업데이트하는 로직을 유지보수 가능하게 관리 할 수 있는지, 거리가 먼 컴포넌트들 간에 state를 공유하는 방법에 대해 배울 것 입니다.

### 이번 챕터에서
- State 변경에 따른 UI 변경은 어떻게 이루어지는걸까요?
- State 구조 잘 짜기
- 컴포넌트들간에 state를 공유하기 위한 state 끌어올리기
- state를 그대로 둘지 리셋할지 컨트롤하는 방법
- 함수 안에 복잡한 state 로직을 통합하는 방법
- “Props drilling”없이 정보를 전달하는 방법
- 앱이 커지면서 state 관리를 확장시키는 방법

## state로 input에 반응하기
리액트에선 코드를 이용해 UI를 직접적으로 변경할 수 없습니다. 예를 들어 “버튼 사용 못 하게 해줘”, ”버튼 사용하게 해줘”, ”성공 시 메시지를 보여줘” 처럼 명령어를 입력할 수 없듯이 말이죠. 대신 컴포넌트에 있는 다양한 state(“initial state”, “typing state”, “success state”)를 이용하여 우리가 보여줄 UI를 묘사해야합니다. 그리고 사용자 input에 반응하여 state의 변화를 일으켜야합니다. 디자이너가 UI를 생각하는 방식과 유사하지요.

리액트로 만들어진 퀴즈 form이 있습니다. Submit 버튼을 활성화 시킬지 시키지 않을지, 성공 메시지를 보여줄지 말지를 결정하기위해 어떻게 `status` state 변수를 이용하는지 주목해보세요.

<https://codesandbox.io/s/vpttuv?file=%2FApp.js&utm_medium=sandpack>

> **이 토픽을 배울 준비가 되었나요?**
> state 중심 사고방식으로 상호작용에 접근하는 방법을 배우고싶다면  **State로 input에 반응하기** 를 읽어보세요.
> [Read More]()

---
##  state 구조 정하기

State 구조를 잘 짜는 것은 수정하기도 버그를 잡기에도 좋은 컴포넌트와 끊임없이 버그가 나오는 컴포넌트간에 차이를 만듭니다. 여기서 가장 중요한 원칙은 state는 불필요하고 중복된 정보를 가지고 있어선 안된다는 것입니다. 만약 불필요한 state가 있다면 업데이트 하는 것을 잊고 버그를 드러내기 쉽습니다!

예를 들어 여기에 불필요한 `fullName` state 변수가 있습니다.

<https://codesandbox.io/s/dnvs2u?file=%2FApp.js&utm_medium=sandpack>

`fullName`을 지우고 코드를 간단히 할 수 있습니다. 컴포넌트를 렌더링하는동안 `fullName`을 계산함으로써  말이죠.

<https://codesandbox.io/s/sc71ts?file=%2FApp.js&utm_medium=sandpack>

이건 작은 변경 같지만, 이 방법으로 리액트 앱에서 일어나는 많은 버그들을 고칠 수 있습니다.

> **이 토픽을 배울 준비가 되었나요?**
> 버그를 피하기위해 어떻게 state를 짜야할지 배우고싶다면  **Choosing the State Structure** 를 읽어보세요.
> [Read More]()

---
## 컴포넌트 간 state 공유하기

종종 항상 함께 변하는 두 컴포넌트의 state를 원할수도 있습니다. 그러기 위해선 양 쪽의 state를 지우고 그들의 가장 가까운 공통 조상 컴포넌트로 state를 옮기세요. 그 다음 props를 통해 그들을 아래로 내리세요. 이것은 소위 “state 끌어올리기”라고 알려져있습니다. 이것이 리액트 코드를 작성할 때 가장 많이 하는 작업 중 하나 입니다.

이 예제에서는 오직 하나의 패널만 활성화해야합니다. 그러기 위해선 각각의 패널 안에 활성화시키는 state를 두는 것이 아니라, 그들의 부모 컴포넌트에 state를 두고 자식 컴포넌트들을 위해 props를 지정하세요.

> **이 토픽을 배울 준비가 되었나요?**
> 어떻게 state를 끌어올리고 컴포넌트들을 동기화 상태로 유지하는 법을 배우고싶다면  **Sharing State Between Components** 를 읽어보세요.
> [Read More]()


---
## State 유지 및 재설정하기

컴포넌트를 다시 렌더링할 때, 리액트는 트리의 어떤 부분이 유지되어야 하는지(업데이트 되어야하는지), 어떤 부분이 삭제되고 또는 맨 처음부터 다시 생성되어야하는지를 결정해야만 합니다. 대부분의 경우, 리액트의 자동화된 동작이 잘 작동합니다. 기본적으로 리액트는 이전에 렌더된 컴포넌트 트리와 “일치”하는 트리의 부분들을 유지합니다.

그러나 이것은 우리가 원하지 않는 것일 수도 있습니다. 이 채팅 앱에서 메시지를 치고나서 수신자를 바꾸어도 input이 재설정(리셋)되지 않습니다. 이것은 사용자가 의도치않게 잘못된 사람에게 메시지를 보내게끔할 수 있습니다.

<https://react.dev/learn/managing-state>

리액트는 기본 동작을 무시하고 `<Chat key={email} />`처럼 다른 `key`를 넘김으로써 컴포넌트의 state를 재구성하게 합니다. 이것이 리액트에게, 만약 수신인이 다르면, 새로운 데이터(그리고 input같은 UI도)와 함께 처음부터 다시 생성되어야 하는 다른 `Chat` 컴포넌트를 주어야할지도 모른다고 말해줍니다. 이제 수신인을 바꾸면 인풋 필드를 재설정하네요 - 동일한 컴포넌트를 렌더링하더라도 말이죠.

<https://codesandbox.io/s/ojpwim?file=%2FApp.js&utm_medium=sandpack>

> **이 토픽을 배울 준비가 되었나요?**
> state의 생명주기와 그것을 다루는 방법을 배우고싶다면  **Preserving and Resetting State** 를 읽어보세요.
> [Read More]()


---
## Reducer로 state로직 추출하기
많은 이벤트 핸들러에 분산된 state 업데이트가 많은 컴포넌트들은 무시무시합니다. 이런 경우에 컴포넌트 외부의 모든 state 업데이트 로직을 “reducer”라고 부르는 단일함수로 통합할 수 있습니다. reducer가 오직 그 사용자의 “actions”만을 지정할 것이기 때문에 이벤트 핸들러는 간결해질 것 입니다. 다음 코드의 아래로 내려가보면 reducer 함수가 각 action에 응답하여 어떻게 state를 업데이하는지에 대한 방법을 지정합니다.

<https://codesandbox.io/s/j8ukrp?file=%2FApp.js&utm_medium=sandpack>


> **이 토픽을 배울 준비가 되었나요?**
> 어떻게 reducer 함수 안에서 로직을 통합할 수 있을지 배우고싶다면  ** Extracting State Logic into a Reducer** 를 읽어보세요.
> [Read More]()

---
## 컨텍스트(Context)로 데이터를 심층적으로 전달하기
항상 prop를 통해서 부모 컴포넌트에서 자식 컴포넌트로 정보를 전달했습니다. 하지만 많은 컴포넌트들에 몇몇 props를 전달해야하거나 여러. 컴포넌트들이 같은 정보가 필요하다면 props를 전달하는것이 불편할 수도 있습니다. 컨텍스트는 props를 정확하게 전달하는 것 없이도 부모 컴포넌트에서 그 아래에 있는 트리의 어떤 컴포넌트에서도 정보들을 사용할 수 있게끔 해줍니다. 깊이가 얼마나 깊은진 상관없습니다.

여기 해당 레벨에 가장 가까운 `Section`을 “요청”하여 향해야할 레벨을 결정하는 `Heading` 컴포넌트가 있습니다. 각각의 `Section`은 부모 `Section`을 요청하고 `Section`을 요청하여 자체 레벨을 추적합니다. 모든 `Section`은 그 아래에 있는 모든 컴포넌트들에 props를 전달하지 않고 정보를 제공합니다.  컨텍스트가 그 일을 하고 있습니다. 

<https://codesandbox.io/s/j0w3hj?file=%2FSection.js&utm_medium=sandpack>

> **이 토픽을 배울 준비가 되었나요?**
> props를 전달하는것 대신 컨텍스트를 쓰는 방법을 배우고싶다면  ** Passing Data Deeply with Context** 를 읽어보세요.
> [Read More]()


---
## Reducer와 컨텍스트로 확장해보기
Reducer는 컴포넌트의 업데이트 로직을 통합시켜줍니다. 컨텍스트는 다른 컴포넌트에 깊게 내려가지 않고 정보를 전달해줍니다. 복잡한 화면의 state를 관리하기 위해 둘을 합칠 수 있습니다.

이러한 접근법으로 복잡한 state를 가진 부모 컴포넌트는 reducer를 이용해 state를 다룹니다. 그 트리안에 얼만큼 깊이 있는 컴포넌트던지 컨텍스트를 이용하여 state를 읽을 수 있습니다. 또한 state를 업데이트 하기위해서 action들을 보낼(dispatch) 수 있습니다.

<https://codesandbox.io/s/4yvi7t?file=%2FApp.js&utm_medium=sandpack>

> **이 토픽을 배울 준비가 되었나요?**
> 앱이 커짐에 따라 state관리를 확장시킬 수 있는 법을 배우고싶다면  **Scaling Up with Reducer and Context** 를 읽어보세요.
> [Read More]()

—

## 다음 배울것
이 챕터를 페이지 별로 읽으려면 [Reacting to Input with State]()로 가세요.
이 토픽들에 익숙하다면 [Escape Hatches]()를 읽어보세요.