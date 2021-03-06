# 类和模块
前面介绍的每个Javascript对象都是一个属性集合，相互之间没有任何联系。在Javascript中也可以定义对象的类，让每个对象都共享某些属性。类的成员或实例都包含一些属性，用以存入或定义它们的状态，其中有些属性定义了它们的行为，这些行为通常由类定义的，而且为所有实例所共享。

在Javascript中，类的实现是基于其原型继承机制的。如果两个实例都从同一个原型对象上继承了属性，则它们就是同一个类的实例。如果两个对象继承自同一个原型，往往意味着(但不是绝对)它们是由同一个构造函数创建并初始化的。

Javascript中类的一个重要特性是动态可继承(dynamically extendable)。

## 9.1 类和原型
在Javascript中，类的所有实例对象都从同一个原型对象上继承属性。因此，原型对象是类的核心。前面定义了inherit()函数，这个函数返回一个新创建的对象，后者继承自某个原型对象，如果定义一个原型对象，然后通过inherit()函数创建一个继承自它的对象，这样就定义了一个Javascript类。通常，类的实例还需要进行一步的初始化，通常是通过定义一个函数来创建并初始化这个对象，如：

```javascript
// range.js 实现一个能表示值的范围的类
// 下面的工厂方法返回一个新的 范围对象
function range(from, to) {
	var r = inherit(range.methods);
	r.from = from;
	r.to = to;
	return r;
}

range.methods = {
	includes: function(x) {
		return this.from <= x && x <= this.to; 
	},
	foreach: function(f) {
		for (var x = Math.ceil(this.from); x <= this.to; x++) f(x);
	},
	toString: function() { return "(" + this.from + "..." + this.to + ")"; }
};

var r = range(1, 3);	
r.includes(2);				// true
r.foreach(console.log);		// 1 2 3	???
console.log(r);				// (1...3)  ???
```

这段代码定义了一个工厂方法range()，range()函数定义了一个属性range.methods，用以存放定义类的原型对象。range()函数给每个范围对象都定义了from和to属性，用以定义范围的起始位置和结束位置，这两个属性是非共享的。

## 9.2 类和构造函数
上面展示了在Javascript中定义类的一种方法，但这种方法并不常用，毕竟它没有定义构造函数，构造函数是用来初始化新创建的对象的。调用构造函数的一个重要特征是，构造函数的prototype属性被用做新对象的原型。这意味着通过同一个构造函数创建的所有对象都继承自一个相同的对象，因此它们都是同一个类的成员：

```javascript
// range2.js 实现一个能表示值的范围的类
function Range(from, to) {
	this.from = from;
	this.to = to;
}

Range.prototype = {
	includes: function(x) { return this.from <= x && x <= this.to; },
	foreach: function(f) { for (var x = Math.ceil(this.from); x <= this.to; x++) f(x); },
	toString: function() { return "(" + this.from + "..." + this.to + ")"; }
};

var r = new Range(1, 3);
r.includes(2);				// true
r.foreach(console.log);		// 1 2 3
console.log(r);				// (1...3)
```

这段代码与上节中比较，工厂方法函数range()转化为构造函数并重命名为Range()。构造对象时，使用new关键字。使用new关键字调用时，会自动使用Range.prototype作为新Range对象的原型。

### 9.2.1 构造函数和类的标识
上文提到原型对象是类的唯一标识：当且仅当两个对象继承自同一个原型对象时，它们才是属于同一个类的实例。而初始化对象的状态的构造函数则不能作为类的标识，两个构造函数的prototype属性可能指向同一个原型对象，那么这两个构造函数创建的实例是属于同一个类的。

构造函数的名字通常用作类名。然而，更根本地讲，当使用instanceof运算符来检测对象是否属于某个类时会用到构造函数。假设这里有一个对象r，要知道r是否是Range对象，这样写：

```javascript
r instanceof Range;			// 如果r继承自Range.prototype，则返回true
```

实际上instanceof运算符并不会检查r是否是由Range()构造函数初始化而来，而会检查r是否继承自Range.prototype。

### 9.2.2 constructor属性
将Range.prototype定义为一个新的对象，这个对象包含类所需要的方法。其实没有必需新创建一个对象，用单个对象直接量的属性就可以方便地定义原型上的方法。任何Javascript函数都可以用做构造函数，并且调用构造函数是需要用到一个prototype属性的。每个Javascript函数(除了Function.bind()方法)都自动拥有一个prototype属性。这个属性的值是一个对象，这个对象包含唯一一个不可枚举属性constructor。constructor属性的值是一个函数对象：

```javascript
var F = function() {};
var p = F.prototype;
var c = p.constructor;
c === F;				// true 对于任意函数F.prototype.constructor === F
```

构造函数的原型中存在预先定义好的constructor属性，这意味着对象通常继承的constructor均指代它们的构造函数。由于构造函数是类的"公共标识"，因此这个constructor属性为对象提供了类。

```javascript
var o = new F();
o.constructor === F;	// true
```

在上面的Range类使用了它自身的一个新对象重写预定义的Range.prototype对象。这人新定义的原型对象不含有constructor属性。因此Range类的实例也不含有constructor属性。可以显式给原型添加一个构造函数：

```javascript
Range.prototype = {
	constructor: Range,
	includes: function(x) { return this.from <= x && x <= this.to; },
	foreach: function(f) { for (var x = Math.ceil(this.from); x <= this.to; x++) f(x); },
	toString: function() { return "(" + this.from + "..." + this.to + ")"; }
};
```

另一种方式就是使用预定义的原型对象，预定义的原型对象包含constructor属性，然后依次给原型对象添加方法：

```javascript
Range.prototype.includes = function(x) { return this.from <= x && x <= this.to; };
Range.prototype.foreach = function(f) { for (var x = Math.ceil(this.from); x < this.to; x ++) f(x); };
Range.prototype.toString = function() { return "(" + this.from + "..." + this.to + ")"; };
```

## 9.3 Javascript中Java式的类继承
在Java或其他类似强类型面向对象语言中，类成员可能是这样的：
> 实例字段
>> 它们是基于实例的属性或变量，用以保存独立对象的状态。
> 实例方法
>> 它们是类的所有实例所共享的方法，由每个独立的实例调用。
> 类字段
>> 这些属性或变量是属于类的，而不是属于类的某个实例的。
> 类方法
>> 这些方法是属于类的，而不是属于类的某个实例的。

Javascript和Java的不同在于，Javascript中的函数都是以值的形式出现的，方法和字段之间并没有太大的区别。如果属性值是函数，那么这个属性就定义一个方法，否则它就是一个普通的属性。Javascript中的类牵扯三种不同的对象，三种对象的属性的行为和下面三种类成员非常相似：
> 构造函数对象
>> 构造函数(对象)为Javascript的类定义了名字。任何添加到这个构造函数对象中的属性都是类字段和类方法。
> 原型对象
>> 原型对象的属性被类的所有所有实例继承，如果原型对象的属性值是函数的话，这个函数就作为类的实例的方法来调用。
> 实例对象
>> 类的每个实例都是一个独立的对象，直接给这个实例定义的属性是不会为所有实例对象所共享的。定义在实例上的非函数属性，实际上是实例的字段。

