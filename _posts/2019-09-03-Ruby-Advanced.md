---
title: 루비 고급
categories: 
  - ruby
---


블록
---
```
def callBlock
  yield
  yield "New World"
end

callBlock { puts "Hello World" }
callBlock { |name| puts "Hello #{name}" }
callBlock do
  puts "Hello World"
end
```
* 블록은 Lambda 개념과 같음
* 'yield'를 통해 블록을 실행
* '|...|' 통해 파라메터를 받음

참고
---
* <http://ruby-doc.com/docs/ProgrammingRuby/>