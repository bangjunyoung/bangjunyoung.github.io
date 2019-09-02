---
layout: post
title: C#에서 F#으로 전환하기
subtitle: 10분만에 이해하는 C#과 F#의 기본적인 차이
tags: [C#, F#]
---

이번 글에서는 C# 프로그램을 단계별로 F# 프로그램으로 변환하면서 두 언어간 차이에 대해 알아보기로 한다. 기본 C# 프로그램은 지난번 글에서 만든 [로또 번호 생성기](https://bangjunyoung.github.io/2019/08/26/함수형-프로그래밍이-어려운-이유/)를 이용한 것으로, 전체 소스는 다음과 같다:

```csharp
using System;
using System.Collections.Generic;

namespace CSharpToFSharp {
    static class Program {
        static List<int> LottoNumbers(int minValue, int maxValue, int count) {
            var rand = new Random((int)DateTime.Now.Ticks);
            var numbers = new List<int>();
            while (numbers.Count != count) {
                var num = rand.Next(minValue, maxValue + 1);
                if (!numbers.Contains(num)) {
                    numbers.Add(num);
                }
            }
            return numbers;
        }

        static void Main(string[] args) {
            List<int> numbers = LottoNumbers(1, 45, 6);
            Console.WriteLine(string.Join(", ", numbers));
        }
    }
}
```

전체 단계는 다음과 같다:

- `namespace` 블록을 선언문으로 대체
- 블록 구분자 `{}`와 문장 끝 `;` 삭제
- `static class`를 `module`로 대체
- `let` 키워드로 함수와 변수 정의
- 타입 지정 형식 변경
- 타입 지정 생략
- 함수 파라미터 형식 변경
- 제어문과 연산자 대체
- `new`와 `return` 키워드 제거
- C# 타입 캐스팅을 F# 함수로 대체
- `[<EntryPoint>]` 어트리뷰트의 사용

쓰고 보니 11단계나 되는데 생각보다 매우 간단하므로 10분안에 이해할 수 있을 것이다.

## `namespace` 블록을 선언문으로 대체

C#에서는 네임스페이스가 블록인데 비해 F#에서는 자바의 패키지 선언과 마찬가지로 한줄 짜리 문장이다. 그리고 `using`문은 반드시 네임스페이스 내부에 있어야 하고 키워드로는 `open`을 쓴다.

```csharp
namespace CSharpToFSharp;

open System;
open System.Collections.Generic;

static class Program {
    static List<int> LottoNumbers(int minValue, int maxValue, int count) {
        var rand = new Random((int)DateTime.Now.Ticks);
        var numbers = new List<int>();
        while (numbers.Count != count) {
            var num = rand.Next(minValue, maxValue + 1);
            if (!numbers.Contains(num)) {
                numbers.Add(num);
            }
        }
        return numbers;
    }

    static void Main(string[] args) {
        List<int> numbers = LottoNumbers(1, 45, 6);
        Console.WriteLine(string.Join(", ", numbers));
    }
}
```

바꾸고 보니 들여쓰기 깊이가 한단계 낮아져 보기가 한결 수월해진 느낌이다. C#도 원래부터 이렇게 만들었으면 좋을 뻔 했다.

## 블록 구분자 `{}`와 문장 끝 `;` 삭제

F#이 C#과 다르게 보이는 가장 근본적인 차이가 블록 들여쓰기 방식이다. F#에서는 C 계열 언어에서처럼 `{}`를 써서 블록을 구분하지 않고 파이썬처럼 공백을 이용한 들여쓰기로 블록을 구분한다. 그리고 문장 끝에 `;`를 붙이지 않아도 된다(한 줄에 두 개 이상의 문장을 쓸 때만 문장 사이에 쓴다). 또한 변수 정의와 같이 모듈과 함수를 정의할 때도 이름 뒤에 `=`를 붙인다.

```fsharp
namespace CSharpToFSharp

open System
open System.Collections.Generic

static class Program =
    static List<int> LottoNumbers(int minValue, int maxValue, int count) =
        var rand = new Random((int)DateTime.Now.Ticks)
        var numbers = new List<int>()
        while (numbers.Count != count)
            var num = rand.Next(minValue, maxValue + 1)
            if (!numbers.Contains(num))
                numbers.Add(num)
        return numbers

    static void Main(string[] args) =
        List<int> numbers = LottoNumbers(1, 45, 6)
        Console.WriteLine(string.Join(", ", numbers))
```

단지 `{}`와 `;'를 삭제했을 뿐인데 여기서부터는 C#보다 F# 쪽에 가까워져 보인다.

참고로 F#에서는 들여쓰기할 때 탭 문자 사용이 금지되어 있어서 모든 공백은 스페이스 문자로만 해야 한다. 이러한 강제 규정 때문에 남이 짠 소스를 내 컴퓨터로 가져왔을 때 원래 것과 다르게 보이거나 들여쓰기가 엉망이 되는 현상(일명 탭 지옥)이 절대 일어나지 않는다.

## `static class`를 `module`로 대체

F#에서 기본적으로 모든 함수는 모듈 내에 두게 되어 있다. 이 모듈은 C#의 정적 클래스와 의미상 같은 것으로 실제로도 정적 클래스로 컴파일된다. 그리고 함수 이름 앞에 `static` 키워드를 붙이지 않는다.

```fsharp
namespace CSharpToFSharp

open System
open System.Collections.Generic

module Program =
    List<int> LottoNumbers(int minValue, int maxValue, int count) =
        var rand = new Random((int)DateTime.Now.Ticks)
        var numbers = new List<int>()
        while (numbers.Count != count)
            var num = rand.Next(minValue, maxValue + 1)
            if (!numbers.Contains(num))
                numbers.Add(num)
        return numbers

    void Main(string[] args) =
        List<int> numbers = LottoNumbers(1, 45, 6)
        Console.WriteLine(string.Join(", ", numbers))
```

## `let` 키워드로 함수와 변수 정의

변수와 함수를 정의할 때는 공통적으로 이름 앞에 `let` 키워드를 쓴다. 그에 따라 변수와 함수의 정의 형태가 아주 유사해지는데, 기본적으로는 이름 뒤에 파라미터가 더 있으면 함수고, 이름 뒤에 아무것도 없으면 변수다(더 자세한 구분은 다음에). 그리고 관례상 함수와 변수의 이름은 소문자로 시작하는 카멜케이스 방식을 따른다(자바와 동일하다). 앞의 소스에 반영하면 다음과 같이 된다:

```fsharp
namespace CSharpToFSharp

open System
open System.Collections.Generic

module Program =
    let List<int> lottoNumbers(int minValue, int maxValue, int count) =
        let rand = new Random((int)DateTime.Now.Ticks)
        let numbers = new List<int>()
        while (numbers.Count != count)
            let num = rand.Next(minValue, maxValue + 1)
            if (!numbers.Contains(num))
                numbers.Add(num)
        return numbers

    let void main(string[] args) =
        let List<int> numbers = lottoNumbers(1, 45, 6)
        Console.WriteLine(string.Join(", ", numbers))
```

여기서 `lottoNumbers`와 `main`은 뒤에 파라미터가 있으므로 함수고, `rand`, `numbers`, `num` 등은 변수다.

## 타입 지정 형식 변경

타입 지정을 변수 이름 앞에 두는 C 계열 언어와 달리 F#에서는 이름 뒤에 `:`과 함께 표기한다. 함수의 리턴 타입은 파라미터 리스트 뒤에 `:`과 함께 표기한다.

```fsharp
namespace CSharpToFSharp

open System
open System.Collections.Generic

module Program =
    let lottoNumbers(minValue: int, maxValue: int, count: int) : List<int> =
        let rand = new Random((int)DateTime.Now.Ticks)
        let numbers = new List<int>()
        while (numbers.Count != count)
            let num = rand.Next(minValue, maxValue + 1)
            if (!numbers.Contains(num))
                numbers.Add(num)
        return numbers

    let main(args: string[]) : void =
        let numbers = lottoNumbers(1, 45, 6)
        Console.WriteLine(string.Join(", ", numbers))
```

## 타입 지정 생략

사실 F#의 타입 추론 기능은 C#보다 훨씬 강력해서 대부분의 경우 타입 지정을 생략해도 문제가 없다. 심지어 함수의 리턴 타입까지 생략해도 된다. 아래처럼 타입 지정을 전부 생략해도 앞의 코드와 의미는 같다:

```fsharp
namespace CSharpToFSharp

open System
open System.Collections.Generic

module Program =
    let lottoNumbers(minValue, maxValue, count) =
        let rand = new Random((int)DateTime.Now.Ticks)
        let numbers = new List<int>()
        while (numbers.Count != count)
            let num = rand.Next(minValue, maxValue + 1)
            if (!numbers.Contains(num))
                numbers.Add(num)
        return numbers

    let main(args) =
        let numbers = lottoNumbers(1, 45, 6)
        Console.WriteLine(string.Join(", ", numbers))
```

F#에서는 특별한 이유가 없다면 타입 지정을 생략하는 것이 관례다. 그래서 언뜻 보면 파이썬 같은 동적 타입/스크립트 언어와 매우 유사한 느낌을 주는데, 실제로는 C#과 같은 정적 타입 기반 언어다.

## 함수 파라미터 형식 변경

F#에서는 함수 파라미터를 `,` 대신 빈 칸으로 구분하고, 파라미터 리스트 전체를 둘러싸는 괄호도 쓰지 않는다. 앞의 소스에서 `,`와 `()`를 제거하면 다음과 같이 된다:

```fsharp
namespace CSharpToFSharp

open System
open System.Collections.Generic

module Program =
    let lottoNumbers minValue maxValue count =
        let rand = new Random((int)DateTime.Now.Ticks)
        let numbers = new List<int>()
        while (numbers.Count != count)
            let num = rand.Next(minValue, maxValue + 1)
            if (!numbers.Contains(num))
                numbers.Add(num)
        return numbers

    let main args =
        let numbers = lottoNumbers 1 45 6
        Console.WriteLine(string.Join(", ", numbers))
```

단, F# 함수가 아닌 다른 .NET 언어에서 가져온 API 메쏘드는 여전히 C# 스타일로 `()`와 `,`를 써서 표기한다. 여기 소스에 보이는 `Random()`, `Next()`, `Contains()` 등이 모두 [.NET BCL](https://docs.microsoft.com/en-us/dotnet/standard/framework-libraries)에서 가져온 것들이다. 더 자세하고 정확한 것은 조만간 따로 다루도록 하겠다.

## 제어문과 연산자 대체

F#에도 `while`과 `if`문이 있는데 C#과 거의 비슷하다. F# 스타일로 고쳐 보면 다음과 같다:

```fsharp
namespace CSharpToFSharp

open System
open System.Collections.Generic

module Program =
    let lottoNumbers minValue maxValue count =
        let rand = new Random((int)DateTime.Now.Ticks)
        let numbers = new List<int>()
        while numbers.Count <> count do
            let num = rand.Next(minValue, maxValue + 1)
            if not (numbers.Contains(num)) then
                numbers.Add(num)
        return numbers

    let main args =
        let numbers = lottoNumbers 1 45 6
        Console.WriteLine(string.Join(", ", numbers))
```

`while`문은 조건 뒤에 `do`를 붙이고, `if`문은 조건 뒤에 `then`을 붙인다.

F#은 `!` 연산자를 다른 용도로 쓰기 때문에 C#의 `!`를 `not` 함수로 대체했다. 여기 `not`을 쓰면서 뒷부분 `numbers.Contains(num)` 전체를 `()`로 감쌌는데, 이렇게 하지 않으면 F# 문법 규칙상 `not`과 `numbers.Contains`가 뒤의 `(num)`보다 먼저 결합하므로 `(not numbers.Contains)(num)`처럼 뜻이 이상해져 버린다(결과는 물론 컴파일 에러). 이 문제는 특히 F# 함수와 C# 메쏘드를 섞어 쓸 때 빈번히 발생하는 문제라 주의가 필요하다.

같지 않음을 비교하는 연산자는 `!=` 대신 `<>`로 쓴다.

## `new`와 `return` 키워드 제거

`new`는 C#과 의미가 같지만 써도 되고 안써도 되는데 대개는 쓰지 않는다(관례상 쓰는 곳이 따로 있다). `return` 키워드는 F#에도 있긴 하지만 함수 리턴값과는 전혀 상관없는 용도로 사용한다. 실제로는 굳이 `return`같은 키워드가 필요하지 않은데, 그 이유는 함수 내에서 가장 마지막에 실행되는 식의 결과가 자동으로 리턴값이 되기 때문이다. 두 키워드를 제거하면 다음과 같이 된다:

```fsharp
namespace CSharpToFSharp

open System
open System.Collections.Generic

module Program =
    let lottoNumbers minValue maxValue count =
        let rand = Random((int)DateTime.Now.Ticks)
        let numbers = List<int>()
        while numbers.Count <> count do
            let num = rand.Next(minValue, maxValue + 1)
            if not (numbers.Contains(num)) then
                numbers.Add(num)
        numbers

    let main args =
        let numbers = lottoNumbers 1 45 6
        Console.WriteLine(string.Join(", ", numbers))
```

## C# 타입 캐스팅을 F# 함수로 대체

F#에서는 `int`, `char`, `float`, ... 등의 기본 타입 이름 자체가 타입 변환 함수로도 쓰인다. 그래서 C#에서 `(int)DateTime.Now.Ticks`처럼 하던 타입 캐스팅이 F#에선 `int DateTime.Now.Ticks`처럼 `int`라는 함수를 호출하는 형태가 된다.

```fsharp
namespace CSharpToFSharp

open System
open System.Collections.Generic

module Program =
    let lottoNumbers minValue maxValue count =
        let rand = Random(int DateTime.Now.Ticks)
        let numbers = List<int>()
        while numbers.Count <> count do
            let num = rand.Next(minValue, maxValue + 1)
            if not (numbers.Contains(num)) then
                numbers.Add(num)
        numbers

    let main args =
        let numbers = lottoNumbers 1 45 6
        Console.WriteLine(String.Join(", ", numbers))
```

또한 맨 마지막 줄의 `string.Join`을 `String.Join`으로 고쳤는데, `int`나 `string` 등을 C#에서처럼 메쏘드 호출에 쓰는 타입 별칭으로는 사용할 수 없다. 그래서 `int.Parse(...)` 같은 용법이 불가능하고 꼭 정식 타입 명칭을 써서 `Int32.Parse(...)`처럼 써야 한다. 이점은 약간 헷갈리는 언어 설계이긴 한데 요약하자면

```fsharp
let a: int = 3
let s: string = "a"
let b = int 3.14
let t = string 42
```

등은 허용되지만

```fsharp
let a = int.Parse("3") // 에러
let s = string.Join(", ", numbers) // 에러
```

는 안된다는 것이다.

## `[<EntryPoint>]` 어트리뷰트의 사용

C#과 달리 F#은 정해진 이름의 메인 함수가 따로 없고 `string[]` 타입의 파라미터를 한개 갖고 리턴 타입이 `int`인 함수에 `[<EntryPoint>]` 어트리뷰트를 붙여 주면 메인 함수가 된다. C#에서는 어트리뷰트를 `[]`로 둘러싸지만 F#에선 `[<>]`로 둘러싸는 것이 다를 뿐이다. 위의 소스에 반영해 보면 다음과 같이 된다:

```fsharp
namespace CSharpToFSharp

open System
open System.Collections.Generic

module Program =
    let lottoNumbers minValue maxValue count =
        let rand = Random(int DateTime.Now.Ticks)
        let numbers = List<int>()
        while numbers.Count <> count do
            let num = rand.Next(minValue, maxValue + 1)
            if not (numbers.Contains(num)) then
                numbers.Add(num)
        numbers

    [<EntryPoint>]
    let main args =
        let numbers = LottoNumbers 1 45 6
        Console.WriteLine(String.Join(", ", numbers))
        0
```

`main` 함수 마지막에 붙인 `0`이 `main` 함수의 리턴값이자 이 프로그램의 리턴값이 된다.

여기까지 하고 나면 F#으로의 변환이 마침내 끝났다! 이제 프로그램을 실행하면

```
42, 20, 22, 12, 25, 3
```

처럼 나오면 잘된 것이다.

## 마치며

지금까지 보인 것처럼 기본적으로 F#은

- 공백으로 구분하는 들여쓰기 방식이 파이썬과 유사하다. 단, 탭 문자는 사용하지 못한다.
- 거의 대부분의 경우 타입 지정을 생략할 수 있어서 언뜻 보면 파이썬 같은 동적 타입/스크립트 언어와 비슷하게 생겼다. 그렇지만 실제로는 C#과 같은 정적 타입 기반 컴파일 언어다.
- 특별히 함수형 프로그래밍을 위해 설계되었지만 기존 명령형 프로그래밍 방식으로도 충분히 짤 수 있다. 글에서는 다루지 않았지만 클래스나 인터페이스 같은 OOP 구문들도 전부 지원한다.
- C#에서 문법상의 각종 군더더기를 싹 걷어낸 형태와 매우 유사하므로 기본 문법을 익히기 쉽다.
- C#을 포함한 다른 .NET 언어로 만든 API를 그대로 가져와 쓸 수 있다. 반대로 F#으로 만든 코드도 C#에서 그대로 불러와 사용 가능하다.

[다음 글]({{ '/2019/08/30/FSharp-함수-이해하기-1부/' | relative_url }})에서는 F#의 함수에 관해 더 자세히 알아보기로 한다.