在Javascript中定义类的步骤可以缩减为一个分三步的算法。第一步，先定义一个构造函数，并设置初始化新对象的实例属性。第二步，给构造函数的prototype对象定义实例的方法。第三步，给构造函数定义类字段和类属性。可以将这三步封装进一个简单的defineClass()函数中：

```javascript
function defineClass(constructor,			// 设置实例的属性的函数
					 methods,				// 实例的方法，复制到原型中
					 statics) {				// 类属性，复制到构造函数中
	if (methods) extend(constructor.prototype, methods);
	if (statics) extend(constructor, statics);
	return constructor;
}

// Range类的另一个实现
var SimpleRange = defineClass(function(f, t) { this.f = f; this.t = t; },
				{
					includes: function(x) { this.f <= x && x <= this.t; },
					toString: function() { return this.f + "..." + this.t; }
				},
				{ upto: function(t) { return new SimpleRange(0, t); } });
```

再如下面的复数类：

```javascript
function Complex(real, imaginary) {
	if (isNaN(real) && isNaN(imaginary))
		throw new TypeError();
	this.r = real;
	this.i = imaginary;
}

Complex.prototype.add = function(that) {
	return new Complex(this.r + that.r, this.i + that.i);
};

Complex.prototype.mul = function(that) {
	return new Complex(this.r * that.r - this.i * that.i,
					   this.r * that.i + this.i * that.r);
};

Complex.prototype.mag = function() {
	return Math.sqrt(this.r * this.r + this.i * this.i);
};

Complex.prototype.neg = function() {
	return new Complex(-this.r, -this.i);
};

Complex.prototype.toString = function() {
	return "{" + this.r + "," + this.i + "}";
};

Complex.prototype.equals = function(that) {
	return that != null &&
			that.constructor === Complex &&
			this.r === that.r &&
			this.i === that.i;
};

Complex.ZERO = new Complex(0, 0);
Complex.ONE = new Complex(1, 0);
Complex.I = new Complex(0, 1);

Complex.parse = function(s) {
	try {
		var m =  Complex._format.exec(s);
		return new Complex(parseFloat(m[1]), parseFloat(m[2]));
	} catch(x) {
		throw new TypeError("Can't parse '" + s + "' as a complex number.");
	}
};

Complex._format = /^\{([^,]+),([^,]+)\}$/;
```

使用如下：

```javascript
var c = new Complex(2, 3);
var d = new Complex(c.i, c.r);
c.add(d).toString();				// "{5,5}"
Complex.parse(c.toString()).add(c.neg()).equals(Complex.ZERO);
```

## 9.4 类的扩充
Javascript中基于原型的继承机制是动态的：对象从其原型继承属性，如果创建对象之后原型的属性发生改变，也会影响到继承这个原型的所有实例对象。这意味着可以通过给原型对象添加新方法扩充Javascript类。如给Complex类添加计算得数的共轭算法：

```javascript
Complex.prototype.conj = function() { return new Complex(this.r, this.i); };
```

Javascript内置类的原型对象也是一样可以扩展，也就是说可以给数字、字符串、数组、函数等数据类型添加方法。如：

```javascript
if (!Function.prototype.bind) {
	Function.prototype.bind = function(o /*, args */) {
		// bind()方法的代码...
	};
}
```

其他例子如：

```javascript
Number.prototype.times = function(f, context) {
	var n = Number(this);
	for (var i = 0; i < n; i ++) { f.call(context, i); }
};

var n = 3;
n.times(function(n) { console.log(n + " hello"); });

String.prototype.trim = String.prototype.trim || function() {
	if (!this) return this;
	return this.replace(/^\s+|\s+$/g, "");
};

Function.prototype.getName = function() {
	return this.name || this.toString().match(/function\s*([^()*])\(/)[1];
};
```

可以给Object.prototype添加方法，从而使所有的对象都可以调用这些方法。但这种做法并不推荐，因为在ECMAScript 5之前，无法将这些新增的方法设置为不可枚举的，如果给Object.prototype添加属性，这些属性是可以被for/in循环遍历到的。使用Object.defineProperty()方法可以安全地扩充Object.prototype。

## 9.5 类和类型
Javascript定义了少量的数据类型：null、undeinfed、布尔值、数字、字符串、函数和对象。typeof运算符可以得出值的类型。然后更希望将类作为类型来对待，这样就可以根据对象所必的类来区分它们。Javascript语言核中的内置对象可以根据它们的class属性来区分彼此，如classof()函数。但当使用本章所提到的技术来定义类时，实例对象的class属性都是Object。

### 9.5.1 instanceof运算符
instanceof运算符左操作数是待检测其类的对象，右操作数是定义类的构造函数。如果o继承自c.prototype，则表达式`o instanceof c`值为true。这里的继承可以上是直接继承，如果o所继承的对象继承自另一个对象，后一个对象继承自c.prototype，这个表达式的运算结果也是true。

构造函数是类的公共标识，但株型是唯一的标识。尽管instanceof运算符的右操作数是构造函数，但计算过程实际上是检测 了对象的继承关系，而不是检测创建对象的构造函数。

如果想检测对象的原型链上是否存在某个特定的原型对象，有没有不使用构造函数作为中介的方法呢？可以使用isPrototypeOf()方法，如：

```javascript
range.methods.isPrototypeOf(r);	// range.methods是原型对象
```

instanceof运算符和isPrototypeOf()方法的缺点是，无法通过对象来获得类名，只能检测对象是否属于指定的类名。在客户端Javascript中还有一个比较严重的问题，在多窗口和多框架子页面的Web应用中兼容性不佳。每个窗口或框架子页面都具有单独的执行上下文，每个上下文都包含独有的全局变量和一组构造函数。在两个不同框架页面中创建的两个数组继承自两个相同但相互独立的原型对象，其中一个框架页面中的数组不是另一个框架页面的Array()构造函数的实例，instanceof运算结果是false。

### 9.5.2 constructor属性
另一个识别对象是否属于某个类的方法是使用constructor属性。因为构造函数是类的公共标识，所以最直接的方法就是使用constructor属性，如：

```javascript
function typeAndValue(x) {
	if (x == null) return "";
	switch(x.constructor) {
		case Number: return "Number: " + x;
		case String: return "String: '" + x + "'";
		case Date: return "Date: " + x;
		case RegExp: return "RegExp: " + x;
		case Complex: return "Complex: " + x;
	}
}
```

在代码中关键字case后的表达式都是函数，如果改用typeof运算符或获取到对象的class属性的话，它们应当改为字符串。使用constructor属性检测对象属于某个类的技术的不足之处和instanceof一样。在多个执行上下文的场景中它是无法正常工作的。在每个框架页面各自拥有独立的构造函数集合，一个框架页面中的Array构造函数和另一个框架页面的Array构造函数不是同一个构造函数。

在Javascript中也并非所有的对象都包含constructor属性。在每个新创建的函数原型上默认会有constructor属性，但常常会忽略原型上的constructor属性。

