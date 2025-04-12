# React19의 비동기 Transition과 Form Action

## 1. Transition이란?

Transition은 state업데이트의 우선순위를 낮추는 매커니즘이다.<br>
대량의 데이터를 불러와서 렌더링할 때 시간이 걸리면 UI가 일시적으로 멈출 수 있지만 Transition을 사용하면 이러한 부하가 큰 렌더링의 우선순위를 낮출 수 있기 때문에 매끄러운 사용자 경험을 제공할 수 있다.

### 1) React18의 Transition

React18 Transition은 실행할 작업에 동기함수만 지정할 수 있다.

##### 코드１：React18 Transition예시

```typescript
function TransitionExample() {
  // useTransition을 사용해 Transition중인 상태를 불러옴 ...(1)
  const [isPending, startTransition] = useTransition();
  // 데이터를 불러오는 중일 때의 상태를 관리 ...(2)
  const [isLoading, setIsLoading] = useState(false);
  const [data, setData] = useState(null);

  const loadData = () => {
    setIsLoading(true);
    fetchHeavyData() // 부하가 큰 데이터를 불러오는 작업
      .then((result) => {
        // 데이터 업데이트를 Transition（낮은 우선순위）으로 지정 ...(3)
        startTransition(() => {
          // 동기함수만 지정할 수 있음 ...(4)
          setData(result);
          setIsLoading(false);
        });
      });
  };

  return (
    <div>
      <button onClick={loadData}>
        {isPending || isLoading ? "読み込み中" : "データ取得"}
      </button>
      {/* 렌더링한 부하가 큰 컴포넌트 ...(5) */}
      {data && <HeavyComponent data={data} />}
    </div>
  );
}
```

useTranstion을 사용하면 Transition실행상태(isPending)를 얻을 수 있다.<br>
(1)이 fetchHeavyData가 실행중(3)일 때는 isPending이 true로 변경되지 않기 때문에 별도의 상태값 isLoading(2)으로 비동기 처리 상태를 관리하고 있다.

(3)에서는 startTransition을 사용하여 setData가 낮은 우선순위로 실행되도록 설정하고 있다.<br>
이 덕분에 HeavyComponent(6)가 렌더링되는 동안에도 사용자는 다른 작업을 수행할 수 있다.

하지만 (4)에서처럼 Transition내부에서는 동기함수만 실행할 수 있으며 비동기 처리를 직접 수행하는 것은 불가능하다.

### 2) React19의 비동기 Transition

React19에서는 Transition이 비동기 함수를 직접 실행할 수 있도록 변경되었다.<br>
Transition이 비동기 함수 시작부터 완료까지 직접 관리해주기 때문에 코드1에서 isLoading같은 비동기 처리 상태추적을 위한 state가 필요없게 되었다.

##### 코드２：React19 비동기 Transition 예시

```typescript
function TransitionExample() {
  // isLoading는 필요없음 ...(1)
  const [isPending, startTransition] = useTransition();
  const [data, setData] = useState(null);

  const loadData = () => {
    //　데이터 패칭과 같은 비동기 처리를 Transition으로 실행 ...(2)
    startTransition(async () => {
      const result = await fetchHeavyData();
      setData(result);
    });
  };

  return (
    <div>
      <button onClick={loadData}>
        {isPending ? "読み込み中" : "データ取得"}
      </button>
      {data && <HeavyComponent data={data} />}
    </div>
  );
}
```

(1) isLoading이 필요없고, JSX의 삼항연산자 조건식도 간결해졌다.<br>
(2) startTransition에 바로 비동기 함수를 전달할 수 있고, 데이터 패칭 또한 Transition 내부에서 await를 사용하여 실행할 수 있기 때문에 then()을 사용하지 않아도 된다.

## 2. Form Action

Action은 비동기 처리를 더 쉽게 관리할 수 있도록 하는 메커니즘이다.<br>
React공식문서에서는 "비동기 Transition을 사용하는 함수를 Action이라고 부르기로 한다"고 설명하고 있다.

