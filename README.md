# react_hooks_nomad

- Nomad - 실전형 리액트 Hooks 10개
- https://nomadcoders.co/react-hooks-introduction

## 0. Introduction

- 레퍼지토리 / 리액트 환경 세팅
  1. npx create-react-app .
  2. 불필요한 파일 삭제

## 1. USESTATE

- 함수형, class형 차이점 코드 비교 (코드 생략)

  - 함수형이 더욱 코드가 짧고 이해하기 쉬움

  ```js
  // 함수형 - simple
  const App = () => {
    const [item, setItem] = useState(1);
    const incrementItem = () => setItem(item + 1);
    const decrementItem = () => setItem(item - 1);
    return (
      <div>
        <h1>Hello {item}</h1>
        <h2>Start editing to see some magic happen!</h2>
        <button onClick={incrementItem}>Increment</button>
        <button onClick={decrementItem}>Decrement</button>
      </div>
    );
  };
  ```

- Hooks
  - 다른 함수(function)에서 이벤트를 처리 할 수 있음
  - 이벤트 처리를 분리된 파일, 다른 entity에 연결해서 처리 할 수 있다.

```js
// useInput (1) : useInput 함수를 만들고, input 태그에 활용하기
const useInput = (initialValue) => {
  const [value, setValue] = useState(initialValue);
  const onChange = (event) => {
    console.log(event.target);
  };
  return { value, onChange };
};

const App = () => {
  const name = useInput("Mr.");
  return (
    <div>
      <h1>Hello</h1>
      <input placeholder="Name" {...name}></input>
    </div>
  );
};
```

```js
// useInput (2) : validator 만들기
const useInput = (initialValue, validator) => {
  const [value, setValue] = useState(initialValue);
  const onChange = (event) => {
    const {
      target: { value },
    } = event;
    let willUpdate = true;
    if (typeof validator === "function") {
      willUpdate = validator(value);
    }
    if (willUpdate) {
      setValue(value);
    }
  };
  return { value, onChange };
};

const App = () => {
  const inputValid = (value) => !value.includes("@");
  const name = useInput("Mr.", inputValid);
  return (
    <div>
      <h1>Hello</h1>
      <input placeholder="Name" {...name}></input>
    </div>
  );
};
```

```js
// useTabs
const content = [
  {
    tab: "Section 1",
    content: "I'm the content of the Section 1",
  },
  {
    tab: "Section 2",
    content: "I'm the content of the Section 2",
  },
];

const useTabs = (initialTab, allTabs) => {
  const [currentIndex, setCurrentIndex] = useState(initialTab);
  if (!allTabs || !Array.isArray(allTabs)) return;
  return {
    currentItem: allTabs[currentIndex],
    changeItem: setCurrentIndex,
  };
};

const App = () => {
  const { currentItem, changeItem } = useTabs(0, content);
  return (
    <div>
      {content.map((section, index) => (
        <button onClick={() => changeItem(index)}>{section.tab}</button>
      ))}
      <div>{currentItem.content}</div>
    </div>
  );
};
```