### 9.5.3 构造函数的名称
为解决在不同执行上下文中存在构造函数的多个副本的问题，使用函数构造函数名字而不是构造函数本身作为类标识符。一个窗口的Array构造函数和另一个窗口的Array构造函数是不相等的，但它们的名字是一样的。在一些Javascript的实现中为函数对象提供了一个非标准的属性name，用来表示函数的名称。对于没有name属性的Javascript实现来说，可以将函数转换为字符串，然后从中提取出函数名。

```javascript
/**
 * 以字符串形式返回o的类型：
 * -如果o是null，返回"null"，如果o是NaN，返回"nan"
 * -如果typeof所返回的值不是"object"，则返回这个值
 * -如果o的类不是"Object"，则返回这个值
 * -如果o包含构造函数并且这个构造函数具有名称，则返回这个名称
 * -否则，一律返回"Object"
 **/
function type(o) {
	var t, c, n;			// type, class, name
	if (o === null) return "null";
	if (o !== o) return "nan";
	if ((t = typeof(o)) !== "object") return t;
	if ((c = classof(o)) !== "Object") return c;
	if (o.constructor && typeof o.constructor === "function" &&
		(n = o.constructor.getName())) return n;
	return "Object";
}

function classof(o) {
	return Object.prototype.toString.call(o).slice(8, -1);
};

Function.prototype.getName = function() {
	if ('name' in this) return this.name;
	return this.name = this.toString.match(/function\s*([^(]*)\(/)[1];
};
```

这种使用构造函数名字来识别对象的类的做法和使用constructor属性一样有一个问题，并不是所有的对象都具有constructor属性，此外，并不是所有的函数都有名字。如果使用不带名字的函数定义表达式定义一个构造函数，getName()方法则会返回空字符串：

```javascript
var Complex = function(x, y) { this.r = x; this.i = y; }
var Range = function Range(f, t) { this.from = f; this.to = t; }
```

### 9.5.4 鸭式辩型
上面的几种方式都会有些问题，至少在客户端Javascript中是如此。解决的办法就是不关注对象的类是什么，而关注对象能做什么。这种思考问题的方式在Python和Ruby中非常普遍，称为鸭式辩型：

```javascript
// 像鸭子一样走路、游泳并且嘎嘎叫的鸟就是鸭子。
```

下面的代码按鸭式辩型理念定义了quacks()函数。quacks()用以检查一个对象是否实现了剩下参数所表示的方法。对于除第一个参数外的每个参数，如果是字符串的话则直接检查是否存在以它命名的演绎法，如果是对象的话则检查第一个对象中的方法是否在这个对象中也具有同名的方法，如果参数是函数，则假定它是构造函数，函数将检查第一个对象实现的方法是否在构造函数的原型对象中也具有同名的方法。

```javascript
function quacks(o /*, ... */) {
	for (var i = 1; i < arguments.length; i ++) {
		var arg = arguments[i];
		switch (typeof arg) {
			case 'string':
				if (typeof o[arg] !== "function") return false;
				continue;
			case 'function':
				arg = arg.prototype;
			case 'object':
				for (var m in arg) {
					if (typeof arg[m] !== "function") continue;
					if (typeof o[m] !== "function") return false;
				}
		}
	}
	return true;
}
```

quacks()函数无法得知这些已经存在的属性的细节信息。

## 9.6 Javascript中的面向对象技术

### 9.6.1 一个例子：集合类
集合(set)是一种数据结构，用以表示非重复值的无序集合。集合的基础方法包括添加值、检测值是否在集合中。Javascript的对象是属性名以及与之对应的值的基本集合。如：

```javascript
function Set() {
	this.values = {};
	this.n = 0;
	this.add.apply(this, arguments);
}

Set.prototype.add = function() {
	for (var i = 0; i < arguments.length; i ++) {
		var val = arguments[i];
		var str = Set._v2s(val);
		if (!this.values.hasOwnProperty(str)) {
			this.values[str] = val;
			this.n++;
		}
	}
	return this;
};

Set.prototype.remove = function() {
	for (var i = 0; i < arguments.length; i ++) {
		var str = Set._v2s(arguments[i]);
		if (this.values.hasOwnProperty(str)) {
			delete this.values[str];
			this.n++;
		}
	}
	return this;
};

Set.prototype.contains = function(value) {
	return this.values.hasOwnProperty(Set._v2s(value));
};

Set.prototype.size = function() {
	return this.n;
};

Set.prototype.foreach = function(f, context) {
	for (var s in this.values) {
		if (this.values.hasOwnProperty(s))
			f.call(context, this.values[s]);
	}
};

Set._v2s = function(val) {
	switch(val) {
		case undefined:	return 'u';
		case null:		return 'n';
		case true:		return 't';
		case false:		return 'f';
		default: 
			switch(typeof val) {
				case 'number': return '#'+val;
				case 'string': return '"'+val;
				default: return '@'+objectId(val);
			}
	}
	function objectId(o) {
		var prop = "|**objectid**|";
		if (!o.hasOwnProperty(prop))
			o[prop] = Set._v2s.next++;
		return o[prop];
	}
};

Set._v2s.next = 100;
```

### 9.6.2 一个例子：枚举类型
枚举类型(enumerated type)是一种类型，它是值的有限集合，如果值定义为这个类型则该值是可列出的。如下单独函数enumeration()它不是构造函数，它并没有定义一个名叫enumeration的类，它是一个工厂方法，每次调用它都　会创建并返回一个新的类，如：

```javascript
var Coin = enumeration({Penny: 1, Nickel: 5, Dime: 10, Quarter: 25});
var c = Coin.Dime;
c instanceof Coin;				// true
c.constructor == Coin			// true
Coin.Quarter + 3 * Coin.Nickel;	// 40
Coin.Dime == 10;				// true
Coin.Dime > Coin.Nickel;		// true
String(Coin.Dime) + ":" + Coin.Dime;	// "Dime:10"
```

```javascript
function enumeration(namesToValues) {
	var enumeration = function() { throw "Can't Instantiate Enumerations"; };
	var proto = enumeration.prototype = {
		constructor: enumeration,
		toString: function() { return this.name; },
		valueOf: function() { return this.value; },
		toJSON: function() { return this.name; }
	};
	enumeration.values = [];
	for (name in namesToValues) {
		var e = inherit(proto);
		e.name = name;
		e.value = namesToValues[name];
		enumeration[name] = e;
		enumeration.values.push(e);
	}
	enumeration.foreach = function(f, c) {
		for (var i = 0; i < this.values.length; i ++) f.call(c, this.values[i]);
	};
	
	return enumeration;
}
```

使用上面的enumeration()函数实现一副扑克牌的类。

