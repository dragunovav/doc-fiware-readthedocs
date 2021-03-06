FIWARE Stream Oriented Generic Enabler - Open API Specification
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

The Stream Oriented API is a resource-oriented API accessed via WebSockets that
uses JSON-RPC V2.0 based representations for information exchange. An RPC call
is represented by sending a **request** message to a server. Each request
message has the following members:

* `jsonrpc`: a string specifying the version of the JSON-RPC protocol. It must
  be exactly `2.0`.

* `id`: an unique identifier established by the client that contains a string
  or number. The server must reply with the same value in the response message.
  This member is used to correlate the context between both messages.

* `method`: a string containing the name of the method to be invoked.

* `params`: a structured value that holds the parameter values to be used
  during the invocation of the method.

When an RPC call is made by a client, the server replies with a **response**
object. In the case of a success, the response object contains the following
members:

* `jsonrpc`: it must be exactly `2.0`.

* `id`: it must match the value of the id member in the request object.

* `result`: structured value which contains the invocation result.

In the case of an **error**, the response object contains the following members:

* `jsonrpc`: it must be exactly `2.0`.

* `id`: it must match the value of the id member in the request object.

* `error`: object describing the error through the following members:

    * `code`: integer number that indicates the error type that occurred

    * `message`: string providing a short description of the error.

    * `data`: primitive or structured value that contains additional
      information about the error. It may be omitted. The value of this member
      is defined by the server.

Therefore, the value of the `method` parameter in the request determines the
type of request/response to be exchanged between client and server. The
following section describes each pair of messages depending of the type of
`method` (namely: `Ping`, `Create`, `Invoke`, `Release`, `Subscribe`,
`Unsubscribe`, and `OnEvent`).


Ping
==== 

In order to warranty the WebSocket connectivity between the client and the
Kurento Media Server, a keep-alive method is implemented. This method is based
on a `ping` method sent by the client, which must be replied with a `pong`
message from the server. If no response is obtained in a time interval, the
client is aware that the connectivity with the media server has been lost.

Request
-------

A `ping` request contains the following parameters:

* `method` (required, string). Value: `ping`.

* `params` (required, object). Parameters for the invocation of the ping
  message, containing these member:

  * interval (required, number). Time out to receive the `pong` message from
    the server, in milliseconds. By default this value is `240000` (i.e. 40
    seconds).

This is an example of `ping`:

+ Body (application/json)

::

   {
       "id": 1,
       "method": "ping",
       "params": {
           "interval": 240000
       },
       "jsonrpc": "2.0"
   }

Response
--------

The response to a `ping` request must contain a `result` object with a `value`
parameter with a fixed name: `pong`. The following snippet shows the `pong`
response to the previous `ping` request:

+ Body (application/json)

::

   {
       "id": 1,
       "result": {
           "value": "pong"
       },
       "jsonrpc": "2.0"
   }

Create
======

Create message requests the creation of an Media Pipelines and Media Elements in
the Media Server. The parameter type specifies the type of the object to be
created. The parameter `params` contains all the information needed to create
the object. Each message needs different parameters to create the object.

Media Elements have to be contained in a previously created Media Pipeline.
Therefore, before creating Media Elements, a Media Pipeline must exist. The
response of the creation of a Media Pipeline contains a parameter called
`sessionId`, which must be included in the next create requests for Media
Elements.

Request
-------

A `create` request contains the following parameters:

* `method` (required, string). Value: `create`.

