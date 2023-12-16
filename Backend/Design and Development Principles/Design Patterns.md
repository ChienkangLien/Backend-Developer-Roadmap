# Design Patterns
## 剖析型模式（Creational Patterns）
關注對象的創建機制
### 工廠方法（Factory Method）
提供一個統一的介面來創建物件，而不暴露創建物件的邏輯。

使用場景： 
1. 日志記錄器：記錄可能記錄到本地硬盤、系統事件、遠程服務器等，用戶可以選擇記錄日志到什麽地方。
2. 數據庫訪問，當用戶不知道最後系統采用哪一類數據庫，以及數據庫可能有變化時。 
3. 設計一個連接服務器的框架，需要三個協議，"POP3"、"IMAP"、"HTTP"，可以把這三個作為產品類，共同實現一個接口。

假設有不同型別的汽車（例如轎車、卡車和摩托車），我們使用工廠模式來創建這些汽車的實例：
```java=
// 定義汽車介面
interface Car {
    void drive();
}

// 不同型別的汽車
class Sedan implements Car {
    public void drive() {
        System.out.println("Sedan car is driving.");
    }
}

class Truck implements Car {
    public void drive() {
        System.out.println("Truck is driving.");
    }
}

class Motorcycle implements Car {
    public void drive() {
        System.out.println("Motorcycle is driving.");
    }
}

// 汽車工廠
class CarFactory {
    public Car createCar(String carType) {
        if (carType.equalsIgnoreCase("sedan")) {
            return new Sedan();
        } else if (carType.equalsIgnoreCase("truck")) {
            return new Truck();
        } else if (carType.equalsIgnoreCase("motorcycle")) {
            return new Motorcycle();
        }
        return null;
    }
}

// 客戶端代碼
public class Main {
    public static void main(String[] args) {
        CarFactory factory = new CarFactory();

        Car sedan = factory.createCar("sedan");
        sedan.drive(); // 輸出：Sedan car is driving.

        Car truck = factory.createCar("truck");
        truck.drive(); // 輸出：Truck is driving.

        Car motorcycle = factory.createCar("motorcycle");
        motorcycle.drive(); // 輸出：Motorcycle is driving.
    }
}
```
Car 是汽車介面，Sedan、Truck 和 Motorcycle 是不同型別的具體汽車類。CarFactory 是負責創建汽車的工廠類，它根據傳入的參數創建對應型別的汽車物件，並呼叫其 drive() 方法。
### 抽象工廠(Abstract Factory)
提供一種創建相關或相互依賴對象家族的方式，而無需指定其具體類型。它屬於工廠模式的一種延伸，以用於創建一系列相關的產品。

在抽象工廠模式中，有兩個抽象層次：抽象工廠和具體工廠。抽象工廠定義了創建相關對象的介面，而具體工廠實現了這個介面，以創建具體的產品。每個具體工廠都可以創建一個完整的產品家族，而這個產品家族通常彼此有關聯。

使用場景： 
1. 人物換造型，一整套一起換。 
2. 生成不同操作系統的程序。

假設我們有兩種不同的形狀（Square 和 Circle）和兩種不同的顏色（Red 和 Blue），我們需要一個工廠來創建這些形狀和顏色的對象。
```java=
// 定義形狀介面
interface Shape {
    void draw();
}

// 不同型別的形狀
class Square implements Shape {
    public void draw() {
        System.out.println("Inside Square::draw() method.");
    }
}

class Circle implements Shape {
    public void draw() {
        System.out.println("Inside Circle::draw() method.");
    }
}

// 定義顏色介面
interface Color {
    void fill();
}

// 不同型別的顏色
class Red implements Color {
    public void fill() {
        System.out.println("Inside Red::fill() method.");
    }
}

class Blue implements Color {
    public void fill() {
        System.out.println("Inside Blue::fill() method.");
    }
}

// 抽象工廠
interface AbstractFactory {
    Shape getShape(String shapeType);
    Color getColor(String colorType);
}

// 具體工廠 Shape Factory
class ShapeFactory implements AbstractFactory {
    public Shape getShape(String shapeType) {
        if (shapeType == null) {
            return null;
        }
        if (shapeType.equalsIgnoreCase("SQUARE")) {
            return new Square();
        } else if (shapeType.equalsIgnoreCase("CIRCLE")) {
            return new Circle();
        }
        return null;
    }

    public Color getColor(String colorType) {
        return null;
    }
}

// 具體工廠 Color Factory
class ColorFactory implements AbstractFactory {
    public Shape getShape(String shapeType) {
        return null;
    }

    public Color getColor(String colorType) {
        if (colorType == null) {
            return null;
        }
        if (colorType.equalsIgnoreCase("RED")) {
            return new Red();
        } else if (colorType.equalsIgnoreCase("BLUE")) {
            return new Blue();
        }
        return null;
    }
}

// 超級工廠 Factory Producer
class FactoryProducer {
    public static AbstractFactory getFactory(String choice) {
        if (choice.equalsIgnoreCase("SHAPE")) {
            return new ShapeFactory();
        } else if (choice.equalsIgnoreCase("COLOR")) {
            return new ColorFactory();
        }
        return null;
    }
}

public class Main {
    public static void main(String[] args) {
        // Get shape factory
        AbstractFactory shapeFactory = FactoryProducer.getFactory("SHAPE");

        // Get an object of Shape Square
        Shape square = shapeFactory.getShape("SQUARE");
        square.draw(); // Output: Inside Square::draw() method.

        // Get color factory
        AbstractFactory colorFactory = FactoryProducer.getFactory("COLOR");

        // Get an object of Color Red
        Color red = colorFactory.getColor("RED");
        red.fill(); // Output: Inside Red::fill() method.
    }
}
```
AbstractFactory 定義了創建形狀和顏色對象的方法，ShapeFactory 和 ColorFactory 分別實現了這個介面並創建了對應的具體對象。FactoryProducer 用於獲取工廠，然後可以使用工廠來獲得所需的產品對象。
### 建造者(Builder)
將複雜對象的構建過程與它的內部屬性和結構分離開來。
1. Builder（建造者）：定義了創建產品的步驟接口。
2. ConcreteBuilder（具體建造者）：實現了Builder接口，負責構建產品的各個部分。
3. Product（產品）：最終的產品對象，通常由多個部分構成。
4. Director（導演）：負責使用Builder接口來創建產品，不直接創建產品，而是通過Builder間接創建。

使用場景： 
1. 需要生成的對象具有覆雜的內部結構。 
2. 需要生成的對象內部屬性本身相互依賴。

