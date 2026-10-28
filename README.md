# Hello World gRPC with Rust and Tonic

A small, complete example of a gRPC service written in Rust. It includes a
Tonic server, a Tonic client, and a Protocol Buffer contract that defines the
service API.

This project is useful as a starting point for learning how to:

- Define an RPC service with Protocol Buffers
- Generate Rust types and service traits during the Cargo build
- Implement a gRPC server with Tonic
- Call the server from a Rust client

## How it works

The service exposes one RPC:

```text
Greeter.SayHello(HelloRequest) -> HelloReply
```

The sample client sends a request containing `World`. The server responds with
`Hello World!`.

```text
Client                                  Server
  |                                       |
  | -- SayHello(name: "World") ---------> |
  | <--------- "Hello World!" ----------- |
```

The server listens locally on `[::1]:50051`.

## Prerequisites

Install the Rust toolchain, including Cargo:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

After installation, verify that Rust is available:

```bash
rustc --version
cargo --version
```

The project uses Rust edition 2024 and requires a current stable Rust release.

## Quick start

Clone the repository and enter the project directory:

```bash
git clone <repository-url>
cd helloworld-tonic
```

Build and validate the project:

```bash
cargo check
```

Start the server in the first terminal:

```bash
cargo run --bin helloworld-server
```

Leave the server running. In a second terminal, enter the same project
directory and run the client:

```bash
cargo run --bin helloworld-client
```

The client prints a response similar to:

```text
RESPONSE=Response { ... message: HelloReply { message: "Hello World!" } ... }
```

The server also logs the received request:

```text
GOT A REQUEST=Request { ... }
```

Stop the server with `Ctrl+C`.

## Project layout

```text
helloworld-tonic/
├── Cargo.toml              # Package metadata, dependencies, and binary names
├── build.rs                # Generates Rust code from the .proto file
├── src/
│   ├── server.rs           # Greeter gRPC server implementation
│   ├── client.rs           # Example Greeter gRPC client
│   └── main.rs             # Separate default package binary
├── proto/
│   └── helloworld.proto    # Service, request, and response definitions
```

The server and client are configured as named binaries in `Cargo.toml`:

```toml
[[bin]]
name = "helloworld-server"
path = "src/server.rs"

[[bin]]
name = "helloworld-client"
path = "src/client.rs"
```

That is why they are started with `cargo run --bin helloworld-server` and
`cargo run --bin helloworld-client`.

## Protocol Buffer contract

The API is defined in [`proto/helloworld.proto`](proto/helloworld.proto):

```protobuf
service Greeter {
    rpc SayHello (HelloRequest) returns (HelloReply);
}
```

`build.rs` runs during the Cargo build and generates the Rust client and server
types. The generated code is included by both `src/server.rs` and `src/client.rs`
through `tonic::include_proto!("helloworld")`; it is not checked into source
control.

## Useful Cargo commands

```bash
# Check compilation without producing a runnable binary
cargo check

# Build debug binaries
cargo build

# Build optimized release binaries
cargo build --release

# Run the server from the release build
cargo run --release --bin helloworld-server

# Run the client from the release build
cargo run --release --bin helloworld-client

# Format Rust source files
cargo fmt

# Run the standard Rust linter
cargo clippy
```

## Troubleshooting

### Address already in use

Another process is already listening on port `50051`. Stop that process or
change the address in both [`src/server.rs`](src/server.rs) and
[`src/client.rs`](src/client.rs)
to use the same available port.

### Connection refused

Start the server before running the client. The client connects to
`http://[::1]:50051`, so the server and client must use the same host and port.

### Cargo cannot find a binary

Run the commands from the directory containing `Cargo.toml`, and use the exact
binary names configured in the manifest:

```bash
cargo run --bin helloworld-server
cargo run --bin helloworld-client
```
