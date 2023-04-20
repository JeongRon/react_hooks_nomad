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

## 2. USEEFFECT

- useEffect란?

  - ComponentDidMount, ComponentWillUnMount, ComponentDidUpdate 역할
  - 1번째 인자 : function으로서의 effect
    - componentDidMount()와 기능이 비슷함
  - 2번쨰 인자 : dependency
    - useEffect()가 dependency 리스트에 있는 값이 변할 때만 실행되게 함
    - componentDidUpdate()와 기능이 비슷함

  ```js
  const App = () => {
    const sayHello = () => console.log("Hello");
    const [number, setNumber] = useState(0);
    const [aNumber, setAnumber] = useState(0);
    useEffect(sayHello, [number]);
    return (
      <div>
        <div>Hi</div>
        <button onClick={() => setNumber(number + 1)}>{number}</button>
        <button onClick={() => setAnumber(aNumber + 1)}>{aNumber}</button>
      </div>
    );
  };
  ```

- useTitle

  ```js
  // useTitle : ComponentDidMount, ComponentDidUpdate
  const useTitle = (initialTitle) => {
    const [title, setTitle] = useState(initialTitle);
    const updateTitle = () => {
      const htmlTitle = document.querySelector("title");
      htmlTitle.innerText = title;
    };
    useEffect(updateTitle, [title]);
    return setTitle;
  };

  const App = () => {
    const titleUpdater = useTitle("Loading ...");
    setTimeout(() => titleUpdater("FriYay"), 3000);
    return (
      <div>
        <div>Hi</div>
      </div>
    );
  };
  ```

- reference 란?

  - component의 특정 부분을 선택할 수 있는 방법
  - documen.getElementById()와 비슷

  ```js
  const App = () => {
    const potato = useRef();
    setTimeout(() => potato.current.focus(), 3000);
    return (
      <div>
        <div>Hi</div>
        <input ref={potato} placeholder="la"></input>
      </div>
    );
  };
  ```

- useClick

  - reference는 component의 어떤 부분을 선택할 수 있는 방법이다.
  - 모든 component는 reference prop을 가지고 있다.
  - useRef()는 document.getElementById() 와 같은 기능을 한다.
  - htmlTag에 ref={이름} 와 같이 사용한다.
  - reference는 {current: HTMLHeadingElement} 의 형식으로 값을 반환한다.
  - useEffect에서 return한 함수는 componentWillUnmount 때 호출된다.
  - useEffect에서 return한 함수를 cleanup function(클린업 함수)라고 부른다.

  ```js
  // useClick
  const useClick = (onClick) => {
    const ref = useRef();
    useEffect(() => {
      const element = ref;
      if (element.current) {
        element.current.addEventListener("click", onClick);
      }
      return () => {
        if (element.current) {
          element.current.removeEventListener("click", onClick);
        }
      };
    });
    if (typeof onClick !== "object") return;
    return ref;
  };

  const App = () => {
    const sayHello = () => console.log("say Hello");
    const title = useClick(sayHello);
    return (
      <div>
        <h1 ref={title}>Hi, there!</h1>
      </div>
    );
  };
  ```

- useConfirm

  ```js
  // useConfirm
  const useConfirm = (message = "", onConfirm, onCancel) => {
    if (typeof onConfirm !== "function") return;
    const confirmAction = () => {
      if (window.confirm(message)) {
        onConfirm();
      } else {
        try {
          onCancel();
        } catch (error) {
          return;
        }
      }
    };
    return confirmAction;
  };

  const App = () => {
    const deleteWorld = () => console.log("Deleting the world");
    const abort = () => console.log("Aborted");
    const confirmDelete = useConfirm("Are you sure", deleteWorld, abort);
    return (
      <div>
        <button onClick={confirmDelete}>Delete the world</button>
      </div>
    );
  };
  ```

- usePreventLeave

  ```js
  const usePreventLeave = () => {
    const listener = (event) => {
      event.preventDefault();
      event.returnValue = "";
    };
    const enablePrevent = () =>
      window.addEventListener("beforeunload", listener);
    const disablePrevent = () =>
      window.removeEventListener("beforeunload", listener);
    return { enablePrevent, disablePrevent };
  };

  const App = () => {
    const { enablePrevent, disablePrevent } = usePreventLeave();
    return (
      <div>
        <button onClick={enablePrevent}>Protect</button>
        <button onClick={disablePrevent}>UnProtect</button>
      </div>
    );
  };
  ```
