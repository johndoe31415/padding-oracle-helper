# padding-oracle-helper
This is a small comand line utility that is intended to allow people to study
the behavior of a PKCS#7 CBC padding oracle "by hand". Consider two blocks Q
and C, where Q is the IV for a CBC-encrypted ciphertext block C. Then Q can be
given on the command line and the CLI will output if the PKCS#7 padding is
correct.

It also supports bruteforcing of nibbles until a valid padding is found by
using the 'x' character within the Q block. Examples:

```
$ ./padding_ora_cli 00000000000000000000000000000000
$ ./padding_ora_cli -vv 00000000000000000000000000000000
Invalid padding at Q: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
               Q ^ P: 49 74 20 72 65 61 6c 6c 79 20 77 6f 72 6b 73 21

Found 0 valid padding(s) and 1 invalid padding(s).
```

You'll see that this Q block did not result in valid padding after decryption.
Let's brute force the last byte:

```
$ ./padding_ora_cli 000000000000000000000000000000xx
Successful padding at Q: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 20
                  Q ^ P: 49 74 20 72 65 61 6c 6c 79 20 77 6f 72 6b 73 01

Found 1 valid padding(s) and 255 invalid padding(s).
```

You'll see that if the last byte of Q is 0x20, the padding will be correct. We
also see what would be typically hidden from us: The actual Q ^ P on the
server, indicating that we actually found a padding of "01". We can also ask
the tool to infer the plaintext:

```
$ ./padding_ora_cli -v 000000000000000000000000000000xx
Successful padding at Q: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 20
                  Q ^ P: 49 74 20 72 65 61 6c 6c 79 20 77 6f 72 6b 73 01
                   D(C): 49 74 20 72 65 61 6c 6c 79 20 77 6f 72 6b 73 21

Found 1 valid padding(s) and 255 invalid padding(s).
```

The last byte of the plaintext therefore is 0x21. Let's manually set it to two:

```
$ python3 -c 'print(hex(0x21^0x02))'
0x23
$ ./padding_ora_cli -v 00000000000000000000000000000023
```

Obviously, this again is a broken padding, because now Q^P ends in "73 02".
Let's brute force the second to last byte:

```
$ ./padding_ora_cli -v 0000000000000000000000000000xx23
Successful padding at Q: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 71 23
                  Q ^ P: 49 74 20 72 65 61 6c 6c 79 20 77 6f 72 6b 02 02
                   D(C): 49 74 20 72 65 61 6c 6c 79 20 77 6f 72 6b 73 21

Found 1 valid padding(s) and 255 invalid padding(s).
```

And so on and so on.

## More challenging ciphertexts
With the default key (all 0), try this ciphertext: `8b 85 b4 13 d5 d8 e5 41  a6 bc 34 5a 56 0c 32 b9`

You can simply do this by using the `-C` command line option.

## License
GNU GPL-3.
