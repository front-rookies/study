# React19의 Action hook & use

## 1. useOptimistic hook: 낙관적 업데이트를 손쉽게 구현

useOptimistic hook은 낙관적(Optimistic)인 업데이트를 쉽게 구현하기 위한 hook이다.

낙관적인 업데이트란, "요청이 실패할 가능성이 낮으니 우선 UI를 업데이트 해버리자"라는 전략으로, UX를 향상시키는 것이 목적이다.<br>
기본적인 흐름은 다음과 같다.

(1) 시간이 걸리는 요청이 있다.<br>
(2) 응답이 오기 전에 UI를 기대값으로 설정해 둔다.<br>
(3: 요청 성공 시) 응답결과를 반영한다.<br>
(3: 요청 실패 시) UI를 요청 전의 상태로 돌린다.

이런 동작을 구현하려면 요청 전후 값, 기대값의 관리, 실패 시 rollback하거나 성공 시 값을 복원하는 등, 예상보다 복잡한 처리가 필요하다.<br>
이걸 최대한 쉽게 구현할 수 있도록 도와주는 게 useOptimistic hook이다.

```typescript
function UpdateTitle({ value, onUpdate }) {
  // useOptimistic로 낙관적 상태와 업데이트 함수를 구조분해할당 ...(1)
  const [optimisticTitle, setOptimisticTitle] = useOptimistic(value);

  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const value = formData.get("title");
      // 낙관적 업데이트를 실행함 ...(2)
      setOptimisticTitle(value);
      const { data, error } = await updateTitle(value);
      if (!error) {
        onUpdate(data);
      }
      return error;
    },
    null // state초기값
  );

  return (
    <form action={submitAction}>
      <input name="title" defaultValue={value} disabled={isPending} />
      <button type="submit" disabled={isPending}>
        更新
      </button>
      {error && <p>{error}</p>}
      <div>値: {value}</div>
      {/* 낙관적 상태를 표시한다 ...(3) */}
      <div>楽観的な値: {optimisticTitle}</div>
    </form>
  );
}
```

(1)에서 useOptimistic을 사용해, state와 업데이트 함수를 구조분해할당하고 있다. 이 state는 낙관적 상태라고 불린다.<br>
(2)에서 실제 요청 전에 state 업데이트 함수를 호출하여 optimisticTitle을 낙관적으로 기대값으로 업데이트 한다. 화면에 표시되는 state는 optimisticTitle로 지정한다.(3)

value를 직접 표시하고 있는 위의 値(값)은 비동기 함수 요청이 완료될 때까지 업데이트 되지 않지만, 낙관적 상태인 optimisticTitle은 버튼을 누르자마자 업데이트 된다. 그리고 비동기 함수 요청이 성공한 경우(error가 아닌 경우)에는 그대로 유지되지만, 요청이 실패한 경우(error가 발생한 경우: 예를 들어 외부 API와의 통신이 실패한 경우)엔 변경 전 값으로 돌아간다.

> ### 칼럼: useOptimistic의 사용처<br>
>
> 항상 낙관적 업데이트가 필요한 건 아니다. 여기서는 form을 사용한 값 업데이트를 예시로 들었지만 업데이트 된 값을 즉시 반영하면 사용자가 "업데이트가 완료됐다"고 오인할 수 있다.<br>
> 좋아요 버튼이나 메시지 앱처럼 요청 후의 상태가 프론트엔드에서 예측이 가능하고, 즉시 반영되지 않으면 사용자 경험에 영향을 미치는 유스케이스에서 유효한 접근법이다.

## 2. useFormStatus hook: form태그 내부의 컴포넌트에서 사용하는 특수한 hook

이 useFormStatus는 지금까지 소개한 hook과는 달리 기존의 처리를 광범위하게 개선하진 않고, 사용도 form태그 내부에 있는 자식 컴포넌트에서만 사용할 수 있다.<br>

```typescript
// 컴포넌트화 하여 전송
function SubmitButton () {
  // useFormStatus에서 제출중 상태와 제출중인 form값을 가져오기...(2)
  const { pending, data } = useFormStatus();
  const value = data?.get("title");
  return (
    <button type="submit" disabled={pending}>
      {value ? `${value} に更新中`　：`更新`}
    ＜/button>
  );
}
function UpdateTitle ({ value, onUpdate }) {
  const [error, submitAction, isPending] = useActionState(
    // ~중략~
  ）;
  return (
    <form action={submitAction}>
      <input name="title" defaultValue={value} disabled={isPending} />
      {/* 반드시＜form>내부에서 사용할 것... (1) */}
      <SubmitButton />
      {error && <p>{error}</p>}
      <div>value: {value}</div>
    </form>
  );
}
```

SubmitButton컴포넌트는 반드시 form내부에서 불러야 한다(1). 그렇게 해야 useFormStatus hook을 사용할 수 있다.<br>
useFormStatus hook은 form이 제출 중인지 아닌지를 표시하는 pending 외에 제출 중인 값을 data(formData형식)로 참조할 수 있기 때문에 이 값을 props로 전달할 필요가 없다(2).<br>
주의할 것은 이 data는 pending이 true일 때만, 즉 form 제출 중일 때만 참조할 수 있다는 점이다(그 외엔 null). 이것은 form제출 처리 중에 표시할 내용을 결정하기 위한 용도라는 점임을 주의하자.

### 1) useActionState와 useFormStatus비교

