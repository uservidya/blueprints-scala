Tinkerpop Blueprints Scala [![Build Status](https://travis-ci.org/anvie/blueprints-scala.png?branch=master)](https://travis-ci.org/anvie/blueprints-scala)
===================================

Scala wrapper for tinkerpop blueprints, this library provide more scalastic code when working with graph database
supported by blueprints.

Usage
---------

For installation see install section.
More working and complete examples can be found on specs test.
This example data based on graph of the gods https://github.com/thinkaurelius/titan/wiki/Getting-Started

![](https://github.com/thinkaurelius/titan/raw/master/doc/images/graph-of-the-gods.png)


Creating new vertex:

```scala
case class Person(name:String, kind:String)

val hercules = Person("hercules", "demigod")

db.save(hercules)
```

If your class subclassing from `DbObject` you will have more sweet code like using `save()` directly:

```scala

case class Person(name:String, kind:String) extends DbObject

val hercules = Person("hercules", "demigod").save()
```

The `save()` method return a Vertex object, if you want to get class back just:

```scala
val herculesObject = hercules.toCC[Person].get
```

Creating edges:

```scala
hercules --> "father" --> jupiter
hercules --> "mother" --> alcmene
```
Or you can also add multiple edges in one line:

```scala
hercules --> "father" --> jupiter --> "father" --> saturn
```

Creating mutual (both) edges:

```scala
jupiter <--> "brother" <--> neptune
```

Or more complex mutual edges:

```scala
jupiter <--> "brother" <--> neptune <--> "brother" <--> pluto
```	

Shorthand property getter and setter:

```scala
jupiter.set("kind", "god")

val kind = jupiter.get[String]("kind").get
```
	
Getter `get` return Option instance, so you can handle unexpected return data efficiently using map:

```scala
jupiter.get[String]("kind") map { kind =>
	// do with kind here
}
```

Or getting value by using default value if empty:

```scala
jupiter.getOrElse[String]("status", "live")
```

Easy getting mutual connection, for example jupiter and pluto is brother both has IN and OUT edges,
is very easy to get mutual connection using `mutual` method:

```scala
val jupitersBrothers = jupiter.mutual("brother")
```
	
Inspect helpers to print vertex list:

```scala
jupitersBrothers.printDump("jupiter brothers:", "name")

// will produce output:

jupiter brothers:
 + neptune
 + pluto
```

Using Gremlin Pipeline like a boss:

```scala
val battled = hercules.pipe.out("battled")
battled.printDump("hercules battled:", "name")

// output:

hercules battled:
 + nemean
 + hydra
 + cerberus
```

Syntactic sugar filtering on Gremlin Pipeline:

```scala
// get monster battled with hercules more than 5 times
val monsters = hercules.pipe.outE("battled").wrap.filter { edge =>
   edge.getOrElse[Int]("time", 0).toString.toInt > 5
}
```

Returning edges from chain by adding `<` on the last line:

```scala
var edge = hercules --> "battled" --> hydra <

edge.set("time", 2)
```

Using transaction:

```scala
transact {
	hercules --> "father" --> jupiter
	hercules --> "mother" --> alcmene
	val edge = hercules --> "battled" --> hydra <

	edge.set("time", 15)
}
```

Want more attributes that not fit for case class constructor parameter?

Use `@Persistent` annotation:

```scala

case class City(name:String){

    @Persistent var province = "Jakarta" // this attribute will be saved
    @Persistent var country = "Indonesia" // this attribute will be saved

    val code = 123 // this not

}

```

Support inheritance as well:

```scala

abstract class City(name:String) {
    @Persistent var province = ""
}

trait Street {
    @Persistent var street = ""
}

trait Code {
    @Persistent var postalCode = 0
}


case class Jakarta extends City("Jakarta") with Street with Code with DbObject

// usage

val address = Jakarta()
address.province = "Jakarta Barat"  // this will be saved
address.street = "KS. Tubun"        // this will be saved
address.postalCode = 12345          // this will be saved
address.save()

// getting back

db.getVertex(address.getVertex.getId).toCC[Jakarta] map { adr =>
    println(adr.province) // will printed `Jakarta Barat`
    println(adr.postalCode) // will printed `12345`
}

// instantiate to parent class
// this is valid

db.getVertex(address.getVertex.getId).toCC[City] map { adr =>
    println(adr.province) // will printed `Jakarta Barat`
}

// this also valid

db.getVertex(address.getVertex.getId).toCC[Street] map { adr =>
    println(adr.street) // will printed `KS. Tubun`
}

```

How it work? All you needs is to adding dependency (see install), import it, and summon Graph db instance with implicit:

```scala
import com.ansvia.graph.BlueprintsWrapper._

// you can use any graph database that support tinkerpop blueprints
// here is i'm using simple in-memory Tinkergraph db for example.
implicit val db = TinkerGraphFactory.createTinkerGraph()

//... work here ....
```

Done, now you can using Scalastic sweet syntactic sugar code.


Install
--------

Add dependency:

	"com.ansvia.graph" % "blueprints-scala" % "0.0.10"


License
---------

[Apache 2](http://www.apache.org/licenses/LICENSE-2.0.html)


***[] Robin Syihab***