假設我們有一個建造汽車的例子，汽車擁有多個部件，包括引擎、車輪、座椅等。我們將使用Builder模式來建造汽車物件。
```java=
// 產品類別: 汽車
class Car {
    private String engine;
    private String wheels;
    private String seats;

    // getter & setter...
    
    public void showInfo() {
        System.out.println("引擎：" + engine);
        System.out.println("輪子：" + wheels);
        System.out.println("座椅：" + seats);
    }
}

// 建造者介面
interface CarBuilder {
    void buildEngine();
    void buildWheels();
    void buildSeats();
    Car getCar();
}

// 具體建造者
class SportsCarBuilder implements CarBuilder {
    private Car car;

    public SportsCarBuilder() {
        car = new Car();
    }

    public void buildEngine() {
        car.setEngine("高性能引擎");
    }

    public void buildWheels() {
        car.setWheels("大尺寸輪胎");
    }

    public void buildSeats() {
        car.setSeats("賽車座椅");
    }

    public Car getCar() {
        return car;
    }
}

// 導演
class CarDirector {
    public Car buildCar(CarBuilder builder) {
        builder.buildEngine();
        builder.buildWheels();
        builder.buildSeats();
        return builder.getCar();
    }
}

public class Main {
    public static void main(String[] args) {
        CarDirector director = new CarDirector();
        CarBuilder sportsCarBuilder = new SportsCarBuilder();

        Car sportsCar = director.buildCar(sportsCarBuilder);
        sportsCar.showInfo(); // 輸出: 引擎：高性能引擎，輪子：大尺寸輪胎，座椅：賽車座椅
    }
}
```
Car 是最終的產品，代表了一輛汽車。CarBuilder 定義了建造汽車的方法，SportsCarBuilder 是具體的建造者，實現了建造方法以構建不同類型的汽車。CarDirector 作為導演，負責使用建造者來創建汽車。通過Builder模式，我們可以將汽車的建造過程封裝在建造者中。
### 原型(Prototype)
通過複製現有對象來創建新對象，而無需依賴常規的構造器。這使得我們可以動態地創建對象，同時避免了在程序中直接將新對象和其相關屬性硬編碼在代碼中。
1. Prototype（原型）：定義了克隆自身的方法(或是implements Cloneable)。
2. ConcretePrototype（具體原型）：實現了Prototype接口，並實現了克隆方法。
3. Client（客戶端）：使用Prototype模式的客戶端，通常使用克隆方法來創建新對象。

使用場景： 
1. 資源優化場景。 
2. 類初始化需要消化非常多的資源，這個資源包括數據、硬件資源等。 
3. 性能和安全要求的場景。 
4. 通過 new 產生一個對象需要非常繁瑣的數據準備或訪問權限，則可以使用原型模式。 
5. 一個對象多個修改者的場景。 
6. 一個對象需要提供給其他對象訪問，而且各個調用者可能都需要修改其值時，可以考慮使用原型模式拷貝多個對象供調用者使用。
7. 在實際項目中，原型模式很少單獨出現，一般是和工廠方法模式一起出現，通過 clone 的方法創建一個對象，然後由工廠方法提供給調用者。原型模式已經與 Java 融為渾然一體，大家可以隨手拿來使用。

```java=
// Prototype（原型）介面
interface Prototype {
    Prototype clone(); // 克隆方法
    String getName();
    void setName(String name);
}

// ConcretePrototype（具體原型）類別
class ConcretePrototype implements Prototype {
    private String name;

    public Prototype clone() {
        ConcretePrototype clone = new ConcretePrototype();
        clone.setName(this.name);
        return clone;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

// Client（客戶端）
public class Main {
    public static void main(String[] args) {
        ConcretePrototype original = new ConcretePrototype();
        original.setName("原始對象");

        // 通過克隆方法創建新對象
        ConcretePrototype cloned = (ConcretePrototype) original.clone();
        cloned.setName("克隆對象");

        System.out.println(original.getName()); // 輸出: 原始對象
        System.out.println(cloned.getName());   // 輸出: 克隆對象
    }
}
```
Prototype 是原型接口，定義了克隆方法和設置名稱的方法。ConcretePrototype 是具體的原型類，實現了克隆方法和其他相關方法。Client 是客戶端，使用Prototype模式來克隆原型對象以創建新對象。透過Prototype模式，我們可以在運行時動態地創建對象。
### 單例(Singleton)
確保一個類別僅有一個實例存在，並提供一個全域訪問點。

使用場景：
1. 要求生產唯一序列號。
2. WEB 中的計數器，不用每次刷新都在數據庫里加一次，用單例先緩存起來。
3. 創建的一個對象需要消耗的資源過多，比如 I/O 與數據庫的連接等。

創建方式：
1. 餓漢式（Eager Initialization）：在類別加載時就創建實例。
```java=
public class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return instance;
    }
}
```
2. 懶漢式（Lazy Initialization）：在第一次調用時才創建實例。
```java=
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
3. 雙重檢查（Double-Checked Locking）：在調用時進行雙重檢查，避免每次都進入同步塊。
```java=
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
## 結構型模式（Structural Patterns）
關注對象之間的結構
### 適配器(Adapter)
將一個類的介面轉換成客戶端所期望的另一個介面。它允許原本不兼容的類可以一起工作，通常用於解決新舊系統介面不一致的問題。
1. 目標介面（Target Interface）：客戶端所期望使用的介面。
2. 適配器（Adapter）：實現了目標介面，同時包裝了被適配者，將其介面轉換為客戶端所需的介面。
3. 被適配者（Adaptee）：需要被適配的類或介面。

使用場景：
有動機地修改一個正常運行的系統的接口，這時應該考慮使用適配器模式。

想像你手上有一些外國購買的家電，但是插頭形狀和你當地的插座不相容。這時你可能需要一個插頭轉接器（適配器），讓這些外國的電器能夠在你的家中使用。

1. 物件適配器：使用組合的方式實現適配器。
```java=
// 目標介面（Target）
interface LocalSocket {
    void plugIn();
}

// 被適配者（Adaptee）
class ForeignAppliance {
    void foreignPlug() {
        System.out.println("Foreign appliance plug");
    }
}

// 適配器（Adapter）
class PlugAdapter implements LocalSocket {
    private ForeignAppliance foreignAppliance;

    public PlugAdapter(ForeignAppliance foreignAppliance) {
        this.foreignAppliance = foreignAppliance;
    }

    public void plugIn() {
        foreignAppliance.foreignPlug();
        System.out.println("Adapter converts foreign plug to local socket");
    }
}

// Client（客戶端）
public class Main {
    public static void main(String[] args) {
        // 假設有一個外國購買的家電
        ForeignAppliance foreignAppliance = new ForeignAppliance();

        // 創建一個插頭轉接器
        PlugAdapter adapter = new PlugAdapter(foreignAppliance);

        // 使用轉接器插入當地插座
        adapter.plugIn();
    }
}
```
LocalSocket 是目標介面，表示你當地的插座。ForeignAppliance 是被適配者，代表外國購買的家電。PlugAdapter 是適配器，將外國家電的插頭形狀轉換成你當地插座所需的形狀。

