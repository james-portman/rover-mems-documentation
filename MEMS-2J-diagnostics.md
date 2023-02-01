5-baud wake up or fast-init (25ms break followed by 25ms non-break)

Only some cables allow breaks to be used properly - FTDI chips always work

Then use 10400 Baud rate, 8 bits, 1 stop bit, NO parity

K-line communication over a single wire

You literally get your own transmissions as echos because it is K-line

Uses KWP-2000


Normal packets structure is - [format] [dest] [source] [length of data bytes] [data bytes ...] [checksum]

The checksum is a SUM of all data bytes and the length byte, then cropped to 1 byte in case it has spilled over 255 (AND it with 0xFF)


After doing fast or slow init, send bytes to the ECU: 0x81, 0x13, 0xF7, 0x81, 0x0C - this is a one-off packet just to init/get in, no length byte sent

ECU should reply 0x03, 0xC1, 0xD5, 0x8F, 0x28 - again this is a one-off packet


After that it is just KWP2000
