---
title: How to handle query parameters?
type: section
description: Handling query parameters with Cask
num: 33
previous-page: web-server-dynamic
next-page: web-server-input
---

{% include markdown.html path="_markdown/install-cask.md" %}

Query parameters are the key-value pairs coming after the question mark in a URL. They can be used for filtering, 
sorting or limiting the results provided by the server. For example, in `<host>/time?city=Paris` URL, the `city` part
is the name of a parameter and `Paris` is its value. Cask allows for reading the query parameters by defining an endpoint
method with arguments matching the names of the expected parameters and not matching any of the URL segments.

In this example the `city` parameter will be optional, which you specify in Cask by giving the argument `Option` type and 
`None` default value. If not provided, the time for the current timezone will be returned.

{% tabs web-server-query-1 class=tabs-scala-version %}
{% tab 'Scala 2' %}
```scala
import java.time.{ZoneId, ZonedDateTime}

object MyApp extends cask.MainRoutes {

  private def getZoneIdForCity(city: String): Option[ZoneId] = {
    import scala.jdk.CollectionConverters._
    ZoneId.getAvailableZoneIds.asScala.find(_.endsWith("/" + city)).map(ZoneId.of)
  }

  @cask.get("/time")
  def dynamicWithParam(city: Option[String] = None): String = {
    city match {
      case Some(value) => getZoneIdForCity(value) match {
        case Some(zoneId) => s"Current date is: ${ZonedDateTime.now().withZoneSameInstant(zoneId)}"
        case None => s"Couldn't find time zone for city $value"
      }
      case None => s"Current date is: ${ZonedDateTime.now()}"
    }
  }

  initialize()
}
```
{% endtab %}
{% tab 'Scala 3' %}
```scala
import java.time.{ZoneId, ZonedDateTime}

object MyApp extends cask.MainRoutes:

  private def getZoneIdForCity(city: String): Option[ZoneId] =
    import scala.jdk.CollectionConverters.*
    ZoneId.getAvailableZoneIds.asScala.find(_.endsWith("/" + city)).map(ZoneId.of)

    @cask.get("/time")
    def dynamicWithParam(city: Option[String] = None): String =
      city match
        case Some(value) => getZoneIdForCity(value) match
          case Some(zoneId) => s"Current date is: ${ZonedDateTime.now().withZoneSameInstant(zoneId)}"
          case None => s"Couldn't find time zone for city $value"
        case None => s"Current date is: ${ZonedDateTime.now()}"

  initialize()
```
{% endtab %}
{% endtabs %}

Run the example as before and access the [endpoint](http://localhost:8080/time?city=Paris) (notice the `?city=Paris` part of the URL).
You should get a result similar to the following one.
```
Current date is: 2024-07-22T10:08:18.218736+02:00[Europe/Paris]
```