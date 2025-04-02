+++
title = 'Qrquiz Refactoring'
date = 2025-03-28T17:10:28+01:00
tags = ["qrquiz", "refactoring", "sqlite", "golang", "silly app"]
technologies = ["golang", "sqlite", "javascript", "gorm", "gin"]
+++

My hobby project [qrquiz](https://qrquiz.sekthor.dev) (described in [blog post](../qrquiz)) has had a pretty ugly hack to how I stored initial "broken" QR codes in the database.
I used to just store JSON marshaled representations of bitmap ([][]bool) in string format.
This has been bugging me from the get go ([#11](https://github.com/sekthor/qrquiz/issues/11)).
I was looking for a less disk space consuming option.
Ideally a more binary format.

I finally got around to implementing a fix.

## Problem

I was not happy, that for each boolean value, I would waste space, by storing four to five bytes, as the string representation is `true` and `false`, rather than a simple `0` or `1`.
I wanted to do this in a BLOB format, but could not be bothered to think of a way to serialize this.
I was laking a good solution to store the size of a row, as I thought I needed that to deserialize the blob back into a two dimensional array.

However I failed to remember, that the "square" nature of a QR code would not even require me to do this.
Since columns and rows are always the same amount for a QR code, I can very easily infer this value by simply calculating the square root of the length of the array.

## Solution

Given this sample bitmap (not a valid QR for brevity)...

```go
bm := Bitmap{
    {true, false, true, false, true},
    {false, true, false, true, false},
    {true, false, true, false, true},
    {false, true, false, true, false},
    {true, false, true, false, true},
}
```

...the Bitmap can be serialized to a simple `[]byte` slice.

```go
bytes := []byte{
    0x01, 0x00, 0x01, 0x00, 0x01,
    0x00, 0x01, 0x00, 0x01, 0x00,
    0x01, 0x00, 0x01, 0x00, 0x01,
    0x00, 0x01, 0x00, 0x01, 0x00,
    0x01, 0x00, 0x01, 0x00, 0x01,
},
```

Storing this as `BLOB` type in sqlite would be 25 bytes in size.

Unmarshaling back into a Bitmap can then be achieved by calculating the size of the "QR" `math.sqrt(len(bytes))` -> `5`.
With this knowledge we can recreate the bitmap from the byte array.

```go
size := int(math.Sqrt(float64(length)))

if size*size != length {
    return errors.New("length of byte array is not perfect square")
}

tmp := Bitmap{}
for i := range size {
    row := []bool{}
    for j := range size {
        if bytes[(i*size)+j] == 0 {
            row = append(row, false)
        } else {
            row = append(row, true)
        }
    }
    tmp = append(tmp, row)
}
```

## Comparison

Our example 5x5 pixel bitmap takes up only 25 bytes of diskspace using the new method.
Representing the Bitmap in JSON string format would have looked like this:

```json
[
    [true, false, true, false, true],
    [false, true, false, true, false],
    [true, false, true, false, true],
    [false, true, false, true, false],
    [true, false, true, false, true]
]
```

With out the pretty-print whitespace, that is still `167` bytes.
So we are now using only about 0.15 time the amount of what we used to.

## To Do

Technically, I will need to write a database migration, to convert existing data into the new schema.
Factually, there are hardly any quizzes in the live DB now.
Might not be worth it?
