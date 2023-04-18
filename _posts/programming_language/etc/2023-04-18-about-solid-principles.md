---
title: "SOLID : 객체 지향 설계의 5원칙"
date: 2023-04-18
categories: [Programming Language, etc_pl]
tags: [til, oop]
---

# 개요

최근 몇 주간 42에서 C++을 사용하여 웹서버를 개발하는 과제를 하고 있다. 오랫동안 C만을 사용해 온 나에게 C++의 객체 지향 프로그래밍은 생소한 개념이었다. 따라서 객체 지향이나 설계에 대한 고려 없이 클래스를 작성하게 되었고, 그 결과로 너무 많은 역할을 가진 클래스를 만들게 되었다. 이 상황에서 OOP의 설계 원칙에 대한 공부의 필요성을 느꼈고 SOLID에 대해 알게 되었다.

# SOLID 5원칙

SOLID는 개발자가 객체 지향 프로그래밍(OOP)을 사용하여 소프트웨어 애플리케이션을 설계하고 구현하는 데 도움이 되는 일련의 원칙들의 두문자어이다. 이 원칙들은 2000년대 초 Robert C. Martin의 연구에 기초하여 Michael Feathers에 의해 명명된 후 소프트웨어 개발 업계에서 널리 채택되어왔다.

SOLID에 내포된 다섯 가지 원칙은 다음과 같다.

- **S**ingle Responsibility Principle (SRP, 단일 책임 원칙)
- **O**pen/Closed Principle (OCP, 개방-폐쇄 원칙)
- **L**iskov Substitution Principle (LSP, 리스코프 치환 원칙)
- **I**nterface Segregation Principle (ISP, 인터페이스 분리 원칙)
- **D**ependency Inversion Principle (DIP, 의존관계 역전 원칙)

## Single Responsibility Principle (SRP)

SRP에 따르면 클래스는 **변경해야 할 이유가 하나**만 있어야 한다. 즉, 각 클래스는 **한 가지 역할**만을 맡아야 한다. 이렇게 하면 코드의 한 부분을 변경해도 다른 부분에 영향을 주지 않기 때문에 코드의 유지보수가 더욱 수월해진다.

예를 들어 다음과 같이 이메일 프로그램을 개발한다고 할 때, 사용자 인증을 처리하는 영역(`UserAuthenticator`)과 이메일을 전송하는 영역(`EmailSender`)은 **별개의 클래스**로 존재해야 한다.

```cpp
class UserAuthenticator {
 public:
  bool authenticateUser(std::string username, std::string password) {
    // 사용자 인증
  }
};

class EmailSender {
 public:
  void sendEmail(std::string recipient, std::string subject, std::string body) {
    // 이메일 전송
  }
};

class EmailProgram {
 private:
  UserAuthenticator authenticator;
  EmailSender sender;
 public:
  void login(std::string username, std::string password) {
    if (authenticator.authenticateUser(username, password)) {
      // 로그인 성공
    } else {
      // 로그인 실패
    }
  }

  void sendEmail(std::string recipient, std::string subject, std::string body) {
    sender.sendEmail(recipient, subject, body);
  }
};
```

## Open/Closed Principle (OCP)

OCP에 따르면 클래스는 **확장에 열려**있는 반면 **수정에는 닫혀**있어야 한다. 그 말인 즉슨, 새로운 기능은 기존의 코드를 수정하는 것이 아니라 **확장을 통해 추가**되어야 한다는 것이다. 이를 통해 코드가 유연해지며 유지보수하기도 쉬워진다.

예를 들어 이커머스 애플리케이션에서 새로운 결제 수단(PayPal)을 추가해야하는 경우, 기존 코드(`CreditCardPayment`)를 수정하는 것이 아니라 기존의 결제 수단 클래스를 확장하는 새로운 클래스(`PayPalPayment`)를 만들어야 한다.

```cpp
class PaymentMethod {
 public:
  virtual void processPayment(double amount) = 0;
};

class CreditCardPayment : public PaymentMethod {
 public:
  void processPayment(double amount) override {
    // 신용카드 결제
  }
};

class PayPalPayment : public PaymentMethod {
 public:
  void processPayment(double amount) override {
    // 페이팔 결제
  }
};
```

## Liskov Substitution Principle (LSP)

LSP에 따르면 프로그램의 정확성에 영향을 주지 않고 상위 클래스의 객체를 하위 클래스의 객체로 대체할 수 있어야 한다. 즉, 하위 클래스는 **상위 클래스의 동작을 변경해서는 안된다.** 이를 통해 코드를 쉽게 재사용하고 유지보수할 수 있다.

