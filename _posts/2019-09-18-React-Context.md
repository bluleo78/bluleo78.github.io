---
title: React Context
categories:
  - React
---

React 를 사용하면 하위 깊은 곳에 있는 컴포넌트까지 props 를 전달하는 경우가 있는데,
이를 React Context 를 이용해서 쉽게 전달하는 방법을 소개한다.

그런데, 가이드에는 React Context 를 사용하면 컴포넌트를 재사용성이 떨어지기 때문에 꼭 필요한 곳에만 쓰라고 나와 있다. 로그인한 유저 정보, 테마, 언어 정도로 예를 들고 있다.

## 예제부터 살펴보자

```
// React Context 를 기본값 'light' 로 생성
const ThemeContext = React.createContext('light');

class App extends React.Component {
  render() {
    return (
      // 컨텍스트 프로바이더로 컴포넌트를 감싸고, 컨텍스트 값을 'dark'로 넘겨줌
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {

  // 구독할 컨텍스트를 설정
  static contextType = ThemeContext;

  render() {
    // 컨텍스트 값을 this.context 를 이용하여 참조
    return <Button theme={this.context} />;
  }
}
```

## 흐름

### 컨텍스트 생성

'React.createContext' 를 이용하여 컨텍스트를 생성
여기서 기본값을 입력받는데, 이 값은 컨텍스트 값을 제공하는 컨텍스트 프로바이더를 찾지 못했을 때 사용되는 값이다.

### 컨텍스트 프로바이더 컴포넌트

컨텍스트 프로바이더 컴포넌트('Provider')는 대상 컴포넌트를 감싸고, 'value' props 의 컨텍스트 값을 설정한다.

```
<MyContext.Provider value={/* 어떤 값 */}>
  ...
</MyContext.Provider>
```

컨텍스트의 스코프 개념이 있어서 컨텍스트 프로바이더 컴포넌트로 감싼 하위 컴포넌트에서 컨텍스트를 구독했을 경우에만 컨텍스트 값을 참조할 수 있다. 다시 말하면 하위 컴포넌트는 컨텍스트 값을 참조할 때, 가장 가까운 상위의 프로바이더 컴포넌트가 제공하는 값을 참조한다.

### 컨텍스트 구독/참조

프로바이더 하위 컴포넌트들에서 컨텍스트 값이 참조하고자 하는 경우, 컨텍스트를 구독해야 한다.

컨텍스트 값 변화를 구독하기 위해서는 컨텍스트의 컨슈머(Consumer) 컴포넌트를 사용해야 한다.

```
function MyFunction(props) {
  return {
    <MyContext.Consumer>
      {context => {/* context 값을 이용한 렌더링 */}}
    </MyContext.Consumer>
  }
}
```

클래스 컴포넌트인 경우에는 클래스의 'static contextType' 에 컨텍스트를 설정해주고 'this.context' 를 이용해서 컨텍스트 값을 좀 더 쉽게 참조할 수 있다. 단 이 경우에는 단인 컨텍스트만 구독할 수 있다.

```
class MyClass extends React.Component {
  static contextType = MyContext;
  render() {
    /* this.context 값을 이용한 렌더링 */
  }
}
```

### 컨텍스트 값 업데이트

컨텍스트 값을 변경하기 위해서는 컨텍스트 프로바이더 컴포넌트를 다시 렌더링해줘야 한다.

프로바이더 컴포넌트를 일반적인 업데이트 흐름은 다음과 같다.

1. 프로바이더 컴포넌트가 포함하는 상위 컴포넌트에서 하위 컴포넌트에 로컬 상태를 변경하는 콜백 전달
2. 하위 컴포넌트에서 이벤트 발생 시 콜백이 실행되어 상위 컴포넌트의 로컬 상태가 변경
3. 상위 컴포넌트가 다시 렌더링되면서 변경된 상태를 프로바이더 컴포넌트의 props 로 설정하여 렌더링

```
class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      value: true,
    };
  }

  changeValue() {
    this.setState(state => !state));
  }

  render() {
    return (
      <MyContext.Provider value={this.state}>
        <MyContextButton onClick = {() => this.changeValue()}/>
      </MyContext.Provider>
    );
  }
}

function MyContextButton() {
  return (
    <MyContext.Consumer>
      {({value}) => (
        <button
          onClick={this.props.onClick}
        >
          {value}
        </button>
      )}
    </MyContext.Consumer>
  );
}
```

다른 업데이트 흐름은 컨텍스트 값에 콜백을 설정하고 하위 컴포넌트에서 컨텍스트에 설정된 콜백을 호출하는 방법이다.

1. 프로바이더 컴포넌트에 로컬 상태를 변경하는 콜백을 컨텍스트 값으로 전달
2. 하위 컴포넌트에서 이벤트 발생 시 컨텍스트 값의 콜백을 실행
3. 상위 컴포넌트의 로컬 상태가 변경
4. 상위 컴포넌트가 다시 렌더링되면서 변경된 상태를 프로바이더 컴포넌트의 props 로 설정하여 렌더링

```
class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      value: true,
      changeValue: () => this.changeValue(),
    };
  }

  changeValue() {
    this.setState(state => !state));
  }

  render() {
    return (
      <MyContext.Provider value={this.state}>
        <MyContextButton />
      </MyContext.Provider>
    );
  }
}

function MyContextButton() {
  return (
    <MyContext.Consumer>
      {({value, changeValuee}) => (
        <button
          onClick={changeValue}
        >
          {value}
        </button>
      )}
    </MyContext.Consumer>
  );
}
```

## 원리(추측)

컨텍스트를 생성할 때 내부적으로 publish/subscribe 관리할 수 있는 구조가 있다
프로바이더 컴포넌트가 렌더링될 때, 컨텍스트 객체에 publisher 로 등록된다.
컨슈머 컴포넌트가 렌더링될 때, 컨텍스트 객체에 상위 컴포넌트가 subscriber 로 등록된다.
프로바이더가 렌더링되면 이를 구독하는 subscriber 컴포넌트들은 다시 렌더링된다.
subscriber 컴포넌트의 컨슈머 컴포넌트가 렌더링될 때 하위 컴포넌트들은 컨텍스트 값을 사용하여 렌더링 한다.

클래스의 경우 contextType 이 정의된 경우 해당 프로바이더를 찾아서 this.context 에 프로바이더의 컨텍스트 값을 연결

## 참고

- <https://ko.reactjs.org/docs/context.html>
