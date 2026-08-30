# Java GUI Design Patterns - Study Guide
### Based on COSC 330 Course Materials

This guide is built **only** from the attached lecture slides on the Observer, Strategy, Adapter, Decorator, Singleton, and Façade patterns.

---

## Table of Contents
1. [Core OO Design Principles (from the slides)](#principles)
2. [Strategy Pattern](#strategy)
3. [Observer Pattern](#observer)
4. [Decorator Pattern](#decorator)
5. [Singleton Pattern](#singleton)
6. [Adapter Pattern](#adapter)
7. [Façade Pattern](#facade)
8. [Pattern Comparison Cheat Sheet](#compare)

---

<a name="principles"></a>
## Core OO Design Principles (from the Slides)

The slides repeatedly summarize these OO principles. Patterns are built on top of them, so know them cold:

- **Encapsulate what varies** — Identify the aspects of the application that vary and separate them from what stays the same.
- **Program to an interface (supertype), not an implementation.**
- **Favor composition over inheritance.** (HAS-A can be better than IS-A.)
- **Strive for loosely coupled designs** between objects that interact.
- **Open-Closed Principle (OCP):** Classes should be open for extension, but closed for modification.
- **Principle of Least Knowledge:** Talk only to your immediate friends. (Minimize dependencies.)
- **Depend on abstractions. Do not depend on concrete classes.**

---

<a name="strategy"></a>
## 1. Strategy Pattern

### Intent
> Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it. — *Gang of Four*

**Problem it solves:** When you have multiple variations of an algorithm/behavior, and inheritance forces all subclasses to inherit behaviors they don't need. The SimUDuck example shows this: putting `fly()` in the `Duck` superclass gave flying ability to `RubberDuck` and `DecoyDuck`, which shouldn't fly. Using interfaces alone forced duplicate code in every subclass. Strategy solves both problems.

### Design Principles Used
- **Encapsulate what varies** — pull varying behaviors (fly, quack) out of the main class.
- **Program to an interface, not an implementation.**
- **Favor composition over inheritance** — Duck *HAS-A* FlyBehavior rather than *IS-A* flying thing.
- **Open-Closed Principle** — add new strategies without modifying the Context class.

### UML Diagram

```
        ┌──────────────────────┐         ┌──────────────────────┐
        │      Context         │◇───────>│ <<interface>>        │
        │----------------------│         │   Strategy           │
        │ - strategy: Strategy │         │----------------------│
        │ + setStrategy(s)     │         │ + execute()          │
        │ + execute()          │         └──────────────────────┘
        └──────────────────────┘                    △
                                                    │
                          ┌─────────────────────────┼─────────────────────────┐
                          │                         │                         │
                ┌─────────┴─────────┐    ┌──────────┴────────┐     ┌──────────┴────────┐
                │ ConcreteStrategyA │    │ ConcreteStrategyB │     │ ConcreteStrategyC │
                │  + execute()      │    │  + execute()      │     │  + execute()      │
                └───────────────────┘    └───────────────────┘     └───────────────────┘
```
- The **Context** has-a Strategy (composition).
- All ConcreteStrategy classes implement the same Strategy interface.

### Real-World Examples (from slides)
- **SimUDuck:** Different fly behaviors (FlyWithWings, FlyNoWay, FlyRocketPowered) and quack behaviors (Quack, Squeak, MuteQuack).
- **Chess characters with weapons:** King, Queen, Knight, Bishop each use a `WeaponBehavior` (KnifeBehavior, BowAndArrowBehavior, AxeBehavior, SpearBehavior).
- **Java Swing Layout Managers:** `Container` has a `LayoutManager` strategy (FormLayout, GridLayout, SelfDefined, etc.).
- **Input validation:** Different `InputValidationStrategy` implementations for UserGroupA/B/C.
- **Swing Borders (the "right way"):** `JComponent` has a `Border` field. `paintBorder()` delegates to whatever Border is set, instead of using a giant `switch` over border types.

### Pattern Recognition (How to spot it in code)
- A **Context class holds a private reference to a Strategy interface** (e.g., `private FlyBehavior flyBehavior;`).
- A **setter method** lets the strategy be changed at runtime (e.g., `setFlyBehavior(...)`).
- The Context's methods **delegate** the work: `performFly()` simply calls `flyBehavior.fly()`.
- Multiple small classes that all implement the same simple interface.
- **Bad sign that Strategy should be used:** big `switch` statements selecting between behaviors (see the "wrong way" Swing border example in the slides).

### Java Implementation
The slides give the canonical structure:

```java
// === The Strategy interface ===
interface IStrategy {
    void execute();
}

// === Concrete Strategies (interchangeable algorithms) ===
class ConcreteStrategyA implements IStrategy {
    public void execute() {
        System.out.println("Called ConcreteStrategyA.execute()");
    }
}

class ConcreteStrategyB implements IStrategy {
    public void execute() {
        System.out.println("Called ConcreteStrategyB.execute()");
    }
}

class ConcreteStrategyC implements IStrategy {
    public void execute() {
        System.out.println("Called ConcreteStrategyC.execute()");
    }
}

// === Context class: HAS-A Strategy ===
class Context {
    IStrategy strategy;  // composition

    // Constructor lets us inject the strategy
    public Context(IStrategy strategy) {
        this.strategy = strategy;
    }

    // Delegates to whichever strategy is set
    public void execute() {
        strategy.execute();
    }
}

// === Main class — picks strategy at runtime ===
public class MainApp {
    public static void main(String[] args) {
        Context context;

        // Same context, three different strategies
        context = new Context(new ConcreteStrategyA());
        context.execute();

        context = new Context(new ConcreteStrategyB());
        context.execute();

        context = new Context(new ConcreteStrategyC());
        context.execute();
    }
}
```

### Scenario-Based Application
**Scenario:** SimUDuck simulation needs many duck types, where some can fly and some can't, and some quack while others squeak or stay mute.

**Why Strategy fits:**
- Pulling `fly()` into the Duck superclass forces all ducks to fly (bad).
- Using `Flyable` / `Quackable` interfaces alone forces every subclass to re-implement fly/quack (duplication).
- Strategy lets each Duck hold a `FlyBehavior` and `QuackBehavior` object. A `RubberDuck` gets a `FlyNoWay` and a `Squeak`; a `MallardDuck` gets `FlyWithWings` and `Quack`. New behaviors (like `FlyRocketPowered`) can be added without changing `Duck` at all — **Open-Closed Principle** satisfied.

### Java GUI Connection
- **LayoutManager** in `Container` — the layout algorithm is a strategy that can be swapped (FlowLayout, BorderLayout, GridLayout…).
- **Swing Borders** — `JComponent` has a `Border` strategy and delegates `paintBorder()` to it, avoiding a giant switch over border types.

---

<a name="observer"></a>
## 2. Observer Pattern

### Intent
> Define a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated automatically. — *GoF, p293*

The slides also call this **"the View part of Model-View-Controller."**

**Problem it solves:** In the Weather Station example, `WeatherData.measurementsChanged()` was hard-coded to call `currentConditionsDisplay.update()`, `statisticsDisplay.update()`, `forecastDisplay.update()` directly. You can't add a new display without modifying `WeatherData`. Observer separates the source of changes (Subject) from the things that react (Observers).

### Design Principles Used (the slides list all four)
- **Encapsulate what varies** — the set of observers and the subject's state can vary.
- **Favor composition over inheritance** — observers are *composed* with the subject at runtime, not set up by an inheritance hierarchy.
- **Program to interfaces, not implementations** — Subject and Observer are interfaces.
- **Strive for loosely coupled designs** — the subject only knows that observers implement the Observer interface; nothing more.

### UML Diagram

```
   ┌─────────────────────────┐         ┌─────────────────────────┐
   │ <<interface>> Subject   │observers│ <<interface>> Observer  │
   │-------------------------│◇───────>│-------------------------│
   │ + registerObserver(o)   │         │ + update()              │
   │ + removeObserver(o)     │         └─────────────────────────┘
   │ + notifyObservers()     │                      △
   └─────────────────────────┘                      │
                △                                   │
                │                       ┌───────────┴────────────┐
   ┌────────────┴────────────┐          │   ConcreteObserver     │
   │   ConcreteSubject       │          │------------------------│
   │-------------------------│          │ + update()             │
   │ + getState()            │          └────────────────────────┘
   │ + setState()            │
   └─────────────────────────┘
```

### Real-World Examples (from slides)
- **Newspaper / magazine subscription model** — publisher publishes, subscribers receive new editions; subscribers can subscribe/unsubscribe at any time.
- **Weather Station:** WeatherData is the Subject; CurrentConditionsDisplay, StatisticsDisplay, ForecastDisplay are Observers.
- **Mailing lists** — when an event happens, all subscribers get notified.
- **MVC** — the View observes the Model.
- **Java Swing GUI events** — JButton is a Subject; ActionListener objects are Observers.

### Pattern Recognition (How to spot it in code)
- A class maintains a **list of listeners/observers** (e.g., `ArrayList observers`).
- Methods named **`registerObserver` / `addXxxListener` / `attach`** and **`removeObserver` / `removeXxxListener` / `detach`**.
- A **`notifyObservers()`** method that iterates and calls `update()` on each.
- Observers implement an interface with an `update()` (or `actionPerformed`, etc.) method.

### Java Implementation
Following the slides' WeatherData example exactly:

```java
import java.util.ArrayList;

// === Subject interface ===
interface Subject {
    void registerObserver(Observer o);
    void removeObserver(Observer o);
    void notifyObservers();
}

// === Observer interface ===
interface Observer {
    void update(float temp, float humidity, float pressure);
}

// === DisplayElement interface (so displays know how to display) ===
interface DisplayElement {
    void display();
}

// === Concrete Subject ===
class WeatherData implements Subject {
    private ArrayList<Observer> observers; // list of registered observers
    private float temperature;
    private float humidity;
    private float pressure;

    public WeatherData() {
        observers = new ArrayList<>();
    }

    // Register a new observer
    public void registerObserver(Observer o) {
        observers.add(o);
    }

    // Unregister an observer
    public void removeObserver(Observer o) {
        int i = observers.indexOf(o);
        if (i >= 0) {
            observers.remove(i);
        }
    }

    // Push the state out to every observer
    public void notifyObservers() {
        for (int i = 0; i < observers.size(); i++) {
            Observer observer = observers.get(i);
            observer.update(temperature, humidity, pressure);
        }
    }

    // Called whenever new data arrives from sensors
    public void measurementsChanged() {
        notifyObservers();
    }

    // Helper for testing
    public void setMeasurements(float t, float h, float p) {
        this.temperature = t;
        this.humidity = h;
        this.pressure = p;
        measurementsChanged();
    }
}

// === Concrete Observer ===
class CurrentConditionsDisplay implements Observer, DisplayElement {
    private float temperature;
    private float humidity;
    private Subject weatherData;

    public CurrentConditionsDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this); // register with subject
    }

    // Called by the subject whenever data changes
    public void update(float temp, float humidity, float pressure) {
        this.temperature = temp;
        this.humidity = humidity;
        display();
    }

    public void display() {
        System.out.println("Current conditions: " + temperature
            + "F degrees and " + humidity + "% humidity");
    }
}

// === Main class to demonstrate ===
public class WeatherStationDemo {
    public static void main(String[] args) {
        WeatherData weatherData = new WeatherData();
        // Wire up an observer
        CurrentConditionsDisplay current = new CurrentConditionsDisplay(weatherData);

        // Simulate sensor updates
        weatherData.setMeasurements(80, 65, 30.4f);
        weatherData.setMeasurements(82, 70, 29.2f);
    }
}
```

### Push vs. Pull (from the slides)
- **Push:** Subject sends all state in `update(...)` arguments (shown above).
- **Pull:** Subject only signals "something changed"; observers call getters on the subject to grab what they need.
- Java's built-in `java.util.Observable` / `java.util.Observer` supports both, but the slides note **problems** with it: `Observable` is a class (you must subclass it, so you can't extend anything else), `setChanged()` is protected, and it forces inheritance over composition.

### Scenario-Based Application
**Scenario:** A stock-tracking app must update multiple display widgets whenever a stock price changes, and new widgets may be added later.

**Why Observer fits:**
- One subject (the stock), many displays (observers) — classic one-to-many.
- New displays plug in by registering — no change to the stock class needed (Open-Closed).
- Loose coupling — stock only knows observers implement `Observer`.

### Java GUI Connection
The slides give a direct mapping for **JButton event handling**:

| Name in Design Pattern | Actual Name in JButton Event Handling |
|------------------------|---------------------------------------|
| Subject | `JButton` |
| Observer | `ActionListener` |
| ConcreteObserver | The class that implements `ActionListener` |
| `attach()` | `addActionListener` |
| `notify()` | `actionPerformed` |

Example from the slides:
```java
JButton button = new JButton("Should I do it?");
button.addActionListener(new AngelListener()); // observer #1
button.addActionListener(new DevilListener()); // observer #2
// When the button is clicked, BOTH listeners' actionPerformed() runs.
```
"Swing makes heavy use of this pattern."

---

<a name="decorator"></a>
## 3. Decorator Pattern

### Intent
> Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

The slides emphasize: **subclassing adds behavior at compile time, whereas decorators provide new behavior at runtime.**

**Problem it solves:** The coffee shop example — if `Beverage.cost()` uses `if (hasMilk()) total += .50; if (hasSoy()) total += .65; ...`, then adding a new topping requires modifying `Beverage`. Decorator wraps the base object in successive layers, each adding its own behavior.

### Design Principles Used
- **Open-Closed Principle** — classes are open for extension, closed for modification. Add new functionality by writing new code, not modifying existing code.
- **Favor composition over inheritance** — the decorator wraps (composes) the component.
- New functionality is added by writing new decorator classes, which "reduces the chances of introducing bugs and causing unintended side effects in pre-existing code."

### UML Diagram

```
                  ┌────────────────────────┐
                  │ <<interface>> Component│
                  │------------------------│
                  │ + operation()          │
                  └────────────────────────┘
                              △
              ┌───────────────┴───────────────┐
              │                               │
   ┌──────────┴────────────┐      ┌───────────┴─────────────┐
   │  ConcreteComponent    │      │      Decorator          │◇──┐
   │-----------------------│      │-------------------------│   │ component
   │ + operation()         │      │ # component: Component  │<──┘
   └───────────────────────┘      │ + operation()           │
                                  └─────────────────────────┘
                                              △
                                  ┌───────────┴─────────────┐
                                  │                         │
                       ┌──────────┴──────┐       ┌──────────┴──────┐
                       │ConcreteDecoratorA│      │ConcreteDecoratorB│
                       │ + operation()    │      │ + operation()    │
                       └──────────────────┘      └──────────────────┘
```
Key feature: the Decorator **both extends** Component **and contains** a Component reference.

### Real-World Examples (from slides)
- **Starbucks coffee** — base Beverage (Espresso, DarkRoast, HouseBlend) decorated with Mocha, Soy, Whip, etc.
- **Java I/O streams** — `FileInputStream`, `BufferedInputStream`, `LineNumberInputStream` wrapping each other. `FilterInputStream` is the abstract decorator.
- **JScrollPane + JTextArea** — `JScrollPane` decorates a component with scrollbars.
- **BorderDecorator** for Swing — wraps a `JComponent` and paints a border around it.
- **Window with scrollbars** — `VerticalScrollBarDecorator` and `HorizontalScrollBarDecorator` wrap a `SimpleWindow`.
- **Text editor evolution** — `Notepad` → `NotepadPlusPlus` (adds format, folding) → `Word` (adds RichText, Image, Table, etc.).
- **Employee responsibilities** — an `EmployeeImpl` can be decorated as `TeamMember`, `TeamLead`, or `Manager`, with responsibilities added/revoked at runtime.
- **University application evaluation** — chaining `GPAEval`, `GREEval`, `TOEFLEval` criteria.

### Pattern Recognition (How to spot it in code)
- A class that **both implements an interface AND holds a reference to that same interface** (e.g., `class WindowDecorator implements Window { protected Window decoratedWindow; }`).
- Methods that **delegate to the wrapped object and then add behavior** before or after:
  ```java
  public void draw() {
      drawVerticalScrollBar();   // added behavior
      decoratedWindow.draw();    // delegate to wrapped
  }
  ```
- Construction looks like nested wrapping:
  ```java
  new LowerCaseInputStream(new BufferedInputStream(new FileInputStream("test.txt")));
  ```
- Type matching, **NOT** "get behaviors" — the slide notes the decorator pattern is about *type compatibility*, not inherited behavior.

### Java Implementation
This is the slides' `Window` example, kept simple and beginner-friendly:

```java
// === Component interface ===
interface Window {
    void draw();                  // draws the Window
    String getDescription();      // describes the Window
}

// === Concrete Component: a basic window with no scrollbars ===
class SimpleWindow implements Window {
    public void draw() {
        // draw the basic window
    }
    public String getDescription() {
        return "simple window";
    }
}

// === Abstract Decorator: implements Window AND holds a Window ===
abstract class WindowDecorator implements Window {
    protected Window decoratedWindow; // the window being decorated

    public WindowDecorator(Window decoratedWindow) {
        this.decoratedWindow = decoratedWindow;
    }
}

// === Concrete Decorator #1: adds a vertical scrollbar ===
class VerticalScrollBarDecorator extends WindowDecorator {
    public VerticalScrollBarDecorator(Window decoratedWindow) {
        super(decoratedWindow);
    }
    public void draw() {
        drawVerticalScrollBar();   // add new behavior
        decoratedWindow.draw();    // then delegate to the wrapped window
    }
    private void drawVerticalScrollBar() {
        // draws the vertical scrollbar
    }
    public String getDescription() {
        return decoratedWindow.getDescription() + ", including vertical scrollbars";
    }
}

// === Concrete Decorator #2: adds a horizontal scrollbar ===
class HorizontalScrollBarDecorator extends WindowDecorator {
    public HorizontalScrollBarDecorator(Window decoratedWindow) {
        super(decoratedWindow);
    }
    public void draw() {
        drawHorizontalScrollBar();
        decoratedWindow.draw();
    }
    private void drawHorizontalScrollBar() {
        // draws the horizontal scrollbar
    }
    public String getDescription() {
        return decoratedWindow.getDescription() + ", including horizontal scrollbars";
    }
}

// === Main class to test it ===
public class DecoratedWindowTest {
    public static void main(String[] args) {
        // Wrap a SimpleWindow with a vertical scrollbar, then with a horizontal one
        Window simpleWindow = new SimpleWindow();
        Window verticalScrollBarWindow = new VerticalScrollBarDecorator(simpleWindow);
        Window decoratedWindow = new HorizontalScrollBarDecorator(verticalScrollBarWindow);

        // Output: "simple window, including vertical scrollbars, including horizontal scrollbars"
        System.out.println(decoratedWindow.getDescription());
    }
}
```

#### Bonus: Writing your own Java I/O decorator (from the slides)
A `LowerCaseInputStream` that converts uppercase characters to lowercase as it reads:

```java
import java.io.*;

// Extend the FilterInputStream, the abstract decorator for all InputStreams
public class LowerCaseInputStream extends FilterInputStream {
    public LowerCaseInputStream(InputStream in) {
        super(in); // pass the wrapped stream up to the parent
    }

    // Read one byte and lowercase it
    public int read() throws IOException {
        int c = super.read();
        return (c == -1 ? c : Character.toLowerCase((char) c));
    }

    // Read into a byte array and lowercase each byte
    public int read(byte[] b, int offset, int len) throws IOException {
        int result = super.read(b, offset, len);
        for (int i = offset; i < offset + result; i++) {
            b[i] = (byte) Character.toLowerCase((char) b[i]);
        }
        return result;
    }
}
```
Usage shows the decorator chain:
```java
InputStream in = new LowerCaseInputStream(
                    new BufferedInputStream(
                        new FileInputStream("test.txt")));
```

### Scenario-Based Application
**Scenario (from slides):** Employees in an organization have responsibilities — team members, team leads, managers. An employee's role can change over time, and a team lead may sometimes also perform team-member duties. Traditional inheritance would force you to destroy and re-create employee objects when roles change.

**Why Decorator fits:**
- Responsibilities can be **added and revoked at runtime** by wrapping/unwrapping decorators.
- The same `EmployeeImpl` object can be wrapped as `TeamMember`, `TeamLead`, or `Manager` — no need to throw away the original.
- New roles can be added as new decorator classes without modifying existing ones.

### Java GUI Connection
The slides give two GUI uses of Decorator:
- **`JScrollPane`** wraps any component (like `JTextArea`) to add scrollbars:
  ```java
  JTextArea area = new JTextArea(10, 25);
  JScrollPane areaScrollPane = new JScrollPane(area);
  ```
- **Borders / `BorderDecorator`** — instead of subclassing every label that needs a border (`JBorderLabel extends JLabel`), wrap any `JComponent` in a `BorderDecorator` that paints the rectangle.
- **Java I/O classes** — the slides note "Java I/O uses a lot of decorator pattern" (FilterInputStream + BufferedInputStream + LineNumberInputStream, etc.).

---

<a name="singleton"></a>
## 4. Singleton Pattern

### Intent
> Ensure a class only has one instance, and provide a global point of access to it.

**Motivation (from slides):** It's important for some classes to have exactly one instance. Although there can be many printers in a system, there should be only one printer spooler. There should be only one file system manager and one window manager.

### Design Principles Used
- **Controlled instantiation** — the class itself controls when (and how many times) it can be instantiated.
- **Global access point** — every part of the system gets the *same* instance via one static method.
- **Encapsulation** of the creation logic.

### UML Diagram

```
   ┌────────────────────────────────┐
   │           Singleton            │
   │--------------------------------│
   │ - instance: Singleton          │   (private static)
   │--------------------------------│
   │ - Singleton()                  │   (private constructor)
   │ + getInstance(): Singleton     │   (public static)
   └────────────────────────────────┘
```

### Real-World Examples (from slides)
- **Window manager**
- **Print spooler** (only one spooler manages many printers)
- **File system / file system manager**
- **Find dialog** (pressing Ctrl-F always shows the same one)
- **Singleton GUI Frame** — a settings or "About" window where you only ever want one instance open

### Pattern Recognition (How to spot it in code)
- The class has a **private constructor** (so `new` is forbidden from outside).
- A **private static field** of the class's own type (often called `instance`).
- A **public static `getInstance()` method** that lazily creates the instance if it's null, then returns it.
- No other way to get an instance — clients always go through `getInstance()`.

### Java Implementation
Straight from the slides:

```java
public class Singleton {
    // Private static reference to the one instance
    private static Singleton instance = null;

    // Private constructor — outside code cannot use `new Singleton()`
    private Singleton() {
    }

    // Public static accessor — creates the instance on first call, then reuses it
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

// === Main / test class ===
public class TestSingleton {
    public static void main(String[] args) {
        Singleton s = Singleton.getInstance();
        // ... use s ...
    }
}
```

### Scenario-Based Application
**Scenario (from slides):** A Java Swing application has a special "Singleton Frame" window that should pop up the same window each time the user clicks any button asking to see it — never a new copy.

**Why Singleton fits:**
- Multiple buttons need access to *the same* frame.
- The frame must be globally accessible from anywhere in the app.
- Creating duplicate frames would waste resources and confuse the user.

### Java GUI Connection
The slides give a complete Swing example:

```java
package singleton;
import javax.swing.*;

// A JFrame that is also a Singleton
public class SingletonFrame extends JFrame {
    private static SingletonFrame myInstance = null;

    // Private constructor — outside code can't create this directly
    private SingletonFrame() {
        this.setSize(400, 100);
        this.setTitle("Singleton Frame. Timestamp:" + System.currentTimeMillis());
        this.setDefaultCloseOperation(JFrame.HIDE_ON_CLOSE);
    }

    // Public accessor — always returns the same instance
    public static SingletonFrame getInstance() {
        if (myInstance == null) {
            myInstance = new SingletonFrame();
        }
        return myInstance;
    }
}
```

And a main frame that has two buttons, both showing **the same** Singleton frame:

```java
package singleton;
import java.awt.*;
import javax.swing.*;
import java.awt.event.*;

public class MyFrame extends JFrame {
    JButton jButton1 = new JButton();
    JButton jButton2 = new JButton();

    public MyFrame() {
        init();
    }

    public static void main(String[] args) {
        MyFrame frame = new MyFrame();
        frame.setSize(300, 250);
        frame.setVisible(true);
    }

    private void init() {
        this.setDefaultCloseOperation(EXIT_ON_CLOSE);

        // Button 1 shows the singleton frame
        jButton1.setText("Show Singleton Frame");
        jButton1.setBounds(new Rectangle(12, 12, 220, 40));
        jButton1.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                SingletonFrame singletonFrame = SingletonFrame.getInstance();
                singletonFrame.setVisible(true);
            }
        });

        // Button 2 shows the SAME singleton frame
        jButton2.setText("Show the same Singleton Frame");
        jButton2.setBounds(new Rectangle(12, 72, 220, 40));
        jButton2.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                SingletonFrame singletonFrame = SingletonFrame.getInstance();
                singletonFrame.setVisible(true);
            }
        });

        this.getContentPane().setLayout(null);
        this.getContentPane().add(jButton1, null);
        this.getContentPane().add(jButton2, null);
    }
}
```
Notice the timestamp in the singleton's title — both buttons show a frame with the **same** timestamp because it's only constructed once.

---

<a name="adapter"></a>
## 5. Adapter Pattern

### Intent
> Convert the interface of a class into another interface clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.

**Motivation (from slides):** Sometimes a toolkit or class library can not be used because its interface is incompatible with the interface required by an application. We can not change the library interface (we may not have its source code). Even if we did have the source code, we probably should not change the library for each domain-specific application.

### Design Principles Used
- **Program to an interface, not an implementation** — the client uses the `Shape` interface; the adapter handles the difference.
- **Encapsulation** — the incompatible interface of the adaptee is hidden inside the adapter.
- The slides distinguish **two kinds**:
  - **Object Adapter** — the adapter class **contains an adaptee object** (composition).
  - **Class Adapter** — the adapter class is **inherited from both** the adaptee and the abstract class (multiple inheritance; not directly available in Java).

### UML Diagram

#### Object Adapter (the one slides show working in Java)
```
   ┌──────────┐                ┌──────────────────┐
   │  Client  │ ─────uses────> │   Shape          │  (Target — what client expects)
   └──────────┘                │ + display()      │
                               │ + fill()         │
                               │ + undisplay()    │
                               └──────────────────┘
                                        △
                                        │
                               ┌────────┴──────────┐
                               │    Circle         │  (Adapter)
                               │-------------------│
                               │ - aCircle:        │◇──────┐
                               │     AnotherCircle │       │
                               │ + display()       │       │
                               │ + fill()          │       │
                               │ + undisplay()     │       │
                               └───────────────────┘       │
                                                           ▼
                                              ┌────────────────────┐
                                              │  AnotherCircle     │  (Adaptee)
                                              │--------------------│
                                              │ + setLocation()    │
                                              │ + drawIt()         │
                                              │ + fillIt()         │
                                              │ + setItColor()     │
                                              │ + unDrawIt()       │
                                              └────────────────────┘
```

### Real-World Example (from slides)
The slides build this scenario step-by-step:
1. You have a `Shape` library (`Line`, `Square`) where each shape has `display()`, `fill()`, `undisplay()`.
2. You want to add `Circle`. Plan: extend `Shape` like the others.
3. Client says: "No — you must use *this* circle library." But that library is from a **different vendor** with a **different interface**: `AnotherCircle` has `setLocation()`, `drawIt()`, `fillIt()`, `setItColor()`, `unDrawIt()`.
4. Solution: write an **Adapter** named `Circle` that *implements `Shape`* but internally holds an `AnotherCircle` and translates each call.

### Pattern Recognition (How to spot it in code)
- A class that **implements one interface** but **internally holds an instance of another, incompatible class**.
- Methods look like simple **translation/forwarding** to the wrapped object's differently-named methods:
  ```java
  public void display() { aCircle.drawIt(); }
  public void fill()    { aCircle.fillIt(); }
  ```
- Names ending in `Adapter` (e.g., `MouseAdapter`) or `Wrapper` are common.

### Java Implementation (Object Adapter — from the slides)

```java
// === Target interface — what the client expects ===
interface Shape {
    void display();
    void fill();
    void undisplay();
}

// === Adaptee — third-party class with the WRONG interface ===
class AnotherCircle {
    public void setLocation() { /* ... */ }
    public void drawIt()      { System.out.println("AnotherCircle: drawing"); }
    public void fillIt()      { System.out.println("AnotherCircle: filling"); }
    public void setItColor()  { /* ... */ }
    public void unDrawIt()    { System.out.println("AnotherCircle: undrawing"); }
}

// === Adapter — translates Shape calls into AnotherCircle calls ===
class Circle implements Shape {
    private AnotherCircle aCircle; // composition: the adapter HAS-A adaptee

    public Circle() {
        aCircle = new AnotherCircle();
    }

    // Each Shape method delegates to the matching AnotherCircle method
    public void display()   { aCircle.drawIt(); }
    public void fill()      { aCircle.fillIt(); }
    public void undisplay() { aCircle.unDrawIt(); }
}

// === Client — works only with the Shape interface ===
public class Client {
    public static void main(String[] args) {
        Shape aShape = new Circle();
        aShape.display();   // actually calls Circle.display() -> aCircle.drawIt()
        aShape.fill();
        aShape.undisplay();
    }
}
```
Client code never sees `AnotherCircle`. As far as the client knows, it's just another `Shape`.

### Scenario-Based Application
**Scenario:** Your drawing app uses a `Shape` interface, but the client requires you to use a third-party circle library with a different (incompatible) API that you can't modify.

**Why Adapter fits:**
- You can't change the third-party library's source.
- You shouldn't change the client (which expects `Shape`).
- An adapter sits in the middle and translates one interface to the other — letting both work together unchanged.

### Java GUI Connection
While the slides' main Adapter example is `Shape`/`AnotherCircle`, Adapter is widely used in Swing for **listener adapters** (e.g., `MouseAdapter`, `KeyAdapter`, `WindowAdapter`) — adapters that provide empty default implementations of listener interfaces so you only override what you need. *(The slides demonstrate the adapter concept primarily through the shape example.)*

---

<a name="facade"></a>
## 6. Façade Pattern

### Intent
> Provide a unified interface to a set of interfaces in a subsystem. Façade defines a higher-level interface that makes the subsystem easier to use.

**Motivation (from slides):** Structuring a system into subsystems helps reduce complexity. A common design goal is to minimize the communication and dependencies between subsystems. Use a façade object to provide a single, simplified interface to the more general facilities of a subsystem.

### What a Façade Can Do (from slides)
- Make a software library **easier to use and understand** — convenient methods for common tasks.
- Make **code that uses the library more readable.**
- **Reduce dependencies** of outside code on the inner workings of a library.
- **Wrap a poorly designed collection of APIs** with a single well-designed API.

### Design Principles Used
- **Principle of Least Knowledge** ("talk only to your immediate friends") — minimize dependencies. The client only knows about the façade, not the subsystem classes.
- **Loose coupling** between the subsystem and clients — you can change subsystem components without affecting clients.
- **Encapsulation** — the complexity of the subsystem is hidden behind one class.

### Benefits (from slides)
- Shields clients from subsystem components by reducing the number of objects clients have to deal with.
- Promotes weak coupling between subsystem and clients.
- **Does not prevent** clients from using the subsystem directly if they need to.

### UML Diagram

```
   ┌──────────┐         ┌─────────────────┐
   │  Client  │────────>│     Façade      │
   └──────────┘         └────────┬────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
       ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
       │ Subsystem 1 │    │ Subsystem 2 │    │ Subsystem 3 │
       └─────────────┘    └─────────────┘    └─────────────┘
```
- The client only has **one friend** — the façade.
- The façade calls into the subsystem classes on the client's behalf.
- If a subsystem grows complicated, you can recursively apply the same principle.

### Real-World Examples (from slides)
- **Watching a movie the hard way** (Head First example) — turning on the projector, lights, sound system, DVD player, etc. A façade `HomeTheaterFacade.watchMovie()` does it all with one call.
- **Service desk** — one number to call, handles routing internally.
- **E-commerce website** — one checkout button hides payment, inventory, shipping subsystems.
- **A façade for a legacy application** — wraps a messy old system with a clean modern API.
- **JDBC design** — a `Connection` class hands back the right info object (`GeneralInfo`, `SpecialInfo`, `PrivateInfo`) based on user privileges, hiding the interface hierarchy behind one façade.

### Pattern Recognition (How to spot it in code)
- A class that **holds references to multiple subsystem classes** as fields.
- Provides **simple high-level methods** that internally **call sequences of methods** on those subsystem objects.
- The client only calls into this one class — never directly into the subsystem classes.
- Names like `XxxFacade`, `XxxManager`, `XxxService` are common.

### Java Implementation
A beginner-friendly version of the slides' "watching a movie" scenario:

```java
// === Subsystem class #1 ===
class DVDPlayer {
    public void on()              { System.out.println("DVD Player on"); }
    public void play(String movie){ System.out.println("Playing: " + movie); }
    public void off()             { System.out.println("DVD Player off"); }
}

// === Subsystem class #2 ===
class Projector {
    public void on()  { System.out.println("Projector on"); }
    public void off() { System.out.println("Projector off"); }
}

// === Subsystem class #3 ===
class SoundSystem {
    public void on()                  { System.out.println("Sound System on"); }
    public void setVolume(int volume) { System.out.println("Volume: " + volume); }
    public void off()                 { System.out.println("Sound System off"); }
}

// === Façade — one simple interface to all three subsystems ===
class HomeTheaterFacade {
    // Façade HAS-A reference to each subsystem
    private DVDPlayer dvd;
    private Projector projector;
    private SoundSystem sound;

    public HomeTheaterFacade() {
        this.dvd = new DVDPlayer();
        this.projector = new Projector();
        this.sound = new SoundSystem();
    }

    // One method that coordinates the whole subsystem
    public void watchMovie(String movie) {
        System.out.println("Getting ready to watch a movie...");
        projector.on();
        sound.on();
        sound.setVolume(10);
        dvd.on();
        dvd.play(movie);
    }

    public void endMovie() {
        System.out.println("Shutting down...");
        dvd.off();
        sound.off();
        projector.off();
    }
}

// === Main class ===
public class FacadeDemo {
    public static void main(String[] args) {
        HomeTheaterFacade theater = new HomeTheaterFacade();
        // The client uses ONE simple call
        theater.watchMovie("Inception");
        theater.endMovie();
    }
}
```

### Scenario-Based Application
**Scenario:** A company has a 20-year-old legacy backend with dozens of confusing, badly-named methods. A new mobile app team needs a clean API to talk to it.

**Why Façade fits:**
- Wraps the legacy mess in a **single well-designed API**.
- New code depends only on the façade, not on internal legacy classes.
- The legacy system can be **refactored or replaced** later without breaking the mobile app, as long as the façade keeps the same surface.

### Java GUI Connection
While the slides' main façade examples are home theater, service desk, e-commerce, and legacy application wrappers, the same principle applies whenever a complex set of GUI subsystem calls (open frame → set up panels → configure listeners → register with manager…) is wrapped behind one easy method.

---

<a name="compare"></a>
## Pattern Comparison Cheat Sheet

The slides give this comparison explicitly:

| Pattern | Intent |
|---------|--------|
| **Adapter** | Converts one interface to another |
| **Decorator** | Doesn't alter the interface, but adds responsibility |
| **Façade** | Makes an interface simpler |

The slides also note the broader differences:
- **Adapter** uses *type matching* (also true of Decorator), **NOT** "getting behaviors" through inheritance.
- **Decorator** is an *alternative to subclassing*; subclassing adds behavior at compile time, decorators add it at runtime.
- **Façade** "talks only to immediate friends" — it shields the client from the subsystem.

Full summary table from the slides:

| Pattern | One-line intent |
|---------|-----------------|
| **Strategy** | Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it. |
| **Observer** | Define a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated automatically. |
| **Decorator** | Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality. |
| **Singleton** | Ensure a class only has one instance, and provide a global point of access to it. |
| **Adapter** | Convert the interface of a class into another interface the clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces. |
| **Façade** | Provide a unified interface to a set of interfaces in a subsystem. Façade defines a higher-level interface that makes the subsystem easier to use. |

### Quick Identification Guide

| If you see... | The pattern is probably... |
|---------------|----------------------------|
| Private constructor + private static instance + `getInstance()` | **Singleton** |
| A class with a list of listeners/observers and `notify…()` / `addXxxListener` | **Observer** |
| A Context class with a strategy field and `setStrategy()` that just delegates | **Strategy** |
| A class that implements an interface AND wraps an object of that same interface, adding behavior | **Decorator** |
| A class that implements one interface but internally calls a different (incompatible) class's methods | **Adapter** |
| A class holding several subsystem objects and offering simple "do everything" methods | **Façade** |

---

## Final Tips for the Exam

- **Memorize each pattern's one-line intent** — the slides give these almost verbatim, and they're easy points.
- **Know the design principles each pattern embodies** — particularly: encapsulate what varies, favor composition over inheritance, program to an interface, OCP (Decorator, Strategy), least knowledge (Façade), loose coupling (Observer).
- **Be able to draw the UML diagram** for each (Context/Strategy, Subject/Observer, Component/Decorator, Singleton, Target/Adapter/Adaptee, Façade/Subsystem).
- **For "identify the pattern from source code"** questions: focus on the structural cues in the "Pattern Recognition" sections above — private constructor + getInstance, list of observers + notify, wraps-and-extends-same-interface, etc.
- **For "apply a pattern to a scenario"** questions: recognize the *problem* the pattern solves (changing behavior at runtime → Strategy or Decorator; one-to-many notification → Observer; one instance only → Singleton; incompatible interfaces → Adapter; complex subsystem → Façade).
- **The Java GUI connections** (JButton↔Observer, LayoutManager/Border↔Strategy, JScrollPane↔Decorator, SingletonFrame↔Singleton) are exactly the kind of GUI-pattern-identification questions the review mentioned.

Good luck!