```javascript
function Card(suit, rank) {
	this.suit = suit;
	this.rank = rank;
}

Card.Suit = enumeration({Clubs: 1, Diamonds: 2, Hearts: 3, Spades: 4});
Card.Rank = enumeration({Two: 2, Three: 3, Four: 4, Five: 5, Six: 6, 
						 Seven: 7, Eight: 8, Nine: 9, Ten: 10, Jack: 11,
						 Queen: 12, King: 13, Ace: 14});
Card.prototype.toString = function() {
	return this.rank.toString() + " of " + this.suit.toString();
};

Card.prototype.compareTo = function(that) {
	if (this.rank < that.rank) return -1;
	if (this.rank > that.rank) return 1;
	return 0;
};

Card.orderByRank = function(a, b) { return a.compareTo(b); };

Card.orderBySuit = function(a, b) {
	if (a.suit < b.suit) return -1;
	if (a.suit > b.suit) return 1;
	if (a.rank < b.rank) return -1;
	if (a.rank > b.rank) return 1;
	return 0;
};

function Deck() {
	var cards = this.cards = [];
	Card.Suit.foreach(function(s) {
							Card.Rank.foreach(function(r) {
									cards.push(new Card(s, r));
							});
	});
}

Deck.prototype.shuffle = function() {
	var deck = this.cards, len = deck.length;
	for (var i = len - 1; i > 0; i --) {
		var r = Math.floor(Math.random() * (i + 1)), temp;
		temp = deck[i], deck[i] = deck[r], deck[r] = temp;
	}
	return this;
};

Deck.prototype.deal = function(n) {
	if (this.cards.length < n) throw "Out of cards";
	return this.cards.splice(this.cards.length - n, n);
};

var deck = (new Deck()).shuffle();
var hand = deck.deal(13).sort(Card.orderBySuit);
```
	
### 9.6.3 标准转换方法
对象类型转换要用到的一些方法是在需要的时候由Javascript解释器自动调用的。不需要为定义的每个类都实现这些方法，但这些方法的确非常重要，如果没有为自定义的类实现这些方法，也应当是有意为之，而不应当因为疏忽而漏掉它们。

最重要的方法首当toString()。这个作用是返回一个可以表示这个对象的字符串。在希望使用字符串的地方用对象的话，Javascript会自动调用这个方法。如果没有实现这个方法，类会默认从Object.prototype中继承toString()方法，这个方法的运算结果是"[object Object]"，这个字符串用处不大。toString()方法应当返回一个可读的字符串，这样最终用户才能将这个输出值利用起来，然而有时候并不一定非要如此，不管怎样，可以返回可读字符串的toString()方法也会让程序调试变得更加轻松。

toLocaleString()和toString()极为类似：toLocaleString()是以本地敏感性的方法来将对象转换为字符串。默认情况下，对象所继承的toLocaleString()方法只是简单地调用toString()方法。有一些内置类型包含有用的toLocaleString()方法用以实际上返回本地化相关的字符串。如果需要为对象到字符串的转换定义toString()，那么同样需要定义toLocaleString()方法用以处理本地化的对象到字符串的转换。

第三个方法是valueOf()，它用来将对象转换为原始值。如，当数学运算符(除了+运算符外)和关系运算符作用于数字文本表示的对象的，会自动调用valueOf()方法。大多数对象都没有合适的原始值来表示它们，也没有定义这个方法。

第四个方法是toJSON()，这个方法是由JSON.stringify()自动调用的。JSON格式用于序列化良好的数据结构，而且可以处理Javascript原始值、数组和纯对象。它和类无关，当对一个对象执行序列化操作时，它会忽略对象的原型和构造函数。如，将Complex对象作用参数传入JSON.stringify()，将会返回诸如{"r":1, "i":-1}之这种字符串。如果将这些字符中传入JSON.parse()，则会得到一个和Complex对象具有相同属性的对象，但这个对象不会包含从Complex继承来的方法。如果一个对象没有toJSON()方法，JSON.stringify()并不会对传入的对象做序列化操作，而会调用toJSON()来执行序列化操作。

Set类并没有定义上述方法中的任何一个。Javascript中没有哪个原始值可以表示集合，因此也没有必要定义valueOf()方法，但该类应当包含toString()、toLocaleString()和toJSON()方法。这里使用extend()来向Set.prototype来添加方法。

```javascript
extend(Set.prototype, {
			toString: function() {
				var s = "{", i = 0;
				this.foreach(function(v) {s += ((i++>0) ? ", " : "") + v; });
				return s + "}";
			},
			toLocaleString: function() {
				var s = "{", i = 0;
				this.foreach(function(v) {
								if (i++ > 0) s += ", ";
								if (v == null) s += v;	// null和undefined
								else s += v.toLocaleString();
							});
				return s + "}";
			},
			toArray: function() {
				var a = [];
				this.foreach(function(v) { a.push(v); });
				return a;
			}
});
Set.prototype.toJSON = Set.prototype.toArray;
```
				
### 9.6.4 比较方法
Javascript的相等运算符比较对象时，比较的是引用而不是值。也就是说，给定两个对象引用，如果要看它们的是否指向同一个对象，不是检查这两个对象是否具有相同的属性名和相同的属性值，而是直接比较两个单独的对象是否相等，或者比较它们的顺序。如果定义一个类，并且希望比较类的实例，应该定义合适的方法来执行比较操作。

为了能让自定义类的实例具备比较的功能，定义一个名收equals()实例方法。这个方法只能接收一个实参，如果这个实参和调用此方法的对象相等的话则返回true。当然，这里所说的相等的含义是根据类的上下文来决定的。对于简单的类，可以通过简单地比较它们的constructor属性来确保两个对象是相同类型，然后比较两个对象的实例属性以保证它们的值相等。为Range类实现这个方法：

```javascript
Range.prototype.constructor = Range;
Range.prototype.equals = function(that) {
	if (that == null) return false;
	if (that.constructor !== Range) return false;
	return this.from == that.from && this.to == that.to;
};
```

给Set类定义equals()方法稍微有些复杂。不能简单地比较两个集合的values属性，还要进行更深层次的比较：

```javascript
Set.prototype.equals = function(that) {
	if (this === that) return true;
	if (!(that instanceof Set) return false;
	if (this.size() != that.size()) return false;
	try {
		this.foreach(function(v) { if (!tha.contains(v)) throw false; });
		return true;
	} catch(x) {
		if ( x === false) return false;
		throw x;
	}
};
```

对于某些类来说，往往需要比较一个实例与另一个实例的大小关系。如果将对象用于Javascript的关系运算符，Javascript会首先调用对象的valueOf()方法，如果这个方法返回一个原始值，则直接比较原始值。多数类没有valueOf()方法，可以定义一个名叫compareTo()方法。如果this对象小于参数对象，应返回小于0的值，如果this对象大于参数对象，应返回大于0的值，如果两个对象相等，应返回0。

给类定义了compareTo()方法，就可以对类的实例组成的数组进行排序了。Array.sort()方法可以接收一具可选的参数，这个参数是一个函数，用来比较两个值的大小，如果这个函数返回值的给定和compareTo()方法保持一致。

### 9.6.5 方法借用
Javascript中的方法没有特别之处：就是一些简单的函数赋值给了对象的属性，可以通过对象来调用它。一个函数可以赋值给两个属性，然后作为两个方法来调用它。如Set类中的toArray()方法和toJSON()方法。

多个类中的方法可以共用一个单独的函数。如，Array类通常定义了一些内置依法，如果定义了一个类，它的实例是类数组的对象，则可以从Array.prototype中将函数复制至所定义的类的原型对象中。

还可以自己定义泛型方法(generic method)。下面的定义了泛型方法toString()和equals()。可以被Range、Complex和Card这些简单的类使用。

