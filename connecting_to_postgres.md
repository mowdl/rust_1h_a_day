# Connecting to a postgres database
We will try connect to a running postgres database from an actix-web using sqlx to query from it.

## Connecting to a db
We first need a connection string:
```rust
impl DatabaseSettings {
    pub fn connection_sting(&self) -> String {
        format!(
            "postgres://{}:{}@{}:{}/{}",
            self.username, self.password, self.host, self.port, self.database_name
        )
    }
}
```

Then we create a connection pool:
```rust
	let db_con_pool = PgPool::connect(
			&configuration.database.connection_sting()
		)
        .await
        .expect("Failed to connect to postgres");
```
 Why create a `PgPool` instead of just a `PgConnection`.
 An actix-web server will be many workers that will handle our endpoint, upon receiving requests these workers may need to query the database at the same time, but to query the database you will need a `&mut PgConnection` wish you can only have one of at the same time. We can protect with a mutex but that will hurt performance.
 So we instead can use `PgPool` which will use an existing not used connection or create a new one each time we perform a query on the database.

## Passing the connection to the handlers
We need to make the connection pool part of our actix-web app state.
```rust
pub fn run(listener: TcpListener, db_con_pool: PgPool) -> Result<Server, std::io::Error> {
    // Wrap connection in a smart pointer since it will be accessed
    // from many parallel workers.
    // This will wrap it in an Arc which is a thread-safe
    // reference-counting pointer
    let db_con_pool = web::Data::new(db_con_pool);

    let server = HttpServer::new(move || {
        App::new()
            .service(health_check)
            .service(subscriptions)
            .app_data(db_con_pool.clone())
    })
    .listen(listener)?
    .run();
    Ok(server)
}
```

And to access it from the handler function we just need to include the extractor as an argument:
```rust
#[post("/subscriptions")]
pub async fn subscriptions(
    form: web::Form<FormData>,
    db_pool: web::Data<PgPool>,
) -> impl Responder {
    /* proccess request */
}
```

## Querying the database
Using the `query!` macro form sqlx. But we need to make sure that `DATABASE_URL` is defined (or defined it in a .env file), so that sqlx runs its compile time checks.
```rust
#[post("/subscriptions")]
pub async fn subscriptions(
    form: web::Form<FormData>,
    db_pool: web::Data<PgPool>,
) -> impl Responder {
    match sqlx::query!(
        r#"
        INSERT INTO subscriptions (id, email, name, subscribed_at)
        VALUES($1, $2, $3, $4)
        "#,
        uuid::Uuid::new_v4(),
        form.email,
        form.name,
        chrono::Utc::now()
    )
    .execute(db_pool.get_ref())
    .await
    {
        Ok(_) => HttpResponse::Ok().finish(),
        Err(e) => {
            println!("Failed to execute query: {e}");
            HttpResponse::InternalServerError().finish()
        }
    }
}
```