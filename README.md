# COMP590 Assignment 1

## Approach

For this assignment, I changed `assgn1` to use a **lossless temporal-difference compression** method.

The video is decoded as grayscale so that each pixel is stored as one 8-bit value from 0 to 255.

The assignment says we must encode **one pixel at a time**, and my program does that by going left to right across each row, then moves to the next row.

I compare each pixel to the pixel in the same location in the previous frame and Then I encode the difference between those two values.


`difference = (current_pixel - prior_frame_pixel + 256) % 256`


After computing the difference, I encode that difference using arithmetic coding.

## Context Model

I use **2 arithmetic coding contexts**:

- `temporal_good_pdf`
- `temporal_bad_pdf`

To decide which context to use, I look at the neighboring pixels that have already been processed. I count how many of those neighbors are marked as **temporally coherent**.

If at least 3 neighbors are coherent, I encode the current pixel difference using the **good** context. Otherwise, I use the **bad** context.

A pixel is marked as temporally coherent if its wrapped temporal difference is close to 0 modulo 256:

- `difference <= 25`
- or `difference >= 231`

This gives the program a simple way to separate areas that are changing very little from areas that are changing a lot.

## Why It Is Lossless

This method is lossless because the decoder follows the exact same steps as the encoder.

It decodes the same wrapped difference values and then rebuilds each pixel using:

`reconstructed_pixel = (prior_frame_pixel + decoded_difference) % 256`

Because of this, the reconstructed pixel is exactly the same as the original pixel.

## Context Limit

The assignment allows at most **256 arithmetic coding contexts**.

My implementation uses 2 contexts and stays within the limit.

## Results

I tested the program on `bourne.mp4`.

Command used:

`cargo run --bin assgn1 -- -check_decode -count 100`

All 100 frames decoded correctly, which confirms that the method is lossless.

Compression results:
- Frames encoded: 100
- Average compressed size: 6,021,110 bits/frame
- Compression ratio: 2.76