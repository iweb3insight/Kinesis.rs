# Troubleshooting (macOS)

If you encounter the `[1] killed` error when running the Kinesis binary on macOS, it is because the file has been quarantined by Apple's Gatekeeper since it is not signed by a registered developer.

## Solution

To resolve this, you need to manually remove the quarantine attribute and ensure the binary has execution permissions.

### Steps:

1. **Remove Quarantine Attribute**:
   Replace `~/Downloads/kinesis-rs` with the actual path where you downloaded the file.
   ```bash
   xattr -d com.apple.quarantine ~/Downloads/kinesis-rs
   ```

2. **Grant Execution Permissions**:
   ```bash
   chmod +x ~/Downloads/kinesis-rs
   ```

3. **Verify**:
   ```bash
   ./kinesis-rs --version
   ```

## Note
Kinesis.rs is a security-focused tool. For maximum safety, we recommend building the binary from source using `cargo build --release` if you are on a development machine.
