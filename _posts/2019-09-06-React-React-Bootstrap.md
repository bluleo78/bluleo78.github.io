---
title: React - React-Bootstrap 기본 가이드
categories: 
  - React
---

설치
---
```
npm install react-bootstrap bootstrap
```


컴포넌트 임포트
---
```
import Button from 'react-bootstrap/Button';

// or less ideally
import { Button } from 'react-bootstrap';
```

컴포넌트 - 버튼(Button)
---
```
<Button onClick = {() => this.props.history.push('/game')}>시작</Button>
```


컴포넌트 - 테이블(Table)
---
```
<Table style = {{marginTop: 20}} size = "sm">
  <thead>
    <tr>
      <th>Name</th>
      <th>Price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>아이폰</td>
      <td>100$</td>
    </tr>
  </tbody>
</Table>
```


컴포넌트 - 네비게이션(Nav)
---
```
<Nav
  activeKey = "/"
  onSelect = {(key) => this.props.history.push(key)} 
>
  <Nav.Item>
    <Nav.Link eventKey = "/">Home</Nav.Link>
  </Nav.Item>
  <Nav.Item>
    <Nav.Link eventKey = "/product">Product</Nav.Link>
  </Nav.Item>
  <Nav.Item>
    <Nav.Link eventKey = "/game">Game</Nav.Link>
  </Nav.Item>
</Nav>
```
* eventKey 대신에 href 를 사용하면 해당 URL로 전환이 이루어져 화면이 다시 그려짐.


컴포넌트 - 레이아웃
---
```
<Container>
  <Row>
    <Col>1 of 2</Col>
    <Col>2 of 2</Col>
  </Row>
</Container>
```

컬럼의 사이즈를 컬럼의 속성(lg, md, sm, xl, xs)을 지정하여 변경할 수 있다
```
<Container>
  <Row>
    <Col xl={2}>1 of 2</Col>
    <Col xl={1}>2 of 2</Col>
  </Row>
</Container>
```
* 첫번째 컬럼은 두번째 컬럼의 두 배


참고
---
* <https://react-bootstrap.github.io>