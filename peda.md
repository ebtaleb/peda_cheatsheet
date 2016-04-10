# Prepare input buffer

## pattern create
pattern create 2000
pattern create 2000 input

## pset arg
pset arg '”A”*200'
pset arg 'cyclic_pattern(200)'

## pset env
pset env EGG 'cyclic_pattern(200)'

# Context display

## Registers
context reg

## Code
context code

## Stack
context stack

# Runtime info

## Virtual memory mapping

vmmap
vmmap binary / libc
vmmap 0xb7d88000

## Register / address (smart deref'd)
xinfo register eax
xinfo 0xb7d88000

## Stack / memory
telescope 40
telescope 0xb7d88000 40

# Search for input buffer

## pattern offset
pattern offset $pc

## pattern search
pattern search

# jmp/call search

## jmpcall
jmpcall
jmpcall eax
jmpcall esp libc

# Generate shellcode/nopsled

## gennop
gennop 500
gennop 500 “\x90”

## shellcode
shellcode x86/linux exec

## assemble
assemble

# Exploit wrapper

## skeleton
skeleton argv exploit.py

## Use with GDB
set exec-wrapper ./exploit.py

# Memory search

## searchmem / find
find “/bin/sh” libc
find 0xdeadbeef all
find “..\x04\x08” 0x08048000 0x08049000

## refsearch
refsearch “/bin/sh”
refsearch 0xdeadbeef

## lookup address
lookup address stack libc

## lookup pointer
lookup pointer stack ld-2

# ASM / ROP search

## asmsearch
asmsearch “int 0x80”
asmsearch “add esp, ?” libc

## ropsearch
ropsearch “pop eax”
ropsearch “xchg eax, esp” libc

## dumprop
```
dumprop
dumprop binary “pop”
```

## ropgadget
ropgadget
ropgadget libc

# ELF headers / symbols

## elfheader / readelf
elfheader
elfheader .got
readelf libc .text

## elfsymbol
elfsymbol
elfsymbol printf

# ret2plt / ROP payload

## payload
payload copybytes
payload copybytes target “/bin/sh”
payload copybytes 0x0804a010 offset

# Other memory operations (1)

dumpmem libc.mem libc

loadmem stack.mem 0xbffdf000

cmpmem 0x08049000 0x0804a000 data.mem

xormem 0x08049000 0x0804a000 “thekey”

patch $esp 0xdeadbeef
patch $eax “the long string”
pattern patch 0xdeadbeef 100
patch (multiple lines)

strings
strings binary 4

hexdump $sp 64
hexdump $sp /20

hexprint $sp 64
hexprint $sp /20

# Other debugging helpers (1)

pdisass main
pdisass $pc /20

nearpc 20
nearpc 0x08048484

pltbreak cpy

deactive setresuid
deactive chdir

unptrace

stepuntil cmp
stepuntil xor
nextcall cpy
nextjmp

## tracecall / ftrace
tracecall
tracecall “cpy,printf”
tracecall “-puts,fflush”

## traceinst / itrace
traceinst 20
traceinst “cmp,xor”

waitfor
waitfor myprog -c

## snapshot
snapshot save
snapshot restore

## assemble
assemble $pc
> mov al, 0xb
> int 0x80
> end

```
procinfo
procinfo fd
```

# Config options

```
pshow
pshow option context

pset option context “code,stack”
pset option badchars “\r\n”
```

## Edit lib/config.py for permanent changes

# Python GDB scripting with PEDA (2)

## Getting help
pyhelp peda
pyhelp hex2str

## One-liner / interactive uses
gdb-peda$ python print peda.get_vmmap()
gdb-peda$ python
> status = peda.get_status()
> while status == “BREAKPOINT”:
>
peda.execute(“continue”)
> end


# Python GDB scripting with PEDA

## External scripts
```python
# myscript.py
def myrun(size):
	argv = cyclic_pattern(size)
	peda.execute(“set arg %s” % argv)
	peda.execute(“run”)

gdb-peda$ source myscript.py
gdb-peda$ python myrun(100)
```

# Extending PEDA

## Special functions
```
PEDA.execute()
PEDA.execute_redirect()
PEDACmd._is_running()
PEDACmd._missing_argument()
utils.execute_external_command()
utils.reset_cache()
```

## Writing new interactive command
```python
class PEDACmd():
    def mycommand(self, *arg):

    (arg1, arg2) = normalize_argv(arg, 2)

    if not arg1:
        self._missing_argument()

    if not self._is_running():
        return

    pid = peda.getpid()
    msg("My command: %d" % pid)

    return
```