| 항목           | useActionState                   | useFormStatus                       |
| -------------- | -------------------------------- | ----------------------------------- |
| 역할           | 서버액션 실행후 경과와 상태 관리 | <form>내부에서 제출 상태를 구독     |
| 사용 위치      | 폼 바깥, 루트 컴포넌트 쪽        | 폼 내부 컴포넌트에서 제출 상태 감지 |
| 주요 사용 목적 | 액션 실행 결과 상태              | 로딩 중 UI처리                      |
| 상태 범위      | 상태 전역 관리에 가깝게 가능     | 현재 <form> 제출 상태에 한정됨      |

## 3. use: 렌더링 중에 리소스를 가져오는 새로운 API

use는 Promise나 Context등 리소스에서 값을 가져오기 위한 새로운 API다. 현재 use에 지정할 수 있는 리소스는 Promise나 Context밖에 없다.

### 1) use로 Context값 가져오기

Context는 React 상태관리 방법 중 하나로, 상위 컴포넌트에서 지정한 값을 하위 컴포넌트에서 참조할 수 있도록 하는 기능이다. 보통 UI테마와 같이 넓은 범위에 영향을 주는 값을 다룰 때 사용하며 props로 전달하는 불편함을 해결하기 위해 사용한다. 다만 전역 변수에 가까운 개념이기 때문에 과도한 사용은 지양하는 게 좋겠다.

지금까지는 하위 컴포넌트로 Context값을 가져올 때 useContext hook을 사용했지만 이젠 use로 대체할 수 있다.

```typescript
// 테마 context
const ThemeContext = createContext("light");
// 테마를 변경하는 컴포넌트
function ThemeChanger() {
  const [theme, setTheme] = useState("light");
  return (
    <div>
      <button onClick={() => setTheme((prev => (prev === "light" ? "dark" : "light"))}>;
        テーマ変更
      </button>
      <ThemeContext value={theme}>
        <ThemeDisplay />
      </ThemeContext>
    </div>
  );
}
//　테마를 사용하는 컴포넌트
function ThemeDisplay() {
  // const theme = useContext(ThemeContext); // 기존 useContext을 사용한 작성방식...(1)
  const theme = use(ThemeContext); //　React 19~　use를 사용한 작성방식...(2)
  return <p>テーマ： {theme}</p>;
}
function App() {
  return <ThemeChanger />;
}
```

위의 예제에서 useContext를 사용한 작성방식(1)을 use를 사용한 작성방식(2)으로 대체할 수 있다. 둘 다 동작은 같다.

hook인 useContext는 반드시 컴포넌트 바로 아래에서 호출해야 하며 if문 안이나 early return 뒤에서는 사용할 수 없다. 반면 use는 hook이 아니므로 if문이나 반복문 내부에서도 호출할 수 있다.<br>
코드4와 같이 use는 early return 이후에도 사용할 수 있어 코드 작성이 더 유연해진다. 따라서 앞으로도 use를 사용하는 것이 권장된다.

##### 코드４：use를 사용하면 early return도 가능

```typescript
// 테마를 사용하는 컴포넌트
function ThemeDisplay() {
  if (someCondition) {
    return null; //　early return
  }
  const theme = use(ThemeContext); // early return 이후에는 useContext호출은 안되지만、use호출은 가능
  return <p>テーマ: {theme}</p>;
}
```

또한 지금까지는 Context엔 Context.Provider를 사용해서 하위 컴포넌트에 적용했지만, React19부터는 Context를 직접 지정하도록 변경되었다. 기존의 Context.Provider는 향후 버전에서 더이상 권장되지 않을 예정이므로 사용시 주의가 필요하다.

##### 코드５：React19이후의 Context사용방법

```typescript
const ThemeContext = React.createContext("light");
function App () {
  return (
    // ＜ThemeContext.Provider value="dark"> {/* React 18까지의 작성방식 */}
    //   <ThemeDisplay />
    // </ThemeContext.Provider>
    <ThemeContext value="dark"> {/* React19이후의 작성방식 */}
    　　＜ThemeDisplay>
    </ThemeContext>
  );

```

### 2) use로 Promise값 가져오기

또 다른 use의 용법은 Promise를 전달하여 그 결과를 가져오는 것이다.<br>
'await를 쓰면 되는 거 아닌가?'라고 생각할 수도 있지만 일반적인 함수 컴포넌트는 동기 함수이기 때문에 직접 await를 사용할 수 없다. 하지만 use를 사용하면 함수 컴포넌트 내부에서도 Promise의 결과를 가져올 수 있게 된다.<br>
use는 React18에서 도입된 Suspense의 동작 방식을 기반으로 하며, 전달된 Promise의 응답을 기다리는 동안 use는 일시중단(Suspend)된다. 즉, use를 제대로 사용하려면 해당 컴포넌트 상위에 Suspense가 존재해야 한다.

##### 코드６：use로 Promise값을 가져오는 예제

```typescript
import { use, Suspense } from "react";

function DataDisplay({ dataPromise }) {
  // use를 사용해서 Promise에서 데이터를 받아온다。
  // ＊Promise응답을 기다리는 동안 렌더링이 중단되고 Suspense의 fallback이 표시된다.
  const dataContent = use(dataPromise);
  return <p>データ： {dataContent}</p>;
}

export function DataContainer({ dataPromise }) {
  return (
    // Promise응답을 기다리는 동안은 fallback이 표시된다.
    <Suspense fallback={<p>読み込み中...</p>}>
      <DataDisplay dataPromise={dataPromise} />
    </Suspense>
  );
}
```