```javascript
var generic = {
	toString: function() {
		var s = '[';
		if (this.constructor && this.constructor.name)
			s += this.contructor.name + ': ';
		var n = 0;
		for (var name in this) {
			if (!this.hasOwnProperty(name)) continue;
			var value = this[name];
			if (typeof value === "function") continue;
			if (n ++ ) s += ", ";
			s += name + "=" + value;
		}
		return s + ']';
	},
	equals: function (that) {
		if (that == null) return false;
		if (this.constructor !== that.constructor) return false;
		for (var name in this) {
			if (name === "|**objectid**|") continue;
			if (!this.hasOwnProperty(name)) continue;
			if (this[name] !== that[name]) return false;
		}
		return true;
	}
};
```

### 9.6.6 私有状态
在经典的面向对象编程中，经常需要将对象的某个状态封装或隐藏在对象内，只能通过对象的方法才能些状态，对外只暴露一些重要的状态变量可以直接读写。在Javascript中可以将变量或者参数闭包在一个构造函数内来模拟实现私有实例字段，调用构造函数会创建一个实例。需要在构造函数内部定义一个函数(因此这人函数可以访问构造函数内部的参数和变量)，并将这个函数赋值给新创建对象的属性。下面的例子对Range类的另一各封装：

```javascript
function Range(from, to) {
	this.from = function() { return from; };
	this.to = function() { return to; };
}

Range.prototype = {
	constructor: Range,
	includes: function(x) { return this.from() < x && x <= this.to(); },
	foreach: function(f) {
		for (var x = Math.ceil(this.from()), max = this.to(); x <= max; x ++) f(x);
	},
	toString: function() { return "(" + this.from() + "..." + this.to() + ")"; }
};
```

### 9.6.7 构造函数的重载和工厂方法
有时候，需要对象的初始化有多种方法。如，通过半径和角度来初始化一个Complex对象等。可以通过重载(overload)这个构造函数让它根据传入参数的不同来执行不同的初始化方法。如：

```javascript
function Set() {
	this.values = {};
	this.n = 0;
	if (argumnets.length == 1 && isArrayLike(arguments[0]))
		this.add.apply(this, arguments[0]);
	else if (arguments.length > 0)
		this.add.apply(this, arguments);
}
```

这段代码定义的Set()构造函数可以将一组元素作为参数列表传入，也可以传入元素组成的数组。但是这个构造函数有多义性，如果集合的某个成员是一个数组就无法通过这个构造函数来创建这个集合了。但可以先创建一个空集合，然后显式调用add()方法：

```javascript
Set.fromArray = function(a) {
	s = new Set();
	s.add.apply(s, a);
	return s;
};
```

可以给工厂方法定义任意的名字，不同名字的工厂方法用以执行不同的初始化。但由于构造函数是类的公有标识，因此每个类只能有一个构造函数。但这并不是一个必须遵守的规则。在Javascript中是可以定义多个构造函数继承自一个原型对象的，如果这样做的话，由这些构造函数的任意一个所创建的对象都属于同一类型。并不推荐这种技术，但下面的示例代码使用这种技术定义了该类型的一个辅助构造函数：

```javascript
function SetFromArray(a) {
	Set.apply(this, a);
}

SetFromArray.prototype = Set.prototype;

var s = new SetFromArray([1, 2, 3]);
s instanceof Set;		// true;
```

## 9.7 子类
在面向对象编程中，类B可以继承自另外一个类A。A称为父类(superclass)，B称为子类(subclass)。B的实例从A继承了所有的实例方法。类B可以定义自己的实例方法，有些方法可以重载A中的同名方法，B中的重载方法可能会调用A中的重载方法，这种做法称为方法链(method chaining)。子类的构造函数B()有时需要调用父类的构造函数A()，这种做法称为构造函数链(constructor chaining)。子类还可以有子类，当涉及类的层次结构时，往往需要定义抽象类(abstract class)。抽象类中定义的方法没有实现。抽象类中的抽象方法是在抽象类的具体子类中实现的。

在Javascript中创建子类的关键之处在于，采用合适的方法对原型对象进行初始化。如果类B继承自类A，B.prototype必须是A.prototype的后嗣。B的实例继承自B.prototype，后者同样也继承自A.prototype。

### 9.7.1 定义子类
Javascript的对象可以从类的原型对象中继承属性。如果O是类B的实例，B是A的子类，那么O也一定从A中继承了属性。为此，首先要确保B的原型对象继承自A的原型对象。通过inherit()函数，可以这样来实现：

```javascript
B.prototype = inherit(A.prototype);
B.prop.constructor = B;
```

这两行代码是在Javascript中创建子类的关键。

```javascript
function defineSubClass(superclass,		// 父类构造函数
						constructor,	// 新的子类构造函数
						methods,		// 实例方法：复制到原型中
						statics)		// 类属性：复制到构造函数中
{
	constructor.prototype = inherit(superclass.prototype);
	constructor.prototype.constructor = constructor;
	if (methods) extend(constructor.prototype, methods);
	if (statics) extend(constructor, statics);
	return constructor;
}

Function.prototype.extend = function(constructor, methods, statics) {
	return defineSubClass(this, constructor, methods, statics);
};
```

下面是不使用defineSubClass()函数来手动实现子类：

```javascript
function SingletonSet(member) {
	this.member = member;
}
SingletonSet.prototype = inherit(Set.prototype);
extend(SingletonSet.prototype, {
			constructor: SingletonSet,
			add: function() { throw "read-only set"; },
			remove: function() { throw "read-only set"; },
			size: function() { return 1; },
			foreach: function(f, context) { f.call(context, this.member); },
			contains: function(x) { return x === this.member; }
		});
```

这里的SingletonSet类是一个比较简单的实现。

### 9.7.2 构造函数和方法链
上一节的SingletonSet类定义了全新的集合实现，而且将它继承自其父类的核心方法全部替换。然而定义子类时，往往希望对父类的行为进行修改或扩充，而不是完全替换掉它们。为此，构造函数和子类的方法需要调用或链接到父类构造函数和父类方法。

```javascript
function NonNullSet() {
	Set.apply(this, arguments);
}

NonNullSet.prototype = inherit(Set.prototype);
NonNullSet.prototype.constructor = NonNullSet;

NonNullSet.prototype.add = function() {
	for (var i = 0; i < arguments.length; i ++) {
		if (argumnets[i] == null)
			throw new Error("Can't add null or undefined to a NonNullSet");
	}
	return Set.prototype.add.apple(this, arguments);
};
```

这个非null集合的概念推而广之，称为过滤后的集合，这个集合中的成员必须首先传入一个过滤函数再执行添加操作。为此，定义一个类工厂函数，传入一个过滤函数，返回一个新的Set子类。还可以定义接收两个参数的类工厂：子类和用于add()方法的过滤函数，这个工厂方法称为filteredSetSubclass()，如：

```javascript
var StringSet = filteredSetSubclass(Set, function(x) { return typeof x === "string" ;});
var MySet = filteredSetSubclass(NonNullSet, function(x) { return typeof x !== "function"; });
```

