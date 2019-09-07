---
title: React 튜토리얼 분석
categories: 
  - React
---


준비 작업
---
우선 기본 리액프 프로젝트 앱을 생성한다.

```
npx create-react-app my-app

```

그리고 소스 디렉토리를 모두 지운다.
```
cd my-app
rm -rf *
```

여기에 다음 2개의 소스파일을 추가한다.
* index.css
* index.js


board.js
---
```javascript
function Square(props) {
  return (
    <button className="square" onClick={props.onClick}>
      {props.value}
    </button>
  );
}


class Board extends React.Component {
  renderSquare(i) {
    return (
      <Square
        value={this.props.squares[i]}
        onClick={() => this.props.onClick(i)}
      />
    );
  }

  render() {
    return (
      <div>
        <div className="board-row">
          {this.renderSquare(0)}
          {this.renderSquare(1)}
          {this.renderSquare(2)}
        </div>
        <div className="board-row">
          {this.renderSquare(3)}
          {this.renderSquare(4)}
          {this.renderSquare(5)}
        </div>
        <div className="board-row">
          {this.renderSquare(6)}
          {this.renderSquare(7)}
          {this.renderSquare(8)}
        </div>
      </div>
    );
  }
}

export default Board;
```
* 그게 Board 아래 9개의 Square 로 구성되어 있다.
* Squire 
 * 함수형 컴포넌트 - 리액트 컴포넌트의 render 에 해당하며 props 를 받음
 * props 는 value, onClick
 * button 태그 사용
* Board
 * React.Component 상속
 * 생성자는 생략 - 단순 super(props) 만 부르기 때문
 * render 에서는 9번의 renderSquire 를 호출하여 Squire 를 생성
 * renderSquire 에서는 Squire 컴포넌트를 생성하고 
  * props 로 현재 상태(this.props.squires[i])와 이벤트 핸들러(() => this.props.onClick(i))를 넘김
 

game.js
---
```javascript
import Board from './Board.js'

class Game extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      history: [
        {
          squares: Array(9).fill(null)
        }
      ],
      stepNumber: 0,
      xIsNext: true
    };
  }

  handleClick(i) {
    const history = this.state.history.slice(0, this.state.stepNumber + 1);
    const current = history[history.length - 1];
    const squares = current.squares.slice();
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    squares[i] = this.state.xIsNext ? "X" : "O";
    this.setState({
      history: history.concat([
        {
          squares: squares
        }
      ]),
      stepNumber: history.length,
      xIsNext: !this.state.xIsNext
    });
  }

  jumpTo(step) {
    this.setState({
      stepNumber: step,
      xIsNext: (step % 2) === 0
    });
  }

  render() {
    const history = this.state.history;
    const current = history[this.state.stepNumber];
    const winner = calculateWinner(current.squares);

    const moves = history.map((step, move) => {
      const desc = move ?
        'Go to move #' + move :
        'Go to game start';
      return (
        <li key={move}>
          <button onClick={() => this.jumpTo(move)}>{desc}</button>
        </li>
      );
    });

    let status;
    if (winner) {
      status = "Winner: " + winner;
    } else {
      status = "Next player: " + (this.state.xIsNext ? "X" : "O");
    }

    return (
      <div className="game">
        <div className="game-board">
          <Board
            squares={current.squares}
            onClick={i => this.handleClick(i)}
          />
        </div>
        <div className="game-info">
          <div>{status}</div>
          <ol>{moves}</ol>
        </div>
      </div>
    );
  }
}

// ========================================

ReactDOM.render(<Game />, document.getElementById("root"));

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6]
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```
* Game 에서는 Board 를 그리고 클릭 이벤트를 핸들한다.
* constructor
 * super(props) 호출은 기본
 * 로컬 스테이트(this.state)는 다음으로 이루어진다.
  * history - 보드 상태 히스토리
  * stepNumber - 현재 단계
  * xIsNext - 다음 차례가 X 인지 여부
* render
 * 현재 보드 상태를 구함 => current
 * 현재 상태에서 누가 이겼는지 구함 => winner
 * 지금까지 게임 이동 히스토리 구함 => moves
  * key 정보를 설정 중요!
  * 리스트 아이템이 클릭되면 jumpTo 가 호출되게
 * 게임 상태 정보 구함 => status
 * Board 컴포넌트와 게임 정보를 표시하는 부분으로 구분된다
 * Board 컴포넌트
  * props 로 현재 보드 상태(current.squires)와 클릭 이벤트 핸들러(i => handleClick(i)) 넘김
 * 게임 정보
  * 게임 상태와 게임 이동 히스토리 표시 
* handleClick
 * 이미 승자가 있거나, 이미 O/X 선택이 된 경우는 그냥 리턴
 * 보드 상태를 변경하고
 * 게임의 로컬 스테이트를 업데이트
  * 히스토리 추가, 단계 추가, 다른 차례 변경
* jumpTo
 * 현재 스탭과 다음 차례 변경 후 로컬 스테이트 업데이트


index.css
---
```css
body {
  font: 14px "Century Gothic", Futura, sans-serif;
  margin: 20px;
}

ol, ul {
  padding-left: 30px;
}

.board-row:after {
  clear: both;
  content: "";
  display: table;
}

.status {
  margin-bottom: 10px;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.square:focus {
  outline: none;
}

.kbd-navigation .square:focus {
  background: #ddd;
}

.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

실행
---
```
npm start
```


참고
---
* <https://reactjs-kr.firebaseapp.com/tutorial/tutorial.html>