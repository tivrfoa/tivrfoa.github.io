---
layout: post
title:  "Random Code Analysis - intel/e1000e/netdev.c"
date:   2021-08-08 14:10:00 -0300
categories: random code analysis linux
---

The goal of "random code analysis" posts are just to
learn more about how interesting/important code are written,
and also to learn more about the language.

`netdev.c` is a big module, with 7924 lines ...<br>
So, I just looked at the first 7% =)

### struct e1000_info

`linux/drivers/net/ethernet/intel/e1000e/e1000.h`

```c
struct e1000_info {
	enum e1000_mac_type	mac;
	unsigned int		flags;
	unsigned int		flags2;
	u32			pba;
	u32			max_hw_frame_size;
	s32			(*get_variants)(struct e1000_adapter *);
	const struct e1000_mac_operations *mac_ops;
	const struct e1000_phy_operations *phy_ops;
	const struct e1000_nvm_operations *nvm_ops;
};
```

It's nice that it has:
  - an enum
  - a function pointer
  - 3 `const` struct pointers

```c
enum e1000_boards {
	board_82571,
	board_82572,
	board_82573,
	board_82574,
	board_82583,
	board_80003es2lan,
	board_ich8lan,
	board_ich9lan,
	board_ich10lan,
	board_pchlan,
	board_pch2lan,
	board_pch_lpt,
	board_pch_spt,
	board_pch_cnp
};

// ...
extern const struct e1000_info e1000_82571_info;
// ...
```

Usage Examples:
 
`linux/drivers/net/ethernet/intel/e1000e/e1000.c`

```c
static const struct e1000_info *e1000_info_tbl[] = {
	[board_82571]		= &e1000_82571_info,
	[board_82572]		= &e1000_82572_info,
	[board_82573]		= &e1000_82573_info,
	[board_82574]		= &e1000_82574_info,
	[board_82583]		= &e1000_82583_info,
	[board_80003es2lan]	= &e1000_es2_info,
	[board_ich8lan]		= &e1000_ich8_info,
	[board_ich9lan]		= &e1000_ich9_info,
	[board_ich10lan]	= &e1000_ich10_info,
	[board_pchlan]		= &e1000_pch_info,
	[board_pch2lan]		= &e1000_pch2_info,
	[board_pch_lpt]		= &e1000_pch_lpt_info,
	[board_pch_spt]		= &e1000_pch_spt_info,
	[board_pch_cnp]		= &e1000_pch_cnp_info,
};
```

I had never saw this `[]` usage before, eg `[board_82571]`.

```c
	const struct e1000_info *ei = e1000_info_tbl[ent->driver_data];
```

So you can use use enums to have key/value pairs!<br>
That's pretty cool!!!

