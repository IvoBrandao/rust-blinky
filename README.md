# Rust Blinky

This project contains the setup of rust tools for embedded systems and a first example for the board STM32 Nucleo-h743zi.


## setup

To start using this project we first need to install the rust tools

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

After the installation process is done, we can check it with:

```shell
rustc -v
```

The output should be something like this:

```shell
rustc 1.65.0 (897e37553 2022-11-02)
```

Then we have to install the toolchains for microcontrollers, to do that we can execute the following commands:

```shell
rustup target add thumbv6m-none-eabi
rustup target add thumbv7m-none-eabi
rustup target add thumbv7em-none-eabi
rustup target add thumbv7em-none-eabih
```

For the Cortex-M0, M0+, and M1 (ARMv6-M architecture), Cortex-M3 (ARMv7-M architecture), Cortex-M4 and M7 without hardware floating point (ARMv7E-M architecture) and Cortex-M4F and M7F with hardware floating point (ARMv7E-M architecture) respectively. You can choose only one you are going to use or install all.

Next we need to install the binary utilites

```shell
cargo install cargo-binutils
```

After that the LLVM compiler

```shell
rustup component add llvm-tools-preview
```

after that we can install also the itmdump tool (Intelligent Trace Macrocell  dump tool):

```shell
cargo install itm
```

We will need gdb (for flashing firmware into the microcontroller and debugging), openocd (to connect to the microcontroller), screen to interact through an UART port (though official guide recommends minicom), and qemu as an emulator. Let's install it:


**FOR Ubuntu**

```shell
sudo apt install gdb-arm-none-eabi openocd screen qemu-system-arm

```

**For MacOS**


```shell
brew install openocd
brew install quemu
brew install screen

```


**For Windows**

TBD


To have an opportunity to create a project from a template, we have to install a crate called cargo-generate:

```shell
cargo install cargo-generate
```

Get to the folder where you are going to store your projects, start the terminal and run the command:

```shell
cargo generate --git https://github.com/rust-embedded/cortex-m-quickstart
```

During the process you will be asked to provide the project name. Choose the one you like, and we will use the name app here, just as in the official manual.

The project based on the https://github.com/rust-embedded/cortex-m-quickstart repository would be generated, thanks to the cortex team.

Now lets run the generated application:

open the file config in the .cargo directory and uncomment the target line that corresponds your microcontroller (I use `target = "thumbv7em-none-eabihf"` here as my NUCLEO board holds STM32H743ZI MCU, which belongs to the Cortex-M7 family).


Now we have to tune the project so it describes our MCU.

Go to the memory.x file and change the lines

```linker
MEMORY
{
  /* FLASH and RAM are mandatory memory regions */
  FLASH  : ORIGIN = 0x08000000, LENGTH = 1024K
  FLASH1 : ORIGIN = 0x08100000, LENGTH = 1024K
  RAM    : ORIGIN = 0x20000000, LENGTH = 128K

  /* AXISRAM */
  AXISRAM : ORIGIN = 0x24000000, LENGTH = 512K

  /* SRAM */
  SRAM1 : ORIGIN = 0x30000000, LENGTH = 128K
  SRAM2 : ORIGIN = 0x30020000, LENGTH = 128K
  SRAM3 : ORIGIN = 0x30040000, LENGTH = 32K
  SRAM4 : ORIGIN = 0x38000000, LENGTH = 64K

  /* Backup SRAM */
  BSRAM : ORIGIN = 0x38800000, LENGTH = 4K

  /* Instruction TCM */
  ITCM  : ORIGIN = 0x00000000, LENGTH = 64K
}

/* The location of the stack can be overridden using the
   `_stack_start` symbol.  Place the stack at the end of RAM */
_stack_start = ORIGIN(RAM) + LENGTH(RAM);

/* The location of the .text section can be overridden using the
   `_stext` symbol.  By default it will place after .vector_table */
/* _stext = ORIGIN(FLASH) + 0x40c; */

```

And finally we can edit the file hello.rs in the examples folder, commenting out the line:

```rust
debug::exit(debug::EXIT_SUCCESS);
```

# Build It  

open the terminal and run 

```shell
cargo build --example hello
```

the output will be similar to this:

```shell

Compiling semver-parser v0.7.0
   Compiling typenum v1.11.2
   Compiling proc-macro2 v1.0.6
   Compiling unicode-xid v0.2.0
   Compiling syn v1.0.11
   Compiling stable_deref_trait v1.1.1
   Compiling vcell v0.1.2
   Compiling cortex-m v0.6.1
   Compiling cortex-m-rt v0.6.11
   Compiling cortex-m-semihosting v0.3.5
   Compiling r0 v0.2.2
   Compiling app v0.1.0 (/home/aleksandr/Projects/Rust/app)
   Compiling panic-halt v0.2.0
   Compiling semver v0.9.0
   Compiling volatile-register v0.2.0
   Compiling rustc_version v0.2.3
   Compiling quote v1.0.2
   Compiling bare-metal v0.2.5
   Compiling generic-array v0.13.2
   Compiling generic-array v0.12.3
   Compiling as-slice v0.1.2
   Compiling aligned v0.3.2
   Compiling cortex-m-rt-macros v0.1.7
warning: unused import: `debug`
 --> examples/hello.rs:9:28
  |
9 | use cortex_m_semihosting::{debug, hprintln};
  |                            ^^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

    Finished dev [unoptimized + debuginfo] target(s) in 35.32s

```
The warning, it's quite ok. If you want it to disappear, you can remove the debug element from the

