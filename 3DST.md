# 3DST File Format

Note for beginners: This format's data is stored in Little Endian. The data is stored such that the order of 
the bytes is arranged least significant to most significant; basically just reversed order. For example, 
512 stored as a DWORD (a 32 bit/4 byte chunk) `00 00 02 00` would be stored as `00 02 00 00`, a value of 3 
`00 00 00 03` stored as a DWORD would be `03 00 00 00` and a 32 bit RGBA value of {127, 63, 31, 255} `1f 3f 7f ff` 
is stored as `ff 7f 3f 1f` (ABGR). The only piece of data not reversed is the first 4 bytes of the file, spelling out in ASCII "3DST"

## Header Data

Here is an example of a 3DST file header. This header is from the vanilla atlas.terrain.meta file. The header is always 32 bytes long, and data seems to be stored in 32-bit chunks.

    33 44 53 54 03 00 00 00 00 00 00 00 00 02 00 00
    00 02 00 00 00 02 00 00 00 02 00 00 03 00 00 00

Here is a table of everything I have figured out the function of:

| Offset | Function | Example from vanilla terrain atlas | Description |
| ------ | -------- | --------- | ----------- |
| `0x00` | "3DST" | `33 44 53 54` | This is just hex for "3DST" |
| `0x04` | Unknown | `03 00 00 00` | Unknown. Seems to always be `03 00 00 00` |
| `0x08` | Unknown | `00 00 00 00` | Unknown. Seems to be unused. |
| `0x0c` | Raw Width | `00 02 00 00` | This value is used to tell how many pixels should make up one line. This value must be a power of 2. |
| `0x10` | Raw Height | `00 02 00 00` | This value is used to tell how many lines of pixels make up the image. This value must be a power of 2. |
| `0x14` | Width | `00 02 00 00` | This value must be equal to or less than the value at offset `0x0c`. This value tells how wide the image should be treated as when loaded; it tells how many pixel columns to keep, starting from the left.
| `0x18` | Height | `00 02 00 00` | This value must be equal to or less than the value at offset `0x10`. This value tells how tall the image should be treated as when loaded; it tells how many pixel rows to keep, starting from the top.
| `0x1c` | Mipmap levels | `03 00 00 00` | This value tells how many mipmap levels there are in the file. All mipped textures are half the width and height as the previous one. This value must be greater than 0.

Some of these will be referenced later by the term used in the Function column.

### Examples
Here are some visual examples, manipulating the header data to show what the header values do

Original vanilla atlas.textures.meta:
![Original](https://github.com/user-attachments/assets/c4833b7f-6bb8-4813-b1b3-d4d8ca92a48c)

Setting Width to 256 `00 01 00 00`, half the original width 
![Half Width](https://github.com/user-attachments/assets/684245e2-3e4f-469f-ae83-e21f25cbf5c8)

Setting Height to 256 `00 01 00 00`, half the original height 
![Half Height](https://github.com/user-attachments/assets/8b767316-f2e4-4190-bf9d-b4454c97c726)

Setting Raw Width to 256 `00 01 00 00`, Raw Height to 1024 `00 04 00 00`, and Width and Height the same as their "raw" counterparts.


## Image Data



## Other Notes
