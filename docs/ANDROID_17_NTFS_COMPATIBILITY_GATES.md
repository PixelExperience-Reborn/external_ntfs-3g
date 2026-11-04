# PixelExperience-Reborn Android 17: optional NTFS source compatibility

Date: 2026-11-05. Scope: platform base / PE historical compatibility. **No image built.**

## Previous PE product contract

Historical source: `PixelExperience-Reborn/external_ntfs-3g` at `7471d6403640f1cd0878cc2c87279571a497bcb7`, based on NTFS-3G **2022.5.17**. Its Soong `Android.bp` exports:
- `libfuse-lite` from `libfuse-lite/*.c`
- `libntfs-3g` with NTFS filesystem parsing and write support
- `fsck.ntfs`, `mkfs.ntfs`, and `mount.ntfs`

The legacy PE product requested these package names. The new AOSP17 `vendor/aosp/config/common.mk` deliberately gates the historical modules behind `PE_A17_PORT_NTFS=true`. Merely enabling this flag cannot make an unmodified 2022 build ABI-correct.

## Source graph and safety

- Official AOSP17 `android-17.0.0_r1` reference manifest has **no** `external/ntfs-3g` checkout. Do not substitute `libjson`, `libfuse`, or system `vold` names.
- Keep `PE_A17_PORT_NTFS` **disabled** in common products until the chosen modern implementation is audited and built.
- Mounting NTFS read-only for removable USB media is separate from running `fsck.ntfs` or `mkfs.ntfs`; the latter modify data. No write/repair or automatic NTFS disk operations may be enabled by this archival review.
- On mondrian, kernel support for NTFS/NTFS3 and actual recovery/vendor mount paths must be checked later, not presumed from a generic kernel version or the historical PE ROM.

## Required Android17 implementation

1. Establish the exact consumer: `vold` public volume support, PE recovery removable-media menu, vendor app or explicit product requirement. Document whether it needs `libntfs-3g`, an executable, or Linux NTFS3. For read-only recovery, a mounted, verified Linux filesystem may avoid a user-space helper entirely.
2. Select **maintained** implementation and compatible license, then carry forward only confirmed PE behavior. Validate FUSE, linker namespaces, seccomp, SELinux, and Soong `vendor`/`recovery` variants. The 2022 PE `Android.bp` does **not** declare `recovery_available` or vendor variant for these binaries.
3. Test images: clean NTFS, dirty bit, corrupted metadata, oversized file, paths and Unicode, partition mount cancellation; enforce read-only handling for untrusted media. Do not write to real userdata or removable drives in preliminary tests.
4. Only after a successful Android17 build and tests may `PE_A17_PORT_NTFS` be enabled for an explicitly supported product.

**Current status:** historical API and packaging identified, optional source omitted safely from the staged Android17 base. NTFS integration and consumer-level behavior are *not completed*.
