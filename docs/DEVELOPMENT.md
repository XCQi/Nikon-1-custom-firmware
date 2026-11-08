# J5 patched v11 Development Notes

English | [简体中文](DEVELOPMENT.zh-CN.md)

This document records the approach and key addresses used in J5 patched v11, including earlier lens/interval features, button assignments, MF magnification controls and menu grouping. Installation is covered in the [J5 guide](../cameras/J5/README.md). Full source, build tools and analysis records are retained separately in the internal handoff.

These addresses apply only to the verified original J5 C1.01 firmware. Other Nikon 1 models require their own analysis and cannot use these addresses directly.

## v11: Menu grouping, controls and MF capture

The two MF captions are localized for all 32 firmware languages. A build-time native-character-index LZ stream stores all 64 captions in 1,068 bytes. The menu clone allocation is 12,026 bytes: its original 9,464-byte layout is unchanged, followed by a decoded UTF-16 text cache. The cache is populated before publishing the clone pointer. Each caption is checked through the actual native decoder and length/metadata paths, with an end-of-allocation canary; native HIF coverage is checked for every character. No new code/text arenas are used. Internal diagnostic stores were removed, without changing capture admission or request decisions. Non-English typography still needs feedback from actual cameras.

The new settings are grouped under a cloned native four-row container (node 329) at the end of Setup. Setup has 19 rows, preserving all 18 native entries, including Firmware version. The container uses its own copy of the native vtable with the generic row renderer; this avoids the Display page's brightness-specific summary behavior. Child nodes 325–328 represent Wi-Fi, Delete, half-press cancellation and MF hold. Header drawing substitutes the text argument based on node ownership without overwriting the native style descriptor.

Wi-Fi/Delete remapping is restricted to the shooting handler at `0x3C3716`. Their assignments use native properties `0xAC`/`0xAD`. The two MF settings use virtual keys `0x442`/`0x443`, backed by reserved configuration bytes `0x80B5C443`/`0x80B5C442`; navigation key `0x444` uses `0x80B5C441`. Writes use the native configuration dirty mechanism; reset defaults are cancellation On and hold Off. The hold switch gates timer-3 expiry at `0x548A78`, leaving the native timer duration unchanged.

With cancellation Off, the MF UI retains half-press/release events and records a logical full press. In the native SYS S1/S2 callback at `0x12B302`, an eligible rising S2 request is submitted once through the original touch-shutter flow at `0x12F91A`. That flow ends MF, waits for completion, and uses native capture requests. The original physical submission branch is skipped for that notification, while native input-cache updates at `0x1876B4` remain. Only the touch gate's S1-held rejection is bypassed during this submission; shared S1 is not modified and other qualifications stay native. This implements a committed single-shot request rather than post-shot magnification restoration. Other input conditions replay the original physical handler.

The release is based on hardware-confirmed custom-menu test15. Diagnostic captions were removed for publication; instruction-level checks and container/CRC/XOR checks were rerun. These checks model the lower RTOS, sensor and storage effects and do not replace camera testing. Regional language visibility edits are included, but Japanese-market hardware remains untested. Retired debug-code/text arenas are fixed and audited; this does not establish free space or compatibility on another body.

## Why several separate edits were needed

Stock restrictions for mechanical lenses are spread across several functions. Allowing entry into focus magnification does not change its timer. Removing the A-mode warning does not grant capture permission. An available ISO menu option also does not establish that metering uses the setting.

patched v07 therefore changes magnification entry, its timer, the A-mode warning, capture eligibility and two Auto-ISO restrictions separately. Aperture remains controlled on the lens; electronic-lens information is not fabricated and the automatic exposure algorithm is not rewritten.

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

An early incorrect edit in `ShootSequenceAct` was reverted. patched v07 retains original `08D0` at image `0xAC8D6`. The on-screen OK hint is controlled by separate UI code and was not changed.

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

## M/A Auto ISO

The original producer at runtime `0x17A5C2` calls `0x178B2C` to obtain the ISO setting, then calls `0x17A4D0` for second-stage handling before writing property `0x2C9`.

Two lens-state checks replace automatic ISO enums `2/3/4` with fixed enum `0x0F`: the first checks `0x3EE`, the second `0x3F8/0x3F9`. patched v07 adds conditional exceptions at both sites:

| Image offset | Runtime address | Byte change | New function entry |
|---|---|---|---|
| `0xFA5EC` | `0x17A5EC` | `022803D0→05F748FF` | `0x80480` |
| `0xFA518` | `0x17A518` | `022C03D0→05F7CAFF` | `0x804B0` |

In v07, `a_auto_iso.s` occupied 256 bytes at image `0x480..0x57F`. v08 uses `am_auto_iso.s`, 260 bytes at image `0x480..0x583`. The two hook addresses and entries remain unchanged; the additional four bytes were zero in both stock and v07.

