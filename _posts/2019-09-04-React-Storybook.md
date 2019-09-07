---
title: React - Storybook 연동
---


스토리북 초기화
---
### 스토리북 추가
'create-react-app' 으로 생성된 프로젝트에 스토리북을 추가할 수 있음
```
cd {프로젝트디렉토리}
npx -p @storybook/cli sb init
```

### 스토리북 동작 확인
```
npm run storybook
```

### 스토리북 설정
'.storybook/config.js' 파일에 다음 설정을 추가한다.

CSS 스타일 파일 임포트
```
import '../src/index.css';
```
* 그렇지 않으면 스토리북 페이지에서 스타일이 하나도 적용 안됨

스토리 로딩 방법 변경
```
const req = require.context('../src/stories', true, /\.stories.js$/);

function loadStories() {
  req.keys().forEach(filename => req(filename));
}
```
* 이 설정을 하지 않으면 기본적으로 'src/stories' 아래 'index.js' 파일을 로딩함
* 'src/stories' 아래 'stories.js'로 끝나는 모든 파일에 포함된 스토리를 로딩하도록 변경

기본으로 제공되는 스토리 예제를 스토리북에서 확인하고 싶으면 아래와 같이 이름을 변경해줘야 함
```
cd src/stories
mv index.js default.stories.js
```


스토리 추가
---
스토리북에 컴포넌트를 추가하고 그 스토리를 추가할 수 있다

스토리를 추가하기 위해서는 'src/stories' 아래 아래와 같은 파일을 추가하면 된다.
```
Board.stories.js
---
import React from 'react';

import { storiesOf } from '@storybook/react';
import { action } from '@storybook/addon-actions';

import Board from '../components/Board';

storiesOf('Board', module)
  .add('initial',() => <Board squares={Array(9).fill(null)} onClick={action('onClick')}/>);

```
* storiesOf('컴포넌트이름', module).add('스토리이름', 스토리에 해당하는 컴포넌트 생성 함수)...
* 생성 함수에서 컴포넌트를 생성할 때, 스토리에 맞는 props 를 넘기면 됨
* 단, 이벤트 핸들러인 경우 action, link 등 스토리북에서 제공하는 함수를 사용할 수 있다
* action 은 스토리북 아래 액션 탭에 표시된다
* linkTo 는 다른 스토리로 이동한다


스토리에 대한 테스트 자동화
---
작성된 스토리에 대해서 스냅샷 기반 테스트를 수행할 수 있다

### 테스트 파일 추가
```
// src/storybook.test.js

import initStoryshots from '@storybook/addon-storyshots';
initStoryshots();
```

### 바벨 메크로 설정 추가
```
// .babelrc

{
  "plugins": ["macros"]
}
```

### 스토리북 설정 변경
바벨 매크로 임포트
```
import requireContext from 'require-context.macro';
```
* 그렇지 않으면 스토리북 페이지에서 스타일이 하나도 적용 안됨

스토리 로딩 방법 변경
```
const req = requireContext('../src/stories', true, /\.stories\.js$/);
```


참고
---
* <https://create-react-app.dev/docs/developing-components-in-isolation>
* <https://www.learnstorybook.com/react/en/simple-component>
