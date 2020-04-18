# 앱 개발 방식

## 1. 네이티브 방식

안드로이드나 iOS 같은 플랫폼 자체에서 제공하는 개발 환경으로 개발한다.  
안드로이드는 개발 도구로 안드로이드 스튜디오를, 갭라 언어로 자바 또는 코틀린(Kotlin)을 사용한다.  
iOS는 맥OS 환경에서만 개발이 가능하며 개발 도구로 엑스코드(XCode), 개발 언어로 스위프트(Swift) 또는 오브젝티브-C(Objective-C)를 사용한다.  
두 플랫폼에 맞는 앱을 각각 네이티브 방식으로 혼자 개발하는 것은 쉬운 일이 아니다.

## 2. 하이브리드 방식

웹 기술로 웹 화면을 만든 후 네이티브 기술로 감싼 앱 형태다.  
기존의 웹 기술을 활용하여 빠르게 앱으로 변환할 수 있기에 빠른 앱 개발이 가능하지만, 네이티브 성능에 미치지 못한다.  
UI 또한 별도로 만들기에 네이티브 앱 느낌을 내지 못한다.

## 3. 크로스 플랫폼 방식

소스 코드 한 번 작성하여 안드로이드와 iOS 등 각 플랫폼용 앱을 만들 수 있다.  
빌드할 때 각 네이티브 코드로 변환되기에, 네이티브 방식으로 갭라했을 때와 거의 같은 성능을 보인다.  
생산성과 품질을 모두 고려했을 때 선호한다.

# 플러터 소개

