# What?

When I save a file in VSCode with the rust-analyzer extension installed, it
causes a full rebuild, **_if_** I opened VSCode through a symlink.

# Steps to repro

Run the following:

```
# fetch this repo
git clone https://github.com/cstrahan-blueshift/vscode_rust_rebuild_repro.git topdir/vscode_rust_rebuild_repro

# create a symlink
ln -s $PWD/topdir/vscode_rust_rebuild_repro vscode_rust_rebuild_repro_link

# open vscode through the symlink
code vscode_rust_rebuild_repro_link

# build the project
cd vscode_rust_rebuild_repro_link
CARGO_LOG=cargo::core::compiler::fingerprint=info cargo build -p demo_bin
```

now that VSCode is open, select `crates/src/main.rs` in the Exploror widget,
give the editor focus and save (`Ctrl`+`s`).

Back in the terminal, cargo build will result in an unexpected rebuild:

```
~/src/vscode_rust_rebuild_repro_link$ CARGO_LOG=cargo::core::compiler::fingerprint=info cargo build -p demo_bin
    Blocking waiting for file lock on build directory
[2022-11-03T22:04:52Z INFO  cargo::core::compiler::fingerprint] dependency on `build_script_build` is newer than we are 1667513089.485227366s > 1667513077.757157622s "/home/cstrahan/.cargo/registry/src/github.com-1ecc6299db9ec823/onnxruntime-sys-0.0.14"
[2022-11-03T22:04:52Z INFO  cargo::core::compiler::fingerprint] fingerprint error for demo_bin v0.1.0 (/home/cstrahan/src/topdir/vscode_rust_rebuild_repro/crates/demo_bin)/Build/TargetInner { name: "demo_bin", doc: true, ..: with_path("/home/cstrahan/src/topdir/vscode_rust_rebuild_repro/crates/demo_bin/src/main.rs", Edition2021) }
[2022-11-03T22:04:52Z INFO  cargo::core::compiler::fingerprint]     err: current filesystem status shows we're outdated
[2022-11-03T22:04:52Z INFO  cargo::core::compiler::fingerprint] fingerprint error for demo_lib v0.1.0 (/home/cstrahan/src/topdir/vscode_rust_rebuild_repro/crates/demo_lib)/Build/TargetInner { ..: lib_target("demo_lib", ["lib"], "/home/cstrahan/src/topdir/vscode_rust_rebuild_repro/crates/demo_lib/src/lib.rs", Edition2021) }
[2022-11-03T22:04:52Z INFO  cargo::core::compiler::fingerprint]     err: current filesystem status shows we're outdated
[2022-11-03T22:04:52Z INFO  cargo::core::compiler::fingerprint] fingerprint error for onnxruntime v0.0.14/Build/TargetInner { ..: lib_target("onnxruntime", ["lib"], "/home/cstrahan/.cargo/registry/src/github.com-1ecc6299db9ec823/onnxruntime-0.0.14/src/lib.rs", Edition2018) }
[2022-11-03T22:04:52Z INFO  cargo::core::compiler::fingerprint]     err: current filesystem status shows we're outdated
[2022-11-03T22:04:52Z INFO  cargo::core::compiler::fingerprint] fingerprint error for onnxruntime-sys v0.0.14/Build/TargetInner { ..: lib_target("onnxruntime-sys", ["lib"], "/home/cstrahan/.cargo/registry/src/github.com-1ecc6299db9ec823/onnxruntime-sys-0.0.14/src/lib.rs", Edition2018) }
[2022-11-03T22:04:52Z INFO  cargo::core::compiler::fingerprint]     err: current filesystem status shows we're outdated
[2022-11-03T22:04:52Z INFO  cargo::core::compiler::fingerprint] fingerprint error for onnxruntime-sys v0.0.14/RunCustomBuild/TargetInner { ..: custom_build_target("build-script-build", "/home/cstrahan/.cargo/registry/src/github.com-1ecc6299db9ec823/onnxruntime-sys-0.0.14/build.rs", Edition2018) }
[2022-11-03T22:04:52Z INFO  cargo::core::compiler::fingerprint]     err: rerun-if-changed output changed: previously ["/home/cstrahan/src/vscode_rust_rebuild_repro_link/target/debug/build/onnxruntime-sys-e77e43d77d973d19/out/onnxruntime-linux-x64-1.8.1.tgz"], now ["/home/cstrahan/src/topdir/vscode_rust_rebuild_repro/target/debug/build/onnxruntime-sys-e77e43d77d973d19/out/onnxruntime-linux-x64-1.8.1.tgz"]
   Compiling onnxruntime-sys v0.0.14
   Compiling onnxruntime v0.0.14
   Compiling demo_lib v0.1.0 (/home/cstrahan/src/topdir/vscode_rust_rebuild_repro/crates/demo_lib)
   Compiling demo_bin v0.1.0 (/home/cstrahan/src/topdir/vscode_rust_rebuild_repro/crates/demo_bin)
    Finished dev [unoptimized + debuginfo] target(s) in 3.19s
```

This appears to be the important part:

```
err: rerun-if-changed output changed: previously ["/home/cstrahan/src/vscode_rust_rebuild_repro_link/target/debug/build/onnxruntime-sys-e77e43d77d973d19/out/onnxruntime-linux-x64-1.8.1.tgz"], now ["/home/cstrahan/src/topdir/vscode_rust_rebuild_repro/target/debug/build/onnxruntime-sys-e77e43d77d973d19/out/onnxruntime-linux-x64-1.8.1.tgz"]
```

A vertical layout might be easier to read, so here are the paths again:

- previously:
  /home/cstrahan/src/vscode_rust_rebuild_repro_link/target/debug/build/onnxruntime-sys-e77e43d77d973d19/out/onnxruntime-linux-x64-1.8.1.tgz
- now:
  /home/cstrahan/src/topdir/vscode_rust_rebuild_repro/target/debug/build/onnxruntime-sys-e77e43d77d973d19/out/onnxruntime-linux-x64-1.8.1.tgz
