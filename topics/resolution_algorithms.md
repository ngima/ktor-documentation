[//]: # (title: Resolution Algorithms)

[Routing](Routing_in_Ktor.md) is organized in a tree with a recursive matching system that is capable of handling quite
complex rules for request processing. The Tree is built with nodes that contain selector and optional handler. Selectors
are used to pick a route based on request. Every selector has quality, usually from 0.0 to 1.0, with one special case
of "transparent quality" (-1.0). This quality is useful when you need to wrap your subtree in a block, but this block
should not change routes priority. Example of it can be `route("/")` block in the middle of a routing tree.

Example tree:
```kotlin
route("a") { // matches first segment with the value "a" and quality 1.0
    route("b") { // matches second segment with the value "b" and quality 1.0
        method(HttpMethod.Get) { // matches GET verb with quality 1.0
            handle { call.respond("get") } // installs handler
        }
        post { call.respond("post") } // matches POST verb with quality 1.0, and installs a handler
    }
    route("/") { // always matches with "transparent" quality
        route("*") { // matches second segment with the any value and 0.5 quality
            handle { call.respond("wildcard") } // installs handler
        }
    }
    route("{...}") { // always matches with quality 0.1
        handle { call.respond("wildcard") } // installs handler
    }
}
```

Routing resolution algorithm consists of two parts.   
Traversing the tree to find matched routes:

1. Select root route's node as current entry
2. If the node has handlers and all path segments are used, add current node to successful matches
3. For every child:
    1. If child's selector does not match or selector quality is not transparent and is less than the best child match
       quality, skip current child
    2. Go to step 2 with child as node
    3. If current child's quality is better than the best child's, update the best matched child

Picking the best match:

1. Collect all paths from the root of the tree to the successful matched nodes
2. From all paths, remove nodes with "transparent" quality 
3. Choose paths with the highest qualities 
4. If there are multiple path with the same quality, chose the first one

[Actual implementation](https://github.com/ktorio/ktor/blob/main/ktor-server/ktor-server-core/jvm/src/io/ktor/routing/RoutingResolve.kt#L112) 
may differ in details for optimization reasons, but its result must be the same, as algorithm described here.

Using the example above, routing resolution for request `GET /a/b` will look like this:

* Check root route `route("a")` -> success, q=1.0, traverse children
    * Check first child `route("b")` -> success, q=1.0, traverse children
        * Check first child `method(Get)` -> success, q=1.0. Route has handler, add it to successful matches
        * Check second child `method(Post)` -> failure
    * Subtree has matches, update the best child match with current child `route("b")`, quality 1.0
    * Check second child `route("/")` -> success, q=-1, traverse children
        * Check first child `route("*")` -> success, q=0.5. Route has handler, add it to successful matches
    * Check third child `route("{...}")` -> success, q=0.1. Quality is less than the best child quality, ignore subtree
* Success matches are `/ a,q=1.0 / b,q=1.0 / <get>,q=1.0` and `a,q=1.0 / </>,q=-1.0 / <*>,q=0.5`
* Ignoring transparent qualities, matches are `/ a,q=1.0 / b,q=1.0 / :get,q=1.0` and `a,q=1.0 / *,q=0.5`
* First result has higher quality for the second node, choose it.
   


