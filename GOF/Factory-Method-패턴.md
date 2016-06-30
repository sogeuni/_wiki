# Factory Method 패턴

객체를 생성하기 위한 인터페이스를 정의하는데, 어떤 클래스의 인스턴스를 만들지는 서브클래스에서 결정하게 만듭니다. Factory Method 패턴을 이용하면 클래스의 인스턴스를 만드는 일을 서브클래스에게 맡기게 됩니다.

@startuml

interface Product <|-- ConcreteProduct
ConcreteProduct <. ConcreteCreator
Creator <|-- ConcreteCreator

class Creator {
    {Abstract} +factoryMethod()
    +anOperation()
}

class ConcreteCreator {
    +factoryMethod()
}

hide empty methods
hide empty fields

@enduml

* **Product**: `factoryMethod()`가 생성하는 객체의 인터페이스를 정의합니다.
* **ConcreteProduct**: **Product** 클래스에 정의된 인터페이스를 실제로 구현합니다. **ConcreateProduct** 클래스에서는 모두 똑같은 인터페이스를 구현해야 합니다. 그래야만 그 제품을 사용할 클래스에서 구상 클래스가 아닌 인터페이스에 대한 레퍼런스를 사용하여 참조가 가능합니다.
* **Creator**: **Pruduct**를 가지고 원하는 일을 하기 위한 모든 메소드를 구현하고 있습니다. 하지만, 제품을 만들어 주는 `factoryMethod()`는 추상 메소드로 가지고 있습니다.
* **ConcreteCreator**: 실제로 제품을 생산하는 `factoryMethod()`를 구현합니다.