No wake up needed for 9600 baud

Optional 10400 baud

Connect at 9600 Baud rate, 8 bits, 1 stop bit, EVEN parity

K-line communication over a single wire

You literally get your own transmissions as echos on K-line


Always start data packets with header 0xB8, 0x13, 0xF7 (format, destination, source)

Reply packets will always start with header 0x13, 0xB8, 0xF7 (format, destination, source)

After the header, add a length byte (data length only), followed by the data, followed by a XOR checksum

The checksum is a XOR of all bytes apart from the checksum byte itself, cropped to 1 byte (AND it with 0xFF)

ECU response checksums are sometimes AND, sometimes XOR

KWP-2000



See ISO 14229-1/ISO 14230-3 (KWP)

Seed/key auth

Ask for seed - send 0x27, 0x01

ECU should reply with 0x67, 0x01, [seed byte 1], [seed byte 2]

Compute the key to reply

Send the key - 0x27, 0x02, [key byte 1], [key byte 2]

ECU should accept if correct - 0x67, 0x02



A 0x7F response is a failure

After that it is just KWP-2000
