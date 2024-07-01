
# Isolating test done on a database
To test a feature that effects a database we might run into the problem where if we perform the test on the same database we will have different results each time, in case our test modifies the state of the db.
So we need a way to isolate the test cases: create a new database for each test.

## Creating a new database and migrating it
```rust
let mut config = get_configuration()
	.expect("Failed to read configuration.");
config.database.database_name = Uuid::new_v4().to_string();
let db_pool = configure_db(&config.database).await;

pub async fn configure_db(config: &DatabaseSettings) -> PgPool {
    // Create database
    let _ = PgConnection::connect(&config.connection_string_without_db())
        .await
        .expect("Failed to connect to db.")
        .execute(format!(r#"CREATE DATABASE "{}";"#, config.database_name).as_str())
        .await
        .expect("Failed to create database.");

    // Create connection pool
    let db_pool = PgPool::connect(&config.connection_string())
        .await
        .expect("Failed to connect to databse.");

    // Migrate database
    sqlx::migrate!("./migrations")
        .run(&db_pool)
        .await
        .expect("Failed to migrate database");

    db_pool
}
```