* 낮은 진입 장벽
* 높은 네이티브 성능 : 다른 크로스 플랫폼 개발 프레임워크와 다르게 화면 구성에 필요한 UI 구성 요소를 플러터가 직접 그려주기에 속도가 빠르다. 초당 60 프레임 애니메이션을 보장한다.
* 훌륭한 개발 도구 지원 : 안드로이드 스튜디오, VS Code, Intellij 모두 플러그인을 지원한다.
* 예쁜 UI : 안드로이드의 [머터리얼 디자인](https://material.io/)과 iOS의 [쿠퍼티노 디자인](https://developer.apple.com/design/)의 UI 구성 요소를 모두 제공한다.

# 1. 안드로이드 개발 환경 구성

1. 플러터 SDK 설치 : [플러터 웹사이트](https://flutter.dev)에서 Flutter SDK를 설치한다.
2. (윈도우) 환경 변수 등록 : 플러터 SDK의 bin 폴더를 환경 변수 Path에 등록한다.
3. 안드로이드 스튜디오 설치
4. 안드로이드 스튜디오에서 Plugins 설정에서 Flutter 플러그인을 설치한다.
5. `flutter doctor` 명령어로 프로젝트 환경 구성 점검하기

## 핫 리로드

핫 리로드는 수정한 코드를 즉시 앱에 반영하는 기능이다.  
소스 코드를 수정하고 저장(`Ctrl + S`) 하면 즉시 기기나 에뮬레이터에 수정된 화면이 반영된다.

## 에뮬레이터 한국어 설정

1. 상단 상태바를 아래로 내려 설정 아이콘을 클릭
2. `System` 메뉴를 선택
3. `Language & Input` 메뉴를 선택
4. `Languages` 메뉴를 선택
5. `Add a language` 메뉴를 선택
6. `korea`를 검색해 한국어를 선택
7. `대한민국`을 선택한다.
8. 오른쪽의 이동 핸들을 클릭해 위로 드래그하여 한국어가 첫 번째가 되도록 설정한다.

# 2. iOS 개발 환경 구성

## iOS용 도구 설정 및 빌드

### 1. 앱 스토어에서 Xcode를 다운로드 받는다.

앱스토어 앱을 열어 Xcode를 검색해 다운로드 받는다.

### 2. Xcode command line tools를 설치한다. Xcode를 처음 실행했을 때 다운로드가 가능하다. 또는 터미널로 직접 설치가 가능하다 

#### Xcode command line tools 설치

```
$ sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
$ sudo xcodebuild -runFirstLaunch

$ sudo xcodebuild -license # 약관 수락
```

### 3. 시뮬레이터로 실행

아래 명령어를 실행하거나 안드로이드 스튜디오(Mac OS 버전)에서도 실행이 가능하다.

```
$ open -a Simulator
```

### 4. 실제 기기 실행

`flutter doctor` 명령어로 iOS 개발 환경 구성을 점검할 수 있다.  
만약 `cocoapods`가 설치되지 않았다면 설치해야 한다.

```
$ sudo gem install cocoapods
```

# 다트 입문하기

## 다트 언어 실습하기

[다음 웹사이트](https://dartpad.dev)에서 코드 실행이 가능하다.

## 주석

```dart
// 한 줄 주석

/**
* 여러 줄 주석
*//

/// 문서 주석
void something() {}
```

## 문장

문장 끝은 `;` 으로 표시한다. 세미클론 누락은 컴파일 오류를 발생시킨다.

```dart
void main() {
    // 주석
    print("Hello, World");
}
```

## 변수

기본 타입은 다음과 같다.

* int: 정수
* double: 실수(소수점)
* String: 문자열
* bool: true or false

```dart
String name;
name = "다람쥐";
name = '다람쥐';
```

문자열을 표시할 때 작은따음표와 큰따음표 모두 사용이 가능하다. 문법 표준 가이드는 **작은따음표**가 표준이다.

int와 double은 num 타입에 포함된다.

```dart
num a = 10;
num b = 10.5;
```

dart 에서는 int 타입과 double 타입의 호환성이 없다.

```dart
int a = 10;
double b = a; // 컴파일 오류
```

## 타입 추론

`var` 키워드로 타입 추론이 가능한 변수 선언을 할 수 있다.

```dart
var i = 10;     // int
var math = 90.5;    // double
var name = 'your name'; // string
var isSignup = true;    // bool
```

## 상수 final, const

final 키워드를 변수 앞에 붙이면 상수로 선언된다.

```dart
final String name = '다람쥐';
name = '청설모'; // 컴파일 오류

final name2 = '다람쥐'; // 타입 생략 가능
```

## 타입 검사

* is : 같은 타입이면 true
* is! : 다른 타입이면 true

## 형변환

형변환은 as 키워드를 사용한다. 상위 개념으로만 변환이 가능하다. as 키워드는 생략이 가능하다.

```dart
var c = 50.1;
int d = c as int; // 컴파일 오류
```

```dart
dynamic a = 50.1;
num n = d; // as num; 생략 가능
```

## 문자열 안 변수 템플릿

${} 기호로 문자열 내에 변수 삽입이 가능하다.

```dart
var _name = '다람쥐';

print('제 이름은 ${_name} 입니다.');
```

## 익명 함수

```dart
(number) {
    return number % 2 == 0;
}
```

## 람다식

```dart
(number) => number % 2 == 0;
```

## 선택 매개변수

함수 매개변수 정의할 때 {} 로 감싼 매개변수는 선택적으로 대입할 수 있다.

```dart
void something(String name, {int age}) { /* Empty */ }

void main() {
    something(name: '다람쥐');
    something(name: '다람쥐', age: 2);
}
```

선택 매개변수는 기본 값을 지정할 수 있다.

```dart
void something(String name, {int age = 1}) { /* Empty */ }

void main() {
    something(name: '다람쥐');
    something(name: '다람쥐', age: 2);
}
```

## switch case

switch 문은 다음과 같이 작성할 수 있다. Dart는 switch 조건 대상으로 enum 타입일 때 실수를 방지하기 위해 모든 타입을 검사하는 것을 강제한다.

```dart
enum Status { Uninitialized, Authenticated, Authenticating, Unauthenticated }

void main() {
    var status = Status.Authenticated;

    switch (status) {
        case Status.Authenticated:
            ~~~
            break;
        case Status.Authenticating:
            ~~~
            break;
        case Status.Unauthenticated:
            ~~~
            break;
        case Status.Untinialized:
            ~~~
            break;
    }
}
```

## 반복문

```dart
var items = ['a', 'b', 'c'];

for (var i = 0; i < items.length; i++) {
    print(items[i]);
}
```

## 클래스 접근 지정자

변수명 앞에 _기호를 붙이면 `private` 변수로 외부에서 접근이 불가능합니다.

```dart
// person.dart
class Person {
    String name;
    int _age;

    void addOneYear() {
        _age++;
    }
}

// main.dart
import 'person.dart';

~~~
var person = Person();
person._age = 10; // 컴파일 오류
```

## 게터와 세터

```dart
// person.dart
class Person {
    String name;
    int _age;

    int get age => _age;
}

// main.dart
import 'person.dart';

~~~
var person = Person();
print(person.age);)
```

```dart
class Rectangle {
    num left, top, width, height;

    Rectangle(this.left, tis.top, this.width, this.height);

    num get right => left + width;
    set right(num value) => left = value - width;

    num get bottom => top + height;
    set bottom(num value) = > top = value - height;
}
```

참고로 생성자 인수를 선택적으로 받는다면, 기본값은 null 이다.

## 상속

`extends` 키워드로 상속이 가능하다. 오버라이딩을 할 시 `@override` 어노테이션으로 재정의할 수 있다.

## 추상 클래스

class 앞에 `abstract` 키워드로 추상 클래스임을 명시적으로 선언한다. 추상 메소드는 메소드 선언은 되지만 정의가 없는 메소드다. 추상 클래스는 그대로 인스턴스로 만들 수 없다.

추상 클래스를 상속하려면, `extends` 키워드 대신에 `implements` 키워드로 사용해야 한다. 추상 클래스 여러개를 모두 구현할 수 있다. 이 때, 모든 추상 메소드를 구현해야 한다.

## 믹스인

`with` 키워드를 사용하면 상속하지 않고 다른 클래스의 기능을 가져오거나 오버라이드할 수 있다.

```dart
abstract class Monster {
    void attack();
}

class Goblin implements Monster {
    @override
    void attack() {
        print('고블린 어택');
    }
}

class DarkGoblin extends Goblin with Hero {

}
```

## 열거 타입

상수를 정의하는 특수한 형태다.

```dart
enum Status { login, logout }
```

## 컬렉션

1. List : 같은 타입의 자료를 여러 개 담을 수 있고 특정 인덱스로 접근이 가능하다.
2. Map : 키(key)와 값(value)의 쌍으로 저장하 수 있고, 키를 통해 값을 얻을 수 있다.
3. Set : 중복이 허용되지 않고, 찾는 값이 있는지 없는지 판단하고자 할 때 사용한다.

### 1. List

다트에서는 배열을 제공하지 않는다.

```dart
List<String> items = ['a', 'b', 'c'];
```

컬렉션과 컬렉션에 담을 요소의 타입도 모두 타입 추론이 가능하다.

```dart
var items = ['a', 'b', 'c'];

items[0] = 'd';

print(items.length); // 3
print(item[2]); // c
print(items[3]); // 컴파일 오류

for (var i = 0; i < items.length; i++) {
    print(items[i]);
}
```

참고로 다트에는 다양한 타입을 받는 `dynamic` 라는 특수한 타입이 있다. 리스트와 함께 사용해 여러 타입을 한 리스트에 저장할 수 있다.  
`List<dynamic>` 대신 `var` 키워드 하나로 대체할 수 있다.

```dart
var list = ['a', 'b', 1, 10.5];
```

### 스프레드 연산자

`...` 연산자는 컬렉션을 펼쳐준다. 다른 컬렉션 안에 컬렉션을 삽입할 때 사용한다.

```dart
var items = ['a', 'b', 'c'];
var items2 = ['z', ...items, 'y'];
```

리스트를 Set에 담으면 중복 제거의 효과를 얻을 수 있다.

```dart
final items = [1, 2, 2, 3, 3, 4, 4, 4, 5, 5, 5];

final numbers = {...items, 6, 7};
print(numbers); // 1, 2, 3, 4, 5, 6, 7
```

### 2. Map

순서가 없고 탐색이 빠른 자료구조다. 키와 값의 쌍으로 이루어진다.  
만약 요청하는 키가 없다면 `null`을 반환한다.

```dart
// Map<String, String> OSLanguageMap = { };
var OSLanguageMap = {
    'iOS': 'Swift',
    'Android': 'Java',
    'Server': 'Java'
};

OSLanguageMap['Android'] = 'Kotlin';

print(OSLanguageMap.length); // 3
print(OSLanguageMap['Server']); // Java
print(OSLanguageMap['Design']); // null

OSLanguageMap['Design'] = 'Adobe'; // 새로운 값 추가
```

### 3. Set

집합을 표현하는 자료구조다. `add()`, `remove()` 메소드로 집합에 추가하거나 삭제할 수 있다. `contains()` 메소드는 집합에 해당 값이 있는지 불리언 타입으로 반환한다. 리스트와 다른 점은 중복을 허용하지 않는다.

빈 Set이나 빈 Map을 작성할 때 문법을 조심해야 한다. 값 없이 그냥 {}만 작성한다면 `Map` 으로 매핑된다.

```dart
var mySet = <String>{}; // Set<String>

var mySet2 = {}; // Map<dynamic, dynamic>
```

키와 값이 아닌 한 데이터만 받는다고 명시했기 때문에 Set으로 인식하는 것 같다.