# MicroPython LittleFS V2 - Directory to .uf2

Pack a directory of files (optionally filtered via a manifest) into a LFSV2 filesystem and save as .uf2 for flashing to the Pico.

Needs LittleFS-Python:
 - https://pypi.org/project/littlefs-python/
 - https://github.com/jrast/littlefs-python


## Current Limitations

- Outputs .uf2 with Pico family ID (non-generic, blocked from flashing to other boards???)
- Only compatible with MicroPython >= 1.23.0

## Usage

```
usage: dir2uf2 [-h] [--filename FILENAME] [--fs-start FS_START] [--fs-size FS_SIZE] [--fs-compact] [--block-size BLOCK_SIZE]
               [--read-size READ_SIZE] [--prog-size PROG_SIZE] [--manifest MANIFEST] [--append-to APPEND_TO] [--debug]
               [--verbose]
               source_dir

positional arguments:
  source_dir            Source directory.

options:
  -h, --help            show this help message and exit
  --filename FILENAME   Output filename.
  --fs-start FS_START   Filesystem offset.
  --fs-size FS_SIZE     Filesystem size.
  --fs-compact          Compact filesystem to used blocks.
  --block-size BLOCK_SIZE
                        LFS block size in Kb.
  --read-size READ_SIZE
                        LFS read size in Kb.
  --prog-size PROG_SIZE
                        LFS prog size in Kb.
  --manifest MANIFEST   Manifest to filter copied files.
  --append-to APPEND_TO
                        uf2 file to append filesystem.
  --debug               Debug output.
  --verbose             Verbose output.
```

## Examples

With a manifest:

```
./dir2uf2 --manifest enviro.txt ~/Downloads/enviro-0.0.5
```

Copy entire dir:

```
./dir2uf2 ~/Downloads/enviro-0.0.5
```

Compact filesystem for a (sometimes *much*) smaller .uf2:

```
./dir2uf2 --fs-compact ~/Downloads/enviro-0.0.5
```

## FAT filesystems

`--fs-type fat` builds a FAT volume, for a region the board exposes over USB mass storage, using
[ffsmake](https://github.com/gadgetoid/ffsmake). ffsmake formats with MicroPython's own oofatfs, so
the volume matches what `vfs.VfsFat.mkfs` makes on the board. Build ffsmake first, and point
`--ffsmake` at it if it is not on `PATH`.

`--block-size` is the sector size, which must match the block device's (4096 for rp2 flash).
`--fs-label` sets the volume label. `--manifest` and `--fs-compact` work as they do for LittleFS.

A region that no block device in the firmware declares is given by address and size, which accept
hex:

```
./dir2uf2 --fs-type fat --fs-label FX --fs-compact --sparse \
    --fs-start 0x10200000 --fs-size 0x6a0000 \
    --append-to firmware.uf2 --filename drive ~/my-drive
```

Appending a FAT volume and a LittleFS filesystem to one .uf2 takes two runs, the second appending to
the first's output. Without `--sparse`, append the lower region first: padding runs from the end of
the firmware to the region being appended, so a later run below an earlier one is refused.
