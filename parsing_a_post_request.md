# Parsing URL encoded data with actix-web
`Form` data helper can be used to decode or encode url-encoded data from a request body.
The extractor can be accessed as an argument to a handler function.
```rust
#[derive(serde::Deserialize)]
struct FormData {
    email: String,
    name: String,
}

#[post("/subscriptions")]
async fn subscriptions(_form: web::Form<FormData>) -> impl Responder {
    HttpResponse::Ok().finish()
}
```

If the extraction failed an `actix_web::Error` is retuned which is turned into a `HttpResponse` with `400`.

All this works through the magic of `serde`.

## serde
serde is a framework for serializing and deserializing Rust data types.
serde itself doesn't provide de/serialization of any specific data format.
serde defines a data model consistent of 29 type.

To implement a library to support serialization for a new data format, you have to provide an implementation of the `Serializer` trait. Which contains methods that specify how each of those 29 types maps to your specific data format.
The same is true for deserialisation, via Deserialize and Deserializer, with a few additional details around lifetimes to support zero-copy deserialisation.

It does all this _efficiently_ because the data model _doesn't exist_, as there is no intermediate conversion to it and from it. (some complicated traits and generics stuff that gets compiled away)

## Summary
- Before calling subscribe actix-web invokes the from_request method for all subscribe’s input arguments: in our case, Form::from_request;
- Form::from_request tries to deserialise the body into FormData according to the rules of URL encoding leveraging serde_urlencoded and the Deserialize implementation of FormData, automatically generated for us by #[derive(serde::Deserialize)];
- if Form::from_request fails, a 400 BAD REQUEST is returned to the caller. If it succeeds, subscribe is invoked and we return a 200 OK