2. 類別適配器：使用繼承的方式實現適配器。
```java=
// 目標介面（Target）
interface LocalSocket {
    void plugIn();
}

// 被適配者（Adaptee）
class ForeignAppliance {
    void foreignPlug() {
        System.out.println("Foreign appliance plug");
    }
}

// 適配器（Adapter）
class PlugAdapter extends ForeignAppliance implements LocalSocket {
    public void plugIn() {
        foreignPlug();
        System.out.println("Adapter converts foreign plug to local socket");
    }
}

// Client（客戶端）
public class Main {
    public static void main(String[] args) {
        // 創建一個插頭轉接器
        LocalSocket adapter = new PlugAdapter();

        // 使用轉接器插入當地插座
        adapter.plugIn();
    }
}
```
PlugAdapter 類別同時擁有 LocalSocket 目標介面和 ForeignAppliance 被適配者的功能。PlugAdapter 直接繼承 ForeignAppliance，同時實現了 LocalSocket 介面。

當客戶端需要將外國購買的家電插入當地的插座時，創建一個插頭轉接器（適配器），然後使用轉接器將外國家電插入當地的插座。這樣，適配器就實現了兩者之間的連接，讓不相容的部分可以一起工作。
### 橋接(Bridge)
將抽象部分與其具體實現部分分離，使它們可以獨立地變化。
1. 抽象部分（Abstraction）：定義了抽象介面，並保存了對實現部分的引用。
2. 具體抽象部分（Refined Abstraction）：擴展了抽象部分，並增加了更多功能。
3. 實現部分（Implementor）：定義了實現部分的介面，供抽象部分調用。
4. 具體實現部分（Concrete Implementor）：實現了實現部分的具體類。

使用場景： 
1. 如果一個系統需要在構件的抽象化角色和具體化角色之間增加更多的靈活性，避免在兩個層次之間建立靜態的繼承聯系，通過橋接模式可以使它們在抽象層建立一個關聯關系。 
2. 對於那些不希望使用繼承或因為多層次繼承導致系統類的個數急劇增加的系統，橋接模式尤為適用。 
3. 一個類存在兩個獨立變化的維度，且這兩個維度都需要進行擴展。

以圖形和不同顏色的繪製為例。圖形有多種類型（如圓形、方形），同時可以擁有不同的顏色（如紅色、藍色）。
```java=
// 實現部分的介面（Implementor）
interface Color {
    void applyColor();
}

// 具體實現部分（Concrete Implementor）
class RedColor implements Color {
    public void applyColor() {
        System.out.println("Applying red color");
    }
}

class BlueColor implements Color {
    public void applyColor() {
        System.out.println("Applying blue color");
    }
}

// 抽象部分（Abstraction）
abstract class Shape {
    protected Color color;

    public Shape(Color color) {
        this.color = color;
    }

    abstract void draw();
}

// 具體抽象部分（Refined Abstraction）
class Circle extends Shape {
    public Circle(Color color) {
        super(color);
    }

    public void draw() {
        System.out.print("Drawing circle: ");
        color.applyColor();
    }
}

class Square extends Shape {
    public Square(Color color) {
        super(color);
    }

    public void draw() {
        System.out.print("Drawing square: ");
        color.applyColor();
    }
}

// Client（客戶端）
public class Main {
    public static void main(String[] args) {
        Color red = new RedColor();
        Color blue = new BlueColor();

        Shape redCircle = new Circle(red);
        Shape blueSquare = new Square(blue);

        redCircle.draw(); // Drawing circle: Applying red color
        blueSquare.draw(); // Drawing square: Applying blue color
    }
}
```
Shape 是抽象部分，Color 是實現部分。Circle 和 Square 是具體抽象部分，分別擴展了 Shape。RedColor 和 BlueColor 是具體實現部分，實現了 Color 介面。透過橋接模式，圖形和顏色的組合可以獨立變化，同時使它們可以組合使用，而不必修改現有程式碼。
### 組合(Composite)
將物件以樹狀結構組織成部分-整體的層次結構，使得使用者可以統一地處理個別物件和組合物件。
1. Component（元件）：定義了組合中所有物件的通用介面。它可以是介面或抽象類別，包含了所有物件共有的操作。
2. Leaf（葉節點）：表示組合中的葉子節點物件，沒有子節點。
3. Composite（複合節點）：表示組合中的複合節點物件，可以包含子節點。

使用場景：
部分、整體場景，如樹形菜單，文件、文件夾的管理。

想像一個組織架構，例如一家公司，它可以被視為組合模式的一個例子：
```java=
// Component（元件）：部門
interface Department {
    void work();
}

// Leaf（葉節點）：員工
class Employee implements Department {
    private String name;

    public Employee(String name) {
        this.name = name;
    }

    public void work() {
        System.out.println(name + " is working");
    }
}

// Composite（複合節點）
class CompositeDepartment implements Department {
    private List<Department> children = new ArrayList<>();

    public void addDepartment(Department department) {
        children.add(department);
    }

    public void removeDepartment(Department department) {
        children.remove(department);
    }

    public void work() {
        for (Department department : children) {
            department.work();
        }
    }
}

// Client（客戶端）
public class Main {
    public static void main(String[] args) {
        Department marketing = new CompositeDepartment();
        marketing.addDepartment(new Employee("Alice"));
        marketing.addDepartment(new Employee("Bob"));

        Department development = new CompositeDepartment();
        development.addDepartment(new Employee("Charlie"));
        development.addDepartment(new Employee("David"));

        CompositeDepartment company = new CompositeDepartment();
        company.addDepartment(marketing);
        company.addDepartment(development);

        company.work();
    }
}
```
Department 是所有部門和員工的通用介面，Employee 是葉節點表示員工，CompositeDepartment 是複合節點表示公司的部門。當work()方法被調用時，公司會統一調用其所包含的子部門或員工的work()方法。
### 裝飾(Decorator)
動態地為物件添加額外的功能，而不需要修改其原始類別的結構。
1. Component（元件）：定義了被裝飾的物件的介面，它可以是介面或抽象類別，包含了所有被裝飾物件共有的操作。
2. ConcreteComponent（具體元件）：實現了Component介面，代表原始的物件。
3. Decorator（裝飾者）：繼承Component介面，同時保存了一個指向Component的參考。
4. ConcreteDecorator（具體裝飾者）：繼承了Decorator，表示具體的裝飾物件。它可以為Component添加額外的功能。

使用場景： 
1. 擴展一個類的功能。
2. 動態增加功能，動態撤銷。