Exceptions apply only to automatic ISO enums `2/3/4`, with successful current staged-view reads of `0x13=3` (A) or `0x13=4` (M), `0x419!=5`, `0x2A2!=1` and `0x3EE=0`. v08 extends only this shared mode check from A to A/M; it does not rewrite automatic exposure or mix in the 4K experiments. Fixed ISO, HDR and other handling remain unchanged. Successful property reads return `0xF0000000`.

Two property interfaces must not be mixed: the public object is obtained through factory `0x2CE0A9`, with its getter at `vtable+0`; the staged-view getter is at `+4`. The staged view holds the state used for this calculation and must not be replaced with potentially unsynchronized public state.

`0x0F` is an internal enum, not a direct numeric ISO value. Its conversion to actual ISO has not been independently traced in full.

## v09: Partial FT1 AF-area unlock

Two edits retain the selected single-point position and allow point-selection input:

| Image offset | Runtime address | Bytes | Effect |
|---|---|---|---|
| `0x12BA5E` | `0x1ABA5E` | `27D0→C046` | Retains the selected point, skipping the lens-type branch that forces the center |
| `0x12C1F4` | `0x1AC1F4` | `02D0→C046` | Removes the final lens-type rejection in point-selection eligibility |

Busy, no-lens, MF and other eligibility checks remain. A J5 with FT1 and a Nikon 16–85mm has focused and saved photographs at some non-central points. The touch entry also changes `1FD0` to `C046` at image `0x12E50E`, runtime `0x1AE50E`, allowing FT1 to continue through native coordinate updates. Hardware feedback confirmed touch selection works, but an out-of-range focus failure could leave the interface stuck.

This revision adds a 548-byte Thumb helper at image `[0x584,0x7A8)`, runtime `[0x80584,0x807A8)`. It uses the native 105-point map at `0x2D0770` to constrain FT1 single-point AF positions:

| Image offset | Runtime address | Wrapped operation |
|---|---|---|
| `0x12E2F0` | `0x1AE2F0` | Touch AF coordinates: run the native conversion first, then snap out-of-range points to the nearest point |
| `0x12C4E6` | `0x1AC4E6` | Directional calculation: retain the original point if the candidate is outside the set |
| `0x12C6C2` | `0x1AC6C2` | Step calculation: skip out-of-range points in the original direction |
| `0x12BBF2`, `0x12EAA2`, `0x12EB88`, `0x12EC72` | `0x1ABBF2`, `0x1AEAA2`, `0x1AEB88`, `0x1AEC72` | Initialization/refresh: correct the working point from saved state, without extra setting writes or notifications |

The helper preserves registers and the calling convention. It does not globally replace the coordinate converter shared with MF magnification. The underlying 105-point map, autofocus algorithm, lens protocol, focus-result notifications and cancellation notifications remain unchanged. Native lenses and MF do not use these bounds.

The 105-point set is not a filled rectangle. The manual calls the fixed on-screen outline the “AF-area brackets”; its footnote only describes visibility in the detailed display. Its geometry has not been matched to the 105-point map, and the reason FT1 cannot complete focus outside the frame remains unresolved. This build does not claim full-frame AF. Auto-area, face-priority and tracking modes remain unchanged.

Offline instruction models of the final helper, native selection callers and coordinate conversion passed 462 touch cases, 8 native-lens touch cases, 924 directional cases, 420 step cases, 512 initialization cases and 46 guard cases. The models do not execute actual autofocus, lens communication or focus-failure exit. Hardware feedback confirms out-of-range touch snapping. Low-contrast touch-shutter focus can still keep the controls unavailable; see the J5 guide.

## v09: One-second interval shooting

Three menu edits lower the minimum:

| Image offset | Runtime address | Bytes |
|---|---|---|
| `0x3D8D7C` | `0x458D7C` | `0520→0120` |
| `0x3D8DA8` | `0x458DA8` | `0528→0128` |
| `0x3D8DAC` | `0x458DAC` | `0520→0120` |

With only the menu edits, a preparation event at a 1- or 2-second interval can arrive while the previous capture is still busy and be skipped; the capture tick still consumes one count. A 22-byte logging slot at image `0x102856..0x10286B`, runtime `0x182856..0x18286B`, is therefore reused for these Thumb instructions:

```text
A8688321890040184088032802D2770800D101270024
```

For 1- and 2-second settings, the preparation-timer argument becomes `max(1 ms, still_delay/2)`. Settings of three seconds or longer retain the native argument. The capture timer, busy protection, AF eligibility, attempt counter and card-writing driver are unchanged. Standalone test02 passed short-exposure RAW tests of 20 photographs each at one and two seconds, but longer sequences or focusing time may still affect the saved count.

## v09: FT1 lens-restriction unlock preview

One byte changes from `01` to `00` at image `0x25966E`, runtime `0x2D966E`, changing `movs r0,#1` to `movs r0,#0`. This leaves the isolated restriction flag at state `+0x50` clear after a lens-list match. Entry conditions, list traversal, hardware errors and communication-error handling remain. Lens identity is not fabricated, and no lens protocol or AF capability is added.

A forum tester reported MF photography with the Nikon 200–500mm, with VR and electronic aperture stop-down during capture working; AF remains unavailable. The Sigma 105mm f/2.8 OS lens error remains unresolved. Only this feature is labeled a preview in the release.

