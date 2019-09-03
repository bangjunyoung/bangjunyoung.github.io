---
layout: post
title: C# LINQ 스타일과 F# 함수형 스타일의 차이
subtitle: 비슷하면서 다른 두 언어의 함수형 프로그래밍 접근 방식
tags: [함수형 프로그래밍, C#, F#]
---

C#의 LINQ는 버전 3.0에 처음 도입된 기능으로, 그 이전까지 단지 **잘 베낀 자바**에 불과했던 C#을 일거에 함수형 프로그래밍 언어로 도약하게 만든 가히 혁명적인 시도라 부를만 하다. LINQ 이후의 C#은 이 언어가 C/C++에 기반을 두고 있는 게 맞나 싶을 정도로 코딩 스타일이 많이 달라졌다. LINQ를 이용하면 기존 명령형 프로그래밍으로 복잡하고 지저분하게 짜야만 했던 코드가 작성하기 쉽고 이해하기 쉬운 함수형 프로그래밍 스타일로 바뀐다.

이 글에서는 LINQ의 기본 아이디어를 알아보고 이것이 F#의 함수형 스타일과는 어떤 차이점과 유사점이 있는지 비교해 보도록 하겠다.

## 확장 메쏘드

사실 LINQ의 문법은 확장 메쏘드라는 아주 간단하고도 기발한 아이디어에서 출발하는데, 이 확장 메쏘드는 더 이상 메쏘드를 추가할 수 없는 기존 타입에 메쏘드가 추가된 것처럼 보이게 만드는 기능이다. 방법은 정적 클래스 안에 정적 메쏘드를 정의하면서 첫번째 파라미터 앞에 `this` 키워드를 붙여주면 된다. 즉, 아래처럼 하면:

```csharp
static class Extension {
    public static int KiloBytes(this int number) => number * 1024;
}

12.KiloBytes() == 12288 // true
```

원래 `int` 타입에는 `KiloBytes()`란 메쏘드가 (당연히) 없지만 메쏘드의 첫번째 파라미터가 인스턴스 위치로 이동해서 마치 인스턴스 메쏘드를 호출하는 것처럼 보이게 되었다. 확장 메쏘드는 기본적으로 정적 메쏘드이기 때문에 아래처럼 기존 문법대로 써도 된다:

```csharp
Console.WriteLine(Extension.KiloBytes(12));
```

## C# 함수형 스타일 로또 생성기

[지난번](https://bangjunyoung.github.io/2019/08/26/함수형-프로그래밍이-어려운-이유/)에 만들었던 함수형 스타일 로또 생성기를 다시 가져와 보자:

```csharp
IEnumerable<TResult> RunInfinite<TResult>(Func<TResult> func) {
    while (true)
        yield return func();
}

IEnumerable<int> LottoNumbers_Functional(int minValue, int maxValue, int count) {
    var rand = new Random((int)DateTime.Now.Ticks);
    return RunInfinite(() => rand.Next(minValue, maxValue + 1))
           .Distinct()
           .Take(count);
}
```

여기서 `Distinct()`와 `Take()`는 `System.Linq.Enumerable` 정적 클래스에 들어 있는 확장 메쏘드들이다. 기존 문법대로 코드를 다시 써보면

```csharp
RunInfinite(() => rand.Next(minValue, maxValue + 1))
.Distinct()
```

는

```csharp
Enumerable.Distinct(
    RunInfinite(() => rand.Next(minValue, maxValue + 1)))
```

로 바뀌게 되고, 뒤의 `.Take(count)`까지 바꾸면

```csharp
Enumerable.Take(
    Enumerable.Distinct(
        RunInfinite(() => rand.Next(minValue, maxValue + 1))),
    count)
```

가 된다. LINQ 스타일로 썼을 때는 가장 마지막에 실행되는 것처럼 보였던 `Take()`가 사실은 가장 먼저 실행되는 것을 볼 수 있다. `RunInfinite()` 메쏘드가 무한대로 실행되지 않는 이유도 그 때문이다. `Take()`가 내부 루프를 통해 `Distinct()`를 반복 호출하고, `Distinct()`는 다시 내부 루프를 통해 `RunInfinite()`를 반복 호출하기 때문에 `RunInfinite()`는 실제로 6번 + 중복제거된 횟수만큼만 실행되는 것이다.

## F# 함수형 스타일 로또 생성기

F# 버전의 로또 생성기는 이미 [지난번](https://bangjunyoung.github.io/2019/09/02/FSharp-함수-이해하기-2부/)에 다룬 바 있다. 파이프라인 연산자 `|>`를 이용해서 아래처럼 만들었던 코드가

```fsharp
open System

let lottoNumbers minValue maxValue count =
    let rand = Random(int DateTime.Now.Ticks)
    Seq.initInfinite (fun _ -> rand.Next(minValue, maxValue + 1))
    |> Seq.distinct
    |> Seq.take count
```

`|>`를 이용하지 않은 형태로 고치면

```fsharp
open System

let lottoNumbers minValue maxValue count =
    let rand = Random(int DateTime.Now.Ticks)
    Seq.take count (
        Seq.distinct (
            Seq.initInfinite (fun _ -> rand.Next(minValue, maxValue + 1))
        )
    )
```

처럼 된다. C# LINQ 스타일과 개념적으로 동일하다는 것을 알 수 있다.

## C# 확장 메쏘드와 F# `|>`의 차이

F#은 단순히 연산자를 정의하는 것만으로 새로운 기능을 만들어낸 반면, C#에서는 언어 자체를 수정해서 확장 메쏘드라는 문법을 새로 만들어야 했다. 그리고 F#은 다른 파라미터들은 그대로 두고 맨 뒤에 있는 파라미터만 `|>` 앞으로 이동하는데 비해 C#은 첫번째 파라미터부터 한칸씩 앞으로 당겨 쓰는 방식이다.

C#에 비해 F#의 구현이 간단한 이유는 기본적으로 커링과 부분 함수 적용이 언어 차원에서 지원되기 때문이다. F#은 함수 파라미터 리스트를 임의의 위치에서 잘라 새로운 함수를 만드는 것이 아주 쉽기 때문에 언어의 유연성과 표현력이 그만큼 높다고 하겠다.
