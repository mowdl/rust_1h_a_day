# Enums
```rust
enum Message {
    Quit,                        // no data
    Move { x: i32, y: i32 },     // named fields
    Write(String),               // a string
    ChangeColor(i32, i32, i32),  // three i32 values
}

// definnig methods on enums
impl Message {
	fn call(&self) {
		// do something
	}
}
```

## A very useful example: IPv4 and IPv6 as an enum
You can put any data inside an enum variant, like structs or even an enum.
```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

## Option enum
Null values are bad, so rust doesn't have any, but some routines might return a no-value, so they return an `Option` enum being either `None` or `Some(T)`.
```rust
enum Option<T> {
    None,
    Some(T),
}
```

# The `match` Control Flow Construct
We can math against variants of an enum, and also can bind over the values in a certain variant.
```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```
This pattern is useful when dealing with the Option enum.

## Matches Are Exhaustive
When using a match we are required to cover all possibilities. But it is possible to make a case that will match a against all values: A catch all pattern:
```rust
let dice_roll = 9;
match dice_roll {
	3 => add_fancy_hat(),
	7 => remove_fancy_hat(),
	other => move_player(other),
	// or we can disregard the value entirely
	// _ => (),
}
```