```rust
use cortex_m_semihosting::{debug, hprintln};
```

line would look like this:

```rust
use cortex_m_semihosting::hprintln;
```

# Flash it

First lets check for if we can access the board :

```shell
openocd -f interface/stlink-v2-1.cfg -f target/stm32h7x.cfg
```
the output should look something like this:

```shell
$ openocd -f interface/stlink-v2-1.cfg -f target/stm32h7x.cfg
Open On-Chip Debugger 0.11.0
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
WARNING: interface/stlink-v2-1.cfg is deprecated, please switch to interface/stlink.cfg
Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : clock speed 1800 kHz
Info : STLINK V2J40M27 (API v2) VID:PID 0483:374B
Info : Target voltage: 3.265522
Info : stm32h7x.cpu0: hardware has 8 breakpoints, 4 watchpoints
Info : starting gdb server for stm32h7x.cpu0 on 3333
Info : Listening on port 3333 for gdb connections
```

The contents may not match exactly but it should get the last line about breakpoints and watchpoints.

> Note: if it does not work check the following page [Rust Embedded book](https://doc.rust-lang.org/beta/embedded-book/intro/install/verify.html)

Open the file `openocd.cfg` and change the lines to the following:

```shell
source [find interface/stlink-v2-1.cfg]

source [find target/stm32h7x.cfg]
```

Now return to the `.cargo/config` file, and uncomment the line

```toml
runner = "arm-none-eabi-gdb -q -x openocd.gdb"
```
Note that is for my system, macos.

In the build section of the `.cargo/config` file uncoment the follwoing line:

```toml
target = "thumbv7em-none-eabihf"     # Cortex-M4F and Cortex-M7F (with FPU)
```

That would allow flashing and running the microcontroller with a single cargo run command.

# Run It

Go to the project directory, and open two terminals windows. 
In the first one run:

```shell
openocd
```

it should show something like this:

```shell
$ openocd
Open On-Chip Debugger 0.11.0
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
WARNING: interface/stlink-v2-1.cfg is deprecated, please switch to interface/stlink.cfg
Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : clock speed 1800 kHz
Info : STLINK V2J40M27 (API v2) VID:PID 0483:374B
Info : Target voltage: 3.265522
Info : stm32h7x.cpu0: hardware has 8 breakpoints, 4 watchpoints
Info : starting gdb server for stm32h7x.cpu0 on 3333
Info : Listening on port 3333 for gdb connections
```



In the second one run:

```shell
arm-none-eabi-gdb -q target/thumbv7em-none-eabihf/debug/examples/hello

```
it should show something like this:

```shell
$ arm-none-eabi-gdb -q target/thumbv7em-none-eabihf/debug/examples/hello
Reading symbols from target/thumbv7em-none-eabihf/debug/examples/hello...
```

Next we have to connect gbd to the open ocd server, execute the follwoing to do that:

```shell
target remote :3333
```

this command should produce a output similar to this:

```shell
Remote debugging using :3333
0x080052be in ?? ()
```

In the open OCD terminal a output similar to this one should apear

```shell
Info : accepting 'gdb' connection on tcp/3333
target halted due to debug-request, current mode: Thread
xPSR: 0x81000000 pc: 0x080052be psp: 0x20009c60
Info : Device: STM32H74x/75x
Info : flash size probed value 2048
Info : STM32H7 flash has dual banks
Info : Bank (0) size is 1024 kb, base address is 0x08000000
Warn : Prefer GDB command "target extended-remote 3333" instead of "target remote 3333"
```
Next, execute the following command in the GBD terminal:

```shell
monitor reset halt
```

command in the gdb window. You'll see the message

```shell
target state: halted
target halted due to debug-request, current mode: Thread
xPSR: 0xc1000000 pc: 0x08001060 msp: 0x20002000
```

Now you can flash your newly created program into the microcontroller by printing

```shell
load
```
in the gdb window.

Now you have to see something like that:

```shell
Loading section .vector_table, size 0xc0 lma 0x8000000
Loading section .text, size 0x1414 lma 0x80000c0
Loading section .rodata, size 0x5c0 lma 0x80014e0
Start address 0x8001060, load size 6804
Transfer rate: 14 KB/sec, 2268 bytes/write.
in the gdb window.
```

Now enter the next command to enable the semihosting

```shell
monitor arm semihosting enable
```

and now (finally!) you can run the program by printing the command

```shell
continue
```

in the gdb window.

Immediately you'll see the message

```shell
Hello, world!
```

in the openocd window.