예를 들어 직사각형의 면적을 구하는 클래스(`Rectangle`)에서 정사각형의 면적을 구하는 하위 클래스(`Square`)가 만들어지는 경우, 새로운 클래스는 기존 클래스의 동작을 변경해서는 안된다.

```cpp
class Shape {
 public:
  virtual double area() = 0;
};

class Rectangle : public Shape {
 public:
  double area() override {
    // 직사각형의 면적 계산
  }
};

class Square : public Shape {
 public:
  double area() override {
    // 정사각형의 면적 계산
  }
};
```

## Interface Segregation Principle (ISP)

ISP에 따르면 클라이언트는 자신이 사용하지 않는 인터페이스에 의존하도록 강요받지 않아야 한다. 다시 말해, 인터페이스는 클라이언트가 **필요로 하는 최소한의 메소드**만을 가져야 한다. 이러한 방식을 사용하면 클라이언트는 자신과 관련된 메소드만 알면 되고, 클래스는 원치않는 동작을 상속받지 않고 여러 인터페이스를 구현할 수 있다.

아래 예시에서 `Printable` 인터페이스는 `print()` 메소드만을 포함하고있으며 `Document`와 `Photo` 각각의 클래스에서 이를 구현하였다. `Printer` 클래스는 `Document`나 `Photo`가 아닌 `Printable` 인터페이스에만 의존하고있기 때문에 코드의 수정과 확장이 용이하다.

```cpp
class Printable {
 public:
  virtual std::string print() = 0;
};

class Document : public Printable {
 public:
  std::string print() override {
    // 문서 프린트
  }
};

class Photo : public Printable {
 public:
  std::string print() override {
    // 사진 프린트
  }
};

class Printer {
 public:
  void printDocument(Printable& printable) {
    std::cout << printable.print() << std::endl;
  }
};
```

## Dependency Inversion Principle (DIP)

DIP에 의하면 상위 계층 모듈은 하위 계층 모듈에 의존해서는 안되고 모두가 **추상화에 의존**해야 한다. 또한 추상화는 세부사항에 의존하면 안되고 **세부사항이 추상화에 의존**해야 한다. 이 원칙을 따르게 되면 더욱 유연하고 재사용 가능하며 테스트 가능한 코드를 작성할 수 있다.

아래 예시에서 `MySQLDatabase` 클래스는 `Database` 클래스를 상속받아서 `saveData()`에 대한 세부 구현을 하는 하위 계층 모듈이다. `DataManager` 클래스는 `MySQLDatabase` 클래스가 아닌 `Database` 추상 클래스에 의존한다. 따라서 이후에 `Database`를 상속받는 또다른 클래스인 `PostgreSQLDatabase`를 만드는 경우에도 `DataManager` 클래스의 생성자로 객체를 넘겨줄 수 있게된다.

```cpp
// 추상 클래스
class Database {
 public:
  virtual void saveData(string data) = 0;
};

// 하위 계층 모듈
class MySQLDatabase : public Database {
 public:
  void saveData(string data) {
    // MySQL 데이터베이스에 저장
  }
};

// 상위 계층 모듈
class DataManager {
 private:
  Database* database;

 public:
  DataManager(Database* database) {
    this->database = database;
  }

  void saveData(string data) {
    database->saveData(data);
  }
};
```

# 결론

이처럼 SOLID 원칙은 개발자가 OOP를 사용하여 고품질의 유지보수 가능한 코드를 작성할 수 있도록 가이드라인을 제공한다. 이러한 원칙을 따름으로써 개발자는 자신의 코드가 수정, 확장, 재사용이 용이하다는 것을 보장할 수 있다. SOLID 원칙을 지금 당장 내 프로젝트에 전반적으로 적용하기 위해서는 상당한 노력이 필요하겠지만, 이를 통해 애플리케이션을 더욱 강건하고 신뢰할 수 있도록 만들 수 있다고 하니 점차적으로 적용해 나갈 생각이다.

# 참고
- [SOLID - Wikipedia](https://en.wikipedia.org/wiki/SOLID)
- [SOLID: The First 5 Principles of Object Oriented Design  \| DigitalOcean](https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)
- [Introduction To SOLID Principles](https://www.c-sharpcorner.com/article/introduction-of-solid-principles/)
- [SOLID Principles — explained with examples \| by Raj Suvariya \| MindOrks \| Medium](https://medium.com/mindorks/solid-principles-explained-with-examples-79d1ce114ace)
