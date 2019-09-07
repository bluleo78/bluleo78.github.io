---
title: React 기본 가이드
categories: 
  - React
---


Hello World
---
```
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```
* 도큐먼트 DOM 을 치환하는 방식인 듯


JSX
---
리액트에서 HTML이나 리액트 컴포넌트를 기술하는 방식
실제로는 React.createElement 함수로 변환됨.
```
const element = <h1>Hello World</h1>;
```

XSS 공격도 방지한다고 함

### JSX 속성 정의
따옴표를 쓰거나 중괄호를 쓰거나
```
const element0 = <div tabIndex="0"></div>;
const element1 = <img src={user.avatarUrl}></img>;
```

추가로 'class' 대신 'className' 사용하고 'tabindex' 대신 'tabIndex'를 사용해야 함.


리액트 렌더링
---
리액트는 변경된 부분만 다시 렌더링합니다.


컴포넌트
---

### 함수 컴포넌트와 클래스 컴포넌트
함수 컴포넌트
```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```
* 간단한 컴포넌트인 경우에 사용함

클래스 컴포넌트
```
class Welcome extends React.Component {
  render() {
    return <h1>Hello World</h1>;
  }
}
```

### 컴포넌트 props 
컴포넌트는 전달된 props를 이용한다.

함수 컴포넌트는 함수 아귀먼트로 props 를 전달받는다.
```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
```

클래스 컴포넌트는 클래서 생성자 아귀먼트로 props 를 전달받는다.
생성자를 오버라이드하는 경우에는 반드시 아래처럼 super(props) 를 호출해야한다.
```
class Welcome extends React.Component {
  contructor(props) {
    super(props);
    this.state = {}
  }

  render() {
    return <h1>Hello World</h1>;
  }
}
```
컴포넌트에 props 를 전달하기 위해서는 컴포넌트의 속성에 기술해야 한다.
```
const element = <Welcome name="Sara" />;
```

props 는 상위 컴포넌트에서 하위 컴포넌트로 전달되고, 컴포넌트 내에서 props 를 수정해서는 안된다.


### 컴포넌트 그룹
여러 컴포넌트를 하나 묶기 위해서 '<div>' 태그를 사용한다.
```
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
```
* 요즘은 '<div>' 태그가 너무 많아서 '<>' 로만 기술하기도 한다.


상태(State)와 라이프사이클(Liftcycle)
---
리액트 컴포넌트는 로컬 상태(this.state)를 갖을 수 있음.
보통 생성자(constructor)에서 초기화시킴
```
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }
  ...
}
```

### 라이프사이클 메소드
* componentDidMount 컴포넌트가 DOM 에 렌더링 완료
* componentWillUnmount 컴포넌트가 언마운트되기 직전

### 상태 올바르게 사용하기
무조건 this.setState 메소드를 사용하여 업데이트
비동기적으로 상태가 업데이트됨

이전 상태를 참조하여 다음 상태를 계산하는 경우 this.setState 에서 함수 인자 사용
```
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

this.setState 는 앝은 병합(Shallow Merge)이므로 업데이트할 것만 기술해도 됨.


이벤트 처리
---

이벤트 핸들러에서 기본 이벤트를 방지하기 위해서 e.preventDefault() 호출해줘야 한다.
(일반 HTML 에서는 false 를 리턴하면 자동으로 해준다고 한다.)
```
function ActionLink() {
  function handleClick(e) {
    e.preventDefault();
    console.log('The link was clicked.');
  }

  return (
    <a href="#" onClick={handleClick}>
      Click me
    </a>
  );
}
```
* 위 경우 href="#" 으로 가는 기본 이벤트가 동작하지 않음

이벤트 핸들러 함수에서 this 를 사용하고 싶으면 생성자에서 bind 를 해줘야 한다.
```
class Toggle extends React.Component {
  constructor(props) {
    ...
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(state => ({
      isToggleOn: !state.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}> Button </button>
    );
  }
...
```

애로우를 써서 정의하면 bind 를 따로 정의할 필요가 없다.
```
class Toggle extends React.Component {
  handleClick() {
    this.setState(state => ({
      isToggleOn: !state.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={() => this.handleClick()}> Button </button>
    );
  }
```


조건부 렌더링
---

### 팁
* 리액트 엘리먼트를 변수에 저장하여 사용
* 중괄호 안에서 && 나 ? 연산자 사용

### 컴포넌트 렌더링 막기
* render 함수에서 null 을 리턴


리스트와 키(Key)
---
여러 개의 같은 컴포넌트에 대한 생성은 map 을 이용함.
```
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
)
```

그런데 위처럼 하면 키를 사용해야한다고 경고 발생

```
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```

키는 렌더링 시 기존 DOM 트리와 새로운 트리 사이에 변경 여부를 체크할 때 효휼적으로 비교할 수 있게 해 줌.
같은 컴포넌트 자식 노드들이 여러 개일 경우에 유용함.


폼(Form)
---
제어 컴포넌트(Controlled Component)는 <form> 안의 <input>, <textarea>, <select> 컴포넌트의 값을 로컬 상태로 관리할 수 있게 폼을 감싸고 있는 구조이다.


상태 끌어올리기
---
두개 이상의 컴포넌트가 같은 데이터를 공유하는 경우, 상위 컴포넌트에 해당 로컬 상태를 추가하고 하위 컴포넌트에 props 로 전달해주는 방식을 사용한다.


합성(Composition)과 상속(Inheritance)
---

### 컴포넌트에 다른 컴포넌트 담기
this.props.children 으로 하위 컴포넌트를 참조할 수 있음
```
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
```

props 로 컴포넌트를 넘길 수도 있다.
```
<SplitPane left={<Contacts />} right={<Chat />} />
```

### 합성
* 다른 컴포넌트를 이용하여 내 컴포넌트를 만드는 것

### 상속
* 의미 없음


참고
---
* </>
