# auth_aes128_md5

### TCP

#### 1. The structure of a handshake request (unencrypted)
```
whole request
+--------+--------+----------+
| part 1 | part 2 |  part 3  |
+--------+--------+----------+
|   7    |   24   | Variable |
+--------+--------+----------+
part 1
+--------+----------+
| Random | HMAC-MD5 |
+--------+----------+
|    1   |     6    |
+--------+----------+
part 2
+-----+----------------------------+----------+
| UID | AES-128-CBC encrypted data | HMAC-MD5 |
+-----+----------------------------+----------+
|  4  |             16             |     4    |
+-----+----------------------------+----------+
part 3
+--------------+------------------+----------+
| Random bytes | Origin SS stream | HMAC-MD5 |
+--------------+------------------+----------+
|   Variable   |     Variable     |     4    |
+--------------+------------------+----------+
AES-128-CBC encrypted data (unencrypted)
+-----+-----+---------------+-------------+---------------------+
| UTC | CID | Connection ID | pack length | Random bytes length |
+-----+---------------------+-------------+---------------------+
|  4  |  4  |       4       |      2      |           2         |
+-----+-----+---------------+-------------+---------------------+
The key of AES128 encryption is:
    Base64(encrypt_key) + salt
    salt is "auth_aes128_md5"
The IV of AES128 encryption is: "\x00" * 16
UTC, Connection ID, pack length, Random bytes length are little-endian
Connection ID is an unsigned 32bit integer, it must +1 after each handshake request sent
CID is randomly generated by SSR client. And Connection ID need to be initialized with a random integer between 0~0xFFFFFF when generate a new CID
Client must keep the CID value until Connection ID exceed 0xFF000000

The encrypt_key is user definition or stream encryption key
```
Notice: part 1 for compatible origin protocol

The HMAC key in part 1 and part 2 is:  
IV + key

The HMAC key in part 3 is user customized or stream encryption key

The HMAC input in part 1 is the Random byte in part 1

The HMAC input in part 2 is whole data in part 2 except HMAC itself

The HMAC input in part 3 is whole data of the handshake request

#### 2. The structure of any other packet
```
+------+----------+--------------+-------------------------+----------+
| size | HMAC-MD5 | Random bytes |         Payload         | HMAC-MD5 |
+------+----------+--------------+-------------------------+----------+
|  2   |     2    |   Variable   | size - Random bytes - 8 |     4    |
+------+----------+--------------+-------------------------+----------+

The "size" is the whole packet size, little-endian
```
Random bytes:  
If the first byte is 255 then `bytes[1] | bytes[2] << 8` is the length of the Random bytes, otherwise bytes[0] is the length of the Random bytes

### UDP
1.Client to server
```
+----------+-----+----------+
|  Payload | UID | HMAC-MD5 |
+----------+-----+----------+
|     2    |  4  |     4    |
+----------+-----+----------+
```
The HMAC key is user customized or stream encryption key

2.Server to client
```
+----------+----------+
|  Payload | HMAC-MD5 |
+----------+----------+
|     2    |     4    |
+----------+----------+
```
The HMAC key is user customized or stream encryption key



# auth_aes128_sha1

Instead of MD5, the HMAC function for `auth_aes128_sha1` is SHA1, and salt is "`auth_aes128_sha1`", rest are the same.