Relative to v08, there are 16 added edit ranges totaling 611 bytes, including the 548-byte selection helper. The ranges do not overlap. The combined file is rebuilt from the official input and complete edit list, with CRCs recalculated together. Reversing the additions reproduces public v08 exactly; reversing all edits reproduces stock exactly. This v09 file is byte-identical to bounds candidate touch-test02, with its filename changed to the release convention. Hardware feedback confirms point snapping and capture at some points; all 105 points and a complete integrated regression have not been covered.

## v10: Independent Wi-Fi/Fn2 assignment

The physical Wi-Fi button decodes to logical key 0x12. At runtime 0x3B106A, the new handler sends native Fn2 UI key 0x31 for configured function values, or native Wi-Fi key 0x39 for value 0. It does not send Fn1 events. Native action 0x3EF3FA already supports separate Fn slots: Fn1 uses property 0xA4 and Fn2 uses 0xAC. The persisted Fn2 field is 0x80B5C5E4; its native default is 4 (ISO). The AC registry maximum at 0xAB54A4 changes from 7 to 8 to admit focus mode.

A lazy hook at 0x3F09B8 copies the menu descriptors to an 8,716-byte heap allocation, appends node 325, and inserts a separate Setup row. The original Fn row retains selection token 0x25; Fn2 uses 0x24. Fn2 binds to AC. Seven native Fn options are copied and Wi-Fi is added as value 0. Fn2 uses the complete native 13-widget ISO page, with only its refresh callback changed to generic Fn handling; the original Fn page remains unchanged. This avoids dynamically appending a widget to the seven-row Fn page. The reused ISO heading remains a known cosmetic limitation.

Three added label IDs (0x3A5–0x3A7) provide Fn2 row/title and a body-font Wi-Fi label across all 32 languages. Five resource lookup hooks handle them without changing the original label count or existing strings. Shared small bitmap 0x94 is replaced with a 32×28 grayscale dot-and-three-arcs glyph; its descriptor length is updated. Other native uses of that small bitmap also receive the new icon.

The 2,766 bytes of helpers and labels occupy audited, unregistered CIPB debug command ranges at 0x11F410–0x11FFF0, with live neighboring code excluded and retired command pointers redirected. No image extension or guessed resource padding is used. Allocation failure keeps the original menu. This release is byte-identical to Fn2 menu test06; only the distribution filename changes.

Independent settings, persistence, Wi-Fi restoration and the earlier display corrections were confirmed on hardware. Final font/icon adjustments have offline verification only. A reported louder FT1+16–85mm focus-motor sound when half-pressing out of the menu has not been compared against stock or attributed to this patch. Native constructor/population models use mocked camera services; they do not prove real-device rendering or all key contexts.

## Where the added code lives

Original image `0x2C0..0x7FF` is zero-filled. The three helpers retained from v08 and the new selection helper at `[0x584,0x7A8)` occupy non-overlapping parts of it. Placement checks covered the boot entry, nearby constants, discovered copy/clear ranges and image write start. Hardware results from v3, v5 and patched v07 also show that the added code was loaded and executed.

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

Offline instruction models checked 5,187 capture combinations, 79,380 timer combinations and 38,400 Auto-ISO combinations for v07. The v08 Auto-ISO model checked 51,200 combinations, including M mode and failed property reads. Reports are retained in the internal handoff. Models execute relevant instructions with mocked property interfaces, not the sensor, actual automatic exposure or flash programming.

The v07 release and restoration files were rebuilt from the official original, checking original bytes, change scope, nested CRC and final SHA-256; all three v07 assembly helpers were recompiled and compared. v08 was also built from the official input with a complete patch list and a recompiled Auto-ISO helper. Its decoded bytes outside the Auto-ISO helper and CRC match v07. The user confirmed v08 works on the camera, including the requested M-mode Auto ISO. See the [J5 guide](../cameras/J5/README.md) for hardware test scope.

Other Nikon 1 models require independent checks of container, address mapping, instruction set, property interfaces, boot process, update checks and target functions. Changing the model name or reusing J5 addresses is insufficient.

| File | SHA-256 |
|---|---|
| Official original | `5fae892c396d4213ebf5a4982fa98559c6c79e00f5e6e0cbe6251084cb9b1b04` |
| patched v08 | `f181a5e089dffbd07e58d4b0b79a952ffdb1dfa87a25ade648ef116e6523688c` |
| patched v09 | `f1ba918e560cbbc2c0802701e3991bca698bbf8c1a0e1d0d9b9de1cd96b58a4b` |
| patched v07 | `94b545bc2b1e6cc3a6599daf73b56bef972fb321da7cfe2b1588f159d6966ab2` |
| Restoration | `0fb4f4115c6f09ca30b87d84affd233ff6891559edc649ac812289a0b0aa31d6` |
| patched v10 | `255fe4c3cf4b99cf3b0e5ee86e1bcfd854d693b08872b32ea679c2f10f7f8a15` |
