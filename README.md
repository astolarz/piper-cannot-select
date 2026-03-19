## rustc-LLVM ERROR: Cannot select: 0x11c366ee0: i32 = Constant<4096>

The Rust compiler for the Xtensa target does not like the use of the number 4096 (aka 0x1000 aka 2<sup>12</sup>) in the piper crate. It fails to compile with the error `rustc-LLVM ERROR: Cannot select: 0x11c366ee0: i32 = Constant<4096>` for all opt-levels in the release profile, and all but opt-level 0 in the  dev profile.

This repo is an attempt at the minimal repro of the issue. The Cargo.toml file includes information on the compilation results with various opt-levels. The project may be able to be slimmed down further by removing the embuild build-dependency and esp-idf-svc dependency, but will fail to link (which may not be necessary to further investigate the compilation issue).

The source of the error is in [this snippet in lib.rs in the piper crate](https://github.com/smol-rs/piper/blob/85da45edce0b9262f00d79d9a42ce0372fb42a10/src/lib.rs#L1121-L1125). Changing 4096 to seemingly any other number results in the code compiling without issue.
```rust
pub fn write_buf(&mut self, max: usize) -> &mut [u8] {
        let n = max
            .min(self.zeroed_until * 2 + 4096) // Don't zero too many bytes when starting.
            .min(self.available_space()) // No more than space in the pipe.
            .min(self.inner.cap - self.inner.real_index(self.tail)); // Don't go past the buffer boundary.
```