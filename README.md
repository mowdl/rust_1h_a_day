# Rust: 1 Hour a Day

This repository is for notes I take while spending 1 hour a day learning about rust.

## Resources

- [The Rust Programming Language](https://doc.rust-lang.org/book/) (Official Rust Book)
- [Zero To Production In Rust](https://www.zero2prod.com)

## Progress Tracking

| Day   | Date       | Notes                                                  | Topics Covered                                                                                 |
| ----- | ---------- | ------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| 1     | 2024-06-23 | [Tests And Tokio](tests_and_tokio.md)                  | Spawning a server as a background task and running requests against it to automate testing.    |
| 2<br> | 2024-06-24 | [Random port](random_port.md)                          | Assigning a random available port to the server.                                               |
| 3     | 2024-06-25 | [Decoding URL encoded data](parsing_a_post_request.md) | Parsing URL encoded data from a post request with actix-web and a primer on how `serde` works. |
| 4     | 2024-06-26 | [PostgreSQL and sqlx](postgres_sqlx.md)                | Spinning up a postgres database with docker and running migrations with sqlx.                  |
| 5     | 2024-06-27 | [Enums in Rust](enums_in_rust.md)                      | enums, option and match.                                                                       |
| 6     | 2024-06-28 | [Configuration](configuration.md)                      | Loading configurations from a file.                                                            |
| 7     | 2024-06-30 | [actix-web, sqlx, postgres](connecting_to_postgres.md) | Connecting to postgres database and running queries from endpoint handlers.                    |
| 8     | 2024-07-01 | [Database Tests Isolation](test_isolation.md)          | Creating a database for each test.                                                             |

> Rust one hour a day, keeps javascript away.