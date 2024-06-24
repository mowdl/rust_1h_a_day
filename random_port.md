# Choosing a random port
Previously in the zero2prod code we hard coded `127.0.0.1:8080` as the port that the server will listen to for requests, this is not ideal when we run our test that spawn a server each time and also because the port might be taken by another service

## Port 0
Port 0 is special-cased at the OS level: trying to bind port 0 will trigger an OS scan for an available port which will then be bound to the application.

## Creating a TcpListener
To bind a port to the sever, instead of calling `bind` on it, we can call `listen` and pass it a `TcpListener`. 
```rust
pub fn run(listener: TcpListener) -> Result<Server, std::io::Error> {
    let server = HttpServer::new(|| App::new().service(health_check))
        .listen(listener)?
        .run();
    Ok(server)
}
```

And by a having the listener we can know to which random port it got bound to.
```rust
fn spawn_app() -> String {
    let listener = TcpListener::bind("127.0.0.1:0")
	    .expect("Failed to bind to random port.");

    let port = listener.local_addr().unwrap().port();

    let server = zero2prod::run(listener)
	    .expect("Failed to create server.");

    // Launch server as a background task
    let _ = tokio::spawn(server);
    format!("http://127.0.0.1:{port}")
}
```
