# 3DST File Format

> [!NOTE]
> This format's data is stored in Little Endian. The data is stored such that the order of the **bytes** is arranged least significant to most significant; basically just reversed order. 
>
> For example, 512 stored as a DWORD (a 32 bit/4 byte chunk) `00 00 02 00` would be stored as `00 02 00 00`
> 
> A value of 3 `00 00 00 03` stored as a DWORD would be `03 00 00 00`
>
> and a 32 bit RGBA value of {127, 63, 31, 255} `1f 3f 7f ff` is stored as `ff 7f 3f 1f` (ABGR).
>
> The only piece of data **not** reversed is the first 4 bytes of the file, spelling out in ASCII "3DST"

## Header Data

Here is an example of a 3DST file header. This header is from the vanilla atlas.terrain.meta file. The header is always 32 bytes long, and data seems to be stored in 32-bit chunks.

    33 44 53 54 03 00 00 00 00 00 00 00 00 02 00 00
    00 02 00 00 00 02 00 00 00 02 00 00 03 00 00 00

Here is a table of everything I have figured out the function of:

| Offset | Function | Example from vanilla terrain atlas | Description | Requirements | Other Notes |
| ------ | -------- | ---------------------------------- | ----------- | ------------ | ----------- |
| `0x00` | "3DST" | `33 44 53 54` | This is just hex for "3DST" | N/A | N/A |
| `0x04` | Unknown | `03 00 00 00` | Unknown. Seems to always be `03 00 00 00` | Breaks if equal to any other value than `03 00 00 00` | Unknown |
| `0x08` | Unknown | `00 00 00 00` | Unknown. Seems to be unused. | Unknown | Unknown |
| `0x0c` | Raw Width | `00 02 00 00` | This value is used to tell how many pixels should make up one line. | This value must be a power of 2. | In the more functional sense, this value is divided by 8 (bitshift right by 3), then used to tell how many pixel "chunks" (see section [Image Data](#image-data)) make up one row. |
| `0x10` | Raw Height | `00 02 00 00` | This value is used to tell how many lines of pixels make up the image. | This value must be a power of 2. | In the more functional sense, this value is divided by 8 (bitshift right by 3), then used to tell how many pixel "chunks" (see section [Image Data](#image-data)) make up one row. |
| `0x14` | Width | `00 02 00 00` |  This value tells how wide the image should be treated as when loaded. | This value must be equal to or less than the value at offset `0x0c`. | This value essentially tells how many pixel columns to load, starting from the left.
| `0x18` | Height | `00 02 00 00` | This value tells how tall the image is should be treated as when loaded. | This value must be equal to or less than the value at offset `0x10`. | This value essentially tells how many pixel rows to load, starting from the top. |
| `0x1c` | Mipmap Levels | `03 00 00 00` | This value tells how many mipmap levels there are in the file. | This value must be greater than 0. | Each mipped texture is half the width and height as the next largest one. |

> [!IMPORTANT]
> Some of these will be referenced later, by the term used in the "Function" column.

### Examples
Here are some visual examples, manipulating the header data to show what the header values do

Original vanilla atlas.textures.meta:

![Original](https://github.com/user-attachments/assets/e1c982d0-4245-4260-8c1f-6ea9614459cc)

Setting "Width" to 256 `00 01 00 00`, half the original width:

![Selection_304](https://github.com/user-attachments/assets/5663fbdc-3e51-4211-bfca-ed0c2ba5d5cb)

Setting "Height" to 256 `00 01 00 00`, half the original height:

![Half-Height](https://github.com/user-attachments/assets/dfebce13-8c61-46a9-b67d-35f000cb3277)

Setting "Width" and "Raw Width" to 256 `00 01 00 00`, "Height" and "Raw Height" to 1024 `00 04 00 00`:

![Selection_301](https://github.com/user-attachments/assets/9e015e6c-d778-4d77-986c-d820279a6a3c)


Setting "Width" and "Raw Width" to 1024 `00 04 00 00`, "Height" and "Raw Height" to 256 `00 01 00 00`:

![Selection_306](https://github.com/user-attachments/assets/eef88231-e5c5-4285-9d38-ee99dfab9d3d)


## Image Data

The image data is stored in chunks. One pixel chunk is 64 pixels, (256 bytes) arranged 8x8, and the first pixel of the image, the first pixel of the first chunk is in the bottom left of the image, but the pixels in the chunks are _not_ arranged in a linear, immediately intuitive fashion.

![Selection_29122](https://github.com/user-attachments/assets/4b11e444-02b2-4b77-95b0-81599a7e543f)

This is a visual representation of how the chunk data is stored. This is linear pixel data written to the start of the image data. Starts at blue, fades to green, then red, then back to blue. 

This is a binary pattern. Let's say $n=\text{AND}(\frac{\text{current offset}-32}{4},63)$[^1]; As $n$ goes from 0 `0b00000000` to 63 `0b00111111`, the $2^4$, $2^2$, and $2^0$[^2] places of $n$ make up the 3 bit $x$ position of pixel $n$ inside the pixel chunk. The $2^5$, $2^3$, and $2^1$[^3] place digits make up the 3 bit $y$ position of pixel $n$ inside the pixel chunk. 

[^1]: Bitwise AND operation. Compares each bit of the same place/value to each other, and returns a value of 1 for the digit only if both bits of the same place are 1.
[^2]: Can be understood as the bitwise AND operation of `0b00010101` and n, then move 4s place to 2s and 16s to 4s to make a 3 bit value, counting linearly from 0 to 7
[^3]: Can be understood as the bitwise AND operation of `0b00101010` and n, then divide by 2 or bitshift to the right one bit , then move 4s place to 2s and 16s to 4s to make a 3 bit value, counting linearly from 0 to 7

## Other Notes