假設有一個咖啡店，咖啡是一個基本的元件，可以被裝飾，例如添加牛奶、糖或奶泡。
```java=
// Component（元件）
interface Coffee {
    void brew();
}

// ConcreteComponent（具體元件）
class SimpleCoffee implements Coffee {
    public void brew() {
        System.out.println("Brewing simple coffee");
    }
}

// Decorator（裝飾者）
class abstract CoffeeDecorator implements Coffee {
    private Coffee coffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
    
    public void brew() {
        coffee.brew();
    }
}

// ConcreteDecorator（具體裝飾者）
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
  
    public void brew() {
        super.brew();
        addMilk();
    }

    private void addMilk() {
        System.out.println("Adding milk");
    }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }

    public void brew() {
        super.brew();
        addSugar();
    }

    private void addSugar() {
        System.out.println("Adding sugar");
    }
}

// Client（客戶端）
public class Main {
    public static void main(String[] args) {
        Coffee simpleCoffee = new SimpleCoffee();
        Coffee milkCoffee = new MilkDecorator(simpleCoffee);
        Coffee sugarMilkCoffee = new SugarDecorator(milkCoffee);

        sugarMilkCoffee.brew();
    }
}
```
Coffee 是元件介面，SimpleCoffee 是具體元件。CoffeeDecorator 是裝飾者，MilkDecorator 和 SugarDecorator 是具體裝飾者，它們可以為咖啡添加額外的功能（牛奶、糖）。當brew()方法被調用時，具體裝飾者會先調用被裝飾物件的brew()方法，然後再添加額外的功能。
### 外觀(Facade)
提供一個統一的接口，用來訪問子系統中的一組介面。
1. Facade（外觀）：為客戶端提供了一個簡單的介面，用來訪問子系統中的一組功能。
2. Subsystem（子系統）：包含了若干個相關的類別或功能。外觀模式將這些類別進行封裝，並提供統一的介面。

使用場景： 
1. 為覆雜的模塊或子系統提供外界訪問的模塊。 
2. 子系統相對獨立。 
3. 預防低水平人員帶來的風險。

一個家庭劇院系統，它包括多個子系統：音響系統、投影系統、燈光控制等等。外觀模式可以將這些子系統封裝起來，提供一個統一的介面供用戶端操作。
```java=
// Subsystem 1（子系統1）
class AudioSystem {
    public void turnOn() {
        System.out.println("Audio system turned on");
    }

    public void turnOff() {
        System.out.println("Audio system turned off");
    }
}

// Subsystem 2（子系統2）
class Projector {
    public void turnOn() {
        System.out.println("Projector turned on");
    }

    public void turnOff() {
        System.out.println("Projector turned off");
    }
}

// Subsystem 3（子系統3）
class LightControl {
    public void dim() {
        System.out.println("Lights dimmed");
    }

    public void brighten() {
        System.out.println("Lights brightened");
    }
}

// Facade（外觀）
class HomeTheaterFacade {
    private AudioSystem audioSystem;
    private Projector projector;
    private LightControl lightControl;

    public HomeTheaterFacade() {
        this.audioSystem = new AudioSystem();
        this.projector = new Projector();
        this.lightControl = new LightControl();
    }

    public void watchMovie() {
        audioSystem.turnOn();
        projector.turnOn();
        lightControl.dim();
    }

    public void endMovie() {
        audioSystem.turnOff();
        projector.turnOff();
        lightControl.brighten();
    }
}

// Client（客戶端）
public class Main {
    public static void main(String[] args) {
        HomeTheaterFacade homeTheater = new HomeTheaterFacade();

        // Watch a movie
        homeTheater.watchMovie();

        // End the movie
        homeTheater.endMovie();
    }
}
```
HomeTheaterFacade 封裝了音響系統、投影系統和燈光控制等子系統的操作，為用戶端提供了 watchMovie() 和 endMovie() 兩個簡單的介面。用戶端只需使用外觀介面來啟動和關閉家庭劇院系統，而不需要了解或直接操作子系統中的細節。
### 享元(Flyweight)
最大程度地減少內存使用或計算開銷，通過共享相似的物件以減少系統中物件的數量。
1. Flyweight（享元）：描述物件需要共享的部分，通常包含多個物件共享的內部狀態。
2. ConcreteFlyweight（具體享元）：實現了享元介面，並實現共享方法。對於可以共享的狀態，它保留了內部狀態。
3. FlyweightFactory（享元工廠）：負責創建和管理享元物件。它可以維護一個享元池，用於存儲已創建的享元物件。

使用場景：
1. 系統有大量相似對象。
2. 需要緩沖池的場景。

有一個文字編輯器，它需要處理大量的文本內容。為了減少內存佔用，可以使用享元模式來共享字符的狀態。
```java=
// Flyweight（享元）
interface TextCharacter {
    void printCharacter();
}

// ConcreteFlyweight（具體享元）
class ConcreteCharacter implements TextCharacter {
    private char symbol;

    public ConcreteCharacter(char symbol) {
        this.symbol = symbol;
    }

    public void printCharacter() {
        System.out.println("Character: " + symbol);
    }
}

// FlyweightFactory（享元工廠）
class CharacterFactory {
    private Map<Character, ConcreteCharacter> characters = new HashMap<>();

    public TextCharacter getCharacter(char symbol) {
        if (!characters.containsKey(symbol)) {
            characters.put(symbol, new ConcreteCharacter(symbol));
        }
        return characters.get(symbol);
    }
}

// Client（客戶端）
public class Main {
    public static void main(String[] args) {
        CharacterFactory factory = new CharacterFactory();

        TextCharacter charA = factory.getCharacter('A');
        charA.printCharacter(); // Character: A

        TextCharacter charB = factory.getCharacter('B');
        charB.printCharacter(); // Character: B

        TextCharacter charC = factory.getCharacter('A');
        charC.printCharacter(); // Character: A (reused)
    }
}
```
ConcreteCharacter 是具體享元，代表一個文字字符。CharacterFactory 是享元工廠，用於管理和創建享元物件。在客戶端中，通過享元工廠可以獲取享元物件。當創建相同字符的時候，如果已經存在該字符的享元物件，則直接返回該物件，實現了共享。
### 代理(Proxy)
提供一個代理物件，控制對於其他物件的訪問。
1. Subject（主題）：定義了RealSubject和Proxy的共用接口，這樣就可以在任何使用RealSubject的地方使用Proxy。
2. RealSubject（真實主題）：代表了真實的物件，是代理所代表的真正物件。
3. Proxy（代理）：保持對RealSubject的引用，並提供與RealSubject相同的介面，使得代理可以代替RealSubject進行控制或附加額外的行為。

使用場景：
1. 遠端代理：用於在本地代表遠端物件，例如在網路上代表一個物件。
2. 虛擬代理：延遲實例化，用於需要大量計算或者大量資源的物件。例如，圖片加載的例子，就是在需要時才實際加載圖片，避免一開始就全部加載造成資源浪費。
3. 保護代理：用於控制對於物件的訪問權限。例如，設置文件的存取權限，只有特定用戶才能訪問。
4. 智能引用：在物件被訪問時，做一些額外的動作。例如，日誌紀錄。

