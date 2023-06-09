# 스냅샷으로써의 state
---
state 변수는 우리가 읽고 쓸수 있는 일반적인 Javascript 변수처럼 보일지도 모릅니다. 하지만 그렇기보다는 state는 마치 스냅샷처럼 동작합니다. state를 설정한다고 우리가 이미 갖고 있는 state 변수를 변화시키진 않습니다. 다만 re-render를 발생시킬 뿐 입니다.

### 이번 챕터에서 배우는 것
- 어떻게 state를 설정하는 것이 re-render를 발생시킬까
- state는 언제, 어떻게 업데이트 될까
- 설정한 직후 상태가 바로 업데이트 되지 않는 이유
- 어떻게 state의 "스냅샷"이 이벤트 핸들러에 접근할까

## state 설정은 render를 불러일으킨다
---
클릭과 같은 사용자 이벤트에 반응해서 인터페이스가 즉시 바뀔거라고 생각할 지도 모릅니다. 이러한 인식과는 다르게 리액트는 조금 다르게 작동합니다. 이전 페이지에서 state를 설정하는것이 re-render를 요청하는 것을 보았습니다. 즉, 인터페이스가 이벤트에 반응하려면 state를 업데이트시켜야 합니다.

버튼을 클릭 했을 때 이런 일이 일어납니다.
1. onSubmit 이벤트 핸들러가 실행됩니다.
2. setIsSent(true) 가 isSent를 true로 설정하고, 새로운 렌더링을 대기열에 넣습니다
3. React는 새로운 isSent 값에 따라 다시 컴포넌트를 렌더링 시킵니다. 
자 state와 렌더링 사이의 관계를 좀더 깊게 살펴봅시다.

## 렌더링은 시간에 따라 스냅샷을 찍는다
---
“렌더링”은 React가 우리가 만든 컴포넌트(컴포넌트는 함수입니다)를 부르는것을 의미합니다. 그 함수로부터 반환된 JSX는 시간에 따른 UI의 스냅샷 같습니다. 그것의 props, 이벤트 핸들러 그리고 로컬 변수들은 렌더링 시점에 state를 이용하여 모두 계산되어집니다.

사진이나 영화의 프레임과는 다르게, 반환되는 UI "스냅샷"은 상호작용적입니다. 인풋에 대한 반응으로 무슨 일이 일어나는지 알려주는 이벤트 핸들러같은 로직을 포함합니다. React는 이 스냅샷과 일치하도록 화면을 업데이트하고 이벤트 핸들러와 연결합니다. 결과적으로 버튼을 누르는것은 JSX로부터 나온 클릭 핸들러를 트리거 할 것입니다.

React가 컴포넌트를 다시 렌더링 시킬 때:
1. React가 함수를 다시 호출합니다.
2. 함수가 새로운 JSX 스냅샷을 반환합니다.
3. React는 우리가 반환한 스냅샷과 일치하도록 화면을 업데이트합니다. 

리액트가 함수를 호출합니다 => 스냅샷을 계산합니다 => DOM 트리를 업데이트 합니다

컴포넌트의 메모리로써의 state는 함수가 반환한 뒤 사라지는 일반적인 변수와는 다릅니다. state는 함수 외부 React 그 자체에 남아 있습니다 - 마치 선반에 있는 것처럼요. React가 컴포넌트를 호출 할 때 해당하는 특정 렌더링에 대한 state의 스냅샷을 제공합니다. 그럼 그 컴포넌트는 JSX안에 새로운 props와 이벤트 핸들러가 있는 UI의 스냅샷을 반환합니다. 그것들은 모두 해당 렌더링의 state값을 사용하여 계산됩니다.

리액트에게 state를 업데이트하라고 지시합니다 => 리액트가 state를 업데이트 합니다 => 리액트가 state값의 스냅샷을 컴포넌트에게 넘깁니다

여기 그러한 것들이 어떻게 작동하는지 설명하는 작은 실험예제가 있습니다. 이 예제에서 우리는 “+3” 버튼을 클릭하면 3을 증가시킬 것이라고 예상지도 모릅니다. 왜냐하면 setNumber(number + 1)을 3번 호출하고 있기 때문에 말이죠. 

그럼 “+3”버튼을 클릭했을 때 무슨 일이 일어나는지 확인해보지요.

number 변수가 매 클릭마다 1씩 증가하는 것을 보세요.

**setState는 오직 다음 렌더링에만 state를 바꿉니다.** 처음으로 렌더링이 되는 중에는 number 변수는 0이었습니다. 그것이 해당 렌더링의 onClick 핸들러에서 setNumber(number + 1)이 호출된 후임에도 불구하고 number 변수의 값이 여전히 0인 이유입니다. 

