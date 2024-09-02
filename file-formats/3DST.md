# 3DST File Format

All documentation and knowledge of 3DST format written in this Markdown file, was observed by manipulating Minecraft New 3DS Edition's 3DST files and observing and documenting the results.


> [!TIP]
> It is best to think of think of the whole file as a list of DWORDs[^DWORD]. All eight values in the header are all DWORDs, and every pixel in the image data is a DWORD.

> [!NOTE]
> If you're already familiar with Little Endian, this note is of little use to you.
> 
> This format's data is stored in Little Endian. The data is stored such that the order of the **bytes** is arranged least significant to most significant; basically just reversed order. 
> 
> For example, a DWORD value of 3 `00 a2 00 03` would be stored as `03 00 a2 00`
>
> and a 32 bit RGBA value of {127, 63, 31, 255} `1f 3f 7f ff` would be stored as `ff 7f 3f 1f` (ABGR).
>
> The only piece of data **not** reversed is the first 4 bytes of the file, spelling out in ASCII "3DST"

## Header Data

Here is an example of a 3DST file header. This header is from the vanilla atlas.terrain.meta file. The header is always 32 bytes long, and data seems to be stored in 32-bit chunks.

     Offset (h)  |  00 01 02 03  04 05 06 07  08 09 0a 0b  0c 0d 0e 0f
    ------------ | ----------------------------------------------------
     0x00000000  |  33 44 53 54  03 00 00 00  00 00 00 00  00 02 00 00
     0x00000010  |  00 02 00 00  00 02 00 00  00 02 00 00  03 00 00 00

Here is a table of the header data, containing all of the information and uses I found:

| Offset (h) | Function | Example from vanilla terrain atlas | Description | Limits / Restrictions | Other Notes |
| ---------- | -------- | ---------------------------------- | ----------- | --------------------- | ----------- |
| `0x00` | "3DST" | `33 44 53 54` | This is just hex for "3DST" | N/A | N/A |
| `0x04` | Unknown | `03 00 00 00` | Unknown. Seems to always be `03 00 00 00` | Breaks if equal to any other value than `03 00 00 00` | Unknown |
| `0x08` | Unknown | `00 00 00 00` | Unknown. Seems to be unused. | Unknown | Unknown |
| `0x0c` | Full Width | `00 02 00 00` | This value is used to tell how many pixels should make up one line. | This value must be a power of 2. $4\times(\text{Full Width}\times\text{Full Height})$ should **not** exceed the file size in bytes. | In the more functional sense, this value is divided by 8 (bitshift right by 3), then used to tell how many pixel "chunks" (see section [Image Data](#image-data)) make up one row. |
| `0x10` | Full Height | `00 02 00 00` | This value is used to tell how many lines of pixels make up the image. | This value must be a power of 2. $4\times(\text{Full Width}\times\text{Full Height})$ should **not** exceed the file size in bytes. | In the more functional sense, this value is divided by 8 (bitshift right by 3), then used to tell how many pixel "chunks" (see section [Image Data](#image-data)) make up one row. |
| `0x14` | Final Width | `00 02 00 00` |  This value tells how wide the image should be treated as when loaded. | This value must be equal to or less than the value at offset `0x0c`. | This value essentially tells how many pixel columns to load, starting from the left.
| `0x18` | Final Height | `00 02 00 00` | This value tells how tall the image is should be treated as when loaded. | This value must be equal to or less than the value at offset `0x10`. | This value essentially tells how many pixel rows to load, starting from the top. |
| `0x1c` | Mipmap Levels | `03 00 00 00` | This value tells how many mipmap levels there are in the file. | This value must be greater than 0. | Each mipped texture is half the width and height as the next largest one. |

> [!IMPORTANT]
> Some of these will be referenced later, by the term used in the "Function" column.

### Examples
Here are some visual examples of manipulating the header data to show what the values do:

Original vanilla atlas.textures.meta:

![Original](https://github.com/user-attachments/assets/e1c982d0-4245-4260-8c1f-6ea9614459cc)

Setting "Final Width" to 256 `00 01 00 00`, half the original width:

![Half-Width](https://github.com/user-attachments/assets/5663fbdc-3e51-4211-bfca-ed0c2ba5d5cb)

Setting "Final Height" to 256 `00 01 00 00`, half the original height:

![Half-Height](https://github.com/user-attachments/assets/dfebce13-8c61-46a9-b67d-35f000cb3277)

Setting "Final Width" and "Full Width" to 256 `00 01 00 00`, "Final Height" and "Full Height" to 1024 `00 04 00 00`:

![Narrow-Tall](https://github.com/user-attachments/assets/af0c4697-5bb7-4197-9723-a6f72adea2c7)

Setting "Final Width" and "Full Width" to 1024 `00 04 00 00`, "Final Height" and "Full Height" to 256 `00 01 00 00`:

![Wide-Short](https://github.com/user-attachments/assets/eef88231-e5c5-4285-9d38-ee99dfab9d3d)


## Image Data 

The image data is stored in chunks. One pixel chunk is 64 pixels, (256 bytes, 64 DWORDs) arranged 8x8, and the first pixel of the image (offset `0x20` in file) which is the first pixel of the first chunk, is in the bottom left of the image, but the pixels in the chunks are _not_ arranged in a linear, immediately intuitive fashion.

![onechunk](https://github.com/user-attachments/assets/34e2a45a-8a11-4f19-b93d-706323d1ec4e)

> [!NOTE]
> The gradient I chose does _not_ repeat every 64 pixels, but instead 48 pixels, giving the illusion that it repeats unevenly.

This is a visual representation of how the chunk data is stored. This is pixel data written to the beginning of the image data. Starts at blue, fades to green, then red, then back to blue. 

This is a binary pattern. Let's say $n=\text{AND}(\frac{\text{current offset}-32}{4},63)$[^bitwise]; As $n$ goes from 0 `0b00000000` to 63 `0b00111111`, the $2^4$, $2^2$, and $2^0$[^footnote-x] places of $n$ make up the 3 bit $x$ position of pixel $n$ inside the pixel chunk. The $2^5$, $2^3$, and $2^1$[^footnote-y] place digits make up the 3 bit $y$ position of pixel $n$ inside the pixel chunk. 

![linechunk](https://github.com/user-attachments/assets/4b11e444-02b2-4b77-95b0-81599a7e543f) 

This is a line of pixel chunks.

[^dword]: a DWORD is a 4 byte (32 bit) piece of data.
[^bitwise]: Bitwise AND operation. Iterates through a binary number, and compares each bit of the same place/value to each other. Returns a value of 1 for the digit if both bits of the same place are 1.
[^footnote-x]: Can be understood as the bitwise AND operation of `0b00010101` and n, then move 4s place to 2s and 16s to 4s to make a 3 bit value, counting linearly from 0 to 7
[^footnote-y]: Can be understood as the bitwise AND operation of `0b00101010` and n, then divide by 2 or bitshift to the right one bit , then move 4s place to 2s and 16s to 4s to make a 3 bit value, counting linearly from 0 to 7

## Other Notes
