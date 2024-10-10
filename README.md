# Nexus-Beta

![ZenRock Banner](https://pbs.twimg.com/profile_banners/1508993482655866881/1723065167/1500x500)

## Links
- [Discord](https://discord.gg/tzTYqBCx)
- [Web](https://beta.nexus.xyz/)
- [Twitter](https://x.com/NexusLabsHQ/status/1800324588116860933)

## System Requirements

- **RAM**: 8 GB
- **CPU**: 4 cores
- **Disk Space**: 100 GB

## Installation ⚙️

# Auto Install

```bash
bash <(curl -s https://file.winsnip.xyz/file/uploads/Nexus-beta.sh)
```
Save prover-id
```bash
cat $HOME/.nexus/prover-id; echo ""
```

# Manual Install

```bash
# install Rush
sudo apt update -y && sudo apt upgrade -y && sudo apt install cmake -y && sudo apt install build-essential -y && curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh


# set cargo
. "$HOME/.cargo/env" && rustup target add riscv32i-unknown-none-elf && cargo install --git https://github.com/nexus-xyz/nexus-zkvm nexus-tools --tag 'v1.0.0' && cargo nexus new nexus-project && cd nexus-project && cd src && rm -rf main.rs && cat <<EOT >> main.rs
#![no_std]
#![no_main]

fn fib(n: u32) -> u32 {
    match n {
        0 => 0,
        1 => 1,
        _ => fib(n - 1) + fib(n - 2),
    }
}

#[nexus_rt::main]
fn main() {
    let n = 7;
    let result = fib(n);
    assert_eq!(result, 13);
}
EOT


# Run Program
cargo nexus run


# Prove your program
cargo nexus prove


# Verify your proof
cargo nexus verify
```

# Cek log
```bash
sudo journalctl -u nexus.service -f
```


Delete
```bash
sudo systemctl stop nexus.service && sudo systemctl disable nexus.service && sudo rm /etc/systemd/system/nexus.service && sudo systemctl daemon-reload && sudo systemctl reset-failed
```

JANGAN LUPA....!!! DOWNLOAD AND SAVE
Prover id di root/.nexus
