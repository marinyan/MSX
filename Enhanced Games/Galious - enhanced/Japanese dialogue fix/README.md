# Galious Enhanced V1.04: Japanese dialogue fixes

This add-on fixes two Japanese UI regressions in Enhanced V1.04:

* DEATH's revival explanation and payment question (messages 00h and 02h)
  display `るれ`, `ろわ`, and `をん` where the original game shows paired
  arrow, coin, and key icons. The legacy codes 54h..59h collide with the
  Enhanced kana font.
* The Japanese YES/NO labels (message 0Dh, also used by the password prompt)
  are 16 pixels above the corresponding English/Spanish labels. The shared
  hand cursor is unchanged. Move the labels from EEA9h (Y104) to EEE9h (Y120).

## Apply

Apply the existing **Galious Enhanced V1.04.ips** to your own supported
original game ROM first. Then apply **Galious Enhanced V1.04 JP fix.ips**
from this directory to that 512 KiB Enhanced ROM. This is an add-on, not a
replacement for the existing enhancement patch. The original patch stays intact.

For a hash-checked, reproducible build, use Python 3 (standard library only):

```sh
python build_patch.py "original.rom" "enhanced-v1.04.rom" --output-dir out
```

The script requires both input identities below and writes a new add-on IPS,
corrected `.rom`, and manifest. It never changes either input in place. The
original ROM is needed to reconstruct the six resource-icon glyphs. No game
ROM is included in this change.

| File | Identity |
|---|---|
| Supported original ROM | SHA-1 `4d51d3c5036311392b173a576bc7d91dc9fed6cb` |
| Existing upstream V1.04 IPS | SHA-256 `f125675dc098385744a65246d447b7ff581ea6dd48694389ac6ee792b6a6963b` |
| Unmodified Enhanced V1.04 ROM | SHA-256 `cdeaa916d198a3ca9891076eb79e05709876fa7ce18dcd929e76f1cf25403f53` |
| Add-on IPS, 4,219 bytes | SHA-256 `b6b9bb0748dc56de39bbff7063ca1a139883b25e889c05f89e2e4e29b074441b` |
| Corrected ROM, 524,288 bytes | SHA-256 `5d85cfc159acd89f5c82b990ddaacc5f5a68df48ea5f24b4c920d4bd3a070253` |

## Binary changes

The ordinary kana glyphs are preserved. The original game's pattern/color RLE
streams at bank 10:6399h/6563h supply the six icons. They occupy blank private
font slots 117..122; only the twelve resource-picture references in messages
00h/02h are remapped. Codes E2h..E7h continue to render the normal kana.

The enlarged 4,096-byte decoded font compresses to 3,615 bytes. Its stream is
placed at bank 12:6000h, an entirely FF-filled unused 8 KiB bank in the pinned
release. Change the font-only loader at RAM 0951h from bank 22 to bank 12.
The original bank-22 stream and all adjacent assets remain intact. Preserve
the source stream's 80h/0000h/00h address-reset/return suffix. No new Z80
instructions, mapper expansion, or ROM-size increase are required.

The second fix changes only the low byte of message 0Dh's destination at
bank 15:B30Ch from A9h to E9h. English and Spanish text and destinations stay
unchanged.

## Validation

The corrected ROM was cold-booted four times in openMSX (C-BIOS MSX2 JP).
Its actual Z80/V9938 drawing routines rendered all 20 dialogue messages in
each of Japanese, English, and Spanish, plus the combined payment question,
YES/NO labels, and common cursor in all three languages.

All 60 isolated dialogue screens match an independent interpretation of the
corrected ROM's text and font. Before/after differences occur only in Japanese
00h, 02h, and 0Dh. All regular font pixels outside the six formerly blank slots
are unchanged, and the actual cold-boot font load matches the relocated stream.
Black palette entries 0 and 14 are treated as equal. The IPS was independently
applied back to the canonical Enhanced image and reproduced the corrected hash.

These are source CPU/VDP fixture checks. Complete ordinary-play revival and
password-entry flows, other MSX configurations, and physical hardware have not
been verified for this patch. Unrelated enhancement or translation issues are
outside its scope.