以網路圖片加載為例，當需要加載一張圖片時，可以使用代理模式來控制圖片的加載行為。
```java=
// Subject（主題）
interface Image {
    void display();
}

// RealSubject（真實主題）
class RealImage implements Image {
    private String filename;

    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }

    private void loadFromDisk() {
        System.out.println("Loading image: " + filename);
    }

    public void display() {
        System.out.println("Displaying image: " + filename);
    }
}

// Proxy（代理）
class ProxyImage implements Image {
    private RealImage realImage;
    private String filename;

    public ProxyImage(String filename) {
        this.filename = filename;
    }

    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename);
        }
        realImage.display();
    }
}

// Client（客戶端）
public class Main {
    public static void main(String[] args) {
        Image image = new ProxyImage("example.jpg");

        // 第一次需加載
        image.display();

        // 不需再加載
        image.display();
    }
}
```
RealImage 是真實的圖片物件，ProxyImage 是代理。當需要顯示圖片時，客戶端可能會直接使用 ProxyImage 進行操作。如果圖片還沒有被加載，ProxyImage 會負責加載圖片，否則它會直接調用 RealImage 來顯示圖片。這樣做可以控制圖片的加載行為，並且能夠在需要時延遲圖片的實際加載。
## 行為型模式（Behavioral Patterns）
關注對象之間的交互和分配職責
### 責任鏈(Chain of Responsibility)
建立一個項目的處理鏈；當接收到請求時，這個請求會依序被一系列物件處理，直到其中一個物件處理該請求為止。
1. Handler（處理者）：定義了處理請求的介面，並保持對下一個處理者的引用。
2. ConcreteHandler（具體處理者）：實現了Handler介面，負責處理特定類型或範圍的請求。如果能處理該請求，就處理它；否則將請求傳遞給下一個處理者。
3. Client（客戶端）：創建一個處理者鏈，並將請求發送到這個鏈上的第一個處理者。

使用場景： 
1. 有多個對象可以處理同一個請求，具體哪個對象處理該請求由運行時刻自動確定。
2. 在不明確指定接收者的情況下，向多個對象中的一個提交一個請求。
3. 可動態指定一組對象處理請求。

以購買商品的例子來說明責任鏈模式。假設有不同級別的折扣，當客戶發出購買請求時，會根據不同等級的折扣來處理該請求。
```java=
// Handler（處理者）
interface DiscountHandler {
    void setNextHandler(DiscountHandler handler);
    void applyDiscount(double amount);
}

// ConcreteHandler（具體處理者）
class SilverDiscountHandler implements DiscountHandler {
    private DiscountHandler nextHandler;

    public void setNextHandler(DiscountHandler handler) {
        nextHandler = handler;
    }

    public void applyDiscount(double amount) {
        if (amount <= 100) {
            System.out.println("Silver discount applied: " + amount * 0.1);
        } else if (nextHandler != null) {
            nextHandler.applyDiscount(amount);
        }
    }
}

class GoldDiscountHandler implements DiscountHandler {
    private DiscountHandler nextHandler;

    public void setNextHandler(DiscountHandler handler) {
        nextHandler = handler;
    }

    public void applyDiscount(double amount) {
        if (amount > 100 && amount <= 500) {
            System.out.println("Gold discount applied: " + amount * 0.2);
        } else if (nextHandler != null) {
            nextHandler.applyDiscount(amount);
        }
    }
}

class PlatinumDiscountHandler implements DiscountHandler {
    public void setNextHandler(DiscountHandler handler) {
        // This is the end of the chain, no next handler
    }

    public void applyDiscount(double amount) {
        if (amount > 500) {
            System.out.println("Platinum discount applied: " + amount * 0.3);
        } else {
            System.out.println("No further discount applied");
        }
    }
}

// Client（客戶端）
public class Main {
    public static void main(String[] args) {
        DiscountHandler silverHandler = new SilverDiscountHandler();
        DiscountHandler goldHandler = new GoldDiscountHandler();
        DiscountHandler platinumHandler = new PlatinumDiscountHandler();

        silverHandler.setNextHandler(goldHandler);
        goldHandler.setNextHandler(platinumHandler);

        // Simulate purchase with different amounts
        silverHandler.applyDiscount(50); // Silver discount applied: 5.0
        silverHandler.applyDiscount(200); // Gold discount applied: 40.0
        silverHandler.applyDiscount(1000); // Platinum discount applied: 300.0
    }
}
```
每個具體處理者（SilverDiscountHandler、GoldDiscountHandler、PlatinumDiscountHandler）都能處理特定範圍內的請求。如果自己無法處理，則將請求傳遞給下一個處理者，直到找到適合處理該請求的處理者為止。
### 命令(Command)
將請求封裝成物件，從而讓你能夠根據不同的請求對客戶端進行參數化，併應用排隊、日誌等功能。
1. Command（命令）：聲明了執行操作的介面。
2. ConcreteCommand（具體命令）：實現了命令介面，並封裝了接收者的一個或多個動作。
3. Receiver（接收者）：實現了命令的相關操作，是具體命令的執行者。
4. Invoker（調用者）：負責將命令發送給接收者以執行請求。
5. Client（客戶端）：創建一個具體命令並設置其接收者。

使用場景：認為是命令的地方都可以使用命令模式，例如：GUI 中每一個按鈕都是一條命令。模擬 CMD。

以家電遙控器為例來說明命令模式。假設有一個遙控器，上面有數個按鈕，每個按鈕對應著不同的家電操作。
```java=
// Command（命令）
interface Command {
    void execute();
}

// ConcreteCommand（具體命令）
class LightOnCommand implements Command {
    private Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    public void execute() {
        light.turnOn();
    }
}

// Receiver（接收者）
class Light {
    public void turnOn() {
        System.out.println("Light is on");
    }

    public void turnOff() {
        System.out.println("Light is off");
    }
}

// Invoker（調用者）
class RemoteControl {
    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void pressButton() {
        command.execute();
    }
}

// Client（客戶端）
public class Main {
    public static void main(String[] args) {
        Light light = new Light();
        Command lightOnCommand = new LightOnCommand(light);

        RemoteControl remoteControl = new RemoteControl();
        remoteControl.setCommand(lightOnCommand);

        // Press the button to turn on the light
        remoteControl.pressButton(); // Output: Light is on
    }
}
```
Command 是命令介面，LightOnCommand 是具體命令，Light 是接收者。RemoteControl 是調用者，可以設置不同的命令並執行它們。這樣做的好處是，可以動態地改變和擴展命令，並將命令的執行與其發送者解耦。
### 迭代器(Iterator)
允許你在不暴露對象內部結構的情況下，按順序訪問聚合對象中的元素。
1. 迭代器（Iterator）：定義了訪問和遍歷元素的接口。
2. 具體迭代器（ConcreteIterator）：實現了迭代器接口，負責實現對元素的叠代操作。
3. 聚合（Aggregate）：定義創建迭代器對象的接口。
4. 具體聚合（ConcreteAggregate）：實現了聚合接口，返回一個合適的迭代器實例。

