################################################################################
#
# Makefile meant to build a demonstration application for micro-ROS on KR260
# File tangled from the README.
#
# Vincent Conus, 2024-04-04
#
################################################################################

# Target, utils, paths
# This is the the final name for the ~*.elf~ binary.

TARGET = micro-ROS_Client_Firmware



# Aliases for usefull tools

RM := rm -rf

SUBDIRS := src

# sources

LD_SRCS += ../src/lscript.ld

# All

all: $(TARGET).elf

# Building the micro-ROS static library

arm_toolchain: $(EXTENSIONS_DIR)/arm_toolchain.cmake.in
	@echo 'Preparing the cmake file'
	rm -f $(EXTENSIONS_DIR)/arm_toolchain.cmake; \
	cat $(EXTENSIONS_DIR)/arm_toolchain.cmake.in | \
		sed "s/@CROSS_COMPILE@/$(subst /,\/,$(CROSS_COMPILE))/g" | \
		sed "s/@FREERTOS_TOPDIR@/$(subst /,\/,$(TOPFOLDER))/g" | \
		sed "s/@ARCH_CPU_FLAGS@/\"$(ARCHCPUFLAGS)\"/g" | \
		sed "s/@ARCH_OPT_FLAGS@/\"$(ARCHOPTIMIZATION)\"/g" \
		> $(EXTENSIONS_DIR)/arm_toolchain.cmake
	@echo ' '

colcon_compile: arm_toolchain
	@echo 'Building the with colcon'
	cd $(UROS_DIR); \
	colcon build \
		--packages-ignore-regex=.*_cpp \
		--metas $(UROS_DIR)/colcon.meta $(UROS_APP_FOLDER)/app-colcon.meta \
		--cmake-args \
		"--no-warn-unused-cli" \
		-DCMAKE_POSITION_INDEPENDENT_CODE=OFF \
		-DTHIRDPARTY=ON \
		-DBUILD_SHARED_LIBS=OFF \
		-DBUILD_TESTING=OFF \
		-DCMAKE_BUILD_TYPE=$(BUILD_TYPE) \
		-DCMAKE_TOOLCHAIN_FILE=$(EXTENSIONS_DIR)/arm_toolchain.cmake \
		-DCMAKE_VERBOSE_MAKEFILE=ON;
	@echo ' '

libmicroros: colcon_compile
	@echo 'Make the built libmicroros available'
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
	@echo ' '

# Building the application

all: $(TARGET).elf

$(TARGET).elf: $(OBJS) ../src/lscript.ld $(USER_OBJS)
	@echo 'Building target: $@'
	@echo 'Invoking: ARM R5 gcc linker'
	armr5-none-eabi-gcc -s -L../microros_lib -mcpu=cortex-r5  -mthumb -mfloat-abi=hard  -mfpu=vfpv3-d16 -Wl,-T -Wl,../src/lscript.ld -L../freertos_libs -o "rpmsg_pingpong_microros_lib.elf" $(OBJS) $(USER_OBJS) $(LIBS)
	@echo 'Finished building target: $@'
	@echo ' '

$(ELFSIZE): $(TARGET).elf
	@echo 'Invoking: ARM R5 Print Size'
	armr5-none-eabi-size -x rpmsg_pingpong_microros_lib.elf  | tee "rpmsg_pingpong_microros_lib.elf.size"
	@echo 'Finished building: $@'
	@echo ' '



# As a second build artifact, the elf size file shall be created too.

secondary-outputs: $(ELFSIZE)



# Recap of all the available option for the Makefile

.PHONY: all clean dependents

# Clean up

clean:
	-$(RM) $(TARGET).elf $(ELFSIZE)
	-@echo ' '
