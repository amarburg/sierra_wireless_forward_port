# Copyright Sierra Wireless 2009 - 2011
#
# Makefile to install the Sierra modules/drivers.
#
# Dependencies: 
#      sierra depends on usbserial (serial driver)
#      sierra_net depends on usbnet (network driver) 
#
# Assumptions:
# 	This makefile assumes that kernel modules are installed in 
#	      /lib/modules/'uname -r'/.
#	Should the assumption be incorrect, please use:
#	    make MODULE_ROOT="/path/to/modules/for/your/kernel"
#	to override assumed path
#
# Serial driver is required, built all the time, the network driver is optional
# and may be removed from the build simply by renaming the sierra_net.c file or
# deleting it completely.

#
# sierra.c is required source, sierra_net.c is optional source
# therefore we build obj-m below conditionaly, based on files present on disk
srcs := sierra.c $(wildcard sierra_net.c)
obj-m := $(srcs:%.c=%.o)

MODULE_ROOT:= /lib/modules/$(shell uname -r)
BUILDDIR   := $(MODULE_ROOT)/build
USBDIR     := $(MODULE_ROOT)/kernel/drivers/usb/serial
NETDIR     := $(MODULE_ROOT)/kernel/drivers/net/usb
PWD        := $(shell pwd)

override install_targets := $(srcs:%.c=install_%)

#-------------------- Rules ------------------

all: default

default:
	@echo "-----------------------------------------------------"
	@echo "Compiling " $(srcs)
	@$(MAKE) -C $(BUILDDIR) SUBDIRS=$(PWD) "obj-m=$(obj-m)" modules
	@echo "-----------------------------------------------------"

.PHONY: default install all clean help $(install_targets)

install_sierra:
	@install -m 644 sierra.ko $(USBDIR)
	@depmod -a
	@modprobe -r sierra
	@modprobe sierra

install_sierra_net:
	@install -b -m 644 sierra_net.ko $(NETDIR)
	@depmod -a
	@modprobe -r sierra_net
	@modprobe sierra_net

# install depends on the installation of current driver set 
install: $(install_targets)

debug:
	@echo
	@echo "srcs="$(srcs)
	@echo "obj-m="$(obj-m)
	@echo "MODULE_ROOT="$(MODULE_ROOT)
	@echo "PWD="$(PWD)
	@echo "install targets: " $(install_targets)
	@echo

help:
	@echo 
	@echo Use: make [target]
	@echo Note:make without a target is the same as \'make default\'
	@echo
	@echo Available targets:
	@echo " default - build drivers in this directory" 
	@echo " all - same as default" 
	@echo " install - install previously built drivers (requires su privileges)" 
	@echo " install_sierra_net - install network/ethernet driver only (su privileges required)"
	@echo " install_sierra - install serial driver only (su privileges required)"
	@echo " help - this help"
	@echo

clean:
	-@rm -rf *.o *.ko sierra*.mod.c Module.symvers .sierra* .tmp_versions modules.order


# Copyright Sierra Wireless 2009 - 2011