使用場景： 
1. 訪問一個聚合對象的內容而無須暴露它的內部表示。
2. 需要為聚合對象提供多種遍歷方式。 
3. 為遍歷不同的聚合結構提供一個統一的接口。

假設我們有一個超市的購物車，我們會使用迭代器來遍歷這個購物車中的商品列表。
```java=
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

// 迭代器（Iterator）
interface Iterator {
    boolean hasNext();
    Object next();
}

// 具體迭代器（ConcreteIterator）
class NameIterator implements Iterator {
    private List<String> list;
    private int index = 0;

    public NameIterator(List<String> list) {
        this.list = list;
    }

    public boolean hasNext() {
        return index < list.size();
    }

    public String next() {
        if (this.hasNext()) {
            return list.get(index++);
        }
        return null;
    }
}

// 聚合（Aggregate）
interface Container {
    Iterator createIterator();
}

// 具體聚合（ConcreteAggregate）
class NameRepository implements Container {
    private List<String> items;

    public ListCollection() {
        items = new ArrayList<>();
    }

    public void addItem(String item) {
        items.add(item);
    }

    public Iterator createIterator() {
        return new NameIterator(items);
    }
}

// 客戶端代碼
public class Main {
    public static void main(String[] args) {
        NameRepository collection = new NameRepository();
        collection.addItem("Robert");
        collection.addItem("John");
        collection.addItem("Julie");

        Iterator iterator = collection.createIterator();

        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```
NameRepository 是具體聚合類，負責添加元素並創建迭代器。NameIterator 是具體迭代器類，實現了對列表元素的迭代操作。在客戶端代碼中，我們創建了一個集合並添加了幾個元素，然後通過迭代器逐個訪問並打印了集合中的元素。
### 中介者(Mediator)
允許對象之間通過中介者對象進行交互，而不必直接相互引用，從而降低了對象之間的耦合性。
1. 中介者（Mediator）：定義了各個對象之間交互的接口。
2. 具體中介者（Concrete Mediator）：實現了中介者接口，協調各個相關對象的行為。
3. 同事類（Colleague）：每個同事類都知道中介者對象，但它們不知道彼此。當一個同事對象改變時，它會通知中介者對象。中介者對象負責處理這些信息，並且可能會轉發給其他同事對象。

使用場景： 
1. 系統中對象之間存在比較覆雜的引用關系，導致它們之間的依賴關系結構混亂而且難以覆用該對象。
2. 想通過一個中間類來封裝多個類中的行為，而又不想生成太多的子類。

以一個聊天室的例子來說明中介者模式。在聊天室中，多個用戶可以相互發送消息。通過中介者模式，用戶並不直接發送消息給其他用戶，而是通過聊天室中介者來處理消息的傳遞。
```java=
import java.util.ArrayList;
import java.util.List;

// 中介者接口
interface ChatMediator {
    void sendMessage(String message, User user);
    void addUser(User user);
}

// 具體中介者
class ChatRoom implements ChatMediator {
    private List<User> users;

    public ChatRoom() {
        this.users = new ArrayList<>();
    }

    public void addUser(User user) {
        users.add(user);
    }

    public void sendMessage(String message, User sender) {
        for (User user : users) {
            if (user != sender) {
                user.receive(message);
            }
        }
    }
}

// 同事類
class User {
    private String name;
    private ChatMediator mediator;

    public User(String name, ChatMediator mediator) {
        this.name = name;
        this.mediator = mediator;
    }

    public void send(String message) {
        mediator.sendMessage(message, this);
    }

    public void receive(String message) {
        System.out.println(name + " received: " + message);
    }
}

// 客戶端代碼
public class Main {
    public static void main(String[] args) {
        ChatMediator chatMediator = new ChatRoom();

        User user1 = new User("Alice", chatMediator);
        User user2 = new User("Bob", chatMediator);
        User user3 = new User("Charlie", chatMediator);

        chatMediator.addUser(user1);
        chatMediator.addUser(user2);
        chatMediator.addUser(user3);

        user1.send("Hello everyone!");
        user2.send("Hey there!");
    }
}
```
ChatMediator 是中介者接口，ChatRoom 是具體中介者，負責協調用戶之間的通信。User 是同事類，每個用戶都知道中介者並且可以發送消息。在客戶端代碼中，我們創建了一個聊天室中介者，並向其中添加了幾個用戶。當用戶發送消息時，中介者會負責將消息傳遞給其他用戶。
### 備忘錄(Memento)
允許你捕獲一個對象的內部狀態並在需要時恢復它，而不暴露其實現細節。
1. 發起人（Originator）：擁有內部狀態的對象。
2. 備忘錄（Memento）：負責捕獲發起人的內部狀態。
3. 管理者（Caretaker）：負責保存和恢復備忘錄。

使用場景： 
1. 需要保存/恢覆數據的相關狀態場景。
2. 提供一個可回滾的操作。

以文本編輯器為例來說明備忘錄模式。在文本編輯器中，當用戶編輯文本時，我們可以保存文本的不同版本作為備忘錄，並在需要時恢復到特定版本。
```java=
// 備忘錄（Memento）
class TextMemento {
    private String text;

    public TextMemento(String text) {
        this.text = text;
    }

    public String getText() {
        return text;
    }
}

// 發起人（Originator）
class TextEditor {
    private String text;

    public void write(String text) {
        this.text = text;
    }

    public TextMemento save() {
        return new TextMemento(text);
    }

    public void restore(TextMemento memento) {
        text = memento.getText();
    }

    public void print() {
        System.out.println("Current Text: " + text);
    }
}

// 管理者（Caretaker）
class TextEditorHistory {
    private List<TextMemento> history = new ArrayList<>();

    public void save(TextMemento memento) {
        history.add(memento);
    }

    public TextMemento get(int index) {
        return history.get(index);
    }
}

// 客戶端代碼
public class Main {
    public static void main(String[] args) {
        TextEditor textEditor = new TextEditor();
        TextEditorHistory history = new TextEditorHistory();

        textEditor.write("Hello");
        history.save(textEditor.save());
        textEditor.print();

        textEditor.write("Hello World");
        history.save(textEditor.save());
        textEditor.print();

        // 恢復到之前的狀態
        textEditor.restore(history.get(0));
        textEditor.print();
    }
}
```
TextEditor 是發起人，負責編輯文本。TextMemento 是備忘錄，用於保存文本的狀態。TextEditorHistory 是管理者，負責保存和恢復備忘錄。在客戶端代碼中，我們編輯了文本並保存了不同版本的備忘錄。最後，我們可以通過使用管理者恢復到之前保存的某個版本的文本狀態。
### 觀察者(Observer)
定義了一種一對多的關係，當一個對象的狀態發生變化時，所有依賴於它的對象都會得到通知並自動更新。
1. 主題（Subject）：具有狀態的對象，可以被觀察。
2. 觀察者（Observer）：當主題的狀態發生變化時，接收通知並執行相應操作的對象。
3. 具體主題（Concrete Subject）：實現主題接口，維護觀察者列表並發送通知。
4. 具體觀察者（Concrete Observer）：實現觀察者接口，接收主題的通知並執行相應操作。

