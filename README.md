# ESC/POS Rust implementation

[![Build status](https://github.com/fabienbellanger/escpos-rs/actions/workflows/CI.yml/badge.svg?branch=main)](https://github.com/fabienbellanger/escpos-rs/actions/workflows/CI.yml)
[![Crates.io](https://img.shields.io/crates/v/escpos)](https://crates.io/crates/escpos)
[![Documentation](https://docs.rs/escpos/badge.svg)](https://docs.rs/escpos)

This crate implements a subset of Epson's ESC/POS protocol for thermal receipt printers.
It allows you to generate and print documents with basic text formatting, cutting, barcodes, QR codes and raster images
on a compatible printer.

![Ticket](resources/documentation/ticket.jpg)  
*Printed on Aures ODP 333*

This project is strongly inspired by [recibo](https://github.com/jamhall/recibo/tree/main) _(
Rust)_, [escposify](https://github.com/local-group/rust-escposify) _(Rust)_
and [escpos](https://github.com/hennedo/escpos) _(Go)_.

## Installation

For standard functionalities (e.g. printing text), no additional dependencies are required:

```toml
[dependencies]
escpos = "0.1.0"
```

If you need all [features](#Features-list), you can use the `full` feature:

```toml
[dependencies]
escpos = { version = "0.1.0", features = ["full"] }
```

Or you can use `cargo add` command:

```bash
cargo add escpos
cargo add escpos -F barcode qrcode graphics
cargo add escpos -F full
```

## Features list

| Name       | Description                                                        |
|------------|--------------------------------------------------------------------|
| `barcode`  | Print barcodes (UPC-A, UPC-E, EAN8, EAN13, CODE39, ITF or CODABAR) |
| `qrcode`   | Print QR codes                                                     |
| `graphics` | Print raster images                                                |
| `full`     | Enable all features                                                |

## Examples

The examples folder contains various examples of how to use `escpos`.
The [docs](https://docs.rs/escpos) (will) also provide lots of code snippets and examples.

To launch an example, use the following command:

```bash
cargo run --example full --features "full"
```

### Simple text formatting

```rust
use escpos::printer::Printer;
use escpos::utils::{protocol::Protocol, *};
use escpos::{driver::*, errors::Result};

fn main() -> Result<()> {
    env_logger::init();

    let driver = NetworkDriver::open("192.168.1.248", 9100)?;
    Printer::new(driver, Protocol::default())
        .debug_mode(Some(DebugMode::Dec))
        .init()?
        .smoothing(true)?
        .bold(true)?
        .underline(UnderlineMode::Single)?
        .writeln("Bold underline")?
        .justify(JustifyMode::CENTER)?
        .reverse(true)?
        .bold(false)?
        .writeln("Hello world - Reverse")?
        .feed()?
        .justify(JustifyMode::RIGHT)?
        .reverse(false)?
        .underline(UnderlineMode::None)?
        .size(2, 3)?
        .writeln("Hello world - Normal")?
        .print_cut()?;

    Ok(())
}
```

### EAN13 (with `barcode` feature enabled)

```rust
use escpos::printer::Printer;
use escpos::utils::{protocol::Protocol, *};
use escpos::{driver::*, errors::Result};

fn main() -> Result<()> {
    env_logger::init();

    let driver = ConsoleDriver::open();
    Printer::new(driver, Protocol::default())
        .debug_mode(Some(DebugMode::Hex))
        .init()?
        .ean13_option(
            "1234567890265",
            BarcodeOption::new("M", "S", "A", BarcodePosition::Below),
        )?
        .feed()?
        .print_cut()?;

    Ok(())
}
```

### QR Code (with `qrcode` feature enabled)

```rust
use escpos::printer::Printer;
use escpos::utils::{protocol::Protocol, *};
use escpos::{driver::*, errors::Result};

fn main() -> Result<()> {
    env_logger::init();

    let driver = ConsoleDriver::open();
    Printer::new(driver, Protocol::default())
        .debug_mode(Some(DebugMode::Hex))
        .init()?
        .qrcode_option(
            "https://www.google.com",
            QRCodeOption {
                model: QRCodeModel::Model1,
                size: 6,
                correction_level: QRCodeCorrectionLevel::M,
            },
        )?
        .feed()?
        .print_cut()?;

    Ok(())
}
```

### Bit image (with `graphics` feature enabled)

```rust
use escpos::printer::Printer;
use escpos::utils::{protocol::Protocol, *};
use escpos::{driver::*, errors::Result};

fn main() -> Result<()> {
    env_logger::init();

    let driver = ConsoleDriver::open();
    Printer::new(driver, Protocol::default())
        .debug_mode(Some(DebugMode::Hex))
        .init()?
        .bit_image_option(
            "./resources/images/rust-logo-small.png",
            BitImageOption::new(Some(128), None, BitImageSize::Normal)?,
        )?
        .feed()?
        .print_cut()?;

    Ok(())
}
```

## Commands list

| Status | Command                | Description                                       | Feature    |
|:------:|------------------------|---------------------------------------------------|------------|
|   ✅    | `init()`               | Initialize printer (`ESC @`)                      |            |
|   ✅    | `print()`              | Print document                                    |            |
|   ✅    | `reset()`              | Hardware reset (`ESC ? LF 0`)                     |            |
|   ✅    | `cut()`                | Paper cut (`GS V A 0`)                            |            |
|   ✅    | `partial_cut()`        | Partial paper cut (`GS V A 1`)                    |            |
|   ✅    | `print_cut()`          | Print and paper cut                               |            |
|   ✅    | `bold()`               | Text bold (`ESC E`)                               |            |
|   ✅    | `underline()`          | Text underline (`ESC -`)                          |            |
|   ✅    | `double_strike()`      | Text double strike (`ESC G`)                      |            |
|   ✅    | `font()`               | Text font (`ESC M`)                               |            |
|   ✅    | `flip()`               | Text flip (`ESC V`)                               |            |
|   ✅    | `justify()`            | Text justify (`ESC a`)                            |            |
|   ✅    | `reserve()`            | Text reserve color (`GS B`)                       |            |
|   ✅    | `size()`               | Text size (`GS !`)                                |            |
|   ✅    | `reset_size()`         | Reset text size (`GS !`)                          |            |
|   ✅    | `smoothing()`          | Smoothing mode (`GS b`)                           |            |
|   ✅    | `feed()`               | Line feed (`ESC d`)                               |            |
|   ✅    | `feeds()`              | Multiple lines feed (`ESC d`)                     |            |
|   ✅    | `line_spacing()`       | Line spacing (`ESC 3`)                            |            |
|   ✅    | `reset_line_spacing()` | Reset line spacing (`ESC 2`)                      |            |
|   ✅    | `upside_down()`        | Upside-down mode (`ESC {`)                        |            |
|   ✅    | `cash_drawer()`        | Generate pulse (`ESC p`)                          |            |
|   ✅    | `write()`              | Write text                                        |            |
|   ✅    | `writeln()`            | Write text and line feed                          |            |
|   ✅    | `motion_units()`       | Set horizontal and vertical motion units (`GS P`) |            |
|   ✅    | `ean13()`              | Print EAN13 with default option                   | `barcode`  |
|   ✅    | `ean13_option()`       | Print EAN13 with custom option                    | `barcode`  |
|   ✅    | `ean8()`               | Print EAN8 with default option                    | `barcode`  |
|   ✅    | `ean8_option()`        | Print EAN8 with custom option                     | `barcode`  |
|   ✅    | `upca()`               | Print UPC-A with default option                   | `barcode`  |
|   ✅    | `upca_option()`        | Print UPC-A with custom option                    | `barcode`  |
|   ✅    | `upce()`               | Print UPC-E with default option                   | `barcode`  |
|   ✅    | `upce_option()`        | Print UPC-E with custom option                    | `barcode`  |
|   ✅    | `code39()`             | Print CODE 39 with default option                 | `barcode`  |
|   ✅    | `code39_option()`      | Print CODE 39 with custom option                  | `barcode`  |
|   ✅    | `codabar()`            | Print CODABAR with default option                 | `barcode`  |
|   ✅    | `codabar_option()`     | Print CODABAR with custom option                  | `barcode`  |
|   ✅    | `itf()`                | Print ITF with default option                     | `barcode`  |
|   ✅    | `itf_option()`         | Print ITF with custom option                      | `barcode`  |
|   ✅    | `qrcode()`             | Print QR code with default option                 | `qrcode`   |
|   ✅    | `qrcode_option()`      | Print QR code with custom option                  | `qrcode`   |
|   ✅    | `bit_image()`          | Print raster bit image with default option        | `graphics` |
|   ✅    | `bit_image_option()`   | Print raster bit image with custom option         | `graphics` |
|   🚧   | `graphic()`            | Print raster graphic with default option          | `graphics` |
|   🚧   | `graphic_option()`     | Print raster graphic with custom option           | `graphics` |
|   ❌    | `?`                    | Print PDF147                                      | `?`        |
|   ❌    | `?`                    | Print GS1                                         | `?`        |
|   ❌    | `?`                    | Print DataMatrix                                  | `?`        |
|   ❌    | `?`                    | Print MaxiCode                                    | `?`        |

- ✅: Ok
- 🚧: In progress
- ❌: Todo

## External resources

- [Epson documentation](https://download4.epson.biz/sec_pubs/pos/reference_en/escpos/ref_escpos_en/introduction.html)

## Todo

- [x] Complete `README.md`
- [ ] Complete documentation
- [ ] Add examples
- [ ] Add other graphic commands (Ex.: `GS 8 L`)
