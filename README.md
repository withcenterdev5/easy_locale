# 다국어


pub.dev 에 좋은 다국어(언어 번역) 패키지들이 있다. 하지만 보다 쉽게 사용 할 수 있도록 `하우스`만의 다국어 기능을 만들었다.

Flutter locale 을 사용하지 않는다. 따라서 Flutter 에 로케일 설정을 할 필요 없다. 단, iOS 의 Info.plist 에 `CFBundleLocalizations` 과 언어 목록을 추가한다. 물론 Flutter locale 을 사용하고 싶으면 해도 된다.


## 설치



### iOS

- 안드로이드와는 다르게, iOS 에서는 사용하고자 하는 언어를 Info.plist 의 `CFBundleLocalizations`에 추가해 주어야 한다. 핸드폰 설정의 기본 언어가 한글이라도, 따로 CFBundleLocalizations 에 추가해 주어야 사용이 가능하다.


## 초기화

- 초기화는 `main()` 함수에서 `runApp` 을 호출하기 전에 할 것을 권한다. 만약 이 위치에 하지 못하는 경우, 앱의  첫 페이지 스크린 위젯의 `initState` 에서 할 수 있다.

- 아래와 같이 init 을 호출하면 핸드폰(장치) 설정의 언어를 사용 할 수 있다. 상황에 따라서 `TranslationService.instance.init()` 앞에 `await` 키워드를 추가해도 된다.

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  TranslationService.instance.init();
  runApp(const MyApp());
}
```

- 아래와 같이 좀 더 자세히 초기화를 할 수 있다.

```dart
TranslationService.instance.init(
    deviceLocale: true,
    defaultLocale: 'ko',
    fallbackLocale: 'en',
    useKeyAsDefaultText: true,
);
```

위 코드에서 `deviceLocale` 에 true 를 지정하면, 현재 장치의 설정에 적용된 언어를 사용하라는 것이다. 기본 값은 `true` 이다.
그리고 만약, `deviceLocale` 에서 언어를 설정하지 못하거나, `deviceLocale` 일 false 인 경우, 기본적으로 `defaultLanguage` 를 사용하게 할 수 있다. 즉, 핸드폰 기본 설정 언어를 사용하지 않고, 내 마음대로 언어를 지정하고 싶다면, `deviceLocale` 에 false 를 주고, `defaultLanguage` 에 "ko" 등의 값을 주어 내가 원하는 언어가 기본 선택되게 할 수 있다.

만약, 다국어에서 번역된 문자열을 찾지 못하면 `fallbackLocale` 의 언어에서 번역된 문자열이 있는지 찾아서 있으면 그 번역 문자열을 사용한다. 기본 값은 `en` 이다.
그리고 `useKeyAsDefaultText` 가 true 이면, 번역된 문자열을 못찾은 경우 언어 코드(키)를 그대로 사용하라는 것이다. 만약, 이 값이 false 이면, 언어 코드(키)를 사용하되 맨 끝에 `.t` 를 붙여서 화면에 표시한다.





## 사용법

`.t` 와 `.tr` 두개의 다국어 함수가 있다.


### 번역 문자열

번역 문자열은 `./lib/translation/translation.text.dart` 의 `translationTexts` 변수에 저장한다.


### 단순 문자열 번역 - t

`.t` 은 단순히 다국어로 표시하는 함수이다.

```dart
Translation.instance.setLocale('ko');
'name'.t // 결과: 이름
```

### 문자열 치환 및 단수와 복수 처리 - tr

`.t` 에 비해서 `.tr` 은 문자열 치환이 가능하며, 단수/복수 처리도 할 수 있다.

기본적인 사용 방법은 `'apple'.tr(args: {'name': 'J', 'n': n}, form: n)` 와 같다.

arg 파라메타에는 `arg: { 키: 값, ... }` 와 같은 형태로 번역 문자열에서 치환하고자 할 키/값을 지정한다.

form 파라메타에는 null 또는 0의 값이면 개체가 없고, 1의 값이면 단수, 2 이상의 값이면 복수로 표현한다.

번역 문자열을 저장 할 때에는 아래와 같은 형식으로 저장되어야 한다.

```dart
final translationTexts = {
    'apple': {
        'en': {
            'zero': '{name} has no apple.',
            'one': '{name} has one apple.',
            'many': '{name} has {n} apples.',
        }
    }
};
```

예제

```dart
TranslationService.instance.set(
  key: 'apple',
  locale: 'en',
  value: {
    'zero': '{name} has no apple.',
    'one': '{name} has one apple.',
    'many': '{name} has {n} apples.',
  },
);

int n = 0;
expect('apple'.tr(args: {'name': 'J'}, form: n), 'J has no apple.');
n = 1;
expect('apple'.tr(args: {'name': 'J'}, form: n), 'J has one apple.');
n = 3;
expect('apple'.tr(args: {'name': 'J', 'n': n}, form: n), 'J has 3 apples.');
```



### 번역 문장 추가 또는 업데이트

`TranslationService.instance.set()` 를 사용하면 기존의 존재하는 번역 문자열을 다른 것 변경 할 수 있다. 또는 기존에 존재하지 않는다면 추가를 하는 것이다. 따라서 앱에서 사용할 번역 문자열을 하우스 다국어 기능을 활용해 번역 할 수 있다.

```dart
TranslationService.instance.set(
key: 'hello',
locale: 'en',
value: 'Hello',
);

expect('hello'.t == 'Hello', true);
```




위와 같이 하면 `name` 이라는 언어 키에 아래와 같이 저장하면 된다. 특히, 단수/복수 처리를 잘 보면 된다.

```dart
final translationTexts = {
    'name': {
        'en': 'Name',
        'ko': '이름',
    },
    'there are {n} apple': {
        'en': {
            'none': 'There is no apple',
            'one': 'There is only {n} apple',
            'many': 'There are {n} apples',
        },
        'ko': '사과가 {n}개 있어요',
    }
}
```

