# ip proto \ah


## BPF

```
000: A = P[12:2]
001: if (A == 2048) goto 2 else goto 5
002: A = P[23:1]
003: if (A == 51) goto 4 else goto 5
004: return 65535
005: return 0
```


## BPF cross-compiled to Lua

```
return function (P, length)
   local A = 0
   if 14 > length then return false end
   A = bit.bor(bit.lshift(P[12], 8), P[12+1])
   if not (A==2048) then goto L4 end
   if 24 > length then return false end
   A = P[23]
   if not (A==51) then goto L4 end
   do return true end
   ::L4::
   do return false end
   error("end of bpf")
end
```


## Direct pflang compilation

```
local cast = require("ffi").cast
return function(P,length)
   if length < 34 then return false end
   if cast("uint16_t*", P+12)[0] ~= 8 then return false end
   return P[23] == 51
end
```

## Native pflang compilation

```
7f58f3365000  4883FE22          cmp rsi, +0x22
7f58f3365004  7C14              jl 0x7f58f336501a
7f58f3365006  0FB7770C          movzx esi, word [rdi+0xc]
7f58f336500a  4883FE08          cmp rsi, +0x08
7f58f336500e  750A              jnz 0x7f58f336501a
7f58f3365010  0FB67717          movzx esi, byte [rdi+0x17]
7f58f3365014  4883FE33          cmp rsi, +0x33
7f58f3365018  7403              jz 0x7f58f336501d
7f58f336501a  B000              mov al, 0x0
7f58f336501c  C3                ret
7f58f336501d  B001              mov al, 0x1
7f58f336501f  C3                ret
```

