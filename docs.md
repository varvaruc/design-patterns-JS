# Design Patterns JS

**[Behavioral](#behavioral)**

-   [Chain Of Resp](#chain-of-resp)
-   [Command](#command)
-   [Interpreter](#interpreter)
-   [Iterator](#iterator)
-   [Mediator](#mediator)
-   [Memento](#memento)
-   [Observer](#observer)
-   [State](#state)
-   [Strategy](#strategy)
-   [Template](#template)
-   [Visitor](#visitor)

**[Creational](#creational)**

-   [Abstract Factory](#abstract-factory)
-   [Builder](#builder)
-   [Factory](#factory)
-   [Prototype](#prototype)
-   [Singleton](#singleton)

**[Structural](#structural)**

-   [Adapter](#adapter)
-   [Bridge](#bridge)
-   [Composite](#composite)
-   [Decorator](#decorator)
-   [Facade](#facade)
-   [Flyweight](#flyweight)
-   [Proxy](#proxy)

## behavioral

### Chain Of Resp

delegates commands to a chain of processing objects.

##### chain-of-resp-es6.js

```Javascript
class ShoppingCart {

  constructor() {
    this.products = [];
  }

  addProduct(p) {
    this.products.push(p);
  };
}

class Discount {

  calc(products) {
    let ndiscount = new NumberDiscount();
    let pdiscount = new PriceDiscount();
    let none = new NoneDiscount();
    ndiscount.setNext(pdiscount);
    pdiscount.setNext(none);
    return ndiscount.exec(products);
  };
}

class NumberDiscount {

  constructor() {
    this.next = null;
  }

  setNext(fn) {
    this.next = fn;
  };

  exec(products) {
    let result = 0;
    if (products.length > 3)
      result = 0.05;

    return result + this.next.exec(products);
  };
}

class PriceDiscount {

  constructor() {
    this.next = null;
  }

  setNext(fn) {
    this.next = fn;
  };

  exec(products) {
    let result = 0;
    let total = products.reduce((a, b) => a + b);

    if (total >= 500)
      result = 0.1;

    return result + this.next.exec(products);
  };
}

class NoneDiscount {
  exec() {
    return 0;
  };
}

export {
  ShoppingCart,
  Discount
};

```

##### chain-of-resp.js

```Javascript
function ShoppingCart() {
  this.products = [];

  this.addProduct = function(p) {
    this.products.push(p);
  };
}

function Discount() {
  this.calc = function(products) {
    var ndiscount = new NumberDiscount();
    var pdiscount = new PriceDiscount();
    var none = new NoneDiscount();

    ndiscount.setNext(pdiscount);
    pdiscount.setNext(none);

    return ndiscount.exec(products);
  };
}

function NumberDiscount() {
  this.next = null;
  this.setNext = function(fn) {
    this.next = fn;
  };

  this.exec = function(products) {
    var result = 0;
    if (products.length > 3)
      result = 0.05;

    return result + this.next.exec(products);
  };
}

function PriceDiscount() {
  this.next = null;

  this.setNext = function(fn) {
    this.next = fn;
  };

  this.exec = function(products) {
    var result = 0;
    var total = products.reduce(function(a, b) {
      return a + b;
    });

    if (total >= 500)
      result = 0.1;

    return result + this.next.exec(products);
  };
}

function NoneDiscount() {
  this.exec = function() {
    return 0;
  };
}

module.exports = [ShoppingCart, Discount];

```

chain-of-resp-es6-test.js

```javascript
const expect = require("chai").expect;

import {
    ShoppingCart,
    Discount,
} from "../src/behavioral/chain-of-resp/chain-of-resp-es6";

describe("chain of resp es6 tests", () => {
    it(" > $ 500", () => {
        const discount = new Discount();

        const sc = new ShoppingCart();
        sc.addProduct(1000);

        let resp = discount.calc(sc.products);

        expect(resp).to.equal(0.1);
    });

    it("more than 3 products", () => {
        const discount = new Discount();

        const sc = new ShoppingCart();
        sc.addProduct(100);
        sc.addProduct(100);
        sc.addProduct(100);
        sc.addProduct(100);

        let resp = discount.calc(sc.products);

        expect(resp).to.equal(0.05);
    });

    it("more than 3 products and > $ 500 ", () => {
        let discount = new Discount();

        let sc = new ShoppingCart();
        sc.addProduct(1000);
        sc.addProduct(100);
        sc.addProduct(100);
        sc.addProduct(100);

        let resp = discount.calc(sc.products);

        expect(resp.toFixed(2)).to.equal("0.15");
    });
});
```

### Command

creates objects which encapsulate actions and parameters.

##### command.js

```Javascript
function Cockpit(command) {
  this.command = command;
}

Cockpit.prototype.execute = function() {
  this.command.execute();
};

function Turbine() {
  this.speed = 0;
  this.state = false;
}

Turbine.prototype.on = function() {
  this.state = true;
  this.speed = 100;
};

Turbine.prototype.off = function() {
  this.speed = 0;
  this.state = false;
};

Turbine.prototype.speedDown = function() {
  if (!this.state) return;

  this.speed -= 100;
};

Turbine.prototype.speedUp = function() {
  if (!this.state) return;

  this.speed += 100;
};

function OnCommand(turbine) {
  this.turbine = turbine;
}

OnCommand.prototype.execute = function() {
  this.turbine.on();
};

function OffCommand(turbine) {
  this.turbine = turbine;
}

OffCommand.prototype.execute = function() {
  this.turbine.off();
};

function SpeedUpCommand(turbine) {
  this.turbine = turbine;
}

SpeedUpCommand.prototype.execute = function() {
  this.turbine.speedUp();
};

function SpeedDownCommand(turbine) {
  this.turbine = turbine;
}

SpeedDownCommand.prototype.execute = function() {
  this.turbine.speedDown();
};

module.exports = [Cockpit, Turbine, OnCommand, OffCommand, SpeedUpCommand, SpeedDownCommand];

```

##### command_es6.js

```Javascript
class Cockpit {

  constructor(command) {
    this.command = command;
  }

  execute() {
    this.command.execute();
  }
}

class Turbine {

  constructor() {
    this.state = false;
  }

  on() {
    this.state = true;
  }

  off() {
    this.state = false;
  }
}

class OnCommand {

  constructor(turbine) {
    this.turbine = turbine;
  }

  execute() {
    this.turbine.on();
  }
}

class OffCommand {

  constructor(turbine) {
    this.turbine = turbine;
  }

  execute() {
    this.turbine.off();
  }
}

export {
  Cockpit,
  Turbine,
  OnCommand,
  OffCommand
};

```

command_es6-test.js

```javascript
const expect = require("chai").expect;

import {
    Cockpit,
    Turbine,
    OnCommand,
    OffCommand,
} from "../src/behavioral/command/command_es6";

describe("command es6 tests", () => {
    it("turn off/on test", () => {
        var turbine = new Turbine();
        const onCommand = new OnCommand(turbine);
        const cockpit = new Cockpit(onCommand);
        cockpit.execute();
        expect(turbine.state).to.be.true;
    });
});
```

### Interpreter

implements a specialized language.

##### interpreter.js

```Javascript
function Sum(left, right) {
  this.left = left;
  this.right = right;
}

Sum.prototype.interpret = function() {
  return this.left.interpret() + this.right.interpret();
};

function Min(left, right) {
  this.left = left;
  this.right = right;
}

Min.prototype.interpret = function() {
  return this.left.interpret() - this.right.interpret();
};

function Num(val) {
  this.val = val;
}

Num.prototype.interpret = function() {
  return this.val;
};

module.exports = [Num, Min, Sum];

```

##### interpreter_es6.js

```Javascript
class Sum {

  constructor(left, right) {
    this.left = left;
    this.right = right;
  }

  interpret() {
    return this.left.interpret() + this.right.interpret();
  }
}

class Min {

  constructor(left, right) {
    this.left = left;
    this.right = right;
  }

  interpret() {
    return this.left.interpret() - this.right.interpret();
  }
}

class Num {

  constructor(val) {
    this.val = val;
  }

  interpret() {
    return this.val;
  }
}

export {
  Num,
  Min,
  Sum
};

```

interpreter_es6-test.js

```javascript
const expect = require("chai").expect;

import { Num, Min, Sum } from "../src/behavioral/interpreter/interpreter_es6";

describe("interpreter tests", () => {
    it("sanity", () => {
        var result = new Sum(new Num(100), new Min(new Num(100), new Num(50)));
        expect(result.interpret()).to.equal(150);
    });
});
```

### Iterator

accesses the elements of an object sequentially without exposing its underlying representation.

##### iterator.js

```Javascript
function Iterator(el) {
  this.index = 0;
  this.elements = el;
}

Iterator.prototype = {
  next: function() {
    return this.elements[this.index++];
  },
  hasNext: function() {
    return this.index < this.elements.length;
  }
};

module.exports = Iterator;

```

##### iterator_es6.js

```Javascript
class Iterator {

  constructor(el) {
    this.index = 0;
    this.elements = el;
  }

  next() {
    return this.elements[this.index++];
  }

  hasNext() {
    return this.index < this.elements.length;
  }
}

export default Iterator;

```

iterator-test.js

```javascript
const expect = require("chai").expect;

const Iterator = require("../src/behavioral/iterator/iterator");

import Iterator6 from "../src/behavioral/iterator/iterator_es6";

describe("iterator tests", () => {
    it("sanity", () => {
        test(Iterator);
    });
});

describe("iterator es6 tests", () => {
    it("sanity", () => {
        test(Iterator6);
    });
});

function test(Iterator) {
    var numbers = new Iterator([1, 2, 3]);

    expect(numbers.next()).to.equal(1);
    expect(numbers.next()).to.equal(2);
    expect(numbers.next()).to.equal(3);
    expect(numbers.hasNext()).to.false;
}
```

### Mediator

allows loose coupling between classes by being the only class that has detailed knowledge of their methods.

##### mediator.js

```Javascript
function TrafficTower() {
  this.airplanes = [];
}

TrafficTower.prototype.requestPositions = function() {
  return this.airplanes.map(function(airplane) {
    return airplane.position;
  });
};

function Airplane(position, trafficTower) {
  this.position = position;
  this.trafficTower = trafficTower;
  this.trafficTower.airplanes.push(this);
}

Airplane.prototype.requestPositions = function() {
  return this.trafficTower.requestPositions();
};

module.exports = [TrafficTower, Airplane];

```

##### mediator_es6.js

```Javascript
class TrafficTower {

  constructor() {
    this.airplanes = [];
  }

  requestPositions() {
    return this.airplanes.map(airplane => {
      return airplane.position;
    });
  }
}

class Airplane {

  constructor(position, trafficTower) {
    this.position = position;
    this.trafficTower = trafficTower;
    this.trafficTower.airplanes.push(this);
  }

  requestPositions() {
    return this.trafficTower.requestPositions();
  }
}

export {
  TrafficTower,
  Airplane
};

```

mediator_es6-test.js

```javascript
const expect = require("chai").expect;

import {
    TrafficTower,
    Airplane,
} from "../src/behavioral/mediator/mediator_es6";

describe("mediator es6 tests", () => {
    it("sanity", () => {
        const trafficTower = new TrafficTower();
        const boeing1 = new Airplane(10, trafficTower);
        const boeing2 = new Airplane(15, trafficTower);
        const boeing3 = new Airplane(55, trafficTower);

        expect(boeing1.requestPositions()).to.deep.equals([10, 15, 55]);
    });
});
```

### Memento

provides the ability to restore an object to its previous state (undo).

##### memento.js

```Javascript
function Memento(value) {
  this.value = value;
}

var originator = {
  store: function(val) {
    return new Memento(val);
  },
  restore: function(memento) {
    return memento.value;
  }
};

function Caretaker() {
  this.values = [];
}

Caretaker.prototype.addMemento = function(memento) {
  this.values.push(memento);
};

Caretaker.prototype.getMemento = function(index) {
  return this.values[index];
};

module.exports = [originator, Caretaker];

```

##### memento_es6.js

```Javascript
class Memento {
  constructor(value) {
    this.value = value;
  }
}

const originator = {
  store: function(val) {
    return new Memento(val);
  },
  restore: function(memento) {
    return memento.value;
  }
};

class Caretaker {

  constructor() {
    this.values = [];
  }

  addMemento(memento) {
    this.values.push(memento);
  }

  getMemento(index) {
    return this.values[index];
  }
}

export {
  originator,
  Caretaker
};

```

memento_es6.js

```javascript
const expect = require("chai").expect;

import { originator, Caretaker } from "../src/behavioral/memento/memento_es6";

describe("memento es6 tests", () => {
    it("sanity", () => {
        const careTaker = new Caretaker();
        careTaker.addMemento(originator.store("hello"));
        careTaker.addMemento(originator.store("hello world"));
        careTaker.addMemento(originator.store("hello world !!!"));

        var result = originator.restore(careTaker.getMemento(1));
        expect(result).to.equal("hello world");
    });
});
```

### Observer

is a publish/subscribe pattern which allows a number of observer objects to see an event.

##### observer.js

```Javascript
function Product() {
  this.price = 0;
  this.actions = [];
}

Product.prototype.setBasePrice = function(val) {
  this.price = val;
  this.notifyAll();
};

Product.prototype.register = function(observer) {
  this.actions.push(observer);
};

Product.prototype.unregister = function(observer) {
  this.actions = this.actions.filter(function(el) {
    return el != observer;
  });
};

Product.prototype.notifyAll = function() {
  return this.actions.forEach(function(el) {
    el.update(this);
  }.bind(this));
};

var fees = {
  update: function(product) {
    product.price = product.price * 1.2;
  }
};

var proft = {
  update: function(product) {
    product.price = product.price * 2;
  }
};

module.exports = [Product, fees, proft];

```

##### observer_es6.js

```Javascript
class Product {
  constructor() {
    this.price = 0;
    this.actions = [];
  }

  setBasePrice(val) {
    this.price = val;
    this.notifyAll();
  }

  register(observer) {
    this.actions.push(observer);
  }

  unregister(observer) {
    this.actions = this.actions.filter(el => !(el instanceof observer));
  }

  notifyAll() {
    return this.actions.forEach(el => el.update(this));
  }
}

class Fees {
  update(product) {
    product.price = product.price * 1.2;
  }
}

class Proft {
  update(product) {
    product.price = product.price * 2;
  }
}

export {
  Product,
  Fees,
  Proft
};

```

observer_es6-test.js

```javascript
const expect = require("chai").expect;
import { Product, Fees, Proft } from "../src/behavioral/observer/observer_es6";

function register(p, f, t) {
    p.register(f);
    p.register(t);
    return p;
}

describe("Observer es6 test", () => {
    it("Subscribers are triggered", () => {
        let product = register(new Product(), new Fees(), new Proft());
        product.setBasePrice(100);
        expect(product.price).to.equal(240);
    });

    it("We are able to unregister a subscriber", () => {
        let product = register(new Product(), new Fees(), new Proft());
        product.unregister(Proft);

        product.setBasePrice(100);
        expect(product.price).to.equal(120);
    });
});
```

### State

allows an object to alter its behavior when its internal state changes.

##### state.js

```Javascript
function Order() {
  this.state = new WaitingForPayment();

  this.nextState = function() {
    this.state = this.state.next();
  };
}

function WaitingForPayment() {
  this.name = 'waitingForPayment';
  this.next = function() {
    return new Shipping();
  };
}

function Shipping() {
  this.name = 'shipping';
  this.next = function() {
    return new Delivered();
  };
}

function Delivered() {
  this.name = 'delivered';
  this.next = function() {
    return this;
  };
}

module.exports = Order;

```

##### state_es6.js

```Javascript
class OrderStatus {
  constructor(name, nextStatus) {
    this.name = name;
    this.nextStatus = nextStatus;
  }

  next() {
    return new this.nextStatus();
  }
}

class WaitingForPayment extends OrderStatus {
  constructor() {
    super('waitingForPayment', Shipping);
  }
}

class Shipping extends OrderStatus {
  constructor() {
    super('shipping', Delivered);
  }
}

class Delivered extends OrderStatus {
  constructor() {
    super('delivered', Delivered);
  }
}

class Order {
  constructor() {
    this.state = new WaitingForPayment();
  }

  nextState() {
    this.state = this.state.next();
  };
}

export default Order;

```

state_es6-test.js

```javascript
const expect = require("chai").expect;

import Order from "../src/behavioral/state/state_es6";

describe("state_es6 tests", () => {
    it("sanity", () => {
        const order = new Order();
        expect(order.state.name).to.equal("waitingForPayment");
        order.nextState();
        expect(order.state.name).to.equal("shipping");
        order.nextState();
        expect(order.state.name).to.equal("delivered");
    });
});
```

### Strategy

allows one of a family of algorithms to be selected on-the-fly at runtime.

##### strategy.js

```Javascript
function ShoppingCart(discount) {
  this.discount = discount;
  this.amount = 0;
}

ShoppingCart.prototype.setAmount = function(amount) {
  this.amount = amount;
};

ShoppingCart.prototype.checkout = function() {
  return this.discount(this.amount);
};

function guestStrategy(amount) {
  return amount;
}

function regularStrategy(amount) {
  return amount * 0.9;
}

function premiumStrategy(amount) {
  return amount * 0.8;
}

module.exports = [ShoppingCart, guestStrategy, regularStrategy, premiumStrategy];

```

##### strategy_es6.js

```Javascript
class ShoppingCart {

  constructor(discount) {
    this.discount = discount;
    this.amount = 0;
  }

  checkout() {
    return this.discount(this.amount);
  }

  setAmount(amount) {
    this.amount = amount;
  }
}

function guestStrategy(amount) {
  return amount;
}

function regularStrategy(amount) {
  return amount * 0.9;
}

function premiumStrategy(amount) {
  return amount * 0.8;
}

export {
  ShoppingCart,
  guestStrategy,
  regularStrategy,
  premiumStrategy
};

```

strategy_es6-test.js

```javascript
const expect = require("chai").expect;

import {
    ShoppingCart,
    guestStrategy,
    regularStrategy,
    premiumStrategy,
} from "../src/behavioral/strategy/strategy_es6.js";

describe("strategy es6 tests", () => {
    it("guest test", () => {
        const guestCart = new ShoppingCart(guestStrategy);
        guestCart.setAmount(100);
        expect(guestCart.checkout()).to.equal(100);
    });

    it("regular test", () => {
        const regularCart = new ShoppingCart(regularStrategy);
        regularCart.setAmount(100);
        expect(regularCart.checkout()).to.equal(90);
    });

    it("premium test", () => {
        const premiumCart = new ShoppingCart(premiumStrategy);
        premiumCart.setAmount(100);
        expect(premiumCart.checkout()).to.equal(80);
    });
});
```

### Template

method defines the skeleton of an algorithm as an abstract class, allowing its subclasses to provide concrete behavior.

##### template.js

```Javascript
function Tax() {}

Tax.prototype.calc = function(value) {
  if (value >= 1000)
    value = this.overThousand(value);

  return this.complementaryFee(value);
};

Tax.prototype.complementaryFee = function(value) {
  return value + 10;
};

function Tax1() {}
Tax1.prototype = Object.create(Tax.prototype);

Tax1.prototype.overThousand = function(value) {
  return value * 1.1;
};

function Tax2() {}
Tax2.prototype = Object.create(Tax.prototype);

Tax2.prototype.overThousand = function(value) {
  return value * 1.2;
};

module.exports = [Tax1, Tax2];

```

##### template_es6.js

```Javascript
class Tax {

  calc(value) {
    if (value >= 1000)
      value = this.overThousand(value);

    return this.complementaryFee(value);
  }

  complementaryFee(value) {
    return value + 10;
  }

}

class Tax1 extends Tax {

  constructor() {
    super();
  }

  overThousand(value) {
    return value * 1.1;
  }
}

class Tax2 extends Tax {

  constructor() {
    super();
  }

  overThousand(value) {
    return value * 1.2;
  }
}

export {
  Tax1,
  Tax2
};

```

template_es6-test.js

```javascript
const expect = require("chai").expect;

import { Tax1, Tax2 } from "../src/behavioral/template/template_es6";

describe("template es6 tests", () => {
    it("sanity", () => {
        const tax1 = new Tax1();
        const tax2 = new Tax2();

        expect(tax1.calc(1000)).to.equal(1110);
        expect(tax2.calc(1000)).to.equal(1210);
        expect(tax2.calc(100)).to.equal(110);
    });
});
```

### Visitor

separates an algorithm from an object structure by moving the hierarchy of methods into one object.

##### visitor.js

```Javascript
function bonusVisitor(employee) {
  if (employee instanceof Manager)
    employee.bonus = employee.salary * 2;
  if (employee instanceof Developer)
    employee.bonus = employee.salary;
}

function Employee() {
  this.bonus = 0;
}

Employee.prototype.accept = function(visitor) {
  visitor(this);
};

function Manager(salary) {
  this.salary = salary;
}

Manager.prototype = Object.create(Employee.prototype);

function Developer(salary) {
  this.salary = salary;
}

Developer.prototype = Object.create(Employee.prototype);

module.exports = [Developer, Manager, bonusVisitor];

```

##### visitor_es6.js

```Javascript
function bonusVisitor(employee) {
  if (employee instanceof Manager)
    employee.bonus = employee.salary * 2;
  if (employee instanceof Developer)
    employee.bonus = employee.salary;
}

class Employee {

  constructor(salary) {
    this.bonus = 0;
    this.salary = salary;
  }

  accept(visitor) {
    visitor(this);
  }
}

class Manager extends Employee {
  constructor(salary) {
    super(salary);
  }
}

class Developer extends Employee {
  constructor(salary) {
    super(salary);
  }
}

export {
  Developer,
  Manager,
  bonusVisitor
};

```

visitor_es6-test.js

```javascript
const expect = require("chai").expect;
import {
    Developer,
    Manager,
    bonusVisitor,
} from "../src/behavioral/visitor/visitor_es6";

describe("visitor es6 tests", () => {
    it("sanity", () => {
        let employees = [];

        const john = new Developer(4000);
        const maria = new Developer(4000);
        const christian = new Manager(10000);

        employees.push(john);
        employees.push(maria);
        employees.push(christian);

        employees.forEach((e) => {
            e.accept(bonusVisitor);
        });

        expect(john.bonus).to.equal(4000);
        expect(christian.bonus).to.equal(20000);
    });
});
```

## creational

### Abstract Factory

provide an interface for creating families of related or dependent objects without specifying their concrete classes.

##### abstract-factory.js

```Javascript
function droidProducer(kind) {
  if (kind === 'battle') return battleDroidFactory;
  return pilotDroidFactory;
}

function battleDroidFactory() {
  return new B1();
}

function pilotDroidFactory() {
  return new Rx24();
}

function B1() {}
B1.prototype.info = function() {
  return "B1, Battle Droid";
};

function Rx24() {}
Rx24.prototype.info = function() {
  return "Rx24, Pilot Droid";
};

module.exports = droidProducer;

```

##### abstract-factory_es6.js

```Javascript
function droidProducer(kind) {
  if (kind === 'battle') return battleDroidFactory;
  return pilotDroidFactory;
}

function battleDroidFactory() {
  return new B1();
}

function pilotDroidFactory() {
  return new Rx24();
}

class B1 {
  info() {
    return "B1, Battle Droid";
  }
}

class Rx24 {
  info() {
    return "Rx24, Pilot Droid";
  }
}

export default droidProducer;

```

abstract-factory-test.js

```javascript
const expect = require("chai").expect;
const droidProducer = require("../src/creational/abstract-factory/abstract-factory");
import droidProducer6 from "../src/creational/abstract-factory/abstract-factory_es6";

describe("abstract factory test", () => {
    it("Battle droid", () => {
        const battleDroid = droidProducer("battle")();
        expect(battleDroid.info()).to.equal("B1, Battle Droid");
    });

    it("pilot droid", () => {
        const pilotDroid = droidProducer("pilot")();
        expect(pilotDroid.info()).to.equal("Rx24, Pilot Droid");
    });

    it("Battle droid es6", () => {
        const battleDroid = droidProducer6("battle")();
        expect(battleDroid.info()).to.equal("B1, Battle Droid");
    });

    it("pilot droid 6", () => {
        const pilotDroid = droidProducer6("pilot")();
        expect(pilotDroid.info()).to.equal("Rx24, Pilot Droid");
    });
});
```

### Builder

separate the construction of a complex object from its representation, allowing the same construction process to create various representations.

##### builder.js

```Javascript
function Request() {
  this.url = '';
  this.method = '';
  this.payload = {};
}

function RequestBuilder() {

  this.request = new Request();

  this.forUrl = function(url) {
    this.request.url = url;
    return this;
  };

  this.useMethod = function(method) {
    this.request.method = method;
    return this;
  };

  this.payload = function(payload) {
    this.request.payload = payload;
    return this;
  };

  this.build = function() {
    return this.request;
  };

}

module.exports = RequestBuilder;

```

##### builder_es6.js

```Javascript
class Request {
  constructor() {
    this.url = '';
    this.method = '';
    this.payload = {};
  }
}

class RequestBuilder {
  constructor() {
    this.request = new Request();
  }

  forUrl(url) {
    this.request.url = url;
    return this;
  }

  useMethod(method) {
    this.request.method = method;
    return this;
  }

  payload(payload) {
    this.request.payload = payload;
    return this;
  }

  build() {
    return this.request;
  }

}

export default RequestBuilder;

```

builder_es6-test.js

```javascript
const expect = require("chai").expect;
import RequestBuilder from "../src/creational/builder/builder_es6";

describe("builder es6 test", () => {
    it("sanity", () => {
        const requestBuilder = new RequestBuilder();
        const url = "http://something/users";
        const method = "GET";
        const request = requestBuilder
            .forUrl(url)
            .useMethod(method)
            .payload(null)
            .build();

        expect(request.method).to.equal(method);
        expect(request.payload).to.be.null;
        expect(request.url).to.equal(url);
    });
});
```

### Factory

define an interface for creating a single object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.

##### factory.js

```Javascript
function bmwFactory(type) {
  if (type === 'X5')
    return new Bmw(type, 108000, 300);
  if (type === 'X6')
    return new Bmw(type, 111000, 320);
}

function Bmw(model, price, maxSpeed) {
  this.model = model;
  this.price = price;
  this.maxSpeed = maxSpeed;
}

module.exports = bmwFactory;

```

##### factory_es6.js

```Javascript
class BmwFactory {

  static create(type) {
    if (type === 'X5')
      return new Bmw(type, 108000, 300);
    if (type === 'X6')
      return new Bmw(type, 111000, 320);
  }
}

class Bmw {
  constructor(model, price, maxSpeed) {
    this.model = model;
    this.price = price;
    this.maxSpeed = maxSpeed;
  }
}

export default BmwFactory;

```

factory_es6-test.js

```javascript
const expect = require("chai").expect;
import BmwFactory from "../src/creational/factory/factory_es6";

describe("Factory es6 test", () => {
    it("We can create a X5 instance", () => {
        const x5 = BmwFactory.create("X5");
        expect(x5.model).to.equal("X5");
    });

    it("The X5 price is properly set", () => {
        const x5 = BmwFactory.create("X5");
        expect(x5.price).to.equal(108000);
    });
});
```

### Prototype

specify the kinds of objects to create using a prototypical instance, and create new objects from the 'skeleton' of an existing object, thus boosting performance and keeping memory footprints to a minimum.

##### prototype.js

```Javascript
function Sheep(name, weight) {
  this.name = name;
  this.weight = weight;
}

Sheep.prototype.clone = function() {
  return new Sheep(this.name, this.weight);
};

module.exports = Sheep;

```

##### prototype_es6.js

```Javascript
class Sheep {

  constructor(name, weight) {
    this.name = name;
    this.weight = weight;
  }

  clone() {
    return new Sheep(this.name, this.weight);
  }
}

export default Sheep;

```

prototype_es6-test.js

```javascript
const expect = require("chai").expect;
import Sheep from "../src/creational/prototype/prototype_es6";

describe("prototype_es6 test", () => {
    it("sanity", () => {
        var sheep = new Sheep("dolly", 10.3);
        var dolly = sheep.clone();
        expect(dolly.name).to.equal("dolly");
        expect(dolly.weight).to.equal(10.3);
        expect(dolly).to.be.instanceOf(Sheep);
        expect(dolly === sheep).to.be.false;
    });
});
```

### Singleton

ensure a class has only one instance, and provide a global point of access to it.

##### singleton.js

```Javascript
function Person() {

  if (typeof Person.instance === 'object')
    return Person.instance;

  Person.instance = this;

  return this;
}

module.exports = Person;

```

##### singleton_es6.js

```Javascript
class Person {
  constructor() {
    if (typeof Person.instance === 'object') {
      return Person.instance;
    }
    Person.instance = this;
    return this;
  }
}

export default Person;

```

singleton_es6-test.js

```javascript
const expect = require("chai").expect;
import Person from "../src/creational/singleton/singleton_es6";

describe("singleton_es6 test", () => {
    it("sanity", () => {
        var john = new Person();
        var john2 = new Person();

        expect(john).to.equal(john2);
        expect(john === john2).to.be.true;
    });
});
```

## structural

### Adapter

allows classes with incompatible interfaces to work together by wrapping its own interface around that of an already existing class.

##### adapter.js

```Javascript
function Soldier(lvl) {
  this.lvl = lvl;
}

Soldier.prototype.attack = function() {
  return this.lvl * 1;
};

function Jedi(lvl) {
  this.lvl = lvl;
}

Jedi.prototype.attackWithSaber = function() {
  return this.lvl * 100;
};

function JediAdapter(jedi) {
  this.jedi = jedi;
}

JediAdapter.prototype.attack = function() {
  return this.jedi.attackWithSaber();
};

module.exports = [Soldier, Jedi, JediAdapter];

```

##### adapter_es6.js

```Javascript
class Soldier {
  constructor(level) {
    this.level = level;
  }

  attack() {
    return this.level * 1;
  }
}

class Jedi {
  constructor(level) {
    this.level = level;
  }

  attackWithSaber() {
    return this.level * 100;
  }
}

class JediAdapter {
  constructor(jedi) {
    this.jedi = jedi;
  }

  attack() {
    return this.jedi.attackWithSaber();
  }
}

export {
  Soldier,
  Jedi,
  JediAdapter
};

```

adapter_es6-test.js

```javascript
const expect = require("chai").expect;

import {
    Soldier,
    Jedi,
    JediAdapter,
} from "../src/structural/adapter/adapter_es6";

describe("adapter_es6 tests", () => {
    it("sanity", () => {
        const stormtrooper = new Soldier(1);
        const yoda = new JediAdapter(new Jedi(10));
        expect(yoda.attack()).to.equal(stormtrooper.attack() * 1000);
    });
});
```

### Bridge

decouples an abstraction from its implementation so that the two can vary independently.

##### bridge.js

```Javascript
function EpsonPrinter(ink) {
  this.ink = ink();
}
EpsonPrinter.prototype.print = function() {
  return "Printer: Epson, Ink: " + this.ink;
};

function HPprinter(ink) {
  this.ink = ink();
}
HPprinter.prototype.print = function() {
  return "Printer: HP, Ink: " + this.ink;
};

function acrylicInk() {
  return "acrylic-based";
}

function alcoholInk() {
  return "alcohol-based";
}

module.exports = [EpsonPrinter, HPprinter, acrylicInk, alcoholInk];

```

##### bridge_es6.js

```Javascript
class Printer {
  constructor(ink) {
    this.ink = ink;
  }
}

class EpsonPrinter extends Printer {
  constructor(ink) {
    super(ink);
  }

  print() {
    return "Printer: Epson, Ink: " + this.ink.get();
  }
}

class HPprinter extends Printer {
  constructor(ink) {
    super(ink);
  }

  print() {
    return "Printer: HP, Ink: " + this.ink.get();
  }
}

class Ink {
  constructor(type) {
    this.type = type;
  }
  get() {
    return this.type;
  }
}

class AcrylicInk extends Ink {
  constructor() {
    super("acrylic-based");
  }
}

class AlcoholInk extends Ink {
  constructor() {
    super("alcohol-based");
  }
}

export {
  EpsonPrinter,
  HPprinter,
  AcrylicInk,
  AlcoholInk
};

```

bridge_es6-test.js

```javascript
const expect = require("chai").expect;

import {
    EpsonPrinter,
    HPprinter,
    AcrylicInk,
    AlcoholInk,
} from "../src/structural/bridge/bridge_es6";

describe("bridge es6 tests", () => {
    it("Epson test", () => {
        const printer = new EpsonPrinter(new AlcoholInk());
        const result = printer.print();
        expect(result).to.equal("Printer: Epson, Ink: alcohol-based");
    });

    it("HP test", () => {
        const printer = new HPprinter(new AcrylicInk());
        const result = printer.print();
        expect(result).to.equal("Printer: HP, Ink: acrylic-based");
    });
});
```

### Composite

composes zero-or-more similar objects so that they can be manipulated as one object.

##### composite.js

```Javascript
// composition
function EquipmentComposition(name) {
  this.equipments = [];
  this.name = name;
}

EquipmentComposition.prototype.add = function(equipment) {
  this.equipments.push(equipment);
};

EquipmentComposition.prototype.getPrice = function() {
  return this.equipments.map(function(equipment) {
    return equipment.getPrice();
  }).reduce(function(a, b) {
    return a + b;
  });
};

function Equipment() {}

Equipment.prototype.getPrice = function() {
  return this.price;
};

// -- leafs
function FloppyDisk() {
  this.name = "Floppy Disk";
  this.price = 70;
}
FloppyDisk.prototype = Object.create(Equipment.prototype);

function HardDrive() {
  this.name = "Hard Drive";
  this.price = 250;
}
HardDrive.prototype = Object.create(Equipment.prototype);

function Memory() {
  this.name = "8gb memomry";
  this.price = 280;
}
Memory.prototype = Object.create(Equipment.prototype);

module.exports = [EquipmentComposition, FloppyDisk, HardDrive, Memory];

```

##### composite_es6.js

```Javascript
//Equipment
class Equipment {

  getPrice() {
    return this.price || 0;
  }

  getName() {
    return this.name;
  }

  setName(name) {
    this.name = name;
  }
}

// --- composite ---
class Composite extends Equipment {

  constructor() {
    super();
    this.equipments = [];
  }

  add(equipment) {
    this.equipments.push(equipment);
  }

  getPrice() {
    return this.equipments.map(equipment => {
      return equipment.getPrice();
    }).reduce((a, b) => {
      return a + b;
    });
  }
}

class Cabinet extends Composite {
  constructor() {
    super();
    this.setName('cabinet');
  }
}

// --- leafs ---
class FloppyDisk extends Equipment {
  constructor() {
    super();
    this.setName('Floppy Disk');
    this.price = 70;
  }
}

class HardDrive extends Equipment {
  constructor() {
    super();
    this.setName('Hard Drive');
    this.price = 250;
  }
}

class Memory extends Equipment {
  constructor() {
    super();
    this.setName('Memory');
    this.price = 280;
  }
}

export {
  Cabinet,
  FloppyDisk,
  HardDrive,
  Memory
};

```

composite_es6-test.js

```javascript
const expect = require("chai").expect;

import {
    Cabinet,
    FloppyDisk,
    HardDrive,
    Memory,
} from "../src/structural/composite/composite_es6";

describe("composity tests", () => {
    it("sanity test", () => {
        const cabinet = new Cabinet();
        cabinet.add(new FloppyDisk());
        cabinet.add(new HardDrive());
        cabinet.add(new Memory());

        expect(cabinet.getPrice()).to.equal(600);
    });
});
```

### Decorator

dynamically adds/overrides behaviour in an existing method of an object.

##### decorator.js

```Javascript
function Pasta() {
  this.price = 0;
}
Pasta.prototype.getPrice = function() {
  return this.price;
};

function Penne() {
  this.price = 8;
}
Penne.prototype = Object.create(Pasta.prototype);

function SauceDecorator(pasta) {
  this.pasta = pasta;
}

SauceDecorator.prototype.getPrice = function() {
  return this.pasta.getPrice() + 5;
};

function CheeseDecorator(pasta) {
  this.pasta = pasta;
}

CheeseDecorator.prototype.getPrice = function() {
  return this.pasta.getPrice() + 3;
};

module.exports = [Penne, SauceDecorator, CheeseDecorator];

```

##### decorator_es6.js

```Javascript
class Pasta {
  constructor() {
    this.price = 0;
  }
  getPrice() {
    return this.price;
  }
}

class Penne extends Pasta {
  constructor() {
    super();
    this.price = 8;
  }
}

class PastaDecorator extends Pasta {
  constructor(pasta) {
    super();
    this.pasta = pasta;
  }

  getPrice() {
    return this.pasta.getPrice();
  }
}

class SauceDecorator extends PastaDecorator {
  constructor(pasta) {
    super(pasta);
  }

  getPrice() {
    return super.getPrice() + 5;
  }
}

class CheeseDecorator extends PastaDecorator {
  constructor(pasta) {
    super(pasta);
  }

  getPrice() {
    return super.getPrice() + 3;
  }
}

export {
  Penne,
  SauceDecorator,
  CheeseDecorator
};

```

decorator_es6-test.js

```javascript
const expect = require("chai").expect;
import {
    Penne,
    SauceDecorator,
    CheeseDecorator,
} from "../src/structural/decorator/decorator_es6";

describe("decorator es6 tests", () => {
    it("sanity test", () => {
        const penne = new Penne();
        const penneWithSauce = new SauceDecorator(penne);
        const penneWithSauceAndCheese = new CheeseDecorator(penneWithSauce);

        expect(penne.getPrice()).to.equal(8);
        expect(penneWithSauce.getPrice()).to.equal(13);
        expect(penneWithSauceAndCheese.getPrice()).to.equal(16);
    });
});
```

### Facade

provides a simplified interface to a large body of code.

##### facade.js

```Javascript
var shopFacade = {
  calc: function(price) {
    price = discount(price);
    price = fees(price);
    price += shipping();
    return price;
  }
};

function discount(value) {
  return value * 0.9;
}

function shipping() {
  return 5;
}

function fees(value) {
  return value * 1.05;
}

module.exports = shopFacade;

```

##### facade_es6.js

```Javascript
class ShopFacade {
  constructor() {
    this.discount = new Discount();
    this.shipping = new Shipping();
    this.fees = new Fees();
  }

  calc(price) {
    price = this.discount.calc(price);
    price = this.fees.calc(price);
    price += this.shipping.calc();
    return price;
  }
}

class Discount {

  calc(value) {
    return value * 0.9;
  }
}

class Shipping {
  calc() {
    return 5;
  }
}

class Fees {

  calc(value) {
    return value * 1.05;
  }
}

export default ShopFacade;

```

facade_es6-test.js

```javascript
const expect = require("chai").expect;

import ShopFacade from "../src/structural/facade/facade_es6";

describe("facade tests", () => {
    it("sanity", () => {
        const shop = new ShopFacade();
        const result = shop.calc(100);
        expect(result).to.equal(99.5);
    });
});
```

### Flyweight

reduces the cost of creating and manipulating a large number of similar objects.

##### flyweight.js

```Javascript
function Color(name) {
  this.name = name;
}

var colorFactory = {
  colors: {},
  create: function(name) {
    var color = this.colors[name];
    if (color) return color;

    this.colors[name] = new Color(name);
    return this.colors[name];
  }
};

module.exports = colorFactory;

```

##### flyweight_es6.js

```Javascript
class Color {
  constructor(name) {
    this.name = name
  }
}

class colorFactory {
  constructor(name) {
    this.colors = {};
  }
  create(name) {
    let color = this.colors[name];
    if (color) return color;
    this.colors[name] = new Color(name);
    return this.colors[name];
  }
};

export {
  colorFactory
};

```

flyweight_es6-test.js

```javascript
const expect = require("chai").expect;

import { colorFactory } from "../src/structural/flyweight/flyweight_es6";

describe("flyweight tests", () => {
    it("sanity", () => {
        const cf = new colorFactory();
        cf.create("RED");
        cf.create("RED");
        cf.create("RED");
        cf.create("YELLOW");
        cf.create("YELLOW");
        expect(Object.keys(cf.colors)).to.have.lengthOf(2);
    });
});
```

### Proxy

provides a placeholder for another object to control access, reduce cost, and reduce complexity.

##### proxy.js

```Javascript
function Car() {
  this.drive = function() {
    return "driving";
  };
}

function CarProxy(driver) {
  this.driver = driver;
  this.drive = function() {
    if (driver.age < 18)
      return "too young to drive";
    return new Car().drive();
  };
}

function Driver(age) {
  this.age = age;
}

module.exports = [Car, CarProxy, Driver];

```

##### proxy_es6.js

```Javascript
class Car {
  drive() {
    return "driving";
  };
}

class CarProxy {
  constructor(driver) {
    this.driver = driver;
  }
  drive() {
    return (this.driver.age < 18) ? "too young to drive" : new Car().drive();
  };
}

class Driver {
  constructor(age) {
    this.age = age;
  }
}

export {
  Car,
  CarProxy,
  Driver
};

```

proxy_es6-test.js

```javascript
const expect = require("chai").expect;

import { Car, CarProxy, Driver } from "../src/structural/proxy/proxy_es6";

describe("proxy es6 tests", () => {
    it("sanity", () => {
        let driver = new Driver(28);
        let kid = new Driver(10);

        let car = new CarProxy(driver);
        expect(car.drive()).to.equal("driving");

        car = new CarProxy(kid);
        expect(car.drive()).to.equal("too young to drive");
    });
});
```
