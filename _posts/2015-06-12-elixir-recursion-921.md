---
author: jeidee
categories:
- elixir
date: '2015-06-12'
guid: https://erlnote.wordpress.com/?p=459
id: 459
permalink: /2015/06/12/elixir-recursion-921/
tags:
- tutorial
title: Elixir Recursion (#9/21)
url: /2015/06/12/elixir-recursion-921
---

> 본 문서는 elixir-lang.org의 [Getting Started](http://elixir-lang.org/getting-started/introduction.html)문서의 한글 번역본입니다.
    
> 자세한 내용은 원문을 참조해 주세요. 

## Loops through recursion

불변성때문에, elixir에서 loop는(모든 함수형 프로그래밍 언어에서 마찬가지로) 다른 언어들과 다르게 작성한다. 예를 들어, c 언어 같은 경우에는 다음과 같을 것이다.

```
  
for(i = 0; i < array.length; i++) {
    
array[i] = array[i] * 2
  
}
  
```

위의 예제가 보여주는 바는, i와 array 둘 모두 값이 변경되는 가변성이다. 가변성은 elixir에서 가능하지 않다. 대신, 함수형 언어는 재귀(recursion)에 의존한다: 함수는 조건이 지속되는 상황에서 종료되는 상황에 이를때까지 재귀적으로 호출된다. 이 처리 과정에서 어떤 데이터도 변경되지 않는다. 임의의 횟수동안 문자열을 출력하는 다음 예제를 살펴보자.

```
  
defmodule Recursion do
    
def print\_multiple\_times(msg, n) when n <= 1 do
      
IO.puts msg
    
end

def print\_multiple\_times(msg, n) do
      
IO.puts msg
      
print\_multiple\_times(msg, n &#8211; 1)
    
end
  
end

Recursion.print\_multiple\_times("Hello!", 3)
  
\# Hello!
  
\# Hello!
  
\# Hello!

```

case와 유사하게 함수는 복수개의 절을 갖고 있다. 특정 절에 일치(매치)하고 해당 절의 가드가 true로 평가될 때에만, 해당 절이 실행된다.

print\_multiple\_times/2 함수가 처음 호출될 때 매개변수 n은 3의 값을 갖는다.

첫 번째 절의 가드는 입력 매개변수인 n이 1과 같거나 작은 경우에만 수행되도록 제한하고 있다. 따라서 초기 호출시 n은 3이므로 이 경우에 해당되지 않으므로 다음 함수 절로 처리가 넘겨진다.

두 번째 절의 정의는 패턴이 일치(매개변수의 개수가 동일)하고 가드가 없기 때문에 실행된다. 먼저 msg를 출력하고 재귀적으로 n의 값을 1 감소(n &#8211; 1, n == 2) 시키고 호출한다.

두 번째 함수 호출에서 n(2)의 값은 여전히 1보다 크기때문에 두 번째 함수 절이 실행되며, 마지막 호출에서 비로소 n(1)이 첫 번째 함수 절의 가드 조건에 부합하므로 msg를 출력하고 재귀를 종료한다.

## Reduce and map algorithms

자, 이제 숫자 리스트의 합계를 구하기 위해 재귀의 힘을 어떻게 사용하는지 살펴보도록 하자.

```
  
defmodule Math do
    
def sum_list([head|tail], accumulator) do
      
sum_list(tail, head + accumulator)
    
end

def sum_list([], accumulator) do
      
accumulator
    
end
  
end

IO.puts Math.sum_list([1, 2, 3], 0) #=> 6
  
```

먼저 리스트 [1, 2, 3]과 초기 누적값 0을 매개변수로 sum_list/2 함수를 호출한다. 우리는 정의된 함수 절과 일치하는 하나를 찾을 때까지 패턴 매치를 수행한다. 이 경우에는, 리스트 [1, 2, 3]이 [head | tail]과 일치하므로, head에 1, tail에 [2, 3], accumulator에 0이 바인드 된다.

그 후, head에 accumulator를 더한 후(1 + 0) sum_list를 재귀적으로 호출하는데, 이 때 tail과 head + accumulator를 매개변수로 전달하게 된다. tail은 다시 한 번 [head | tail]과 일치하고 첫 번째 매개변수인 리스트가 빈 리스트([])가 될 때까지 반복하게 된다.

```
  
sum_list [1, 2, 3], 0
  
sum_list [2, 3], 1
  
sum_list [3], 3
  
sum_list [], 6
  
```

리스트가 빈 리스트가 되면, 마지막 절에 일치하게 되고 최종 결과인 6을 반환하게 된다.

리스트를 가져와 하나의 값으로 감소시키는 처리를 reduce algorithm이라고 하며 함수형 프로그래밍의 핵심이다.

만약 우리가 리스트의 각 값을 두 배로 만들길 원한다면?

```
  
defmodule Math do
    
def double_each([head|tail]) do
      
[head * 2|double_each(tail)]
    
end

def double_each([]) do
      
[]
    
end
  
end
  
```

```
  
iex math.exs
  
```

```
  
iex> Math.double_each([1, 2, 3]) #=> [2, 4, 6]
  
```

위의 예는 리스트의 각 요소를 탐색하고, 값을 두배로 만들어 새로운 리스트를 반환하는 재귀함수를 보여준다. 리스트를 가져와 각 요소별로 맵핑하는(mapping over) 처리를 map algorithm이라고 한다.

재귀와 [tail cal &#8211; 스택을 사용하지 않는 최적화된 재귀호출, 함수의 중간에서 재귀호출이 일어나선 안된다 &#8211;](http://en.wikipedia.org/wiki/Tail_call) 최적화는 elixir에서 중요한 부분이고 loop를 생성할 때 일반적으로 사용된다. 하지만, elixir에서 프로그래밍할 때 리스트를 처리하기 위해 위와 같은 재귀호출을 사용하는 것은 드물 것이다.

다음 챕터에서 살펴볼 Enum 모듈은 리스트를 편리하게 처리할 수 있는 많은 기능을 제공하고 있다. 일례로, 위의 예제는 다음과 같이 재작성할 수 있다.

```
  
iex> Enum.reduce([1, 2, 3], 0, fn(x, acc) -> x + acc end)
  
6
  
iex> Enum.map([1, 2, 3], fn(x) -> x * 2 end)
  
[2, 4, 6]
  
```

또는 캡춰 구문을 사용할 수도 있다.

```
  
iex> Enum.reduce([1, 2, 3], 0, &+/2)
  
6
  
iex> Enum.map([1, 2, 3], &(&1 * 2))
  
[2, 4, 6]
  
```

## 참고

  * [Elixir Recursion](http://elixir-lang.org/getting-started/recursion.html)