## Required hardware

### OrangeCrab

[Documentation](https://gregdavill.github.io/OrangeCrab/r0.2/)

[Get one](https://store.groupgets.com/collections/frontpage/products/orangecrab)

### hs-probe

[Project page](https://github.com/probe-rs/hs-probe)

[Get one](https://github.com/probe-rs/hs-probe/issues/7#issuecomment-717924548)

### USB-serial cable

### Keyboard FeatherWing

[Documentation](https://www.solder.party/docs/keyboard-featherwing/)

Get one: [tindie](https://www.tindie.com/products/arturo182/keyboard-featherwing-qwerty-keyboard-26-lcd/) or [pimoroni](https://shop.pimoroni.com/products/keyboard-featherwing-qwerty-keyboard-2-6-lcd)

## Requried software

* Git
* Python3
* [FPGA toolchain](https://github.com/open-tool-forge/fpga-toolchain/releases) ([direct link](https://github.com/open-tool-forge/fpga-toolchain/releases/download/nightly-20201214/fpga-toolchain-linux_x86_64-nightly-20201214.tar.xz)) ~100MB
* [RISC-V toolchain](https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2020.04.0-x86_64-linux-ubuntu14.tar.gz) ~220MB
* [PyCharm](https://www.jetbrains.com/pycharm/download/) ~450MB
* [LiteX](https://github.com/enjoy-digital/litex/)
* [Rust](https://www.rust-lang.org/tools/install)
* [ecpdap](https://github.com/adamgreig/ecpdap)

## Set up toolchains

```console
wget https://github.com/open-tool-forge/fpga-toolchain/releases/download/nightly-20201214/fpga-toolchain-linux_x86_64-nightly-20201214.tar.xz
tar xf fpga-toolchain-linux_x86_64-nightly-20201214.tar.xz
PATH=$PATH:$(pwd)/fpga-toolchain/bin

wget https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2020.04.0-x86_64-linux-ubuntu14.tar.gz
tar xf riscv64-unknown-elf-gcc-8.3.0-2020.04.0-x86_64-linux-ubuntu14.tar.gz
PATH=$PATH:$(pwd)/riscv64-unknown-elf-gcc-8.3.0-2020.04.0-x86_64-linux-ubuntu14/bin
```

## Install Rust

```console
rustup default stable
rustup target add riscv32i-unknown-none-elf
```

## Install ecpdap

```console
git clone https://github.com/adamgreig/ecpdap
cd ecpdap
cargo install --path .
PATH=$PATH:$HOME/.cargo/bin
```

## Set up LiteX environment

```console
mkdir workspace
cd workspace
python3 -m venv .venv
source .venv/bin/activate

wget https://raw.githubusercontent.com/enjoy-digital/litex/master/litex_setup.py
chmod +x litex_setup.py
./litex_setup.py init install
```

## Build the example

```console
cd litex-boards/litex_boards/targets/
./orangecrab.py --build
ecpdap program --freq=50000 build/orangecrab/gateware/orangecrab.bit
```

## Serial connection

```
Cable Rx (white) -- FPGA Tx (1)
Cable Tx (green) -- FPGA Rx (0)
```

## Flash boot

```python
spi_pads = platform.request("spiflash4x")
self.submodules.lxspi = spi_flash.SpiFlashDualQuad(spi_pads, dummy=6, endianness="little")
self.lxspi.add_clk_primitive(platform.device)
self.register_mem("spiflash", self.mem_map["spiflash"], self.lxspi.bus, size=16 * 1024 * 1024)

self.constants["FLASH_BOOT_ADDRESS"] = self.mem_map['spiflash'] + 0x00100000
```
или

```python
self.add_spi_flash(...)
```

+update `boot.c`
