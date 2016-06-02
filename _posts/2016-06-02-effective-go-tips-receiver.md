---
layout: post
title: effective go tips - receiver
---

This is about interface and implements of interface tips : )

<!--more-->

Since almost anything can have methods attached, almost anything can satisfy an interface. One illustrative example is in the http package, which defines the Handler interface. Any object that implements Handler can serve HTTP requests.

{% highlight golang %}

type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}

{% endhighlight %}

So I can support any simple implement of http.Handler, for example
{% highlight golang %}

// Simple counter server.
type Counter struct {
    n int
}

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ctr.n++
    fmt.Fprintf(w, "counter = %d\n", ctr.n)
}

{% endhighlight %}

We can use Counter as Handler to process http request like this
{% highlight golang %}

import "net/http"
...
ctr := new(Counter)
http.Handle("/counter", ctr)

{% endhighlight %}

Here is the question, why we have to use Counter as struct? Of course, we don't have to.
In golang, any object can implement interface, We can do like this
{% highlight golang %}

// Simpler counter server.
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    *ctr++ 
    fmt.Fprintf(w, "counter = %d\n", *ctr)
}

{% endhighlight %}

Such implement of http.Handler above, int type alias is named Counter.
We use a Counter pointer as recevier, in order to modify **real** value of ctr, not copy of it, and Counter implement http.Handler

What if your program has some internal state that needs to be notified that a page has been visited? Tie a channel to the web page.
{% highlight golang %}

// A channel that sends a notification on each visit.
// (Probably want the channel to be buffered.)
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ch <- req
    fmt.Fprint(w, "notification sent")
}

{% endhighlight %}

Finally, how to use function to implement a interface? We can define HandlerFunc type like this
{% highlight golang %}

// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers.  If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler object that calls f.
type MyHandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(c, req).
func (f MyHandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
    f(w, req)
}

{% endhighlight %}


We can write any logical statement in a function which type is MyHandlerFunc, use reference of this function as recevier, it also 
implement http.handler.

{% highlight golang %}

func main() {
    hl := net.Listen("tcp", "127.0.0.1:9999")
    var handler MyHandlerFunc
    handler = func(w http.ResponseWriter, req *http.Request) {
        w.Write([]byte("hehe"))
    }

    // we can use HandlerFunc as params in http.HandleFunc()
    // http.HandleFunc("/", handler)
    // http.Serve(hl, nil)

    // cast to http.Handler, use http.Handle()
    // http.Handle("/", http.HandlerFunc(handler))
    // http.Serve(hl, nil)

    // ofcourse, handler implement ServeHttp method defined in http.Handler
    http.Serve(hl, handler)
}

{% endhighlight %}

refer: [effective go](https://golang.org/doc/effective_go.html#interface_methods)