使用場景：
1. 一個抽象模型有兩個方面，其中一個方面依賴於另一個方面。將這些方面封裝在獨立的對象中使它們可以各自獨立地改變和覆用。
2. 一個對象的改變將導致其他一個或多個對象也發生改變，而不知道具體有多少對象將發生改變，可以降低對象之間的耦合度。
3. 一個對象必須通知其他對象，而並不知道這些對象是誰。
4. 需要在系統中創建一個觸發鏈，A對象的行為將影響B對象，B對象的行為將影響C對象……，可以使用觀察者模式創建一種鏈式觸發機制。

以一個簡單的示例來說明觀察者模式。假設有一個主題（天氣信息），它維護了許多觀察者（不同的天氣預報應用），當天氣狀態發生變化時，所有的觀察者都會得到通知。
```java=
import java.util.ArrayList;
import java.util.List;

// 主題（Subject）
interface Subject {
    void registerObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers();
}

// 具體主題（Concrete Subject）
class WeatherStation implements Subject {
    private int temperature;
    private List<Observer> observers;

    public WeatherStation() {
        observers = new ArrayList<>();
    }

    public void setTemperature(int temperature) {
        this.temperature = temperature;
        notifyObservers();
    }

    public void registerObserver(Observer observer) {
        observers.add(observer);
    }

    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(temperature);
        }
    }
}

// 觀察者（Observer）
interface Observer {
    void update(int temperature);
}

// 具體觀察者（Concrete Observer）
class WeatherApp implements Observer {
    private int temperature;

    public void update(int temperature) {
        this.temperature = temperature;
        display();
    }

    public void display() {
        System.out.println("Temperature is " + temperature + "°C");
    }
}

// 客戶端代碼
public class Main {
    public static void main(String[] args) {
        WeatherStation weatherStation = new WeatherStation();

        WeatherApp app1 = new WeatherApp();
        WeatherApp app2 = new WeatherApp();

        weatherStation.registerObserver(app1);
        weatherStation.registerObserver(app2);

        // 改變天氣狀態
        weatherStation.setTemperature(25);
        weatherStation.setTemperature(30);
    }
}
```
WeatherStation 是主題，維護了觀察者列表並通知觀察者。WeatherApp 是觀察者，當收到主題的通知時，它會更新自身的狀態並顯示新的天氣信息。在客戶端代碼中，我們創建了一個天氣站對象和兩個天氣應用，並註冊了觀察者。当天氣狀態改變時，所有的觀察者都會得到通知並更新。
### 狀態(State)
允許對象在內部狀態改變時改變它的行為，看起來就像是改變了它的類型。
1. 狀態（State）：定義了一個介面，用於封裝與特定狀態相關的行為。
2. 具體狀態（Concrete State）：實現了狀態介面，表示對象目前的狀態。
3. 上下文（Context）：維護一個當前狀態的對象，並提供一個介面來設置和改變當前狀態。

使用場景： 
1. 行為隨狀態改變而改變的場景。 
2. 條件、分支語句的代替者。

以一個簡單的自動售貨機示例來說明狀態模式。自動售貨機根據存貨情況和使用者的行為來改變自身的狀態。
```java=
// 狀態（State）
interface VendingMachineState {
    void insertCoin();
    void selectItem();
    void dispenseItem();
}

// 具體狀態（Concrete State）
class NoCoinState implements VendingMachineState {
    public void insertCoin() {
        System.out.println("Coin inserted");
    }

    public void selectItem() {
        System.out.println("Please insert a coin first");
    }

    public void dispenseItem() {
        System.out.println("Please insert a coin first");
    }
}

class HasCoinState implements VendingMachineState {
    public void insertCoin() {
        System.out.println("You can't insert another coin");
    }

    public void selectItem() {
        System.out.println("Item selected");
    }

    public void dispenseItem() {
        System.out.println("Item dispensed");
    }
}

// 上下文（Context）
class VendingMachine {
    private VendingMachineState currentState;

    public VendingMachine() {
        currentState = new NoCoinState();
    }

    public void setCurrentState(VendingMachineState state) {
        currentState = state;
    }

    public void insertCoin() {
        currentState.insertCoin();
    }

    public void selectItem() {
        currentState.selectItem();
    }

    public void dispenseItem() {
        currentState.dispenseItem();
    }
}

// 客戶端代碼
public class Main {
    public static void main(String[] args) {
        VendingMachine vendingMachine = new VendingMachine();

        vendingMachine.insertCoin();
        vendingMachine.selectItem();
        vendingMachine.dispenseItem();

        vendingMachine.setCurrentState(new HasCoinState());

        vendingMachine.insertCoin();
        vendingMachine.selectItem();
        vendingMachine.dispenseItem();
    }
}
```
VendingMachineState 是狀態介面，定義了不同狀態下的行為。NoCoinState 和 HasCoinState 是具體狀態，實現了狀態介面並定義了具體狀態下的行為。VendingMachine 是上下文，維護了當前的狀態，可以改變狀態並執行相應的行為。在客戶端代碼中，我們使用自動售貨機並模擬了插入硬幣、選擇商品和出售商品等動作。
### 策略(Strategy)
定義了一系列算法，並使每個算法能夠獨立地在不影響客戶端的情況下替換。
1. 策略（Strategy）：定義了所有支持的算法的公共接口。
2. 具體策略（Concrete Strategy）：實現了策略接口，包含特定的算法實現。
3. 上下文（Context）：維持一個對策略對象的引用，並且可以在運行時動態地改變所使用的具體策略

使用場景：
1. 如果在一個系統里面有許多類，它們之間的區別僅在於它們的行為，那麽使用策略模式可以動態地讓一個對象在許多行為中選擇一種行為。
2. 一個系統需要動態地在幾種算法中選擇一種。
3. 如果一個對象有很多的行為，如果不用恰當的模式，這些行為就只好使用多重的條件選擇語句來實現。

