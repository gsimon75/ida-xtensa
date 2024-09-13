# ida-xtensa
This is a processor plugin for IDA 7.x (backported to 6.x too), to support the Xtensa core found in
Espressif ESP8266 and ESP32.

This is a beta release, for your hacking pleasure.

## Limitations
It does not support other configurations of the Xtensa architecture, but that is probably
(hopefully) easy to implement.

The most annoying missing architectural thing is the ELF loader support for Tensilica (type=94).
IDA 6.x doesn't know about it at all, 7.x recognizes it but still doesn't handle relocations.

This means that if you try to disassemble a `.o` file (eg. `app_main.o` from `libmain.a`), it won't be too usable, as the external function references won't be displayed by name.

For example, the function `call_user_start_local` looks like this:

```
.text.call_user_start_local:00001960 ; ===========================================================================
.text.call_user_start_local:00001960
.text.call_user_start_local:00001960 ; Segment type: Pure code
.text.call_user_start_local:00001960 off_1960        .int 0x000003fc
.text.call_user_start_local:00001964 dword_1964      .int 0
.text.call_user_start_local:00001968
.text.call_user_start_local:00001968 ; =============== S U B R O U T I N E =======================================
.text.call_user_start_local:00001968
.text.call_user_start_local:00001968
.text.call_user_start_local:00001968 call_user_start_local:
.text.call_user_start_local:00001968                 addi           a1, a1, 0xF0        ; a1 -= 0x10
.text.call_user_start_local:0000196B                 s32i           a0, a1, 0xC         ; [a1 + 0x0c] = a0
.text.call_user_start_local:0000196E                 l32r           a0, off_1960        ; a0 = [pc + ...] = [off_1960]
.text.call_user_start_local:00001971                 callx0         a0                  ; call a0
.text.call_user_start_local:00001974                 l32r           a0, dword_1964      ; a0 = [pc + ...] = [dword_1964]
.text.call_user_start_local:00001977                 callx0         a0                  ; call a0
.text.call_user_start_local:0000197A                 l32i.n         a0, a1, 0xC         ; a0 = [a1 + 0x0c]
.text.call_user_start_local:0000197C                 addi           a1, a1, 0x10        ; a1 += 0x10
.text.call_user_start_local:0000197F                 ret.n
.text.call_user_start_local:0000197F ; End of function call_user_start_local
```

With `objdump` we can see that that `0x3fc` actually means `user_start+0x3fc` and that `0` means `ets_run+0`:

```
RELOCATION RECORDS FOR [.text.call_user_start_local]:
OFFSET   TYPE              VALUE

00000000 R_XTENSA_32       .text.user_start
0000000e R_XTENSA_SLOT0_OP  .text.call_user_start_local
0000000e R_XTENSA_ASM_EXPAND  .text.user_start+0x000003fc

00000004 R_XTENSA_32       ets_run
00000014 R_XTENSA_SLOT0_OP  .text.call_user_start_local+0x00000004
00000014 R_XTENSA_ASM_EXPAND  ets_run
```

Now, the support for these `R_XTENSA_<whatever>` records is completely missing from the IDA ELF loader.

So, either we write an ELF loader too :D, or the best we can do is to actually link these
objects into a firmware (that loads to a known address, and has no relocations, but has
debug symbols), and load that `.elf`.


## Usage
Copy the file to the `procs/` directory in your IDA install. Ta-daa!

## License
GPLv2!
