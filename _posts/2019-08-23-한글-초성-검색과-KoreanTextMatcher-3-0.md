---
layout: post
title: 한글 초성 검색과 KoreanTextMatcher 3.0
subtitle: 5년만의 새 버전에서 달라진 점
tags: [한글, 자바]
---

자바/안드로이드용 한글 초성 검색 라이브러리인 KoreanTextMatcher의 [새 버전](https://github.com/bangjunyoung/KoreanTextMatcher)을 5년만에 내놓았다. 기능상의 큰 변화는 없지만 API를 수정하면서 하위 호환성이 깨졌기 때문에 버전 번호를 2.0에서 3.0으로 올렸다. 바뀐 기능은 다음과 같다:

- `KoreanTextMatcher.matches()`가 `List<KoreanTextMatch>` 대신 `Iterable<KoreanTextMatch>`를 리턴하도록 변경. 이렇게 함으로써 검색 결과를 다양한 형태로 받아쓸 수 있다.
- `KoreanChar.xxxCompatibilityChoseong()`을 `KoreanChar.xxxCompatChoseong()`으로 이름 줄임. `Compatibility`란 단어가 너무 길어서 쓰기 불편했는데 `Compat`으로 줄여도 가독성에 문제가 없는 듯하다.
- Gradle 기반 빌드. 이제 KoreanTextMatcher를 JDK와 Gradle이 설치되어 있는 곳이면 어디서나 빌드할 수 있다.
- 문서화 개선. 지난 버전은 API 레퍼런스를 군데군데 빼먹은 곳이 많았는데, 지금은 전부 다 문서화했다. 미리 생성한 [Javadoc 문서](https://github.com/bangjunyoung/KoreanTextMatcher/releases/download/3.0/KoreanTextMatcher-3.0-javadoc.zip)도 제공하므로 웹브라우저에서 볼 수 있다.
- JDK 8과 JDK 12에서 테스트됨. 안드로이드가 공식 지원하는 JDK는 버전 8이고 오라클 자바는 JDK 12가 최신인데 둘 다 지원한다.

한글 초성 검색이 무엇인지 모르는 분을 위해 잠깐 덧붙이자면 대략 다음과 같은 일을 할 수 있다:

![안드로이드에서 한글 초성 검색후 하일라이트]({{ '/img/HightlightTextViewDemo.png' | relative_url }}){: .center-block :}

실제 소스 코드는 다음과 같다:

``` java
String text = "바닥에 남은 차가운 껍질에 뜨거운 눈물을 부어";

TextView tv = new TextView(getBaseContext());
tv.setTextSize(25F);
tv.setText(text, TextView.BufferType.SPANNABLE);

Spannable spannedText = (Spannable) tv.getText();

KoreanTextMatcher matcher = new KoreanTextMatcher("ㄱㅇ");
for (KoreanTextMatch match : matcher.matches(text)) {
    spannedText.setSpan(new ForegroundColorSpan(Color.RED),
        match.index(), match.index() + match.length(),
        Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
}
```

더 자세한 용법은 [프로젝트 홈](https://github.com/bangjunyoung/KoreanTextMatcher)에 나와 있다.

## 로드맵

KoreanTextMatcher는 아마도 이번 버전이 마지막이 될 것 같다. 초성 매칭 검색 이외에 한글 처리 관련 다양한 기능을 제공하는 라이브러리를 .NET용으로 만들고 있는데, 재미있고 유용한 기능들이 많다. 앞으로 하나씩 블로그를 통해 소개하도록 하겠다.