以一個簡單的支付系統為例來說明策略模式。假設我們有不同的支付方式（信用卡支付、支付寶支付、現金支付等），我們可以根據用戶的選擇來執行不同的支付算法。
```java=
// 策略（Strategy）
interface PaymentStrategy {
    void pay(double amount);
}

// 具體策略（Concrete Strategy）
class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    private String cvv;

    public CreditCardPayment(String cardNumber, String cvv) {
        this.cardNumber = cardNumber;
        this.cvv = cvv;
    }

    public void pay(double amount) {
        System.out.println("Paid " + amount + " using Credit Card");
    }
}

class AlipayPayment implements PaymentStrategy {
    private String alipayAccount;

    public AlipayPayment(String alipayAccount) {
        this.alipayAccount = alipayAccount;
    }

    public void pay(double amount) {
        System.out.println("Paid " + amount + " using Alipay");
    }
}

// 上下文（Context）
class ShoppingCart {
    private PaymentStrategy paymentStrategy;

    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void checkout(double amount) {
        paymentStrategy.pay(amount);
    }
}

// 客戶端代碼
public class Main {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();

        cart.setPaymentStrategy(new CreditCardPayment("123456789", "456"));
        cart.checkout(100.0);

        cart.setPaymentStrategy(new AlipayPayment("example@alipay.com"));
        cart.checkout(50.0);
    }
}
```
PaymentStrategy 是策略接口，定義了支付的公共行為。CreditCardPayment 和 AlipayPayment 是具體的支付策略，分別實現了不同的支付方式。ShoppingCart 是上下文，維護了一個對策略對象的引用，並且可以動態地改變支付策略。在客戶端代碼中，我們模擬了使用信用卡支付和支付寶支付兩種不同的支付方式。
### 模板方法(Template Method)
在一個方法中定義一個算法的骨架，將一些步驟推遲到子類中實現。這樣，子類可以在不改變算法結構的情況下重定義該算法的某些步驟。
1. 抽象類（Abstract Class）：定義了一個模板方法，它定義了算法的骨架，並包含了一系列抽象步驟，讓子類來實現。
2. 具體類（Concrete Class）：繼承自抽象類，實現了抽象類中定義的抽象步驟。

使用場景：
1. 有多個子類共有的方法，且邏輯相同。
2. 重要的、覆雜的方法，可以考慮作為模板方法。

以一個簡單的食譜示例來說明模板方法模式。一個烹飪食物的過程可以被抽象成模板方法，不同的菜品可以通過繼承模板並重寫其中的特定步驟來實現。
```java=
// 抽象類（Abstract Class）
abstract class Recipe {
    public final void cook() {
        prepareIngredients();
        cookIngredients();
        serve();
    }

    abstract void prepareIngredients();
    abstract void cookIngredients();

    void serve() {
        System.out.println("Serve the dish");
    }
}

// 具體類（Concrete Class）
class PizzaRecipe extends Recipe {
    void prepareIngredients() {
        System.out.println("Prepare pizza ingredients");
    }

    void cookIngredients() {
        System.out.println("Cook pizza");
    }
}

class PastaRecipe extends Recipe {
    void prepareIngredients() {
        System.out.println("Prepare pasta ingredients");
    }

    void cookIngredients() {
        System.out.println("Cook pasta");
    }
}

// 客戶端代碼
public class Main {
    public static void main(String[] args) {
        Recipe pizza = new PizzaRecipe();
        pizza.cook();

        Recipe pasta = new PastaRecipe();
        pasta.cook();
    }
}
```
Recipe 是抽象類，定義了烹飪食物的模板方法 cook()，並包含了幾個抽象步驟（prepareIngredients() 和 cookIngredients()）。PizzaRecipe 和 PastaRecipe 是具體類，繼承了 Recipe 並實現了抽象步驟。在客戶端代碼中，我們實例化了不同的菜品類並調用了它們的 cook() 方法。這樣，具體的烹飪步驟被推遲到了子類中實現，同時保留了烹飪食物的整體結構。
### 訪問者(Visitor)
允許你將算法與它作用的對象分開。通常，當你有一個對象結構，並且希望根據不同的對象執行不同的操作時，可以使用訪問者模式。
1. 訪問者（Visitor）：定義了一系列訪問方法，每個方法對應一種對象結構中的元素。
2. 具體訪問者（Concrete Visitor）：實現了訪問者接口中的方法，具體定義了對每種元素的訪問行為。
3. 元素（Element）：定義了一個accept方法，該方法接收一個訪問者作為參數，讓訪問者能夠對其進行操作。
4. 具體元素（Concrete Element）：實現了元素接口，即accept方法，用於接收訪問者並調用訪問者的方法。
5. 對象結構（Object Structure）：包含一組元素，可以被訪問者訪問。

使用場景：
1. 對象結構中對象對應的類很少改變，但經常需要在此對象結構上定義新的操作。
2. 需要對一個對象結構中的對象進行很多不同的並且不相關的操作，而需要避免讓這些操作"污染"這些對象的類，也不希望在增加新操作時修改這些類。

假設有一個動物園，裡面有不同類型的動物。我們可以使用訪問者模式來實現一個計算動物總數和不同種類的動物數量的功能。
```java=
// 訪問者（Visitor）
interface AnimalVisitor {
    void visit(Lion lion);
    void visit(Bear bear);
    // Add more visit methods for other animals
}

// 具體訪問者（Concrete Visitor）
class AnimalCounter implements AnimalVisitor {
    private int lionCount = 0;
    private int bearCount = 0;

    public void visit(Lion lion) {
        lionCount++;
    }

    public void visit(Bear bear) {
        bearCount++;
    }

    public void displayResults() {
        System.out.println("Total Lions: " + lionCount);
        System.out.println("Total Bears: " + bearCount);
    }
}

// 元素（Element）
interface Animal {
    void accept(AnimalVisitor visitor);
}

// 具體元素（Concrete Element）
class Lion implements Animal {
    public void accept(AnimalVisitor visitor) {
        visitor.visit(this);
    }
}

class Bear implements Animal {
    public void accept(AnimalVisitor visitor) {
        visitor.visit(this);
    }
}

// 對象結構（Object Structure）
class Zoo {
    private List<Animal> animals;

    public Zoo() {
        animals = new ArrayList<>();
    }

    public void addAnimal(Animal animal) {
        animals.add(animal);
    }

    public void accept(AnimalVisitor visitor) {
        for (Animal animal : animals) {
            animal.accept(visitor);
        }
    }
}

// 客戶端代碼
public class Main {
    public static void main(String[] args) {
        Zoo zoo = new Zoo();
        zoo.addAnimal(new Lion());
        zoo.addAnimal(new Bear());
        zoo.addAnimal(new Lion());
        
        AnimalCounter counter = new AnimalCounter();
        zoo.accept(counter);
        counter.displayResults();
    }
}
```
AnimalVisitor 是訪問者接口，定義了不同動物的訪問方法。AnimalCounter 是具體訪問者，實現了對獅子和熊的訪問方法並記錄它們的數量。Animal 是元素接口，定義了accept方法，用於接受訪問者並調用相應的訪問方法。Lion 和 Bear 是具體元素，實現了accept方法並調用訪問者的訪問方法。Zoo 是對象結構，包含了不同類型的動物，並提供了接受訪問者的方法。在客户端代碼中，我們創建了一個動物園，將不同的動物添加到動物園中，然後創建一個訪問者來計算動物的數量。