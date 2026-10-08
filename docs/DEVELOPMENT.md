# J5 Patched-v07 Development Notes

English | [简体中文](DEVELOPMENT.zh-CN.md)

This document records the approach and key addresses used in J5 Patched-v07. Installation is covered in the [J5 guide](../cameras/J5/README.md). Full source, build tools and analysis records are retained separately in the internal handoff.

These addresses apply only to the verified original J5 C1.01 firmware. Other Nikon 1 models require their own analysis and cannot use these addresses directly.

## Why several separate edits were needed

Stock restrictions for mechanical lenses are spread across several functions. Allowing entry into focus magnification does not change its timer. Removing the A-mode warning does not grant capture permission. An available ISO menu option also does not establish that metering uses the setting.

Patched-v07 therefore changes magnification entry, its timer, the A-mode warning, capture eligibility and two Auto-ISO restrictions separately. Aperture remains controlled on the lens; electronic-lens information is not fabricated and the automatic exposure algorithm is not rewritten.

## Firmware container and addresses

The main image starts at decoded package offset `0x50`. Low-address code in this document uses `runtime=image offset+0x80000`; code copied into high RAM requires a separate mapping.

The first 32 container bytes remain unchanged. Subsequent bytes are XORed using three 256-byte tables:

```text
raw[i] XOR t0[i&255] XOR t1[(i>>8)&255] XOR t2[(i>>16)&255]
```

The tables come from historical `Firmware1.cs` tooling. The same transformation decodes and re-encodes the container. Image offsets and patch bytes below refer to decoded content.

## Update recognition and version display

The main-image `+0x800` header changes from `Ver.1.0100` to `Ver.1.0101`, corresponding to `30→31` at package offset `0x859`. Runtime defaults at `image+0x16490C` and `image+0xA33CF0` remain unchanged. The version screen therefore still shows C1.01, and the update entry may remain available.

Earlier A/B tests found no update entry when only the outer entry name changed, but an entry when only the main-image header changed. Static version-comparison analysis and repeated installations also support this approach.

An update entry establishes only candidate acceptance. Actual features must be checked to confirm installation. This approach does not establish that an arbitrary higher version can be downgraded to C1.01.

## Focus magnification entry

At image `0xAC71E`, runtime `0x12C71E`, `3FD0→C046` replaces the conditional branch blocking magnification in `CameraNonCPULens` with a Thumb NOP.

An early incorrect edit in `ShootSequenceAct` was reverted. Patched-v07 retains original `08D0` at image `0xAC8D6`. The on-screen OK hint is controlled by separate UI code and was not changed.

## Removing the magnification timeout

The added function `mf_timer_am.s` starts at image `0x2C0`. Code and constants occupy 228 bytes, with space reserved through `0x3AF`.

The timer hook is at image `0x4C8A78`, runtime `0x548A78`. `B1F699FF032892D1→004B1847C1020800` uses an absolute jump to Thumb entry `0x802C1`. An old-style Thumb BL cannot be used directly because the target is outside its range.

Timeout is suppressed only for timer 3, with all property reads successful and the following conditions met:

- M: `(0x13,0x2BB)=(4,3)`; A: `(0x13,0x2BB)=(3,2)`.
- `0x3EE=0`, `0x419!=5`, `0x2A2!=1`.

Other states retain original timer handling. Non-expired and expired continuations are `0x5489A6` and `0x5489F2`, respectively. Half-press and OK exit handling is unchanged.

## A-mode capture

The edit `07→41` at image `0x15A78A`, runtime `0x1DA78A`, removes the A-mode warning. This alone allowed metering but still did not allow capture.

Capture eligibility is checked separately at image `0x15FD02`, runtime `0x1DFD02`. `032803D0→A0F655FB` calls `0x803B0` with BL. The added function `a_capture_c2.s` occupies 196 bytes at image `0x3B0..0x473`.

The edit retains the original allowance for effective mode 3 and adds effective mode 2. The added branch requires the original C2 byte at entry to be zero and successful public-property reads of:

```text
0x13=3
0x3EE=0
0x419!=5
0x2A2!=1
0x2BB=2
```

