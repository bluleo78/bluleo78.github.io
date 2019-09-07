---
title: React 고급 가이드
categories: 
  - React
---


비제어 컴포넌트
---
사용자 입력이 존재하는 <form> 내에 <input> DOM 의 값을 리액트 레퍼런스를 이용하여 직접 접근하는 컴포넌트

다음 예제에서는 <input> DOM의 값에 접근하고 있다.
```
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
    this.input = React.createRef();
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.current.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={this.input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```
* 생성자에서 'this.input' 에 레퍼런스를 할당하
* 렌더링에서 <input> DOM 에 'this.input' 레퍼런스를 설정
* 이벤트 핸들러에서 'this.input' 을 사용


참고
---
* </>
