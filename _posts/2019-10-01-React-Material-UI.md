---
title: React Material UI
categories: 
  - CSS
---

'Material UI' 는 React 에서 가장 많이 사용하는 UI


설치
---
```
npm install @material-ui/core
```

기본 적용
---
### CssBaseline
브라우져/디바이스 간 불일치 문제를 어느정도 해결해준다고 함.
```
ReactDOM.render((
  <React.Fragment>
    <CssBaseline />
    <App />
  </React.Fragment>
), document.getElementById('root'));
```

컴포넌트 스타일링
---
withStyles 이라는 HOC 컴포넌트를 제공

```
import { withStyles } from '@material-ui/core/styles';

const styles = (theme) => ({
  button: {
    backgroundColor: theme.palette.common.white,
  },
});

class MyComponent extends React.Component {

}

export defaults withStyles(styles)(MyComponent);
```

Typography - 타이포그래피
---
HTML 텍스트 태그를 직접 사용하지 않고 Typography 로 감싸서 사용.
텍스트 관련 일관성 때문인 듯.
```
        <Typography component="h1" variant="h5">
          Sign in
        </Typography>
```

아이콘
---
아이콘을 사용하기 위하서는 아래 모듈 설치 필요
```
npm install @material-ui/icons
```

아래처럼 임포트로 가져와서 사용
```
import LockOutlinedIcon from '@material-ui/icons/LockOutlined';

...
...<LockOutlinedIcon />
```

아이콘은 https://material-ui.com/components/material-icons/ 에서 검색할 수 있음



공용 props
---

### fullWidth 너비 최대
### color 색깔
### variant 컴포넌트 모양
* contained
* outlined
### required 필수


Container - 컨테이너
---


            variant="contained"
            color="primary"
            className={classes.submit}