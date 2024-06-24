## Split project into a library and a binary
Its better if most of our project is a library and only a slim main function is what gets compiled into a binary
```toml
[package]
name = "zero2prod"
version = "0.1.0"
edition = "2021"

[lib]
path = "src/lib.rs"

[[bin]]
path = "src/main.rs"
name = "zero2prod"
```

## Test dependencies
Dev dependencies are used exclusively when running tests or examples
They do not get included in the final application binary!
```toml
[dev-dependencies]
reqwest = "0.11"
```

## Writing tests
we can write tests in `tests/`
`tokio::spawn` can essentially `await` in the background which will launch the server and make it possible to pass requests to it.
```rust
fn spawn_app() {
    let server = zero2prod::run().expect("Failed to create server.");
    // Launch server as a background task
    let _ = tokio::spawn(server);
}

#[actix_web::test]
async fn health_check_works() {
    // Arrange
    spawn_app();
    let client = reqwest::Client::new();

    // Act
    let response = client
        .get("http://127.0.0.1:8080/health_check")
        .send()
        .await
        .expect("Failed to execute request.");

    // Assert
    assert!(response.status().is_success());
    assert_eq!(Some(0), response.content_length());
}
```
`tokio::test` spins up a new runtime at the beginning of each test case and it shuts it down at the end of each test case.