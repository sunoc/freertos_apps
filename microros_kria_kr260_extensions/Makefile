################################################################################
#
# File tangled from the README.
#
################################################################################

# Target
# This is the the final name for the ~*.elf~ binary.

TARGET = micro-ROS

# Utils
# Aliases for usefull tools

RM := rm -rf

# paths

SUBDIRS := src

# sources

LD_SRCS += ../src/lscript.ld

# All

all: $(TARGET).elf

# libmicroros

libmicroros: colcon_compile
	mkdir -p $(UROS_DIR)/libmicroros; cd $(UROS_DIR)/libmicroros; \
	for file in $$(find $(UROS_DIR)/install/ -name '*.a'); do \
		folder=$$(echo $$file | sed -E "s/(.+)\/(.+).a/\2/"); \
		mkdir -p $$folder; cd $$folder; ar x $$file; \
		for f in *; do \
			mv $$f ../$$folder-$$f; \
		done; \
		cd ..; rm -rf $$folder; \
	done ; \
	ar rc libmicroros.a *.obj; mkdir -p $(BUILD_DIR); cp libmicroros.a $(BUILD_DIR); ranlib $(BUILD_DIR)/libmicroros.a; \
	cd ..; rm -rf libmicroros;

# Clean

clean:
	-$(RM) $(TARGET).elf

# .PHONY
# Recap of all the available option for the Makefile

.PHONY: all clean
