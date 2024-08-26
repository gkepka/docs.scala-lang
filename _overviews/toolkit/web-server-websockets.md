---
title: How to use websockets?
type: section
description: Using websockets with Cask
num: 35
previous-page: web-server-input
next-page: web-server-cookies-and-decorators
---

{% include markdown.html path="_markdown/install-cask.md" %}

You can create a WebSocket endpoint by using the `@cask.websocket` annotation. The endpoint method can return either a
`cask.WsHandler` instance defining how the communication should take place, or a `cask.Response`, which rejects the 
attempt of forming a WebSocket connection.

The connection can also be closed by sending `cask.Ws.close()` message through the WebSocket channel.

Create an HTML file named `websockets.html` with the following content and place it in `resources ` directory.

```html
<!DOCTYPE html>
<html>
<body>
<div>
    <input type="text" id="input" placeholder="Provide city name">
    <button onclick="sendMessage()">Send</button>
</div>
<div id="time"></div>
<script>
    const ws = new WebSocket('ws://localhost:8080/websocket');
    ws.onmessage = function(event) {
        receiveMessage(event.data);
    };

    ws.onclose = function(event) {
        receiveMessage('The connection has been closed');
    };

    function sendMessage() {
        const inputElement = document.getElementById('input');
        const message = inputElement.value;
        ws.send(message);
    }

    function receiveMessage(message) {
        const timeElement = document.getElementById('time');
        timeElement.textContent = message;
    }
</script>
</body>
</html>
```

The JavaScript code opens a WebSocket connection using the `ws://localhost:8080/websocket` endpoint. The `ws.onmessage`
event handler is executed when server pushes a message to the browser and `ws.onclose` when the connection is closed. 

Create an endpoint for serving static files using `@cask.staticResources` annotation and the endpoint for handling
the WebSocket connection.

{% tabs web-server-websocket-1 class=tabs-scala-version %}
{% tab 'Scala 2' %}
```scala
@cask.staticResources("/static")
def static() = "."

private def getZoneIdForCity(city: String): Option[ZoneId] = {
  import scala.jdk.CollectionConverters._
  ZoneId.getAvailableZoneIds.asScala.find(_.endsWith("/" + city)).map(ZoneId.of)
}

@cask.websocket("/websocket")
def websocket(): cask.WsHandler =
  cask.WsHandler { channel =>
    cask.WsActor {
      case cask.Ws.Text("") => channel.send(cask.Ws.Close())
      case cask.Ws.Text(city) =>
        val text = getZoneIdForCity(city) match {
          case Some(zoneId) => s"Current date is: ${ZonedDateTime.now().withZoneSameInstant(zoneId)}"
          case None => s"Couldn't find time zone for city $city"
        }
        channel.send(cask.Ws.Text(text))
    }
  }
```
{% endtab %}
{% tab 'Scala 3' %}
```scala
@cask.staticResources("/static")
def static() = "."

private def getZoneIdForCity(city: String): Option[ZoneId] =
  import scala.jdk.CollectionConverters.*
  ZoneId.getAvailableZoneIds.asScala.find(_.endsWith("/" + city)).map(ZoneId.of)

@cask.websocket("/websocket")
def websocket(): cask.WsHandler =
  cask.WsHandler { channel =>
    cask.WsActor {
      case cask.Ws.Text("") => channel.send(cask.Ws.Close())
      case cask.Ws.Text(city) =>
        val text = getZoneIdForCity(city) match
          case Some(zoneId) => s"Current date is: ${ZonedDateTime.now().withZoneSameInstant(zoneId)}"
          case None => s"Couldn't find time zone for city $city"
        channel.send(cask.Ws.Text(text))
    }
  }
```
{% endtab %}
{% endtabs %}

In the `cask.WsHandler` we define a `cask.WsActor` which reacts to events (of `cask.util.Ws.Event` type) and uses
WebSocket channel to send messages. In this example we receive a name of city and return the current time there. If server
receives an empty message, the connection is closed.