```javascript
function filteredSetSubclass(superclass, filter) {
	var constructor = function() {
		superclass.apply(this, arguments);
	};
	var proto = constructor.prototype = inherit(superclass.prototype);
	proto.constructor = constructor;
	proto.add = function() {
		for (var i = 0; i < arguments.length; i ++) {
			var v = arguments[i];
			if (!filter(v)) throw ("value " + v + " rejected by filter");
		}
		superclass.prototype.add.apply(this, arguments);
	};
	return constructor;
}
```

上面用一个函数将创建子类的代码包装起来，这样就可以在构造函数和方法链中使用父类的参数，而不是通过写死某个父类的名字来使用它的参数。如果想修改父类，只须修改一处代码即可，而不必对每个用到父类类名的地方都做修改。

```javascript
var NonNullSetj = (function() {
	var superclass = Set;
	return superclass.extend(
		function() { superclass.apply(this, arguments); },
		{
			add: function() {
				for (var i = 0; i < arguments.length; i ++)
					if (arguments[i] == null)
						throw new Error("Can't add null or undefined");
				return subclass.prototype.add.apple(this, arguments);
			}
		});
}());
```

### 9.7.3 组合VS子类
可以利用组合的原理定义一个新的集合实现，它包装了另外一个集合对象，在将受限制的成员过滤掉之后会用到这个集合对象。

```javascript
var FilteredSet = Set.extend(
	function FilteredSet(set, filter) { // constructor function
		this.set = set;
		this.filter = filter;
	},
	{
		add: function() {
			if (this.filter) {
				for (var i = 0; i < arguments.length; i ++) {
					var v = arguments[i];
					if (!this.filter(v))
						throw new Error("FilteredSet: value " + v +
										" rejected by filter");
				}
			}
			this.set.add.apply(this.set, arguments);
			return this;
		},
		remove: function() {
			this.set.remove.apply(this.set, arguments);
			return this;
		},
		contains: function(v) { return this.set.contains(v); },
		size: function() { return this.set.size(); },
		foreach: function(f, c) { this.set.foreach(f, c); }
});
```

上面使用组合的一个好处是，只须创建一个单独的FilteredSet子类即可。如：

```javascript
var s = new FilteredSet(new Set(), function(x) { return x !== null; });
var t = new FilteredSet(s, function(x) { return !(x instanceof Set); };
```

### 9.7.4 类的层次结构和抽象类
上节给出组合优于继承的原则，但为了将这条原则阐述清楚，创建了Set的子类。这样做的原因是最终得到的类是Set的实例，它会从Set继承有用的辅助方法，比如toString()和equals()。尽管这是一个很实际的原因，但不用创建类似Set类这种具体类的子类也可以很好的用组合来实现范围。

假定定义了一个AbstractSet类，其中定义了一些辅助方法比如toString()，但并没有实现诸如foreach()的核心依法。这样，实现的Set、SingletonSet和FilteredSet都是这个抽象类的子类，FilteredSet和SingletonSet都不必再实现为某个不相关的类的子类了。

在这个思路上进一步，定义一个层次结构的抽象的集合类。AbstractSet只定义了一个抽象方法：contains()。任何类只要声称是一个表示范围的类，就必须至少定义这个contains()方法。然后，定义AbscractSet的子类AbstractEnumerableSet。这个类增加了抽象的size()和foreach()依法，而且定义了一些有用的非抽象方法，AbstractEnumerableSet并没有定义add()和remove()方法，它只代表只读集合。SingletonSet可以实现为非抽象子类。最后定义了AbstractEnumerableSet的子类AbstractWritableSet。这个final抽象集合定义了抽象方法add()和remove()，并实现了诸如union()和intersection()非具体方法，这两个方法调用add()和remove()。AbstractWriteableSet是Set和FilteredSet类相应的父类。

```javascript
// 这个函数可以用做任何抽象方法，非常方便
function abstractmethod() { throw new Error("abstract method"); }

// class AbstractSet
function AbstractSet() { throw new Error("Can't instantiate abstract classes"); }
AbstractSet.prototype.contains = abstractmethod;

// class NotSet
var NotSet = AbstractSet.extend(
	function NotSet(set) { this.set = set; },
	{
		contains: function(x) { return !this.set.contains(x); },
		toString: function(x) { return "~" + this.set.toString(); },
		equals: function(that) {
			return that instanceof NotSet && this.set.equals(that.set);
		}
	}
);

// class AbstractEnumerableSet 
var AbstractEnumerableSet = AbstractSet.extend(
	function() { throw new Error("Can't instantiate abstract classes"); },
	{
		size: abstractmethod,
		foreach: abstractmethod,
		isEmpty: function() { return this.size() == 0; },
		toString: function() {
			var s = "{", i = 0;
			this.foreach(function(v) {
				if (i ++ > 0) s += ", ";
				s += v;
			});
			return s + "}";
		},
		toLocaleString: function() {
			var s = "{", i = 0;
			this.foreach(function(v) {
				if (i ++ > 0) s += ", ";
				if (v == null) s += v;
				else s += v.toLocaleString();
			});
			return s + "}";
		},
		toArray: function() {
			var a = [];
			this.foreach(function(v) { a.push(v); });
			return s;
		},
		equals: function(that) {
			if (!(that instanceof AbstractEnumerableSet)) reutrn false;
			if (this.size() != that.size()) return false;
			try {
				this.foreach(function(v) { if (!that.contains(v)) throw false; });
				return true;
			} catch (x) {
				if (x === false) return false;
				throw x;
			}
		}
});

// class SingletonSet
var SingletonSet = AbstractEnumerableSet.extend(
	function SingletonSet(member) { this.member = member; },
	{
		contains: function(x) { return x === this.member; },
		size: function() { return 1;},
		foreach: function(f, ctx) { f.call(ctx, this.member); }
	}
);

// class AbstractWritableSet
var AbstractWritableSet = AbstractEnumerableSet.extend(
	function() { throw new Error("Can't instantiate abstract classes"); },
	{
		add: abstractmethod,
		remove: abstractmethod,
		union: function(that) {
			var self = this;
			that.foreach(function(v) { self.add(v); });
			return this;
		},
		intersection: function(that) {
			var self = this;
			this.foreach(function(v) { if (!that.contains(v)) self.remove(v); });
			return this;
		},
		difference: function(that) {
			var self = this;
			that.foreach(function(v) { self.remove(v); });
			return this;
		}
});

// class ArraySet
var ArraySet = AbstractWritableSet.extend(
	function ArraySet() {
		this.values = [];
		this.add.apply(this, arguments);
	},
	{
		contains: function(v) { return this.values.indexOf(v) != -1; },
		size: function() { return this.values.length; },
		foreach: function(f, c) { this.values.forEach(f, c); },
		add: function() {
			for (var i = 0; i < arguments.length; i ++) {
				var arg = arguments[i];
				if (!this.contains(arg)) this.values.push(arg);
			}
			return this;
		},
		remove: function() {
			for (var i = 0; i < arguments.length; i ++) {
				var p = this.values.indexOf(arguments[i]);
				if (p == -1) continue;
				this.values.splice(p, 1);
			}
			return this;
		}
	}
);
```

## 9.8 ECMAScript 5中的类
ECMAScript 5给属性特性增加了方法支持，而且增加了对象可扩展性的限制。

### 9.8.1 让属性不可枚举
ECMAScript 5通过设置属性为不可枚举来让属性不会遍历到。如：

