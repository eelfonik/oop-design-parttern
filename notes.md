## Code reuse: strategy pattern
**Inheritance** is bad because if you add sth in the superclass, every subclass inherits it will have that method.

And if you choose to add **interface**, then you need to implement those methods in every subclass that needs the methods (as Java interface don't have implementation code -- same for typescript interface).

### Encapsulate what varies
So we need to **seperate** what is *constant* and what is *varing*, so we can changing the least possible codes when adding new things or changing current behaviours. We can also say it as **take the parts that change and encapsulate them**.

So we won't put the changing parts in the superclass, nor adding interfaces that need to be directly implements by the subclasses.

### Favor composition over inheritance
Instead, we create new classes that implement seperately those parts by **satisfying** the interfaces. So the subclasses can *delegate* (代理) those classes, instead of using a concrecte implementation (either in the superclass or the subclass itself). That way we acheived the **polymorphism** at *runtime*.

And with all those classes, we can say we're *composing* them instead of *inheriting* them.

### Program to interfaces, not implementations


Some examples:

```java
// behaviour interface
public interface QuackBehaviour {
  quack();
}

// abstract superclass
public abstract class Duck {
  quackBehaviour: QuackBehaviour;
  //...
  //some common unchanging behaviours
  public void performQuack() {
    quackBehaviour.quack();
  }

  // we can add a setter in the superclass to define the quackBehaviour at runtime
  public void setQuackBehaviour(QuackBehaviour qb) {
    quackBehaviour = qb;
  }
}

// seperated behaviour classes that do the implementation
class Quack implements QuackBehaviour {
  public void quack() {
    System.out.println('quackkkkkkkk!')
  }
}

class SuperQuack implements QuackBehaviour {
  public void quack() {
    System.out.println('super quack! gagaga')
  }
}

// Then the subclass that extends the superclass by defining which quack behaviour it wants
public class RubberDuck extends Duck {
  public RubberDuck() {
    // N.B. here in the constructor we're still making a new instance of a concrete implementation class, but we can dynamically change it at runtime
    quackBehaviour = new Quack();
  }
}

// now when we use the RubberDuck on runtime
public class DuckSimulator {
  public static void main(String[] args) {
    Duck rubber = new RubberDuck();
    rubber.performQuack(); 
    // 'quackkkkkkkk!'
    rubber.setQuackBehaviour(new SuperQuack());
    rubber.performQuack(); 
    // 'super quack! gagaga'
  }
}
```

So, `HAS-A` can be better than `IS-A`. Or rather, favor composition over inheritance.

### Quiz
Later if we want to have a duck call device that doesn't inherits `Duck` class, but only do the quack to mimic the call of duck, how we can do it ?

```java
public class DuckCall {
  quackBehaviour: QuackBehavoir;

  public void setQuack(QuackBehavoir: qb) {
    quackBehaviour = qb;
  }
  public void performQuack() {
    quackBehaviour.quack();
  }
}

public class DuckCallSimulator {
  public static void main(String[] args) {
    DuckCall duckCall = new DuckCall();
    duckCall.setQuackBehaviour(new SuperQuack());
    duckCall.performQuack(); 
    // 'super quack! gagaga'
  }
}
```

### Sumup
We have the first OO pattern here, **Strategy parttern** : that is, defines *a group of algorithms* (behaviours classes like the example above), and make them *interchangeable* (you can replace `Quack` with `SuperQuack` without changing anything else as long as they implement the same interface).

By doing so we achieved the *polymorphism* at runtime, and we compose those algorithms instead of inherit the class that implement them. (N.B. the composition is done by using `setters` in the abstract superclass).

## Loosely coupling: observer pattern
- We have a publisher (Subject)
- and multiple subscribers (Observers)

Everytime there's update on the data, the subject publish a notification, then all the observers who subscirbe to it get notified. And observers can register/unregister anytime they want.

So we have a typical *one-to-many* dependency. That is, one object (Subject) changes state, all of its dependents are notified.

Becaus the subject don't care about how many objects are subscirbed to its data, it will deliver notifications to any object that implements the Observer interface. That's why we say they are *loosely coupled*.

### Manual version of obersver pattern:

```java
// how we can define a data struct ?
class Data {
  private float temprature;
  private float pressure;
}

public interface Subject {
  addObserver(Observer o);
  removeObserver(Observer o);
  notifyChanges();
}

public interface Observer {
  update(Data data);
}

// the Subject class
public class CenteredData implements Subject {
  // we use an array to keep track of all subscribed observers
  private ArrayList<Observer> observers;
  pirvate Data data

  public CenteredData() {
    observers = new ArrayList<Observer>();
  }
  public void addObserver(Observer o) {
    observers.add(o);
  }
  public void removeObserver(Observer o) {
    int i = observers.indexOf(o);
    if (i >= 0) {
      observers.remove(i)
    }
  }
  public void notifyChanges() {
    // here's the fun part, how we notify the observers
    for (Observer observer : observers) {
      observer.update()
    }
  }
  public void onDataChanges(Data changedData) {
    this.data = changedData;
    notifyChanges();
  }

  // and as we are using pull model, we need to set getter for the observers
  public Data getData() {
    return data;
  }
}

public class ObserverA implements Observer {
  private Data data;
  private Subject centeredData
  public ObserverA(Subject centeredData) {
    // here we keep a reference to the centeredData object, so later we can call the `removeObserver` if needed
    this.centeredData = centeredData;
    centeredData.addObserver(this);
  }
  public void update() {
    // here it's the observer pulling data from observable
    this.data = this.centeredData.getData();
  }
}
```

### Java built-in observer pattern
Java has built-in support of observer pattern, in the `util`, which already handled the add/remove/notify part of it. We can re-implement the above like this:

```java
import java.util.Observable;
import java.util.Observer;

public class CenteredData extends Observable {
  pivate Data data;
  public CenteredData() {}
  public void notifyChanges() {
    // both methods are specified by the Observable superclass
    setChanged();
    // N.B. here if we pass nothing to the notify method, it use automatically the `pull` model, that is, the observers are responsible to pull the data from the subject, instead of the subject pushing data to the observers
    notifyObservers();
  }

  public void onDataChanges(Data changedData) {
    this.data = changedData;
    notifyChanges();
  }

  // and as we are using pull model, we need to set getter for the observers
  public Data getData() {
    return data;
  }
}

public class ObserverA implements Observer {
  Observable observable;
  private Data data;

  public ObserverA(Observable observable) {
    this.observable = observable;
    observable.addOberserver(this);
  }

  public void update(Observable obs, Object arg) {
    if (obs instanceof CenteredData) {
      // type casting in Java, as CenteredData is a subclass of Observable, it's called "explicit upcasting".
      CenteredData centeredData = (CenteredData)obs;
      this.data = centeredData.getData();
    }
  }
}
```

N.B. the order of notifications is not the same as the order in our code. The `notifyObservers()` method implements the order differently, and we can do nothing to change that: the built-in `Observable` class is a class not an interface:
- you need to *subclass* it to use, and it doesn't have an interface you can implements, so you're stuck with the current implementation of this superclass.
- it has a protected method `setChanged()`, so you cannot "compose" it but only "inherit" it.

### Anther example of observer pattern in Java world
- `AbstractButton` superclass in Swing is a `observable`, it has lots of methods to add/remove listeners (AKA `observers`). And an `ActionListener` can *listen in* on any types of actions on a button.

### Sumup
When using **Observer pattern** as a way of achieving loose coupling, generally it's considered more "correct" to use the **pull mode** instead of **push mode**.