These are called **Designated Initializers** [https://gcc.gnu.org/onlinedocs/gcc/Designated-Inits.html](https://gcc.gnu.org/onlinedocs/gcc/Designated-Inits.html)

>In ISO C99 you can give the elements in any order, specifying the array indices or structure field names they apply to, and GNU C allows this as an extension in C90 mode as well. This extension is not implemented in GNU C++.
>
>To specify an array index, write ‘[index] =’ before the element value. For example,
>
>```c
>int a[6] = { [4] = 29, [2] = 15 };
>```
>is equivalent to
>
>```c
>int a[6] = { 0, 0, 15, 0, 29, 0 };
>```


### Initializing an array of struct

```c
struct e1000_reg_info {
	u32 ofs;
	char *name;
};

static const struct e1000_reg_info e1000_reg_info_tbl[] = {
	/* General Registers */
	{E1000_CTRL, "CTRL"},
	{E1000_STATUS, "STATUS"},
	{E1000_CTRL_EXT, "CTRL_EXT"},

	/* Interrupt Registers */
	{E1000_ICR, "ICR"},
	// ...
	
    /* List Terminator */
    // Interesting use to make it easy to know it's the end of the array
    // This way you don't need to know the array size
    {0, NULL}          
};                       

```

`E1000_CTRL` is defined in regs.h: `e1000.h:#include "hw.h" -> hw.h:#include "regs.h"`

```c
#define E1000_CTRL	0x00000	/* Device Control - RW */
```

### Unused parameter ??

```c
static void __ew32_prepare(struct e1000_hw *hw)
{
	s32 i = E1000_ICH_FWSM_PCIM2PCI_COUNT;

	while ((er32(FWSM) & E1000_ICH_FWSM_PCIM2PCI) && --i)
		udelay(50);
}
```

I asked on the netdev mailing list, and here is the explanation:

> If you have a look at the definition of er32() (which is a macro and
> is defined in e1000.h, you'll see that the hw parameter is used
> there without being a parameter of the macro itself. Thus if you'd
> rename the parameter you'd get a build error.Not really the best
> code to look at when you want to learn coding, because that's an
> example how not to do things, IMHO.
>
> -michael


er32 is defined as:

```c
#define er32(reg)   __er32(hw, E1000_##reg)
```

That's why I didn't find any definition for `FWSM` ... because of the
token concatenation: [https://gcc.gnu.org/onlinedocs/cpp/Concatenation.html](https://gcc.gnu.org/onlinedocs/cpp/Concatenation.html)

I think this way it would be more clear:

```c
#define er32(hw, reg)   __er32(hw, reg)
```

I don't know if they did it that way just to avoid typing, or if there is some
other reason.

That is one advantage of Rust: macros have an ! at the end, eg: `println!`
So you can easily distinguish macros from functions.


How to create this macro in Rust:

```rust
#![feature(concat_idents)]

const E1000_FWSM: u32 = 10;

struct e1000_hw {
	hw_addr: u32,
}

#[inline]
fn __er32(hw: &e1000_hw, reg: u32) -> u32
{
    hw.hw_addr + reg
}

macro_rules! er32 {
	($hw:ident, $reg:ident) => {
		__er32($hw, concat_idents!(E1000_, $reg));
	}
}

fn main() {
	let hw1 = e1000_hw { hw_addr: 5 };
	let hw = &hw1;

	println!("{}", er32!(hw, FWSM));
}
```

Rust also prevents the probleme of capturing local variables:

>Hygienic macros are macros whose expansion is guaranteed not to cause the accidental capture of identifiers. They are a feature of programming languages such as Scheme,Dylan, Rust, Nim, and Julia.
>
> [https://en.wikipedia.org/wiki/Hygienic_macro](https://en.wikipedia.org/wiki/Hygienic_macro)

[This answer](https://stackoverflow.com/a/63349812/339561) on Stack Overflow shows a good example:

>Rust macros are hygienic, which prevents identifiers from one scope leaking into another and causing all manner of mayhem. But you can solve this particular problem by passing param from generate_func to generate_body:

```rust
macro_rules! generate_func {
    ($name:ident) => {
        fn $name(param: i32) {
            generate_body!(param)
        }
    };
}

macro_rules! generate_body {
    ($param:ident) => {
        println!("{}", &$param);
    }
}

generate_func!(foo);
```

[https://stackoverflow.com/questions/63349678/is-there-a-way-to-reference-a-local-variable-within-a-rust-macro](https://stackoverflow.com/questions/63349678/is-there-a-way-to-reference-a-local-variable-within-a-rust-macro)

### writel

`writel` is used in lots of places, what does it do?


Look at this macro! It creates a function!

```c
#define build_mmio_write(name, size, type, reg, barrier) \
static inline void name(type val, volatile void __iomem *addr) \
{ asm volatile("mov" size " %0,%1": :reg (val), \
"m" (*(volatile type __force *)addr) barrier); }


build_mmio_write(writel, "l", unsigned int, "r", :"memory")
```

Let's try this (`create-function.c`):

```c 
#include <stdio.h>

// https://elixir.bootlin.com/linux/latest/source/include/linux/compiler_types.h#L11
# define __iomem	__attribute__((noderef, address_space(__iomem)))

// https://elixir.bootlin.com/linux/latest/source/include/linux/compiler_types.h#L24
# define __force	__attribute__((force))

// https://elixir.bootlin.com/linux/latest/source/arch/x86/include/asm/io.h#L52
#define build_mmio_write(name, size, type, reg, barrier) \
static inline void name(type val, volatile void __iomem *addr) \
{ asm volatile("mov" size " %0,%1": :reg (val), \
"m" (*(volatile type __force *)addr) barrier); }


build_mmio_write(writel, "l", unsigned int, "r", :"memory")

typedef unsigned int ui;

int main() {

	ui a = 5;
	ui *b = &a;
	writel(10, b);
	printf("a = %d\n", a); // prints 10

	return 0;
}
```

Compiling with `gcc -E create-function.c` shows the macro expansion!<br>
Pretty cool command.

```c
# 10 "create-function.c"
static inline void writel(unsigned int val, volatile void __iomem *addr)
{
	asm volatile("mov" "l" " %0,%1": :"r" (val), "m" (*(volatile unsigned int __force *)addr) :"memory");
}
```

So it seems `writel` writes a value to a memory address.

If you include the `#include <stdio.h>` you'll see
many interesting things! eg:

```c
extern FILE *stdin;
extern FILE *stdout;
extern FILE *stderr;


extern int printf (const char *__restrict __format, ...);


# 246 "/usr/include/stdio.h" 3 4
extern FILE *fopen (const char *__restrict __filename,
      const char *__restrict __modes) ;
```


### More macros ...

`regs.h`

```c
/* Convenience macros
 * 
 * Note: "_n" is the queue number of the register to be written to
 *  
 * Example usage:
 * E1000_RDBAL_REG(current_rx_queue) 
 */ 
#define E1000_RDBAL(_n) ((_n) < 4 ? (0x02800 + ((_n) * 0x100)) : \
             (0x0C000 + ((_n) * 0x40))) 
#define E1000_RDBAH(_n) ((_n) < 4 ? (0x02804 + ((_n) * 0x100)) : \
             (0x0C004 + ((_n) * 0x40))) 

```

`netdev.c`

```c
/**                                                                           
 * e1000_regdump - register printout routine                                  
 * @hw: pointer to the HW structure                                           
 * @reginfo: pointer to the register info table                               
 **/                                                                          
static void e1000_regdump(struct e1000_hw *hw, struct e1000_reg_info *reginfo)
{                                                                             
    int n = 0;                                                                
    char rname[16];                                                           
    u32 regs[8];                                                              
                                                                              
    switch (reginfo->ofs) {                                                   
    case E1000_RXDCTL(0):                                                     
        for (n = 0; n < 2; n++)                                               
            regs[n] = __er32(hw, E1000_RXDCTL(n));                            
        break;                                                                
    case E1000_TXDCTL(0):                                                     
        for (n = 0; n < 2; n++)                                               
            regs[n] = __er32(hw, E1000_TXDCTL(n));                            
        break;                                                                
    case E1000_TARC(0):                                                       
        for (n = 0; n < 2; n++)                                               
            regs[n] = __er32(hw, E1000_TARC(n));                              
        break;                                                                
    default:                                                                  
        pr_info("%-15s %08x\n",                                               
            reginfo->name, __er32(hw, reginfo->ofs));                         
        return;                                                               
    }                                                                         
                                                                              
    snprintf(rname, 16, "%s%s", reginfo->name, "[0-1]");                      
    pr_info("%-15s %08x %08x\n", rname, regs[0], regs[1]);                    
}                                                                             
```

### You can have a struct inside a function ...

... I didn't know that ...

```c
static void e1000e_dump(struct e1000_adapter *adapter)
{                                                     
    struct net_device *netdev = adapter->netdev;      
    struct e1000_hw *hw = &adapter->hw;               
    struct e1000_reg_info *reginfo;                   
    struct e1000_ring *tx_ring = adapter->tx_ring;    
    struct e1000_tx_desc *tx_desc;                    
    struct my_u0 {                                    
        __le64 a;                                     
        __le64 b;                                     
    } *u0;                                            
// ...
}
```

It's interesting that it also creates a pointer to a struct it
just defined.

u0 receives the value through a cast:
```c
u0 = (struct my_u0 *)tx_desc;
```

### `e1000_tx_desc` cool and complex data structure

```c
/* Transmit Descriptor */
struct e1000_tx_desc {
    __le64 buffer_addr;      /* Address of the descriptor's data buffer */
    union {
        __le32 data;
        struct {
            __le16 length;    /* Data buffer length */
            u8 cso; /* Checksum offset */
            u8 cmd; /* Descriptor control */
        } flags;
    } lower;
    union {
        __le32 data;
        struct {
            u8 status;     /* Descriptor status */
            u8 css; /* Checksum start */
            __le16 special;
        } fields;
    } upper;
};
```

It turns out in C you can cast a structure to a number.

eg:

```c
#include <stdio.h>
#include <stdlib.h>

typedef unsigned long long ull;

char* bin(ull n)
{
	char *buf = malloc(65);
	int p = 0;
	ull i;
	buf[p++] = '0';
	for (i = 1L << 62; i > 0; i = i / 2)
	{
	  if((n & i) != 0) {
		buf[p++] = '1';
	  } else {
		buf[p++] = '0';
	  }
	}
	return buf;
}

int main() {

	struct {
		int a;
		int b;
	} s1;

	s1.a = 1987;
	s1.b = 34;

	printf("%s|%s\n", bin(s1.a), bin(s1.b));


	ull *l = (ull *) &s1;
	printf("%lld\n", *l);
	printf("%s\n", bin(*l));

	return 0;
}
```

This outputs:

```out
0000000000000000000000000000000000000000000000000000011111000011|0000000000000000000000000000000000000000000000000000000000100010
146028890051
0000000000000000000000000010001000000000000000000000011111000011
```

Can you spot one interesting things in this output?! ...<br>
The order is reversed to the order declared in the structure, that's
why the `e1000_tx_desc` struct first has `lower` then `upper` variables. 

### One more of a variable storing more than one information

```c
/**                                                                         
 * e1000_rx_checksum - Receive Checksum Offload                             
 * @adapter: board private structure                                        
 * @status_err: receive descriptor status and error fields                  
 * @skb: socket buffer with received data                                   
 **/                                                                        
static void e1000_rx_checksum(struct e1000_adapter *adapter, u32 status_err,
                  struct sk_buff *skb)                                      
{                                                                           
    u16 status = (u16)status_err;                                           
    u8 errors = (u8)(status_err >> 24);                                     
// ...
}
```

