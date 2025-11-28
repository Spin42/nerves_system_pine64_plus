# Pine64+ (64-bit)

This is the base Nerves System configuration for the Pine64+ single-board computer (with WIFI/BT module).

![Pine64+ Board](assets/images/pine64-plus.jpg)
<br><sup>[Pine A64+](https://www.pine64.org/devices/single-board-computers/pine-a64/)</sup>

| Feature              | Description                      |
| -------------------- | -------------------------------- |
| CPU                  | 1.152 GHz quad-core Cortex-A53 (64-bit mode) |
| Memory               | 2 GB LPDDR3 SDRAM                |
| Storage              | MicroSD                          |
| Linux kernel         | 6.6.93 with Allwinner/SUNXI patches |
| IEx terminal         | UART `ttyS0`                     |
| GPIO, I2C, SPI       | Yes - [Elixir Circuits](https://github.com/elixir-circuits) |
| ADC                  | No                               |
| PWM                  | Yes, but no Elixir support       |
| UART                 | 1 available - `ttyS0`            |
| Display              | HDMI                             |
| Ethernet             | Yes - Gigabit Ethernet           |
| WiFi                 | Yes - RTL8723BS (onboard module) |
| Bluetooth            | Yes - RTL8723BS (onboard module) |
| Audio                | HDMI/3.5mm stereo jack           |

## Using

The most common way of using this Nerves System is create a project with `mix nerves.new` and to export `MIX_TARGET=pine64_plus`. See the [Getting started guide](https://hexdocs.pm/nerves/getting-started.html#creating-a-new-nerves-app) for more information.

If you need custom modifications to this system for your device, clone this repository and update as described in [Making custom systems](https://hexdocs.pm/nerves/customizing-systems.html).

## Supported WiFi devices

The base image includes drivers for the onboard Pine64+ WiFi module (`rtl8723bs` driver). This driver is built into the kernel and supports the Realtek RTL8723BS WiFi/Bluetooth combo chip.

To use WiFi, you'll need to configure it using [VintageNet](https://github.com/nerves-networking/vintage_net). Add `vintage_net_wifi` to your project dependencies and configure it in your application.

## Audio

The Pine64+ has multiple options for audio output, including HDMI and the 3.5mm stereo jack. The Linux ALSA drivers are used for audio output.

To configure audio output, use `amixer` commands. Here's an example configuration for the headphone jack:

```elixir
cmd("amixer cset numid=6 160")    # Volume DAC
cmd("amixer cset numid=2 160")    # Volume AIF1
cmd("amixer cset numid=7 50")     # Volume Headphone
cmd("amixer cset numid=41 1")     # DAC ON
cmd("amixer cset numid=29 1")     # AIF1 Slot 0 ON
cmd("amixer cset numid=33 1")     # Headphone ON
cmd("amixer cset numid=32 0,0")   # Routing DAC
```

Test audio output with:

```elixir
cmd("speaker-test -t sine -f 440 -c 2 -l 1")
```

## Provisioning devices

This system supports storing provisioning information in a small key-value store outside of any filesystem. Provisioning is an optional step and reasonable defaults are provided if this is missing.

Provisioning information can be queried using the Nerves.Runtime KV store's [`Nerves.Runtime.KV.get/1`](https://hexdocs.pm/nerves_runtime/Nerves.Runtime.KV.html#get/1) function.

Keys used by this system are:

| Key                    | Example Value     | Description
| :--------------------- | :---------------- | :----------
| `nerves_serial_number` | `"12345678"`      | By default, this string is used to create unique hostnames and Erlang node names. If unset, it defaults to part of the board's unique ID.

The normal procedure would be to set these keys once in manufacturing or before deployment and then leave them alone.

For example, to provision a serial number on a running device, run the following and reboot:

```elixir
iex> cmd("fw_setenv nerves_serial_number 12345678")
```

This system supports setting the serial number offline. To do this, set the `NERVES_SERIAL_NUMBER` environment variable when burning the firmware. If you're programming MicroSD cards using `fwup`, the commandline is:

```sh
sudo NERVES_SERIAL_NUMBER=12345678 fwup path_to_firmware.fw
```

Serial numbers are stored on the MicroSD card so if the MicroSD card is replaced, the serial number will need to be reprogrammed. The numbers are stored in a U-boot environment block. This is a special region that is separate from the application partition so reformatting the application partition will not lose the serial number or any other data stored in this block.

Additional key value pairs can be provisioned by overriding the default provisioning.conf file location by setting the environment variable `NERVES_PROVISIONING=/path/to/provisioning.conf`. The default provisioning.conf will set the `nerves_serial_number`, if you override the location to this file, you will be responsible for setting this yourself.

## Linux kernel configuration

The Linux kernel compiled for Nerves is a stripped down version of the default Allwinner/SUNXI Linux kernel. This is done to remove unnecessary features, select some Nerves-specific features like F2FS and SquashFS support, and to save space.
