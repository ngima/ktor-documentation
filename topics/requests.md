[//]: # (title: Requests)

Ktor allows you to handle incoming requests and send [responses](responses.md) inside [route handlers](Routing_in_Ktor.md#define_route). You can perform various actions when handling requests:
* Get [request information](#request_information), such as headers, cookies, and so on.
* Get [route parameter](#route_parameters) values.
* Get parameters of a [query string](#query_parameters).
* Receive [body contents](#body_contents), for example, data objects, form parameters, and files.

## General request information {id="request_information"}
Inside a route handler, you can get access to a request using the [call.request](https://api.ktor.io/%ktor_version%/io.ktor.application/-application-call/request.html) property. This returns the [ApplicationRequest](https://api.ktor.io/%ktor_version%/io.ktor.request/-application-request/index.html) instance and provides access to various request parameters. For example, the code snippet below shows how get a request URI:
```kotlin
routing {
    get("/") {
        val uri = call.request.uri
        call.respondText("Request uri: $uri")
    }
}
```
> The [call.respondText](responses.md#plain-text) method is used to send a response back to the client.

The [ApplicationRequest](https://api.ktor.io/%ktor_version%/io.ktor.request/-application-request/index.html) object allows you to get access to various request data, for example:
* Headers  
  To access all request headers, use the [ApplicationRequest.headers](https://api.ktor.io/%ktor_version%/io.ktor.request/-application-request/headers.html) property. You can also get access to specific headers using dedicated extension functions, such as `acceptEncoding`, `contentType`, `cacheControl`, and so on.
* Cookies  
  The [ApplicationRequest.cookies](https://api.ktor.io/%ktor_version%/io.ktor.request/-application-request/cookies.html) property provides access to cookies related to a request. To learn how to handle sessions using cookies, see the [Sessions](sessions.md) section.
* Connection details  
  Use the [ApplicationRequest.local](https://api.ktor.io/%ktor_version%/io.ktor.request/-application-request/local.html) property to get access to connection details such as a host name, port, scheme, and so on.
* `X-Forwarded-` headers  
  To get information about a request passed through an HTTP proxy or a load balancer, install the [](forward-headers.md) feature and use the [ApplicationRequest.origin](https://api.ktor.io/%ktor_version%/io.ktor.features/origin.html) property.


## Route parameters {id="route_parameters"}
When handling requests, you can get access to [route parameter](Routing_in_Ktor.md#route_parameter) values using the `call.parameters` property. For example, `call.parameters["login"]` in the code snippet below will return _admin_ for the `/user/admin` path:
```kotlin
```
{src="snippets/_misc/RouteParameter.kt"}


## Query parameters {id="query_parameters"}

To get access to parameters of a <emphasis tooltip="query_string">query string</emphasis>, you can use the [ApplicationRequest.queryParameters](https://api.ktor.io/%ktor_version%/io.ktor.request/-application-request/query-parameters.html) property. For example, if a request is made to `/products?price=asc`, you can access the `price` query parameter in this way:

```kotlin
```
{src="snippets/_misc/QueryParameter.kt"}

You can also obtain the entire query string using the [ApplicationRequest.queryString](https://api.ktor.io/%ktor_version%/io.ktor.request/query-string.html) function.


## Body contents {id="body_contents"}
This section shows how to receive body contents sent with `POST`, `PUT`, or `PATCH`:

### Objects {id="objects"}
Ktor provides a [ContentNegotiation](serialization.md) feature to negotiate the media type of request and deserialize content to an object of a required type. To receive and convert content for a request, call the [receive](https://api.ktor.io/%ktor_version%/io.ktor.request/receive.html) method that accepts a data class as a parameter: 
```kotlin
```
{src="snippets/_misc/SerializationReceiveObject.kt"}

You can learn more from [](serialization.md).

### Form parameters {id="form_parameters"}
Ktor allows you to receive form parameters sent with both `x-www-form-urlencoded` and `multipart/form-data` types using the [receiveParameters](https://api.ktor.io/%ktor_version%/io.ktor.request/receive-parameters.html) function. The example below shows an [HTTP client](https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html) `POST` request with parameters passed in a body:
```HTTP
POST http://localhost:8080/post
Content-Type: application/x-www-form-urlencoded

first_name=Jet&last_name=Brains
```
You can obtain the `first_name` and `last_name` parameter values in code as follows:
```kotlin
```
{src="snippets/_misc/FormParameter.kt"}

### Multipart form data {id="form_data"}
If you need to receive a file sent as a part of a multipart request, call the [receiveMultipart](https://api.ktor.io/%ktor_version%/io.ktor.request/receive-multipart.html) function and then loop over each part as required. In the example below, `PartData.FileItem` is used to receive a file as a byte stream.
```kotlin
```
{src="/snippets/upload-file/src/main/kotlin/com/example/UploadFile.kt" include-symbol="main"}

Learn how to run this sample from [upload-file](https://github.com/ktorio/ktor-documentation/tree/main/codeSnippets/snippets/upload-file).


### Raw payload {id="raw"}
If you need to access the raw body payload and parse it by yourself, you can use the following functions:
* [receiveChannel](https://api.ktor.io/%ktor_version%/io.ktor.request/-application-request/receive-channel.html)
* [receiveStream](https://api.ktor.io/%ktor_version%/io.ktor.request/receive-stream.html)
* [receiveText](https://api.ktor.io/%ktor_version%/io.ktor.request/receive-text.html)
