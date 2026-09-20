
# #HTB 


![[Pasted image 20260920210240.png|700]]


# HTB Challenge — Espresso Firmware

**Category:** Hardware / Reverse Engineering  

---

## Challenge Description

> Someone leaked the new Espresso firmware, can you try to figure out what it does?

**Given file:** `firmware.bin` (4 MiB)

---

## Step 1 — Initial Triage

First, check what kind of file this is:

```bash
file firmware.bin
# Output: data
```

`file` doesn't recognize it. Check the hex dump manually:

```python
python3 -c "
with open('firmware.bin','rb') as f:
    data = f.read(512)
print(data)
"
```

First 0x1000 bytes are all `0xFF` — classic erased flash padding. The real content starts at offset `0x1000`.

---

## Step 2 — Identify the Format

Use `esptool` to inspect the image:

```bash
pip install esptool

# Extract the app partition first
python3 -c "
import struct
data = open('firmware.bin','rb').read()
# Find partition table at 0x8000
off = 0x8000
while True:
    entry = data[off:off+32]
    if entry[:2] != b'\xaaP': break
    magic, ptype, subtype, addr, size = struct.unpack('<HBBII', entry[:12])
    label = entry[12:28].split(b'\x00')[0].decode()
    print(hex(off), ptype, subtype, hex(addr), hex(size), label)
    off += 32
"
```

**Output:**

```
0x8000  1 2  0x9000   0x6000  nvs
0x8020  1 1  0xf000   0x1000  phy_init
0x8040  0 0  0x10000  0x100000 factory
```

This is a **full ESP32 flash dump** with a standard partition table. Extract the `factory` app:

```bash
python3 -m esptool --chip esp32 image-info firmware.bin
```

**Key output:**

```
Project name: espresso
App version:  2c1ec8fd-dirty
Compile time: Feb 28 2026 03:10:39
Chip ID:      0 (ESP32)
Entry point:  0x400814ac
```

---

## Step 3 — Static Analysis (Strings)

```bash
strings -n 6 firmware.bin | grep -v "^ESP_ERR\|IDF/components"
```

Three suspicious strings stand out — clustered together in the binary:

```
flag did not generate correctly.
It seems you are running the firmware on cloned hadware.
Buy the real hardware, or perhaps try to emulate it. ;)
```

The third string is a hint from the challenge author:

> **"try to emulate it"** 👈

No flag string exists anywhere in the binary. This means the **flag is generated at runtime** from the chip's hardware identity (eFuse MAC address), not stored statically.

Further string analysis reveals:

- TAG = `"main"` — the logging tag for the main function
- `get_efuse_factory_mac` — function that reads the unique chip MAC from eFuse BLK0
- Anti-clone logic: if running on genuine hardware → generate flag; otherwise → print error

---

## Step 4 — Build Espressif QEMU

Standard QEMU doesn't support ESP32. Espressif maintains a fork with ESP32 machine support.

```bash
# Install dependencies
sudo apt install -y libglib2.0-dev libpixman-1-dev libgcrypt-dev \
  libslirp-dev libfdt-dev zlib1g-dev libssl-dev cmake ninja-build

# Clone Espressif QEMU
git clone https://github.com/espressif/qemu.git
cd qemu

# Configure (no --enable-debug to avoid -Werror breaking build)
./configure --target-list=xtensa-softmmu --enable-gcrypt

# Build (~15 minutes)
make -j$(nproc)
```

---

## Step 5 — Emulate the Firmware

```bash
cd ~/Downloads/hw_espresso

qemu/build/qemu-system-xtensa \
  -machine esp32 \
  -drive file=firmware.bin,if=mtd,format=raw \
  -nographic \
  -serial mon:stdio
```

**UART Output:**

```
I (3627) app_init: Project name:     espresso
I (3628) app_init: App version:      2c1ec8fd-dirty
I (3629) app_init: Compile time:     Feb 28 2026 03:10:39
I (3637) efuse_init: Chip rev:       v0.0
I (3971) main_task: Calling app_main()
I (3991) main: HTB{3mul4ting_hw_is_s0_c00l!!!}
I (3991) main_task: Returned from app_main()
```

---

## Flag

```
HTB{3mul4ting_hw_is_s0_c00l!!!}
```

---

## Summary

|Step|Action|
|---|---|
|1|Identified file as ESP32 full flash dump|
|2|Parsed partition table — found `factory` app at `0x10000`|
|3|String analysis revealed anti-clone check + emulation hint|
|4|Built Espressif's QEMU fork with Xtensa/ESP32 support|
|5|Emulated firmware — flag printed over virtual UART|

## Key Takeaway

The firmware reads the ESP32's unique **eFuse MAC address** and derives the flag from it at runtime. The flag is never stored in the binary — it only appears when the firmware believes it's running on real (or emulated) Espressif hardware. The challenge hint _"try to emulate it"_ pointed directly to the solution: use Espressif's QEMU fork to satisfy the hardware check and obtain the flag.

---

_Written by: Pasindu (W.P.S.D. Wijesinghe)_  
_Blog: pasindu-sd.github.io/writeups-blog_