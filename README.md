# 📌 GitHub Repository Guide

## 🚀 작성 방법

이 저장소는 팀원들이 발표한 주제를 효율적으로 정리하고 공유하기 위한 공간입니다. 아래 가이드라인을 참고하여 내용을 작성해 주세요.

### 📂 폴더 구조

1. **큰 주제별 폴더 생성**

   - 예시: `JavaScript/`, `React/`, `CS/` 등
   - 관련된 주제끼리 한 폴더에 정리해 주세요.

2. **발표 주제별 파일 생성**

   - 폴더 내에 해당 주제의 내용을 정리한 Markdown 파일 (`.md`)을 생성합니다.
   - 파일명은 발표 날짜\_발표 주제\_본인 식별로 설정합니다.

   ✅ 예시:

   ```
   📂 React
   ├── 20250406_useState(강인권).md
   ├── 20250406_useEffect_Chuck.md
   📂 CS
   ├── 20250406_Operating_System(강인권).md
   ├── 20250406_Data_Structure(Chuck).md
   ```

### 📝 Markdown 작성 가이드

각 문서 작성 시 아래의 형식을 참고하면 통일성을 유지할 수 있습니다.

#### 1️⃣ 제목 작성

```md
# 📌 useState란?
```

#### 2️⃣ 개념 정리

```md
## 🧐 useState란?

React에서 상태 관리를 위해 사용하는 Hook 중 하나로, 컴포넌트 내에서 상태를 선언하고 변경할 수 있습니다.
```

#### 3️⃣ 코드 예시

````md
## 💻 코드 예시

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>현재 카운트: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```
````

#### 4️⃣ 참고 자료

```md
## 🔗 참고 자료

- [React 공식 문서 - useState](https://react.dev/reference/react/useState)
```

---

모두가 쉽게 찾아보고 활용할 수 있도록 일관된 스타일로 작성해 주세요! 🚀
