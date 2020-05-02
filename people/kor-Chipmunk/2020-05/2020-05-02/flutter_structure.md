# 플러터 프로젝트 구조와 앱 구조

플러터의 기본 예제 프로젝트는 다음과 같이 구성돼있다.

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget { ... }

class MyHomePage extends StatefulWidget { ... }

class _MyHomePageState extends State<MyHomePage> { ... }
```

`_MyHomePageState` 클래스에 보통 로직을 작성한다.

`package:flutter/material.dart` 패키지에는 머티리얼 디자인 위젯들이 포함되어 있다.

## StatelessWidget 클래스

`StatelessWidget` 클래스는 상태가 없는 위제를 정의한다. `MyApp` 클래스가 이 클래스의 서브클래스다.

상태를 가지지 않는다는 의미는 한 번 그려진 후 다시 그리지 않는 경우를 뜻한다. 프로퍼티로 변수를 가지지 않는다. 상수는 가질 수 있다.

`StatelessWidget` 클래스는 `build()` 메소드를 가지고 있다. `build()` 메소드는 위젯을 생성할 때 호출된다. 실제로 화면에 그릴 위젯을 작성해 반환한다.

따라서 `StatelessWidget` 클래스를 상속받은 `MyApp` 클래스는 `MaterialApp` 클래스의 인스턴스를 만들어 반환한다.

## MaterialApp 클래스

```dart
return MaterialApp(
    title: 'Flutter Demo',
    theme: ThemeData(
        primarySwatch: Colors.blue,
    ),
    home: MyHomePage(title: 'Flutter Demo Home Page'),
);
```

`title`, `theme`, `home` 세 가지 이름이 있는 인수를 설정한다. 플러터에서 이름 있는 인수는 클래스의 프로퍼티에 ㄱ밧을 할당한다.

## StatefulWidget 클래스

상태가 있는 위젯을 정의할 때 사용한다.

상태를 생성하는 메소드를 오버라이딩 한다.

```dart
@override
_MyHomePageState createState() => _MyHomePageState();

class _MyHomePageState extends State<MyHomePage> {
    int _counter = 0; // 변경 가능한 상태

    ...

    @override
    Widget build(BuildContext context) {
        return Scaffold(...)
    }
}
```

`MyHomePage` 클래스에는 상속받은 `createState()` 메소드를 재정의해 `_MyHomePageState` 클래스의 인스턴스를 반환한다. `StatefulWidget`이 생서오딜 때 한 번만 실행되는 메소드다.

`State` 클래스를 상속받은 클래스를 상태 클래스라고 부른다. 상태 클래스는 변경 가능한 상태를 프로퍼티 변수로 표현한다. 나중에 값을 변경하면 화면을 다시 그리게 된다.

`_MyHomePageState` 클래스의 `build(...)` 메소드로 화면에 그려질 부분을 정의한다.

## 위젯에서 위젯으로 값 전달

```dart
return MaterialApp(
    title: 'Flutter Demo',
    theme: ThemeData(
        primarySwatch: Colors.blue,
    ),
    home: MyHomePage(title: 'Flutter Demo Home Page'),
);
```

코드 중

```dart
home: MyHomePage(title: 'Flutter Demo Home Page'),
```

`MyHomePage` 위젯에 값을 전달했다. 위젯 사이의 데이터 전달은 생성자를 활용한다.

## 상태 변경

```dart
class _MyHomePageState extends State<MyHomePage> {
    int _counter = 0;

    void _incrementCounter() {
        setState(() {
            _counter++;
        });
    }

    @override
    Widget build(BuildContext context) {
        return Scaffold(...)
    }
}
```

`_incrementCounter()` 메소드에서 `setState()` 메소드를 실행한다. 이 메소드의 인수로 입력 인수가 없고 반환값이 없는 익명 함수를 전달했다. 익명 함수의 내용은 `_counter`를 1 증가하는 식이다.

`setState()` 메소드는 익명 함수를 실행한 뒤 화면을 다시 그리게 해준다. 화면은 `build()` 메소드가 실행되며 그려진다. 따라서 `setState()` 메소드는 `build()` 메소드를 다시 실행시킨다. `setState()` 메소드는 `State` 클래스가 제공하는 메소드다.

`MyHomePage` 클래스는 `StatefulWidget`의 서브클래스로 상태를 가질 수 있다. 상태는 `State` 클래스의 서브클래스로 정의한다.

## Scaffold 클래스와 AppBar 클래스

`Scaffold` 클래스는 머티리얼 디자인 앱을 만들 때 뼈대가 되는 위젯이다. 즉 `MaterialApp -> Scaffold`가 기본 형태다.

```dart
Scaffold(
    appBar: AppBar(
        title: Text(widget.title),
    ),
    body: ...,
    floatingActionButton: ...
);
```

```dart
int _counter = 0;
...
body: Center(
    child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
            Text(
                'You have pushed the button this mant times:',
            ),
            Text(
                '$_counter',
                style: Theme.of(context).textTheme.display1,
            ),
        ],
    ),
),
```

`_counter` 정수형 변수 앞에 `$` 기호를 붙임으로 문자열 형태로 변환한다.

## FloatingActionButton 클래스

```dart
FloatingActionButton: FloatingActionButton(
    onPressed: _incrementCounter,
    tooltip: 'Increment',
    child: Icon(Icons.add),
),
``

`onPressed` 프로퍼티는 버튼이 눌러지면 실행된다. 함수도 값으로 사용할 수 있는 특성에 대입이 가능하다. 또는 익명 함수로 전달해도 된다.

```dart
onPressed: () => _incrementCounter(),

onPressed: () {
    return _incrementCounter();
}
```

## 기본 예제 코드 전문

[dartpad.dev]() 에서 플러터 실습을 진행할 수 있다.

```dart
import 'package:flutter/material.dart';

// 앱 시작 부분
void main() => runApp(MyApp());

// 시작 클래스. 머티리얼 디자인 앱 설정
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

// 시작 클래스가 실제로 표시하는 클래스. 카운터 앱 화면
class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);

  final String title;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

// 위 MyHomePage 클래스의 상태를 나타내는 State 클래스
class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0; // 화면에 표시할 상탯값인 카운터 숫자

  // counter 변수를 1 증가시키고 화면을 다시 그리는 메소드
  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  // 화면에 UI를 그리는 메소드. 그려질 위젯을 반환
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // 머티리얼 디자인 기본 뼈대 위젯
      appBar: AppBar(
        // 상단 앱바
        title: Text(widget.title),
      ),
      body: Center(
        // 표시할 내용
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter', // _counter 변수를 표시
              style: Theme.of(context).textTheme.headline4,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter, // 클릭 시 _incrementCounter() 메소드 실행
        tooltip: 'Increment',
        child: Icon(Icons.add), // 상단 앱바
      ),
    );
  }
}
```

# 참고 도서

오준석의 플러터 생존코딩