# PlayWarts

[![Build Status](https://travis-ci.org/danielnixon/playwarts.svg?branch=master)](https://travis-ci.org/danielnixon/playwarts)
[![Dependency Status](https://www.versioneye.com/user/projects/5418232b54ffbda60b000061/badge.svg?style=flat)](https://www.versioneye.com/user/projects/5418232b54ffbda60b000061)
[![Codacy Badge](https://api.codacy.com/project/badge/f641de970f1f4a98a1900ee38250bb7d)](https://www.codacy.com/app/danielnixon/playwarts)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/org.danielnixon/playwarts_2.11/badge.svg)](https://maven-badges.herokuapp.com/maven-central/org.danielnixon/playwarts_2.11)

[WartRemover](https://github.com/typelevel/wartremover) warts for [Play Framework](https://www.playframework.com/) and [Slick](http://slick.typesafe.com/) (and some bonus warts).

## Versions

| PlayWarts version | WartRemover version | Play version       | Play Slick version  | Scala version |
|-------------------|---------------------|--------------------|---------------------|---------------|
| 0.28              | 1.1.x               | 2.5.x              | 2.0.x               | 2.11.x        |
| 0.15 ([README](https://github.com/danielnixon/playwarts/blob/77b01471e016d2d494224dd838715eeff6e19ebf/README.md))     | 0.14                | 2.4.x              | 1.1.x               | 2.11.x        |

## Usage

1. Setup [WartRemover](https://github.com/typelevel/wartremover).
2. Add the following to your `plugins.sbt`:

    ```scala
    addSbtPlugin("org.danielnixon" % "sbt-playwarts" % "0.28")
    ```

3. Add the following to your `build.sbt`:
    ```scala
    // Play Framework
    wartremoverWarnings ++= Seq(
      PlayWart.AssetsObject,
      PlayWart.CookiesPartial,
      PlayWart.FlashPartial,
      PlayWart.FormPartial,
      PlayWart.HeadersPartial,
      PlayWart.JavaApi,
      PlayWart.JsLookupResultPartial,
      PlayWart.JsReadablePartial,
      PlayWart.LangObject,
      PlayWart.MessagesObject,
      PlayWart.PlayGlobalExecutionContext,
      PlayWart.SessionPartial,
      PlayWart.WSResponsePartial)

    // Slick
    wartremoverWarnings ++= Seq(
      PlayWart.BasicStreamingActionPartial)

    // Bonus Warts
    wartremoverWarnings ++= Seq(
      PlayWart.DateFormatPartial,
      PlayWart.EnumerationPartial,
      PlayWart.FutureObject,
      PlayWart.GenMapLikePartial,
      PlayWart.GenTraversableLikeOps,
      PlayWart.GenTraversableOnceOps,
      PlayWart.LegacyDateTimeCode,
      PlayWart.ScalaGlobalExecutionContext,
      PlayWart.StringOpsPartial,
      PlayWart.TraversableOnceOps,
      PlayWart.UntypedEquality)
    ```

## Warts

### Play Framework

#### AssetsObject

The `controllers.Assets` object depends on global state. Declare a dependency on an instance of the `controllers.Assets` class instead.

#### CookiesPartial

`play.api.mvc.Cookies` has an `apply` method that can throw. Use `Cookies#get` instead.

#### FlashPartial

`play.api.mvc.Flash` has an `apply` method that can throw. Use `Flash#get` instead.

#### FormPartial

`play.api.data.Form` has a `get` method which will throw if the form contains
errors. The program should be refactored to use `play.api.data.Form#fold` to
explicitly handle forms with errors and successful form submissions.

#### HeadersPartial

`play.api.mvc.Headers` has an `apply` method that can throw. Use `Headers#get` instead.

#### JavaApi

The Java API in the `play` package is disabled. Use the Scala API under `play.api` instead.

#### JsLookupResultPartial

`play.api.libs.json.JsLookupResult` has a `get` method which can throw. Use `JsLookupResult#getOrElse` instead.

#### JsReadablePartial

`play.api.libs.json.JsReadable` has an `as` method which can throw. Use `JsReadable#asOpt` instead.

#### LangObject

The `play.api.i18n.Lang` object is disabled. Use `play.api.i18n.Langs` instead.

#### MessagesObject

`play.api.i18n.Messages.Implicits` exists to allow you to continue to use static controller objects in Play 2.4.x. Use controller classes with dependency injection instead. See [Migration24#I18n](https://www.playframework.com/documentation/2.4.x/Migration24#I18n).

#### PlayGlobalExecutionContext

Play's global execution context `play.api.libs.concurrent.Execution#defaultContext` is disabled. Declare a dependency on an `ExecutionContext` instead. See [MUST NOT hardcode the thread-pool / execution context](https://github.com/alexandru/scala-best-practices/blob/master/sections/4-concurrency-parallelism.md#411-must-not-hardcode-the-thread-pool--execution-context).

#### SessionPartial

`play.api.mvc.Session` has an `apply` method that can throw. Use `Session#get` instead.

#### WSResponsePartial

The `play.api.libs.ws.WSResponse` trait defines `json` and `xml` methods that will throw if the response body can't be parsed as JSON or XML respectively (the default `AhcWSResponse` implementation of this trait throws `JsonParseException` and `SAXException`). You can wrap these unsafe methods in an implicit class that might look something like this:

```scala
implicit class WSResponseWrapper(val response: WSResponse) extends AnyVal {
  @SuppressWarnings(Array("org.danielnixon.playwarts.WSResponsePartial"))
  def jsonOpt: Option[JsValue] = catching[JsValue](classOf[JsonParseException]) opt response.json

  @SuppressWarnings(Array("org.danielnixon.playwarts.WSResponsePartial"))
  def xmlOpt: Option[Elem] = catching[Elem](classOf[SAXException]) opt response.xml
}
```

### Slick

#### BasicStreamingActionPartial

`slick.profile.BasicStreamingAction` has a `head` method which will fail if the stream is empty (i.e. if the `SELECT` SQL query returned zero rows). Use `headOption` instead.

### Bonus Warts

#### DateFormatPartial

`java.text.DateFormat#parse` is disabled because it can throw a `ParseException`. You can wrap it in an implicit that might look like this:

```scala
implicit class DateFormatWrapper(val dateFormat: DateFormat) extends AnyVal {
  @SuppressWarnings(Array("org.danielnixon.playwarts.DateFormatPartial"))
  def parseOpt(source: String): Option[Date] = nonFatalCatch[Date] opt dateFormat.parse(source)
}
```

#### EnumerationPartial

`scala.Enumeration#withName` is disabled because is will throw a `NoSuchElementException` if there is no value matching the specified name. You can wrap it in an implicit that might look like this:

```scala
implicit class EnumerationWrapper[A <: Enumeration](val enum: A) extends AnyVal {
  @SuppressWarnings(Array("org.danielnixon.playwarts.EnumerationPartial"))
  def withNameOpt(s: String): Option[A#Value] = {
    catching[A#Value](classOf[NoSuchElementException]) opt enum.withName(s)
  }
}
```

#### FutureObject

`scala.concurrent.Future` has a `reduce` method that can throw `NoSuchElementException` if the collection is empty. Use `Future#fold` instead.

#### GenMapLikePartial

`scala.collection.GenMapLike` has an `apply` method that can throw ` NoSuchElementException` if there is no mapping for the given key. Use `GenMapLike#get` instead.

#### GenTraversableLikeOps

WartRemover's [ListOps](https://github.com/puffnfresh/wartremover#listops) wart only applies to `scala.collection.immutable.List`. The `GenTraversableLikeOps` wart extends it to everything that implements `scala.collection.GenTraversableLike`.

`scala.collection.GenTraversableLike` has:

* `head`,
* `tail`,
* `init` and
* `last` methods,

all of which will throw if the list is empty. The program should be refactored to use:

* `GenTraversableLike#headOption`,
* `GenTraversableLike#drop(1)`,
* `GenTraversableLike#dropRight(1)` and
* `GenTraversableLike#lastOption` respectively,

to explicitly handle both populated and empty `GenTraversableLike`s.

#### LegacyDateTimeCode

The `Date`, `TimeZone` and `Calendar` classes in the `java.util` package are disabled. Use `java.time.*` instead. See [Legacy Date-Time Code](https://docs.oracle.com/javase/tutorial/datetime/iso/legacy.html).

#### ScalaGlobalExecutionContext

Scala's global execution context `scala.concurrent.ExecutionContext#global` is disabled. Declare a dependency on an `ExecutionContext` instead. See [MUST NOT hardcode the thread-pool / execution context](https://github.com/alexandru/scala-best-practices/blob/master/sections/4-concurrency-parallelism.md#411-must-not-hardcode-the-thread-pool--execution-context).

#### StringOpsPartial

`scala.collection.immutable.StringOps` has
* `toBoolean`,
* `toByte`,
* `toShort`,
* `toInt`,
* `toLong`,
* `toFloat` and
* `toDouble` methods,

all of which will throw `NumberFormatException` (or `IllegalArgumentException` in the case of `toBoolean`) if the string cannot be parsed.

You can hide these unsafe `StringOps` methods with an implicit class that might look something like this:

```scala
implicit class StringWrapper(val value: String) extends AnyVal {
  import scala.util.control.Exception.catching

  @SuppressWarnings(Array("org.danielnixon.playwarts.StringOpsPartial"))
  def toIntOpt: Option[Int] = catching[Int](classOf[NumberFormatException]) opt value.toInt
}
```

#### TraversableOnceOps

`scala.collection.TraversableOnce` has a `reduceLeft` method that will throw if the collection is empty. Use `TraversableOnce#reduceLeftOption` or `TraversableOnce#foldLeft` instead.

`scala.collection.TraversableOnce` has

* `max`,
* `min`,
* `maxBy` and
* `minBy` methods, 

all of which will throw `UnsupportedOperationException` if the collection is empty. You can wrap these unsafe methods in an implicit class that might look something like this:

```scala
implicit class TraversableOnceWrapper[A](val traversable: TraversableOnce[A]) extends AnyVal {
  @SuppressWarnings(Array("org.danielnixon.playwarts.TraversableOnceOps"))
  def maxOpt[B >: A](implicit cmp: Ordering[B]): Option[A] = {
    if (traversable.isEmpty) None else Some(traversable.max(cmp))
  }
}
```


#### UntypedEquality

`scala.Any` and `scala.AnyRef` contain a number of untyped equality methods:

* `equals`
* `eq`
* `ne`

All of which are disabled. Use a typesafe alternative (such as [Scalaz's Equal typeclass](http://eed3si9n.com/learning-scalaz/Equal.html) or [Heiko Seeberger's solution](http://hseeberger.github.io/blog/2013/05/30/implicits-unchained-type-safe-equality-part1/)) instead.
