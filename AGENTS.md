```markdown
## Набор проверок (Health Stack)

- typecheck: `cargo build --workspace --lib`
- lint: `cargo clippy --workspace --all-targets -- -D warnings`
- test: `cargo test --workspace`
- shell: `shellcheck scripts/install.sh`

```
## Health Stack

- typecheck: cargo build --workspace --lib
- lint: cargo clippy --workspace --all-targets -- -D warnings
- test: cargo test --workspace
- shell: shellcheck scripts/install.sh