Allowed capture continues at `0x1DFD0E`; otherwise execution continues at original `0x1DFD06`. The function saves and restores registers and stack without changing original exposure calculations or lens aperture data.

## A-mode Auto ISO

The original producer at runtime `0x17A5C2` calls `0x178B2C` to obtain the ISO setting, then calls `0x17A4D0` for second-stage handling before writing property `0x2C9`.

Two lens-state checks replace automatic ISO enums `2/3/4` with fixed enum `0x0F`: the first checks `0x3EE`, the second `0x3F8/0x3F9`. Patched-v07 adds conditional exceptions at both sites:

| Image offset | Runtime address | Byte change | New function entry |
|---|---|---|---|
| `0xFA5EC` | `0x17A5EC` | `022803D0→05F748FF` | `0x80480` |
| `0xFA518` | `0x17A518` | `022C03D0→05F7CAFF` | `0x804B0` |

The added function `a_auto_iso.s` occupies 256 bytes at image `0x480..0x57F`.

Exceptions apply only to automatic ISO enums `2/3/4`, with successful current staged-view reads of `0x13=3`, `0x419!=5`, `0x2A2!=1` and `0x3EE=0`. Fixed ISO, HDR and other handling remain unchanged. Successful property reads return `0xF0000000`.

Two property interfaces must not be mixed: the public object is obtained through factory `0x2CE0A9`, with its getter at `vtable+0`; the staged-view getter is at `+4`. The staged view holds the state used for this calculation and must not be replaced with potentially unsynchronized public state.

`0x0F` is an internal enum, not a direct numeric ISO value. Its conversion to actual ISO has not been independently traced in full.

## Where the added code lives

Original image `0x2C0..0x7FF` is zero-filled. The three added functions use non-overlapping parts of this space. Placement checks covered the boot entry, nearby constants, discovered copy/clear ranges and image write start. Hardware results from v3, v5 and Patched-v07 also show that the added code was loaded and executed.

Static searches cannot exhaust all indirect references. Further use of this space requires checking occupancy again, along with ARM/Thumb state, function-pointer low bits, complete instruction boundaries, registers, flags, LR, stack alignment and return addresses.

## CRC and restoration firmware

CRC uses `binascii.crc_hqx(data,0)`, stored big-endian:

| Checksum | Decoded package location | Coverage |
|---|---|---|
| Embedded CRC | `0x175A6E0..E1` | `[0x50,0x175A6E0)` |
| Outer CRC | `0x175A6E2..E3` | `[0,0x175A6E2)` |

Write the embedded CRC first, then calculate the outer CRC including it. Each appended CRC produces a zero remainder. Re-encode afterward and preserve other trailing bytes.

Restoration firmware is regenerated from stock. Only the version character at package offset `0x859` and two embedded CRC bytes change; the recalculated outer CRC happens to remain unchanged. All functional code matches stock, and restoration has been tested successfully. Use still requires a working update menu.

## Verification and porting

Offline instruction models checked 5,187 capture combinations, 79,380 timer combinations and 38,400 Auto-ISO combinations. Reports are retained in the internal handoff. Models execute relevant instructions with mocked property interfaces, not the sensor, actual automatic exposure or flash programming.

Both release files were rebuilt from the official original, checking original bytes, change scope, nested CRC and final SHA-256. All three assembly helpers were recompiled and compared. See the [J5 guide](../cameras/J5/README.md) for hardware test scope.

Other Nikon 1 models require independent checks of container, address mapping, instruction set, property interfaces, boot process, update checks and target functions. Changing the model name or reusing J5 addresses is insufficient.

| File | SHA-256 |
|---|---|
| Official original | `5fae892c396d4213ebf5a4982fa98559c6c79e00f5e6e0cbe6251084cb9b1b04` |
| Patched-v07 | `94b545bc2b1e6cc3a6599daf73b56bef972fb321da7cfe2b1588f159d6966ab2` |
| Restoration | `0fb4f4115c6f09ca30b87d84affd233ff6891559edc649ac812289a0b0aa31d6` |
