
# vrEmu6502

6502/65c02 emulator written in C

Initially created for the [HBC-56 Emulator](https://github.com/visrealm/hbc-56)

Includes:
* Support for standard 6502, 65C02, WDC65C02 and R65C02.
* All WDC and Rockwell-specific instructions.
* Correct handling of Decimal mode.
* Accurate instruction timing.
* User-supplied I/O callbacks.
* IRQ and NMI signals.
* Multiple CPU instances.

## Test suite
Includes a test program which was designed to run [Klaus Dormann's 6502 tests](https://github.com/Klaus2m5/6502_65C02_functional_tests). Currently passing:
* 6502_functional_test
* 6502_decimal_test (with valid bcd only)
* 65C02_extended_opcodes_test

On Windows, run `test\test.bat`

## Quick start

```C
#include "vrEmu6502.h"

uint8_t ram[0x8000];
uint8_t rom[0x8000];

uint8_t My6502MemoryReadFunction(uint16_t addr, bool isDbg)
{
  if (addr < 0x8000)
  {
    return ram[addr];
  }
  return rom[addr & 0x7fff];
}

void My6502MemoryWriteFunction(uint16_t addr, uint8_t val)
{
  if (addr < 0x8000)
  {
    ram[addr] = val;
  }
}

/* fill rom with something that makes sense here */

VrEmu6502 *my6502 = vrEmu6502New(CPU_W65C02, My6502MemoryReadFunction, My6502MemoryWriteFunction);

if (my6502)
{
  /* if you want to interrupt the CPU, get a handle to its IRQ "pin" */
  vrEmu6502Interrupt *irq = vrEmu6502Int(my6502);

  /* reset the cpu (technically don't need to do this as vrEmu6502New does reset it) */
  vrEmu6502Reset(my6502);

  while (1)
  {
    /* call me once for each clock cycle (eg. 1,000,000 times per second for a 1MHz clock) */
    vrEmu6502Tick(my6502);
        
    /* interrupt it? */
    if (myHardwareWantsAttention)
    {
      *irq = IntRequested;
      
      /* at some point, the hardware will be happy and it will need to release the interrupt */
    }
  }

  vrEmu6502Destroy(my6502);
  my6502 = NULL;
}
```

See  [HBC-56](https://github.com/visrealm/hbc-56) for real-life example usage.


## License
This code is licensed under the [MIT](https://opensource.org/licenses/MIT "MIT") license