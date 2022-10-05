# An async Rust client for SurrealDB's RPC endpoint
This crate serves as a temporary yet complete implement of an async Rust client to connect to a remote SurrealDB instance
via its RPC endpoint until the official SurrealDB client crate comes out.

The crate is aimed to be used in Rust backends and was not tested in a WASM environment.
_It probably doesn't work at all_

# Example
```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
  let mut client = SurrealClient::new("ws://127.0.0.1:8000/rpc")
    .await
    .expect("RPC handshake error");

  client.signin("root", "root").await.expect("Signin error");
  client.use_namespace("modspot", "modspot").await.expect("Namespace error");

  client.send_query("create User set username = $username".to_owned(), json!({ "username": "John" }),)
    .await
    .unwrap();

  let some_user: Option<User> = client.find_one("select * from User".to_owned(), Value::Null)
    .await
    .unwrap();

  if let Some(user) = some_user {
    print!("found user: {:?}", user);
  }
}
```

The `SurrealClient` type possesses utility functions to:
 - send a query in order to get a raw, unparsed response: `client.send_query()`
 - send a query and get the first element of type `<T>` from the response: `client.find_one()`
 - send a query and get the many elements of type `<T>` in the form of a `Vec<T>` from the response: `client.find_many()`

You can find a complete example in the [`./tests`](/tests) directory.