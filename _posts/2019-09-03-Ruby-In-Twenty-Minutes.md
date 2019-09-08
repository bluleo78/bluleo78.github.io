---
title: 루비 20분 가이드 요약
categories: 
  - ruby
---

루비쉘(irb, interactive ruby shell)
---
> irb

헬로우월드
---
```
puts "Hello World"
```

함수
---
```
def h
  puts "Hello World"
end
def h1(name)
  puts "Hello #{name}"
end
```

배열
---
```
names = []
names = Array.new
names = ["a", "b", "c"]
names                     => ["a", "b", "c"]
names.join(",")           => "a,b,c"
names[0]                  => "a"
```

해시(Hash)
---
```
h = {
  'a' => 'A',
  'b' => 'B'
}
h['a']                    => "A"
```


제어
---
```
if @names.nil?
  ...
elsif @names.respond_to?("join")
  ...
else
  ...
end

while a < 100
  ...
end

a = a * 2 while a < 100
```


반복
---
```
names.each do |name|
  puts "Hello #{name}"
end
```
* 참고로 each 다음에는 블록이 넘겨지고, each 내부에서 yield 를 실행하는 구조



모듈
---
```
Math.sqrt(a+b)
```
* Math 모듈의 sqrt 메소드 호출


클래스
---
```
class Greeter
  def initialize(name = "World")
    @name = name
  end
  def sayHi()
    puts "Hello #{@name}
  end
end

g = Greeter.new("Pat")
g.sayHi()
```
* 'initialize' 생성자
* '@' 멤버변수
* 'new(...)' 객체 생성


클래스 - 멤버 접근 제어
---
```
class Greeter
  attr_accessor :name
  def initialize(name = "World")
    @name = name
  end
  def sayHi()
    puts "Hello #{@name}"
  end
end

g = Greeter.new("Pat")
g.name
g.name = "Pet"
g.sayHi()
```

NULL 체크
---
```
name.nil?
```

멤버 존재 여부 체크
---
```
g.respond_to?("sayHi")
```



__FILE__ 변수
---
현재 파일 이름
```
if __FILE__ === $0

end
```
* 참고로 '$0' 시작 루비 파일


주석 #
---
```
# say hi
```