```javascript
// 将代码包装在一个匿名函数中，这样定义的变量就在这个函数作用域内
(function() {
	Object.defineProperty(Object.prototype, "objectId", {
							get: idGetter,
							enumerable: false,
							configurable: false
						});
						
	function idGetter() {
		if (!(idprop in this)) {
			if (!Object.isExtensible(this))
				throw Error("Can't define id for nonextensible objects");
			Object.defineProperty(this, idprop, {
									value: nextid++,
									writable: false,
									enumerable: false,
									configurable: false
								});
		}
		return this[idprop];
	};
	var idprop = "|**objectId**|";
	var nextid = 1;
}());
```

### 9.8.2 定义不可变的类
除了可以设置属性为不可枚举的，还可以设置属性为只读的。下面使用Object.defineProperties()和Object.create()定义不可变的Range类。它同样使用Object.defineProperties()来为类创建原型对象，并实例方法设置为不可枚举的，就像内置灰的方法一样。不仅如此，它还将这些实例方法设置为只读和不可删除的，这样就可以防止对类做任何修改。最后，展示了一个技巧，其中实现的构造函数也可以用做工厂函数，这样不论调用函数之前是否带有new关键字，都可以正确地创建实例。

```javascript
function Range(from, to) {
	var props = {
		from: {value: from, enumerable: true, writable: false, configurable: false},
		to: {value: to, enumerable: true, writable: false, configurable: false}
	};
	if (this instantceof Range)
		Object.defineProperties(this, props);
	else
		return Object.create(Range.prototype, props);
}

Object.defineProperties(Range.prototype, {
	includes: {
		value: function(x) { reutrn this.from <= x && x <= this.to; }
	},
	foreach: {
		value: function(f) { for (var x = Math.ceil(this.from); x <= this.to; x++) f(x);
	},
	toStirng: {
		value: function() { return "(" + this.from + "..." + this.to + ")"; }
	}
});
```

属性描述符对象让错码的可读性变得差，可以将修改已定义属性的特性的操作定义为一个工具函数：

```javascript
function freezeProps(o) {
	var props = (arguments.length == 1)
		? Object.getOwnPropertyNames(o)
		: Array.prototype.splice.call(arguments, 1);
	props.forEach(function(n) {
		if (!Object.getOwnPropertyDescriptor(o, n).configurable) return;
		Object.defineProperty(o, n, { writable: false, configurable: false });
	});
	return o;
}

function hidePros(o) {
	var props = (arguments.length == 1)
		? Object.getOwnPropertyNames(o)
		: Array.prototype.splice.call(arguments, 1);
	props.forEach(function(n) {
		if (!Object.getOwnPropertyDescriptor(o, n).configurable) return;
		Object.defineProperty(o, n, { enumerable: false });
	});
	return o;
}
```

Object.defineProperty()和Object.defineProperties()可以用来创建新属性，也可以修改已有属性的特性。当用它们创建新的属性时，默认的属性特性的值都是false。但当用它们修改已经存在的属性时，默认的属性特性依然保持不变。

使用这些工具函数，就可以充分利用ECMAScript 5的特性来实现一个不可变的类，而且不用动态地修改这个类。如：

```javascript
function Range(from, to) {
	this.from = from;
	this.to = to;
	freezeProps(this);
}

Range.prototype = hideProps({
	constructor: Range,
	includes: function(x) { return this.from <= x && x <= this.to; },
	foreach: function(f) { for(var x = Math.ceil(this.from); x <= this.to; x++) f(x); },
	toString:: function() { return "(" + this.from + "..." + this.to + ")"; }
});
```

### 9.8.3 封装对象状态
通过定义属性getter和setter方法将状态变量更健壮地封装起来，这两个方法是无法删除的：

```javascript
function Range(from, to) {
	if (from > to) throw new Error("Range: from must be <= to");
	function getFrom() { return from; }
	function getTo() { return to; }
	function setFrom(f) {
		if (f <= to) from = f;
		else throw new Error("Range: from must be <= to");
	}
	function setTo(t) {
		if (t >= from) to = t;
		else throw new Error("Range to must be >= from");
	}
	Object.defineProperties(this, {
		from: {get: fetFrom, set: setFrom, enumerable: true, configurable: false },
		to: { get: getTo, set: setTo, enumerable: true, configurable: false }
	});
}

Range.prototype = hideProps({
	constructor: Range,
	includes: function(x) { return this.from <= x && x <= this.to; },
	foreach: function(f) { for(var x = Math.ceil(this.from); x <= this.to; x++) f(x); },
	toString:: function() { return "(" + this.from + "..." + this.to + ")"; }
});
```

### 9.8.4 防止类的扩展
通常认为，通过给原型对象添加方法可以动态地对类进行扩展，这是Javascript本身的特性。ECMAScript 5可以根据需要对此特性加以限制。Object.preventExtensions()可以将对象设置为不可扩展的，也就说不能给对象添加任何新属性。Object.seal()则更加强大，它除了能阻止用户给对象添加新属性，还能将当前已有的属性设置为不可配置的，这样就不能删除这些属性了(但不可配置的属性可以是可写的，也可以转换为只读属性)。Javascript的另外一个动态特性是对象的可以随时替换：

```javascript
var original_sort_method = Array.prototype.sort;
Array.prototype.sort = function() {
	var start = new Date();
	original_sort_method.apply(this, arguments);
	var end = new Date();
	console.log("Array sort took " + (end - start) + " milliseconds.");
};
```

可以通过将实例方法设置为只读来防止这类修改，一种方法就是使用上面代码所定义的freezeProps()工具函数，另外一种方法是使用Object.freeze()，它的功能和Object.seal()完全一样，它同样会把所有属性都设置为只读的和不可配置的。

理解类的只读属性的特性至关重要的。如果对象o继承了只读属性p，那么给o.p的赋值操作将会失败，就不会给o创建新属性。如果你想重写一个继承来的只读属性，就必须使用Object.defineProperty()、Object.defineproperties()或Object.create()来创建这个新属性。对只读属性的重写更加困难。

### 9.8.5 子类和ECMAScript 5
定义AbscractWritableSet类的子类来说明。使用Object.create()创建原型对象，这个原型对象继承自父类的原型，同时给新创建的对象定义属性。这个例子使用Object.create()创建对象时传入了参数null，这个创建的对象没有任何继承任何成员。这个对象用来存储集合的成员，同时，这个对象没有原型，可以直接使用in运算符，而不须使用hasOwnProperty()方法。

```javascript
function StringSet() {
	this.set = Object.create(null);
	this.n = 0;
	this.add.apply(this, arguments);
}

StringSet.prototype = Object.create(AbstractEnumerableSet.prototype, {
	constructor: { value: StringSet },
	contains: { value: function(x) { return x in this.set; } },
	size: { value: function(x) { return this.n; } },
	foreach: { value: function(f, c) { Object.keys(this.set).forEach(f, c); } },
	add: {
		value: function() {
			for (var i = 0; i < arguments.length; i ++) {
				if (!(arguments[i] in this.set)) {
					this.set[arguments[i]] = true;
					this.n++;
				}
			}
			return this;
		}
	},
	remove: {
		value: function() {
			for (var i = 0; i < arguments.length; i ++) {
				if (arguments[i] in this.set) {
					delete this.set[arguments[i]];
					this.n--;
				}
			}
			return this;
		}
	}
});
```