기존 React에서 데이터 전송이나 비동기처리 상태를 관리할 때는 각 컴포넌트의 내부에서 개별적으로 처리해야 했기 때문에 코드가 복잡해지는 경우가 많았지만 Action을 사용하면 비동기처리 상태, 에러핸들링, 성공/실패의 피드백 표시 등을 더욱 간편하게 관리할 수 있다.

### 1) 기존 React의 Form관리

##### 코드３：React를 사용한 심플한 데이터 업데이트 구현 예시

```typescript
function UpdateTitle({ Value, onUpdate }) {
  const [error, setError] = useState(null); // 에러 정보
  const [isPending, setIsPending] = useState(false); // 대기중 상태값

  const handleSubmit = async (event) => {
    event.preventDefault();
    // 대기 중 상태로 변경
    setIsPending(true);
    // form 입력값을 반영하여 업데이트 실행
    const title = event.target.title.value;
    const { data, error } = await updateTitle(title);
    //　대기 중 상태 해제
    setIsPending(false);
    if (error) {
      // 에러일 경우, 에러정보 표시
      setError(error);
      return;
    }
    onUpdate(data);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="title" defaultValue={value} disabled={isPending} />
      <button type="submit" disabled={isPending}>
        更新
      </button>
      {error && <p>{error}</p>}
      <div>value: {value}</div>
    </form>
  );
}
```

### 2) React19에서의 Form관리

useActionState는 Action상태를 관리하는 새로운 hook이다. React19의 새로운 API중에서도 특히 활용도가 높은 핵심 기능이다.

또한 React19에서는 form에 Action을 직접 전달할 수 있어 훨씬 더 간결하게 제어할 수 있다.<br>
React 공식 문서에서도 useActionState는 form Action과 함께 사용하는 것을 전제로 하고 있다.

##### 코드4：useActionState을 사용한 데이터 갱신처리

```typescript
function UpdateTitle({ value, onUpdate }) {
  // useActionState의 반환값은 state, action, 대기 중 상태 ...(1)
  const [error, submitAction, isPending] = useActionState(
    // 비동기 함수를 action에 전달 ...(2)
    async (previousState, formData) => {
      // 첫번째 인자：변경 전 state, 두번째 인자：｀FormData｀　...(3)
      const value = formData.get("title");
      const { data, error } = await updateTitle(value);
      if (!error) {
        onUpdate(data);
      }
      return error;
    },
    null // state의 초기값
  );

  return (
    // form action에 Action을 직접 전달할 수 있음 ...(4)
    <form action={submitAction}>
      <input name="title" defaultValue={value} disabled={isPending} />
      <button type="submit" disabled={isPending}>
        更新
      </button>
      {error && <p>{error}</p>}
      <div>value: {value}</div>
    </form>
  );
}
```

동작 자체는 useTransition을 사용할 때와 비교하면 크게 다르진 않다.

#### (1) useActionState 반환값 - state, action, isPending

지금까지 관리했던 error state와 대기중 상태(isPending) 모두 useActionState hook에서 가져오고 있다.<br>
여기서 useActionState의 두번째 반환값인 Action 내부에서 실행되는 state업데이트는 우선순위가 낮은 업데이트로 간주된다.<br>
또한 여기서 관리하는 state는 useState처럼 객체 등과 같은 값을 사용할 수도 있다.

#### (2) useActionState의 인자 - 비동기 함수, state초기값

첫번째 인자는 로직을 실행하는 비동기 함수이다.<br>
useTransition와 마찬가지로 비동기 함수를 전달하면 Promise가 완료될 때까지 isPending이 true로 유지되기 때문에 전송중 상태를 간결하게 관리할 수 있다.<br>
두번째 인자에는 useState처럼 state초기값을 지정한다.

#### (3) Action 비동기 함수의 인자 - 변경 전 state, FormData

첫번째 인자는 변경 전 state로 state값을 이전 값을 바탕으로 변경해야하는 경우에 사용할 수 있다.<br>
두번째 인자는 form태그 Action에서 전달된 Web API FormData객체이므로 DOM요소 값을 직접 참조할 필요가 없다.

useActionState hook에서 반환된 Action을 form태그 action속성에 직접 전달하는 것이 form Action기능의 핵심이다.

##### 출처: https://codezine.jp/article/detail/20822
