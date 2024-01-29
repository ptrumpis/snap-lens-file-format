# 👻 Snap Lens File Format 
Documentation of the Snapchat / Snap Camera Lens File Format (lens.lns / \*.lns) used by Snap Inc.

## Introduction
These files appear as `lens.lns` files in the cache folder of the Snap Camera desktop application, which officially ceased operations on **January 25, 2023**.
It appears to be a proprietary but uniform format used by Snap Inc. to publish and deliver its AR Lenses to its own applications.

Not much was known about this file format until now. All findings in this document are based on reverse engineering and trial and error.

## Application Usage
The file is known to be used by these Snap Inc. owned applications
- Snapchat
  - Used for the various AR camera filters that the app is famous for.
- Lens Studio - Snap Ar
  - Used as the final file format to publish developed AR Lenses on the official website.
- Snap Camera
  - Used for offline caching of the various AR camera filters.


## File Format
The file itself consists of a **Main Header**, a **File Allocation Table (FAT)**, a very small **Sub Header**, and BLOB of **Compressed Data** using the Zstandard algorithm.

*Zstandard, commonly known by the name of its reference implementation zstd, is a lossless data compression algorithm developed by Yann Collet at Facebook.*
The [zstd source code](https://github.com/facebook/zstd) is dual-licensed under BSD and GPLv2.

The basic structure of the file can be divided into 4 sections as follows.

| File Structure            | Size    |
| ------------------------- | ------- |
| Main Header               | 72 byte |
| File Allocation Table     |  N byte |
| Small Sub Header          |  8 byte |
| Zstandard Compressed Data |  N byte |

The main header is always 72 bytes in size and begins with the ASCII signature `LZC` terminated with `0x00`.

The File Allocation Table addresses the uncompressed data and varies in size. The exact size of the FAT is specified in the last 4 bytes of the Main Header.

The small Sub Header consists of only 8 bytes, with the last 4 bytes specifying the size of the following data block compressed with Zstandard.

The format of the data compressed with ZStandard is most detailed in the official documentation and not scope of this documentation.
Link to the offical documentation: [Zstandard Compression Format](https://github.com/facebook/zstd/blob/dev/doc/zstd_compression_format.md)

## Detailed File Structure
A more detailed description of each file section follows.

Important Note: The byte order is little endian.

### Main Header
Description of the first 72 bytes referred to as Main Header section.

Hex offset | Description                 | Size    | Static Value | Comment 
---------- | --------------------------- | ------- | ------------ | --------
0x0000     | Signature                   |  4 byte | 4c 5a 43 00  | LZC 0x00 (Zero terminated ASCII String)
0x0004     | Version                     |  4 byte | 01 00 00 00  | Version 1
0x0008     | FAT Record count            |  4 byte |       -      | Total number of FAT Records and files in this archive
0x000C     | Sub Header Offset           |  4 byte |       -      | Points to the first byte of the sub header
0x0010     | Unknown                     |  4 byte | 01 00 00 00  | Value is always 1
0x0014     | Unknown                     |  4 byte | 01 00 00 00  | Value is always 1
0x0018     | Uncompressed File Size      |  4 byte |       -      | Size of all files inside this archive uncompressed
0x001C     | Compressed File Size        |  4 byte |       -      | Size of all files inside this archive compressed
0x0020     | Zero Padding                | 32 byte | 00 00 00 00  | 32 Zero Byte padding
0x0040     | Unknown                     |  4 byte | 02 00 00 00  | Value is always 2
0x0044     | FAT Size                    |  4 byte |       -      | Dynamic FAT size in bytes


### File Allocation Table (FAT)
The FAT consists of a repetitive data structure referred to as FAT record.

#### FAT Record
A single FAT Record has a dynamic size of minimum 16 bytes. The maximum size of a single record is estimated at 255/260 (Windows MAX_PATH) + 16 bytes, but that is not certain.

Hex offset | Description                | Size    | Comment 
---------- | -------------------------- | ------- | --------
0x0048     | Path/File Name Length      |  4 byte | File name length and the total size of the FAT Record + 16 bytes
......     | Path/File Name             |  n byte | Dynamic length specified by the first 4 bytes of this structure
......     | Compressed File Size       |  4 byte | This field presumably holds the compressed file size
......     | Uncompressed File Size     |  4 byte | Original file size
......     | File Offset (Uncompressed) |  4 byte | Offset of the first byte of this file inside the uncompressed BLOB

The first record starts at offset `0x0048`, the next record will start right after the first record without any padding.
The total amount of records the FAT holds is specified in the Main Header at offset `0x0008` the total size in bytes is specified at offset `0x0044`.

The beginning of the Sub Header marks the end of the File Allocation Table.

### Sub Header
The Sub Header has a static size of 8 bytes. Its first 4 bytes start with the static value of 1.

The exact location of this header is specified at offset `0x000C` in the Main Header, which represents a pointer to the first byte.

Hex offset | Description         | Size    | Static Value | Comment 
---------- | ------------------- | ------- | ------------ | --------
......     | Version/Unknown     |  4 byte | 01 00 00 00  | Could represent version or reserved for future use
......     | Zstandard Data Size |  4 byte |       -      | Dynamic size of compressed BLOB in bytes

### Zstandard Compressed Data
See here: [Zstandard Compression Format](https://github.com/facebook/zstd/blob/dev/doc/zstd_compression_format.md)

## Proof of Concept
I wrote a small web based file unpacker tool in JavaScript to decompress the data and export the archive to ZIP format. Since there is currently no working unpackers.

My working implementations
- [https://github.com/ptrumpis/snap-lens-file-extractor](https://github.com/ptrumpis/snap-lens-file-extractor)
- [https://github.com/ptrumpis/snap-lens-tool](https://github.com/ptrumpis/snap-lens-tool)

## Links and Resources
- [https://github.com/facebook/zstd](https://github.com/facebook/zstd)
- [https://github.com/facebook/zstd/blob/dev/doc/zstd_compression_format.md](https://github.com/facebook/zstd/blob/dev/doc/zstd_compression_format.md)

## Special Thanks
I would like to thank [encode.su](https://encode.su/) and especially the user mariush for providing very accurate first header analysis.
[https://encode.su/threads/4010-Help-with-unknown-file-archive-starting-with-LZC-Header](https://encode.su/threads/4010-Help-with-unknown-file-archive-starting-with-LZC-Header)

---

Source code available at GitHub:  
[Snap Lens File Format](https://github.com/ptrumpis/snap-lens-file-format)

© 2023-2024 [Patrick Trumpis](https://github.com/ptrumpis)
