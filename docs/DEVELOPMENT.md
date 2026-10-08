# Nikon 1 custom firmware development: J5 v7

English | [简体中文](DEVELOPMENT.zh-CN.md)

## Scope and reproduction

This document is for developers. Users can download the BIN and follow the installation guide. Patches apply only to the exact J5 C1.01 input, not other Nikon 1 bodies. This document publishes the implementation approach and key addresses; full source, build tools and detailed analysis material are retained in the internal handoff.

## Container and addresses

The main image starts at decoded package offset 0x50. Low-address code in this document uses runtime=image offset+0x80000; copied high-RAM sections require separate mapping. The first 32 package bytes remain unchanged. Subsequent bytes use three 256-byte XOR tables: raw[i] XOR t0[i&255] XOR t1[(i>>8)&255] XOR t2[(i>>16)&255]. The same transform re-encodes the package; the three tables come from historical Firmware1.cs tooling. Patch bytes refer to decoded content.

## Update recognition and runtime version

The main-image +0x800 header changes from Ver.1.0100 to Ver.1.0101: package byte 0x859 changes 30→31. Runtime defaults at image+0x16490C and +0xA33CF0 remain unchanged. A/B probes found no update entry when only the outer entry name changed, but an entry when only the image header changed. Static comparison and repeated updates support this finer candidate-version scheme. The version screen still shows C1.01 and the update entry may remain. An entry proves only precheck acceptance; feature and restoration tests provide separate installation evidence. This is not a general downgrade method.

## MF magnification entry

At image 0xAC71E / runtime 0x12C71E, 3FD0→C046 replaces a branch with a Thumb NOP in CameraNonCPULens. The early incorrect ShootSequenceAct edit was reverted; image 0xAC8D6 retains original 08D0. The on-screen OK hint belongs to a separate UI path and was not added.

## Persistent magnification

mf_timer_am.s starts at image 0x2C0, with 228 text bytes reserved through 0x3AF. The hook at image 0x4C8A78 / runtime 0x548A78 changes B1F699FF032892D1→004B1847C1020800. An absolute jump enters Thumb 0x802C1, avoiding the old Thumb BL range limit. Only timer=3 with all reads successful can match: M uses (0x13,0x2BB)=(4,3), A uses (3,2), and both require 0x3EE=0, 0x419!=5, 0x2A2!=1. Matches suppress timeout; other states retain original processing. Non-expired/expired continuations are 0x5489A6/0x5489F2. Half-press and OK exits remain available.

## A-mode warning and capture

At image 0x15A78A / runtime 0x1DA78A, 07→41 handles the warning, but this edit alone enabled metering without capture. The capture hook at image 0x15FD02 / runtime 0x1DFD02 changes 032803D0→A0F655FB, branching to 0x803B0. a_capture_c2.s occupies 196 bytes at image 0x3B0..0x473. It retains the original effective-mode-3 allowance. Added mode 2 also requires the original C2 byte at the hook to be zero and successful public reads of 0x13=3, 0x3EE=0, 0x419!=5, 0x2A2!=1 and 0x2BB=2. Allow/original continuations are 0x1DFD0E/0x1DFD06. Registers and stack are preserved; AE is not rewritten and lens aperture data is not fabricated.

## A-mode Auto ISO

The original ISO producer at runtime 0x17A5C2 calls 0x178B2C for configuration, then 0x17A4D0 for second-stage handling before writing 0x2C9. Two lens-state clamps convert automatic enums 2/3/4 to 0x0F: one checks 0x3EE, the other 0x3F8/0x3F9. v7 changes 022803D0→05F748FF at image 0xFA5EC / runtime 0x17A5EC to enter 0x80480, and 022C03D0→05F7CAFF at image 0xFA518 / runtime 0x17A518 to enter 0x804B0. a_auto_iso.s occupies 256 bytes at image 0x480..0x57F.

## Property interfaces and exception scope

The ISO exception matches only automatic enums 2/3/4 and requires successful current staged-view reads of 0x13=3, 0x419!=5, 0x2A2!=1 and 0x3EE=0. Other original ISO/HDR handling remains. Success is 0xF0000000. Public-object factory 0x2CE0A9 uses vtable+0 for its getter; the staged view uses +4. These interfaces must not be mixed, and potentially unsynchronized public state should not replace staged state. 0x0F is an enum; its conversion to numeric ISO has not been independently traced in full.

## Code placement

Original image 0x2C0..0x7FF is zero-filled. Analysis checked the boot entry, nearby literals, discovered copy/clear ranges and image write start; the three helpers do not overlap. Working v3/v5/v7 hardware behavior supports loading and execution, but static searches do not exhaustively prove the absence of all indirect references. Recheck placement when expanding usage. Verify ARM/Thumb state, function-pointer low bits, complete instruction boundaries, registers, flags, LR, stack alignment and continuation semantics.

## Nested CRC and restoration

Use binascii.crc_hqx(data,0), stored big-endian. Embedded CRC at decoded package 0x175A6E0..E1 covers [0x50,0x175A6E0). Outer CRC at 0x175A6E2..E3 covers [0,0x175A6E2), including the written embedded CRC. Each appended CRC gives a zero syndrome; encode afterward and preserve other trailing bytes. Restoration is regenerated from stock: only package 0x859 changes 30→31 and two embedded CRC bytes change. Outer CRC happens to remain unchanged; all functional code remains stock. Restoration was tested successfully; it is not brick recovery.

## Verification and porting limits

Offline models cover 5,187 capture cases, 79,380 timer cases and 38,400 Auto-ISO cases; reports are retained in the internal handoff. They execute relevant instructions with mocked property interfaces, not the sensor, real AE or flash safety. See the J5 guide for the scope of hardware feedback. Empty-FT1 Auto ISO did not vary; this does not prove zero effect on all electronic lenses. Other Nikon 1 bodies require independent container, mapping, instruction-set, property-contract, boot, update and target-path checks. Reusing J5 addresses or merely changing the model name is not a port.


## Release files and evidence limits

Users download the BIN directly from Releases; no build is required. The internal builder starts from the exact official input and checks original bytes, change scope, nested CRC and final SHA-256. All three assembly helpers were recompiled and compared. Offline models are not full-camera emulation, and hardware feedback comes from one J5; see the [J5 guide](../cameras/J5/README.md) for untested scope.

Original input SHA-256: `5fae892c396d4213ebf5a4982fa98559c6c79e00f5e6e0cbe6251084cb9b1b04`.

v7 SHA-256: `94b545bc2b1e6cc3a6599daf73b56bef972fb321da7cfe2b1588f159d6966ab2`.

Restoration SHA-256: `0fb4f4115c6f09ca30b87d84affd233ff6891559edc649ac812289a0b0aa31d6`.