* `params` (required, object). Parameters for the invocation of the create
  message, containing these members:

    * type (required, string). Media pipeline or media element to be
      created. The allowed values are the following:

        * `MediaPipeline`: Media Pipeline to be created.

        * `WebRtcEndpoint`: This media element offers media streaming
          using WebRTC.

        * `RtpEndpoint`: Media element that provides bidirectional
          content delivery capabilities with remote networked peers through RTP
          protocol. It contains paired sink and source MediaPad for audio and
          video.

        * `HttpPostEndpoint`: This type of media element provides
          unidirectional communications. Its MediaSource are related to HTTP
          POST method. It contains sink MediaPad for audio and video, which
          provide access to an HTTP file upload function.

        * `PlayerEndpoint`: It provides function to retrieve contents
          from seekable sources in reliable mode (does not discard media
          information) and inject them into KMS. It contains one MediaSource
          for each media type detected.

        * `RecorderEndpoint`: Provides function to store contents in
          reliable mode (doesn't discard data). It contains MediaSink pads for
          audio and video.

        * `FaceOverlayFilter`: It detects faces in a video feed. The
          face is then overlaid with an image.

        * `ZBarFilter`: This Filter detects QR and bar codes in a
          video feed. When a code is found, the filter raises a CodeFound.

        * `GStreamerFilter`: This is a generic Filter interface, that
          creates GStreamer filters in the media server.

        * `Composite`: A Hub that mixes the audio stream of its
          connected sources and constructs a grid with the video streams of its
          connected sources into its sink.

        * `Dispatcher`: A Hub that allows routing between arbitrary
          port pairs.

        * `DispatcherOneToMany`: A Hub that sends a given source to
          all the connected sinks.

    * `constructorParams` (required, object). Additional parameters. For
      example:

        * `mediaPipeline` (optional, string): This parameter is only
          mandatory for Media Elements. In that case, the value of this
          parameter is the identifier of the media pipeline which is going to
          contain the Media Element to be created.

        * `uri` (optional, string): This parameter is only required
          for Media Elements such as `PlayerEndpoint` or `RecorderEndpoint`. It
          is an URI used in the Media Element, i.e. the media to be played (for
          `PlayerEndpoint`) or the location of the recording (for
          `RecorderEndpoint`).

        *  `properties` (optional, object): Array of additional
           objects (key/value).

    * `sessionId` (optional, string). Session identifier. This parameter
      is not present in the first request (typically the media pipeline
      creation).


The following example shows a request message requesting the creation of an
object of the type `MediaPipeline`:

+ Body (application/json)

::

   {
       "id": 2,
       "method": "create",
       "params": {
           "type": "MediaPipeline",
           "constructorParams": {},
           "properties": {}
       },
       "jsonrpc": "2.0"
   }

The following example shows a request message requesting the creation of an
object of the type `WebRtcEndpoint` within an existing Media Pipeline
(identified by the parameter `mediaPipeline`). Notice that in this request, the
`sessionId` is already present, while in the previous example it was not (since
at that point was unknown for the client):

+ Body (application/json)

::

   {
       "id": 3,
       "method": "create",
       "params": {
           "type": "WebRtcEndpoint",
           "constructorParams": {
               "mediaPipeline": "6ba9067f-cdcf-4ea6-a6ee-d74519585acd_kurento.MediaPipeline"
           },
           "properties": {},
           "sessionId": "bd4d6227-0463-4d52-b1c3-c71f0be68466"
       },
       "jsonrpc": "2.0"
   }

Response
--------

The response message contains the identifier of the new object in the field
value. As usual, the message `id` must match with the request message. The
`sessionId` is also returned in each response. A `create` response contains the
following parameters:

* `result` (required, object). Result of the create invocation:

    * `value` (required, number). Identifier of the created media element.

    * `sessionId` (required, string). Session identifier.

The following examples shows the responses to the previous request messages
(respectively, the response to the `MediaPipeline` create message, and then the
response to the to `WebRtcEndpoint` create message). In the first example, the
parameter ``value`` identifies the created Media Pipelines, and ``sessionId``
is the identifier of the current session.

+ Body (application/json)

::

   {
       "id": 2,
       "result": {
           "value": "6ba9067f-cdcf-4ea6-a6ee-d74519585acd_kurento.MediaPipeline",
           "sessionId": "bd4d6227-0463-4d52-b1c3-c71f0be68466"
       },
       "jsonrpc": "2.0"
   }

In the second response example, the parameter ``value`` identifies the created
Media Element (a ``WebRtcEndpoint`` in this case). Notice that this value also
identifies the Media Pipeline in which the Media Element is contained. The
parameter ``sessionId`` is also contained in the response.

+ Body (application/json)

::

   {
       "id": 3,
       "result": {
           "value": "6ba9067f-cdcf-4ea6-a6ee-d74519585acd_kurento.MediaPipeline/087b7777-aab5-4787-816f-f0de19e5b1d9_kurento.WebRtcEndpoint",
           "sessionId": "bd4d6227-0463-4d52-b1c3-c71f0be68466"
       },
       "jsonrpc": "2.0"
   }

Invoke
======

Invoke message requests the invocation of an operation in the specified object.
The parameter object indicates the identifier of the object in which the
operation will be invoked. The parameter operation carries the name of the
operation to be executed. Finally, the parameter `operationParams` contains the
parameters needed to execute the operation.

Request
-------

An `invoke` request contains the following parameters:

* `method` (required, string). Value is `invoke`.

* `params` (required, object)

    * `object` (required, number). Identifier of the source media element.

    * `operation` (required, string). Operation invoked. Allowed Values:
 
        * `connect`. Connect two media elements.

        * `play`. Start the play of a media (`PlayerEndpoint`).

        * `record`. Start the record of a media (`RecorderEndpoint`).

        * `setOverlayedImage`. Set the image that is going to be
          overlaid on the detected faces in a media stream
          (`FaceOverlayFilter`).

        * `processOffer`. Process the offer in the SDP negotiation
          (`WebRtcEndpoint`).

        * `gatherCandidates`. Start the ICE candidates gathering to
          establish a WebRTC media session (`WebRtcEndpoint`).

        * `addIceCandidate`. Add ICE candidate (`WebRtcEndpoint`).

    * `operationParams` (optional, object).
 
        * `sink` (required, number). Identifier of the sink media
          element.

        * `offer` (optional, string). SDP offer used in the WebRTC SDP
          negotiation (in `WebRtcEndpoint`).

    * `sessionId` (required, string). Session identifier.

The following example shows a request message requesting the invocation of the
operation connect on a `PlayerEndpoint` connected to a `WebRtcEndpoint`:

+ Body (application/json)

::

   {
       "id": 5,
       "method": "invoke",
       "params": {
           "object": "6ba9067f-cdcf-4ea6-a6ee-d74519585acd_kurento.MediaPipeline/76dcb8d7-5655-445b-8cb7-cf5dc91643bc_kurento.PlayerEndpoint",
           "operation": "connect",
           "operationParams": {
               "sink": "6ba9067f-cdcf-4ea6-a6ee-d74519585acd_kurento.MediaPipeline/087b7777-aab5-4787-816f-f0de19e5b1d9_kurento.WebRtcEndpoint"
           },
           "sessionId": "bd4d6227-0463-4d52-b1c3-c71f0be68466"
       },
       "jsonrpc": "2.0"
   }


Response
--------

The response message contains the value returned while executing the operation
invoked in the object or nothing if the operation doesn’t return any value.

An `invoke` response contains the following parameters:

* `result` (required, object)

    * `sessionId` (required, string). Session identifier.

    * `value` (optional, object). Additional object which describes the
      result of the `Invoke` operation. For example, in a `WebRtcEndpoint` this
      field is the SDP response (WebRTC SDP negotiation).

The following example shows a typical response while invoking the operation
connect:

+ Body (application/json)

::

   {
       "id": 5,
       "result": {
           "sessionId": "bd4d6227-0463-4d52-b1c3-c71f0be68466"
       },
       "jsonrpc": "2.0"
   }

Release
=======

Release message requests the release of the specified object. The parameter
`object` indicates the id of the object to be released:

Request
-------

A `release` request contains the following parameters:

* `method` (required, string). Value is `release`.

* `params` (required, object).

    * `object` (required, number). Identifier of the media element or
      pipeline to be released.

    * `sessionId` (required, string). Session identifier.

+ Body (application/json)

::

   {
       "id": 36,
       "method": "release",
       "params": {
           "object": "6ba9067f-cdcf-4ea6-a6ee-d74519585acd_kurento.MediaPipeline",
           "sessionId": "bd4d6227-0463-4d52-b1c3-c71f0be68466"
       },
       "jsonrpc": "2.0"
   }

Response
--------

A `release` response contains the following parameters:

* `result` (required, object)

    * `sessionId` (required, string). Session identifier.

The response message only contains the `sessionId`. The following example shows
the typical response of a release request:

+ Body (application/json)

::

   {
       "id": 36,
       "result": {
           "sessionId": "bd4d6227-0463-4d52-b1c3-c71f0be68466"
       },
       "jsonrpc": "2.0"
   }

Subscribe
=========

Subscribe message requests the subscription to a certain kind of events in the
specified object. The parameter object indicates the id of the object to
subscribe for events. The parameter type specifies the type of the events. If a
client is subscribed for a certain type of events in an object, each time an
event is fired in this object, a request with method onEvent is sent from
Kurento Media Server to the client. This kind of request is described few
sections later.

Request
-------

A `subscribe` request contains the following parameters:

* `method` (required, string). Value is `subscribe`.

* `params` (required, object). Parameters for the invocation of the subscribe
  message, containing these members:

    * `type` (required, string). Media event to be subscribed. The allowed
      values are the following:

        * `CodeFoundEvent`: raised by a `ZBarFilter` when a code is
          found in the data being streamed.

        * `ConnectionStateChanged`: Indicates that the state of the
          connection has changed.

        * `ElementConnected`: Indicates that an element has been
          connected to other.

        * `ElementDisconnected`: Indicates that an element has been
          disconnected.

        * `EndOfStream`: Event raised when the stream that the element
          sends out is finished.

        * `Error`: An error related to the MediaObject has occurred.

        * `MediaSessionStarted`: Event raised when a session starts.
          This event has no data.

        * `MediaSessionTerminated`: Event raised when a session is
          terminated. This event has no data.

        * `MediaStateChanged`: Indicates that the state of the media
          has changed.

        * `ObjectCreated`: Indicates that an object has been created
          on the media server.

        * `ObjectDestroyed`: Indicates that an object has been
          destroyed on the media server.

        * `OnIceCandidate`: Notify of a new gathered local candidate.

        * `OnIceComponentStateChanged`: Notify about the change of an
          ICE component state.

        * `OnIceGatheringDone`: Notify that all candidates have been
          gathered.

    * `object` (required, string). Media element identifier in which the
      event is subscribed.

    * `sessionId` (required, string). Session identifier.

The following example shows a request message requesting the subscription of the
event type `EndOfStream` on a `PlayerEndpoint` Media Element:

+ Body (application/json)

::

   {
       "id": 11,
       "method": "subscribe",
       "params": {
           "type": "EndOfStream",
           "object": "6ba9067f-cdcf-4ea6-a6ee-d74519585acd_kurento.MediaPipeline/76dcb8d7-5655-445b-8cb7-cf5dc91643bc_kurento.PlayerEndpoint",
           "sessionId": "bd4d6227-0463-4d52-b1c3-c71f0be68466"
       },
       "jsonrpc": "2.0"
   }

Response
--------

The response message contains the subscription identifier. This value can be
used later to remove this `subscription`.

A `subscribe` response contains the following parameters:

* `result` (required, object). Result of the subscription invocation. This
  object contains the following members:

    * `value` (required, number). Identifier of the media event.

    * `sessionId` (required, string). Session identifier.

The following example shows the response of subscription request. The `value`
attribute contains the subscription identifier:

+ Body (application/json)

::

   {
       "id": 11,
       "result": {
           "value": "052061c1-0d87-4fbd-9cc9-66b57c3e1280",
           "sessionId": "bd4d6227-0463-4d52-b1c3-c71f0be68466"
       },
       "jsonrpc": "2.0"
   }

Unsubscribe
===========

Unsubscribe message requests the cancellation of a previous event subscription.
The parameter `subscription` contains the subscription id received from the
server when the subscription was created.

Request
-------

An `unsubscribe` request contains the following parameters:

* `method` (required, string). Value is `unsubscribe`.

* `params` (required, object).

    * `object` (required, string). Media element in which the subscription
      is placed.

    * `subscription` (required, number). Subscription identifier.

    * `sessionId` (required, string). Session identifier.

The following example shows a request message requesting the cancellation of the
`subscription` `353be312-b7f1-4768-9117-5c2f5a087429`:

+ Body (application/json)

::

   {
       "id": 38,
       "method": "unsubscribe",
       "params": {
           "subscription": "052061c1-0d87-4fbd-9cc9-66b57c3e1280",
           "object": "6ba9067f-cdcf-4ea6-a6ee-d74519585acd_kurento.MediaPipeline/76dcb8d7-5655-445b-8cb7-cf5dc91643bc_kurento.PlayerEndpoint",
           "sessionId": "bd4d6227-0463-4d52-b1c3-c71f0be68466"
       },
       "jsonrpc": "2.0"
   }

Response
--------

The response message only contains the `sessionId`. The following example shows
the typical response of an unsubscription request:

An `unsubscribe` response contains the following parameters:

* `result` (required, object)

    * `sessionId` (required, string). Session identifier.

For example:

+ Body (application/json)

::

   {
       "id": 38,
       "result": {
           "sessionId": "bd4d6227-0463-4d52-b1c3-c71f0be68466"
       },
       "jsonrpc": "2.0"
   }

OnEvent
=======

When a client is `subscribed` to a type of events in an object, the server sends
an onEvent request each time an event of that type is fired in the object. This
is possible because the Stream Oriented open API is implemented with WebSockets
and there is a full duplex channel between client and server.

Request
-------

An `OnEvent` request contains the following parameters:

* `method` (required, string). Value is `onEvent`.

* `params` (required, object). 

    * `value` (required, object)

        * `data` (required, object)

            * `source` (required, string). Source media element.

            * `tags` (optional, string array). Metadata for the
              media element.

            * `timestamp` (required, number). Media server time
              and date (in Unix time, i.e., number of seconds since 01/01/1970).

            * `type` (required, string). Same type identifier
              described on `subscribe` message (i.e.: `CodeFound`,
              `ConnectionStateChanged`, `ElementConnected`,
              `ElementDisconnected`, `EndOfStream`, `Error`,
              `MediaSessionStarted`, `MediaSessionTerminated`,
              `MediaStateChanged`, `ObjectCreated`, `ObjectDestroyed`,
              `OnIceCandidate`, `OnIceComponentStateChanged`,
              `OnIceGatheringDone`)

        * `object` (required, object).Media element identifier.

        * `type` (required, string). Type identifier (same value than
          before)

The following example shows a notification sent for server to client to notify
an event of type `EndOfStream` in a `PlayerEndpoint` object:

+ Body (application/json)

::

   {
       "jsonrpc": "2.0",
       "method": "onEvent",
       "params": {
           "value": {
               "data": {
                   "source": "6ba9067f-cdcf-4ea6-a6ee-d74519585acd_kurento.MediaPipeline/76dcb8d7-5655-445b-8cb7-cf5dc91643bc_kurento.PlayerEndpoint",
                   "tags": [],
                   "timestamp": "1461589478",
                   "type": "EndOfStream"
               },
               "object": "6ba9067f-cdcf-4ea6-a6ee-d74519585acd_kurento.MediaPipeline/76dcb8d7-5655-445b-8cb7-cf5dc91643bc_kurento.PlayerEndpoint",
               "type": "EndOfStream"
           }
       }
   }

Notice that this message has no `id` field due to the fact that no response is
required.

Response
--------

There is no response to the `onEvent` message.
