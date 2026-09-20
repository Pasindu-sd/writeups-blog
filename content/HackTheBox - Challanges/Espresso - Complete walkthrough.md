
# #HTB 


![[Pasted image 20260920210240.png|700]]


# HTB Espresso Writeup

**Category:** Hardware  
**Difficulty:** Easy  
**Platform:** HackTheBox

---

## Challenge Description

> Someone leaked the new Espresso firmware, can you try to figure out what it does?

We are provided with a single file:

- `firmware.bin` (4 MiB)

The objective is to analyze the firmware, understand what it does, and extract the flag.

---

## Initial Analysis

First, check what kind of file this is:

```bash
file firmware.bin
# Output: data
```

`file` command doesn't recognize it. Manually inspect the hex:

```python
python3 -c "
with open('firmware.bin','rb') as f:
    data = f.read()

# Find first non-FF byte
i = 0
while i < len(data) and data[i] == 0xFF:
    i += 1
print('First non-FF byte at:', hex(i))
print(data[i:i+64])
"
```

**Output:**

```
First non-FF byte at: 0x1000
b'\xe9\x03\x02 D\x06\x08@\xee...v6.1-dev-2748-g490691bc6'
```

![[Pasted image 20260920212458.png]]

First `0x1000` bytes are `0xFF` — classic erased flash padding. Real content starts at `0x1000`. The magic byte `0xE9` is the ESP32 bootloader signature.

---

## Identifying the Format

Use `esptool` to parse the image:

```bash
pip install esptool
python3 -m esptool --chip esp32 image-info firmware.bin
```

**Output:**

```
Project name: espresso
App version:  2c1ec8fd-dirty
Compile time: Feb 28 2026 03:10:39
Chip ID:      0 (ESP32)
Entry point:  0x400814ac
ESP-IDF:      v6.1-dev-2748-g490691bc61
```

This is a **full ESP32 flash dump**. Parse the partition table at `0x8000`:

```python
import struct
data = open('firmware.bin','rb').read()
off = 0x8000
while True:
    entry = data[off:off+32]
    if entry[:2] != b'\xaaP': break
    magic, ptype, subtype, addr, size = struct.unpack('<HBBII', entry[:12])
    label = entry[12:28].split(b'\x00')[0].decode()
    print(hex(addr), hex(size), label)
    off += 32
```

**Output:**

```
0x9000   0x6000   nvs
0xf000   0x1000   phy_init
0x10000  0x100000 factory   ← main app
```

|Partition|Offset|Size|Purpose|
|---|---|---|---|
|nvs|0x9000|24 KB|Non-volatile storage|
|phy_init|0xF000|4 KB|WiFi PHY calibration|
|factory|0x10000|1 MB|**Main application**|

---

## Static Analysis — Strings

```bash
strings -n 6 firmware.bin | grep -v "^ESP_ERR\|IDF/components"
```

Three suspicious strings stand out, clustered together in the binary:

```
flag did not generate correctly.
It seems you are running the firmware on cloned hadware.
Buy the real hardware, or perhaps try to emulate it. ;)
```

The third string is a direct hint from the challenge author:

> **"try to emulate it"** 👈

Further string analysis reveals the internal logic:

```
get_efuse_factory_mac    ← reads unique chip MAC from eFuse BLK0
main                     ← logging TAG used for ESP_LOG
```

**Key finding:** No flag string exists anywhere in the binary. The flag is **generated at runtime** from the chip's eFuse MAC address — it only appears when the firmware detects it is running on genuine Espressif hardware.

The logic flow is:

```
Boot
 └─► read eFuse MAC address
       ├─► MAC valid (genuine chip) → generate flag → print over UART
       └─► MAC invalid (clone)      → print "cloned hardware" error
```

---

## Building Espressif QEMU

Standard QEMU does not support ESP32. Espressif maintains an official fork with full Xtensa/ESP32 machine emulation.

### Install Dependencies

```bash
sudo apt install -y libglib2.0-dev libpixman-1-dev libgcrypt-dev \
  libslirp-dev libfdt-dev zlib1g-dev libssl-dev cmake ninja-build
```

### Clone and Build

```bash
git clone https://github.com/espressif/qemu.git
cd qemu

# Note: omit --enable-debug to avoid -Werror breaking build on GCC 15
./configure --target-list=xtensa-softmmu --enable-gcrypt

make -j$(nproc)
# Build takes ~15 minutes
```

---

## Emulating the Firmware

```bash
cd ~/Downloads/hw_espresso

qemu/build/qemu-system-xtensa \
  -machine esp32 \
  -drive file=firmware.bin,if=mtd,format=raw \
  -nographic \
  -serial mon:stdio
```

![[Pasted image 20260920211716.png]]


### UART Output

```
I (3627) app_init: Project name:     espresso
I (3628) app_init: App version:      2c1ec8fd-dirty
I (3629) app_init: Compile time:     Feb 28 2026 03:10:39
I (3634) efuse_init: Min chip rev:   v0.0
I (3637) efuse_init: Chip rev:       v0.0
I (3931) main_task: Started on CPU0
I (3971) main_task: Calling app_main()
I (3991) main: HTB{**************************}
I (3991) main_task: Returned from app_main()
```

![[Pasted image 20260920211652.png]]

The QEMU ESP32 emulator satisfies the hardware genuineness check — the eFuse MAC read succeeds, the flag is generated and printed over the virtual UART.

---

## Result

```
HTB{**************************************}
```

---

## Challenge Solved 


![[Pasted image 20260920211623.png|700]]


---

## Communication Flow

```
firmware.bin
      │
      ▼
┌─────────────────────────┐
│  ESP32 Flash Layout     │
├─────────────────────────┤
│  Bootloader  @ 0x1000   │
│  Part. Table @ 0x8000   │
│  NVS         @ 0x9000   │
│  Factory App @ 0x10000  │
└─────────────────────────┘
      │
      ▼
┌─────────────────────────┐
│  app_main()             │
│  get_efuse_factory_mac()│
│  → flag generation      │
│  → UART print           │
└─────────────────────────┘
      │
      ▼
 HTB{********************************}
```

---

## Conclusion

Although this is a hardware challenge, **HTB Espresso** requires no physical hardware at all. The key insight is recognizing that:

1. The firmware performs an **eFuse-based hardware authenticity check**
2. The flag is **never stored statically** — it is derived at runtime from the chip MAC
3. The challenge hint _"try to emulate it"_ directly points to the solution

Using Espressif's official QEMU fork, the ESP32 environment is faithfully emulated — including eFuse reads — allowing the firmware to pass its hardware check and reveal the flag over the virtual serial port.

This challenge is an excellent introduction to **firmware analysis**, **embedded systems emulation**, and the ESP32 platform commonly seen in IoT hardware challenges.

---

_Written by: Pasindu (W.P.S.D. Wijesinghe)_  
_Blog: pasindu-sd.github.io/writeups-blog_