이것이 버튼의 click 핸들러가 리액트에게 지시하는 일입니다.

1. setNumber(number + 1) : number 는 0 이기 떄문에 setNumber(0 + 1).- 리액트는 다음 렌더링에 number를 1로 바꿀 준비를 합니다. 
2. setNumber(number + 1) : number 는 0 이기 떄문에 setNumber(0 + 1).- 리액트는 다음 렌더링에 number를 1로 바꿀 준비를 합니다. 
3. setNumber(number + 1) : number 는 0 이기 떄문에 setNumber(0 + 1).- 리액트는 다음 렌더링에 number를 1로 바꿀 준비를 합니다. 
setNumber(number + 1)를 3번 호출했을에도 불구하고 이번 렌더링에서 이벤트 핸들러의 number는 여전히 0이므로 state 1을 3번 설정합니다. 이것이 이벤트 핸들러가 완료 된 후, 리액트가 number가 3이 아닌 1인 컴포넌트를 렌더링 하는 이유입니다.    

state 변수들의 값을 대체해서 확인해 볼 수 있습니다.이번 렌더링에서 number state 변수가 0이기 때문에 이 이벤트 핸들러는 이렇게 보입니다:

다음 렌더링에서 number는 1 이므로 이번 이벤트 핸들러는 이렇게 생겼습니다.

이것이 다음버튼을 눌러도 계속해서 2,3 .. 으로 설정되는 이유입니다.

## 시간 경과에 따른 state

재밌었죠? 이번에도 버튼을 클릭하면 어떤 것이 나올지 예상해보세요.

이전에 치환법을 사용했다면 0이 나오리란 것을 예상했을 것 입니다.

하지만 컴포넌트가 렌더링된 후에만 실행되는 alert에 timer를 설정하여 넣으면 어떻게 될까요? “0” 이 나올까요 “5”가 나올까요. 
예상해보세요!

놀랍죠? 치환법을 사용했으면 alert에 전해진 state의 스냅샷을 볼 수 있습니다.

리액트에 저장된 state는 alert가 실행되는 시점에 맞추어 변할 지도 모릅니다. 하지만 실제로는 사용자가 상호작용하는 시점의 state의 스냅샷을 사용하여 예약됩니다.

그 변수의 이벤트 핸들러 코드가 비동기인 경우에도 **state 변수의 값은 렌더링하는 동안 절대로 변경되지 않습니다.** 그 렌더링의 onClick 내부에서 setNumber(number + 5) 가 호출된 후에도 number값은 계속해서 0일 것 입니다. 컴포넌트가 호출됨으로써 React가 UI의 “스냅샷을 찍을 때” 그 값은 “고정”됩니다. 

여기 이벤트 핸들러가 타이밍 실수를 어떻게 줄 일 수 있는지에 대한 예가 있습니다. 아래에 5초 뒤 메시지를 보내는 form이 있습니다.

1. “Send” 버튼을 누르면 Alice 에게 “Hello”를 보냅니다.
2. 5초가 끝나기전에, “To” 필드의 값을 “Bob”으로 바꿉니다. 

리액트는 한번의 렌더링 내에 이벤트 핸들러 안에서 state값을 “고정”시킵니다. 코드가 실행되는 동안 state가 바뀌었는지 안 바뀌었는지 걱정할 필요가 없습니다.

다시 렌더링 되기 전에 최신 state값을 읽어내고 싶다면 어떨까요? State updater function을 사용하고 싶을겁니다. 다음 페이지에서 다뤄보겠습니다!

복습
- state를 설정하는 것 (setState)은 새로운 렌더링을 요청합니다.
- 리액트는 컴포넌트 바깥에서 state를 저장합니다. 선반에 두는 것처럼요.
- useState를 호출할 때, 리액트는 해당 렌더링에서의 state 스냅샷을 제공합니다.
- 변수나 이벤트 핸들러들은 다시 렌더링 될떄 남아있지않습니다. 매 렌더링에는 자체 이벤트 핸들러들이 있습니다.
- 매번 렌더링에는 리액트가 해당 렌더링에 제공한 state의 스냅샷을 볼 수 있습니다.
- 마음 속으로 이벤트 핸들러안의 state를 대체해볼 수 있습니다. 렌더링 된 JSX에 대해 생각해보는것과 유사합니다.
- 과거에 만들어진 이벤트 핸들러들은 그것들이 생성되었던 렌더링에서의 state 값을 갖습니다.

