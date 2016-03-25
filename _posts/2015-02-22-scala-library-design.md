---
title: "Scala library design"
date: "2015-02-22"
categories: [engineering]
tags: [scala, api, playframework, elasticsearch]
excerpt: >
  We've been using Play Framework with Elasticsearch using elastic4s for a
  while. Getting it all to work together requires some boilerplate code, though.
  We decided to create a small library that would make things easier in future
  projects. We learned some lessons about building Scala libraries along the way.
meta:
  description: "Good practices for designing library APIs"
  keywords: "scala, api, programming, library, design, playframework, elasticsearch"
---
We've been using [Play Framework] with [Elasticsearch] using [elastic4s] for a
while. Getting it all to work together requires some boilerplate code, though.
We decided to create a small library that would make things easier in future
projects. We learned some lessons about building Scala libraries along the way.

The library itself is [hosted on GitHub]. It's a work in progress, but any
comments are welcome. There is also an [activator project] with a small example;
it should be added to the activator templates directory in a matter of days.

The Problems
------------

There are two main issues we wanted our library to solve.

The first one is the necessity to define two types of JSON formatters for the
domain objects. For any type `T`, Play requires `Reads[T]` and/or `Writes[T]`
from `play.api.libs.json` ([play JSON docs]). In `elastic4s`, on the other hand,
one should provide `Indexable[T]` (see [indexing from classes]) for inserting
documents and `HitAs[T]` (see [search conversion]) for parsing search results.
The only real difference between the two approaches is the fact that Play parser
is fed with JSON only, while `HitAs` can consume additional metadata returned by
Elasticsearch server (such as magic `_id`, `_index` or `_score` fields). The
difference is small enough to consider writing both formatters a violation of
DRY principle. We could definitely do something to avoid it.

The second feature of our dreamed-of library was integration with Play lifecycle
and configuration management. We got accustomed to setting all the configuration
vars in `application.conf` and having all the necessary wiring done behind the
scenes.

Initial Approach
----------------

The library started as a simple one-day hackathon without too much designing. We
simply exploited the golden rule of Object Oriented Programming:

> Every issue can be solved by introducing an extra layer of indirection.

So at the core of our module was a new `ElasticClientWrapper` class. It had
plenty of methods and a few clever tricks, but for the purpose of this post we
might as well assume it looked similar to this:

```scala
import org.sksamuel.elastic4s.ElasticClient
import play.api.libs.json._

class ElasticClientWrapper(val underlying: ElasticClient) {

    def index[T: Writes](id: String, index: String, doc: T): Future[Boolean] = { ... }
    def getById[T: Reads](id: String, index: String): Future[Option[T]] = { ... }
    def search[T: Reads](query: QueryDefinition, index: String): Future[List[T]] = { ... }
}
```

The main task of this interface layer was to accept communication with
Elasticsearch using our already written Play JSON formatters. The methods
defined on `ElasticClientWrapper` used Play JSON formatters to create valid
`elastic4s` queries.

Having dealt with JSON formatting, we moved on to configuration management. We
decided the parameters should be loaded from `application.conf` and injected
transparently. So we defined an extra node in the configuration file and
provided a module which would read the cluster definitions and provide named
bindings for the connections. A relevant node in `application.conf` was:

```
# application.conf

elastic4s {
  clusters {
    fooCluster {
      uri: "elasticsearch://foo-es-host.myorg.com:9300"
    }
    barCluster {
      uri: "elasticsearch://bar-es-host.myorg.com:9300"
    }
  }
}
```

The proper clients were then injected using [named bindings]:

```scala

class FooDao @Inject() (@Named("fooCluster") client: ElasticClientWrapper) {
  def get(id: String): Future[Option[Foo]] = client.getById(id, "foos")
  ...
}
```

And it worked like a charm. At least until we wanted to plug it into one of our
real projects.

Why it didn't Work
------------------

We happily plugged the library in one of the projects in order to remove the
boilerplate mentioned at the beginning of the post. It very quickly turned out
that the small API provided by our wrapper is not enough and we had to add more
methods. As we did that, we realized that it's a path to nowhere. Every user of
the library would eventually form the same conclusion:

> In order to profit from implicit JSON conversions, I have to stick to the
> methods exposed on the wrapper.

But the wrapper interface was extremely small compared to the capabilities of
underlying elastic4s library. We had to do better.

Goals Revisited
---------------

So we reviewed our goals again, and focused on achieving them while keeping our
library as small as possible. This time, it lead to the following decisions:

### Keep the Concepts Separated

The library was supposed to handle two issues (JSON serialization and
configuration management). The initial approach tried to solve them both at the
same time. In the revised design, we kept those issues separate, to the point
that users might use any one of them without the other.

### Don't Make Users Think

Instead of providing a carefully crafted set of classes and methods, we kept our
API as small as possible. Instead of providing our own methods for interacting
with ES cluster, we rely on original elastic4s API and only provide a mixin that
automatically derives elastic4s typeclasses based on Play JSON formatters. The
mixin API, in fact, can be described in just a few lines:

```scala
import play.api.libs.json.{Reads, Writes}
import com.sksamuel.elastic4s.HitAs
import com.sksamuel.elastic4s.source.Indexable

trait PlayElasticJsonSupport {
    implicit def playReadsToHitAs[T](reads: Reads[T]): HitAs[T] = { ... }
    implicit def playWritesToIndexable[T](writes: Writes[T]): Indexable[T] = { ... } 
} 
    
```

And that's it. The only think the user has to do is extend the trait. Under the
hood, quite a few things have to happen, because the typeclasses have slightly
different interfaces. From the user's perspective, though, everything is simple
- they use the original elastic4s API without having to provide the typeclass
instances by hand.

Takeaway
--------

From a user's perspective, including a library in their project is always a
trade-off. The more libraries in a project, the more difficult it is to keep
them all up-to-date and resolve conflicts. The libraries get old and support
gets dropped. Having that all in mind, forcing the users to learn another new
API is an additional cost that might tip the scale.

  [Play Framework]: http://playframework.com
  [Elasticsearch]: http://elastic.co
  [elastic4s]: http://google.com
  [hosted on GitHub]: https://github.com/evojam/play-elastic4s
  [activator project]: https://github.com/evojam/play-elastic-template
  [play JSON docs]: https://www.playframework.com/documentation/2.4.x/ScalaJson
  [indexing from classes]: https://github.com/sksamuel/elastic4s#indexing-from-classes
  [search conversion]: https://github.com/sksamuel/elastic4s#search-conversion
  [named bindings]: https://www.playframework.com/documentation/2.4.x/ScalaDependencyInjection#Programmatic-bindings
