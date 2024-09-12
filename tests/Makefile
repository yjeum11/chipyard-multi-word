#################################
# RISCV Toolchain
#################################

TARGET = riscv64-unknown-elf

GCC = $(TARGET)-gcc
CXX = $(TARGET)-g++
CP = $(TARGET)-objcopy
OBJDUMP = $(TARGET)-objdump
DG = $(TARGET)-gdb
SIZE = $(TARGET)-size


#################################
# Flags
#################################

# SoC Settings
ARCH = rv64imafdc
ABI = lp64d
ARCHFLAGS = -march=$(ARCH) -mabi=$(ABI)

CFLAGS  = -std=gnu99 -O3 -fno-common -fno-builtin-printf -Wall
CFLAGS += $(ARCHFLAGS)
LDFLAGS = -static

include libgloss.mk

# PROGRAMS = pwm blkdev accum charcount nic-loopback big-blkdev pingd \
#            streaming-passthrough streaming-fir nvdla spiflashread spiflashwrite fft gcd \
#            hello mt-hello symmetric
PROGRAMS = multi-word/rocc/time_ntt multi-word/rocc/time_add multi-word/rocc/time_ntt16 multi-word/rocc/time_ntt_1k multi-word/rocc/time_ntt_1k_int128

.DEFAULT_GOAL := default


#################################
# Build
#################################

spiflash.img: spiflash.py
	python3 $<

%.o: %.S
	$(GCC) $(CFLAGS) -D__ASSEMBLY__=1 -c $< -o $@

multi-word/vhd128/vhd128.o: multi-word/vhd128/vhd128.c multi-word/vhd128/vhd128.h
	$(GCC) $(CFLAGS) -c $< -o $@

%.o: %.c mmio.h spiflash.h
	$(GCC) $(CFLAGS) -c $< -o $@

%.riscv: %.o $(libgloss) multi-word/vhd128/vhd128.o
	$(GCC) $(LDFLAGS) multi-word/vhd128/vhd128.o $< -o $@

%.dump: %.riscv
	$(OBJDUMP) -D $< > $@


#################################
# Recipes
#################################

.PHONY: clean
clean:
	rm -f *.riscv *.o *.dump
	find . -type f \( -name "*.riscv" -o -name "*.o" -o -name "*.dump" \) -delete
	$(if $(libgloss),rm -rf $(libgloss_builddir)/)

.PHONY: default
default: $(addsuffix .riscv, $(PROGRAMS)) spiflash.img

.PHONY: dumps
dumps: $(addsuffix .dump, $(PROGRAMS))
