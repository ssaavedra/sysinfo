# sysinfo ![img_github_ci] [![][img_crates]][crates] [![][img_doc]][doc]

`sysinfo` is a crate used to get a system's information.

## Supported Oses

It currently supports the following OSes (alphabetically sorted):

 * Android
 * FreeBSD
 * iOS
 * Linux
 * macOS
 * Raspberry Pi
 * Windows

You can still use `sysinfo` on non-supported OSes, it'll simply do nothing and always return
empty values. You can check in your program directly if an OS is supported by checking the
[`SystemExt::IS_SUPPORTED`] constant.

The minimum-supported version of `rustc` is **1.54**.

## Usage

⚠️ Before any attempt to read the different structs' information, you need to update them to
get up-to-date information because for most of them, it works on diff between the current value
and the old one.

Which is why, it's much better to keep the same instance of [`System`] around instead of
recreating it multiple times.

You have an example into the `examples` folder. You can run it with `cargo run --example simple`.

Otherwise, here is a little code sample:

```rust
use sysinfo::{NetworkExt, NetworksExt, ProcessExt, System, SystemExt};

// Please note that we use "new_all" to ensure that all list of
// components, network interfaces, disks and users are already
// filled!
let mut sys = System::new_all();

// First we update all information of our `System` struct.
sys.refresh_all();

// We display all disks' information:
println!("=> disks:");
for disk in sys.disks() {
    println!("{:?}", disk);
}

// Network interfaces name, data received and data transmitted:
println!("=> networks:");
for (interface_name, data) in sys.networks() {
    println!("{}: {}/{} B", interface_name, data.received(), data.transmitted());
}

// Components temperature:
println!("=> components:");
for component in sys.components() {
    println!("{:?}", component);
}

println!("=> system:");
// RAM and swap information:
println!("total memory: {} KB", sys.total_memory());
println!("used memory : {} KB", sys.used_memory());
println!("total swap  : {} KB", sys.total_swap());
println!("used swap   : {} KB", sys.used_swap());

// Display system information:
println!("System name:             {:?}", sys.name());
println!("System kernel version:   {:?}", sys.kernel_version());
println!("System OS version:       {:?}", sys.os_version());
println!("System host name:        {:?}", sys.host_name());

// Number of processors:
println!("NB processors: {}", sys.processors().len());

// Display processes ID, name na disk usage:
for (pid, process) in sys.processes() {
    println!("[{}] {} {:?}", pid, process.name(), process.disk_usage());
}

```

By default, `sysinfo` uses multiple threads. However, this can increase the memory usage on some
platforms (macOS for example). The behavior can be disabled by setting `default-features = false`
in `Cargo.toml` (which disables the `multithread` cargo feature).

### Running on Raspberry Pi

It'll be difficult to build on Raspberry Pi. A good way-around is to cross-build, then send the
executable to your Raspberry Pi.

First install the arm toolchain, for example on Ubuntu:

```bash
> sudo apt-get install gcc-multilib-arm-linux-gnueabihf
```

Then configure cargo to use the corresponding toolchain:

```bash
cat << EOF > ~/.cargo/config
[target.armv7-unknown-linux-gnueabihf]
linker = "arm-linux-gnueabihf-gcc"
EOF
```

Finally, cross compile:

```bash
rustup target add armv7-unknown-linux-gnueabihf
cargo build --target=armv7-unknown-linux-gnueabihf
```

### Linux on Docker & Windows Subsystem for Linux (WSL)

Virtual Linux systems, such as those run through Docker and Windows Subsystem for Linux (WSL), do
not receive host hardware information via `/sys/class/hwmon` or `/sys/class/thermal`. As such,
querying for components may return no results (or unexpected results) when using this library on
virtual systems.

### Use in binaries running inside the macOS or iOS Sandbox/stores

Apple has restrictions as to which APIs can be linked into binaries that are distributed through the app store.
By default, `sysinfo` is not compatible with these restrictions. You can use the `apple-app-store`
feature flag to disable the Apple prohibited features. This also enables the `apple-sandbox` feature. 
In the case of applications using the sandbox outside of the app store, the `apple-sandbox` feature 
can be used alone to avoid causing policy violations at runtime.

### How it works

I wrote a blog post you can find [here][sysinfo-blog] which explains how `sysinfo` extracts information
on the diffent systems.

[sysinfo-blog]: https://blog.guillaume-gomez.fr/articles/2021-09-06+sysinfo%3A+how+to+extract+systems%27+information

### C interface

It's possible to use this crate directly from C. Take a look at the `Makefile` and at the
`examples/simple.c` file.

To build the C example, just run:

```bash
> make
> ./simple
# If needed:
> LD_LIBRARY_PATH=target/release/ ./simple
```

### Benchmarks

You can run the benchmarks locally with rust **nightly** by doing:

```bash
> cargo bench
```

## Donations

If you appreciate my work and want to support me, you can do it with
[github sponsors](https://github.com/sponsors/GuillaumeGomez) or with
[patreon](https://www.patreon.com/GuillaumeGomez).

[img_github_ci]: https://github.com/GuillaumeGomez/sysinfo/workflows/CI/badge.svg
[img_crates]: https://img.shields.io/crates/v/sysinfo.svg
[img_doc]: https://img.shields.io/badge/rust-documentation-blue.svg

[crates]: https://crates.io/crates/sysinfo
[doc]: https://docs.rs/sysinfo/