### 9.8.6 属性描述符
给Object.prototype添加properties()方法(不可枚举方法)，其返回一个对象，用以表示属性的列表，并定义了有用的方法用来输出属性和属性特性，用来锋利属性描述符以及用来设置属性的特性。

```javascript
(function namespace() {
	function properties() {
		var names;
		if (arguments.length == 0) 
			names = Object.getOwnPropertyNames(this);
		else if (arguments.length == 1 && Array.isArray(arguments[0]))
			names = arguments[0];
		else
			names = Array.prototype.splice.call(arguments, 0);
		return new Properties(this, names);
	}
	Object.defineProperty(Object.prototype, "properties", {
		value: properties, enumerable: false, writable: true, configurable: true
	});
	function Properties(o, names) {
		this.o = o;
		this.names = names;
	}
	Properties.prototype.hide = function() {
		var o = this.o, hidden = { enumerable: false };
		this.names.forEach(function(n) {
							if (o.hasOwnProperty(n))
								Object.defineProperty(o, n, hidden);
						});
		return this;
	};
	Properties.prototype.freeze = function() {
		var o = this.o, frozen = { writable: false, configurable: false };
		this.names.forEach(function(n) {
							if (o.hasOwnProperty(n)) 
								Object.defineProperty(o, n, frozen);
						});
		return this;
	};
	Properties.prototype.descriptors = function() {
		var o = this.o, desc = {};
		this.names.forEach(function(n) {
							if (o.hasOwnProperty(n))
								desc[n] = Object.getOwnPropertyDescriptor(o, n);
						});
		return this;
	};
	Properties.prototype.toString = function() {
		var o = this.o;
		var lines = this.names.map(nameToString);
		return "{\n " + lines.join(",\n ") + "\n}";
		
		function nameToString(n) {
			var s = "", desc = Object.getOwnPropertyDescriptor(o, n);
			if (!desc) return "nonexistent " + n + ": undefined";
			if (!desc.configurable) s += "permanent ";
			if ((!desc.get && !desc.set)) || !desc.writable) s += "readonly ";
			if (!desc.enumerable) s += "hidden ";
			if (desc.get || desc.set) s += "accessor " + n
			else s += n + ": " + ((typeof desc.value === "function") ? "function" : desc.value);
			return s;
		}
	};
	Properties.prototype.properties().hide();
}());
```

## 9.9 模块
将代码组织到类中的一个重要原因是，让代码更加模块化，可以在很多不同场景中实现代码的重用。但类不是唯一的模块化代码的方式。一般来讲，模块是一个独立的Javascript文件。模块文件可以包含一个类定义、一组相关的类、一个实用函数库或者是一些待执行的代码。只要以模块的形式编写代码，任何Javascript代码段就可以当做一个模块。

模块化的目标是支持大规模的程序开发，处理分散源中代码的组装，并且能让代码正确运行。为了做到这一点，不同的模块必须避免修改全局执行上下文，因此后续模块应当在它们所期望运行的原始上下文中执行。这意味着模块应当尽可能少地定义全局标识。理想的状况是，所有模块都不应当定义超过一个全书标识。

### 9.9.1 用做命名空间的对象
在模块创建过程中避免污染全局变量的一种方法是使用一个对象作为命名空间。它将函数和值作为命名空间对象属性存储起来，而不是定义全局函数和变量。如Set类，它定义一个全局构造函数Set()。然后给这个类定义了很多实例方法，但将这些实例方法存储为Set.prototype的属性，因此这些方法不是全局的。

基于这种保持干净的全局命名空间的观点，一种更好的做法是将集合类定义为一个单独的全局对象：

```javascript
var sets = {};
```

这个sets对象是模块的命名空间，并且将每个集合类都定义为这个对象的属性：

```javascript
sets.SingletonSet = sets.AbstractEnumerableSet.extend(...);
```

如果要使用这样定义的类，需要通过命名空间来调用所需的构造函数：

```javascript
var s = new sets.SingletonSet(1);
```

模块的作者并不知道他的模块会和哪些其他模块一起工作，因此尤为注意这种命名空间的用法带的命名冲突，然而，使用这个模块的开发者是知道它用了哪些模块，用到了哪些名字的。可以这样使用将名字导入：

```javascript
var Set = sets.Set;
var s = new Set(1, 2, 3);
```

### 9.9.2 作为私有命名空间的函数
模块对外导出一些公用API，这些API提供给其他程序员使用的，它包括函数、类、属性和方法。但模块的实现往往需要一些额外的辅助函数和方法，这些函数和方法并不需要在模块外部可见。可以通过将模块定义在某个函数的内部来实现。如：

```javascript
var Set = (function invocation() {
	function Set() {
		this.values = {};
		this.n = 0;
		this.add.apply(this, arguments);
	}
	Set.prototype.contains = function(value) {
		return this.values.hasOwnProperty(v2s(value));
	};
	Set.prototype.size = function() { return this.n; };
	Set.prototype.add = function() { /*...*/ };
	Set.prototype.remove = function() { /*...*/ };
	Set.prototype.foreach = function(f, context) { /*...*/ };
	function v2s(val) { /*...*/ };
	function objectId(o) { /*...*/ };
	var nextId = 1;
	return Set;
}());
```

注意，这里使用了立即执行的匿名函数，这在Javascript中是一种惯用法。如果想让代码在一个私有命名空间中运行，只须给这段代码加上前缀`(function(){`和后缀`}())`。一旦将模块代码封装进一个函数，就需要一些方法导出共公用API，以便在模块函数的外部调用它们。如上面的代码返回构造函数，这个构造函数随后赋值给一个全局变量。将值返回已经清楚地表明API已经导出在函数作用域之外。如果模块API包含多个单元，则它可以返回命名空间对象。如：

```javascript
var collections;
if(!collections) collections = {};
collections.sets = (function namespace() {
	/* ... */
	return {
		AbstractSet: AbstractSet,
		NotSet: NotSet,
		AbstractEnumerableSet: AbstractEnumerableSet,
		SingletonSet: SingletonSet,
		AbstractWritableSet: AbstractWritableSet,
		ArraySet: ArraySet
	};
}());
```

另外一种类似的技术是将模块函数当做构造函数，通过new来调用，通过将它们赋值给this来将其导出：

```javascript
var collections;
if (!collections) collections = {};
collections.sets = (new function namespace() {
	/* ... */
	this.AbstractSet = AbstractSet;
	this.NotSet = NotSet;
}());
```

作为一种替代方案，如果已经定义了全局命名空间对象，这个模块函数可以直接设置那个对象的属性，不用返回任何内容：

```javascript
var collections;
if (!collections) collections = {};
collections.sets = {};
(function namespace() {
	/* ... */
	collections.sets.AbstractSet = AbstractSet;
	collections.sets.NotSet = NotSet;
	// no return statement
}());
```

有些框架实现了模块加载功能，其中包括其他一些导出模块API的方法。

Author website: [furzoom](http://furzoom.com/about-us/ "Furzoom")
