# Simple ADB Protocol Implementation

Basic documentation on implementing the ADB protocol using raw tcp sockets. See [here](https://github.com/cstyan/adbDocumentation) for more in depth information and [here](https://android.googlesource.com/platform/system/core/+/refs/heads/android10-mainline-a-release/adb/protocol.txt) for the official documentation
## Connection steps
1. Send `CNXN` packet (`CNXN\x00\x00\x00\x01\x00\x10\x00\x00\x07\x00\x00\x00\x32\x02\x00\x00\xBC\xB1\xA7\xB1host::\x00`)
2. Receive `AUTH` packet and payload of token
3. Send `AUTH` packet and payload
    - Either sign token from previous AUTH packet with own private key and send back with type 2
    - or send own public key with type 3 
4. Receive 2 packets
    1. `CNXN` packet
    2. `device::` packet (contains information about the device)
5. Can now send any other packets

### Packets
Messages consist of a 24 byte header (shown below) and an optional payload. Each argument in the heaeder a 32bit word (`int32_t`) in little endian format
```C
struct message {
    unsigned command;       /* command identifier constant (A_CNXN, ...) */
    unsigned arg0;          /* first argument                            */
    unsigned arg1;          /* second argument                           */
    unsigned data_length;   /* length of payload (0 is allowed)          */
    unsigned data_crc32;    /* crc32 of data payload                     */
    unsigned magic;         /* command ^ 0xffffffff                      */
};
```

### Useful functions
```python
def str_byte(strr):
    return sum(c << (i * 8) for i, c in enumerate(bytearray(strr)))
    
def calc_crc(x):
    crc = 0
    for i in range(length):
        crc = (crc + x[i]) & 0xFFFFFFFF
    return crc
```
### `AUTH` packet
```python3
cmd = str_byte(b"AUTH")
arg0 = 3 # Use 2 for signed token, 3 for public key
arg1 = 0
payload = key # signed token or public key
length = len(payload)
crc = calc_crc(payload)
magic = cmd ^ 0xFFFFFFFF
auth_packet = struct.pack(b'<6I', cmd, arg0, arg1, length, crc, magic)
s.send(auth_packet)
s.send(payload)
```
### `OPEN` packet
```python3
cmd = str_byte(b"OPEN")
arg0 = str_byte(b"\xA5") # Non zero value
arg1 = 0
payload = b"shell:(shell cmd)"
length = len(payload)
crc = calc_crc(payload)
magic = cmd ^ 0xFFFFFFFF
auth_packet = struct.pack(b'<6I', cmd, arg0, arg1, length, crc, magic)
s.send(auth_packet)
s.send(payload)
```
