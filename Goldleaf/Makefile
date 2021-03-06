.SUFFIXES:

ifeq ($(strip $(DEVKITPRO)),)
$(error "Please set DEVKITPRO in your environment. export DEVKITPRO=<path to>/devkitpro")
endif

TOPDIR ?= $(CURDIR)
include $(DEVKITPRO)/libnx/switch_rules

APP_TITLE   := Goldleaf
APP_AUTHOR  := XorTroll
APP_VERSION := 0.6
APP_TITLEID	:= 050032A5CF12E000

TARGET      := Goldleaf
ICON		:= Icon.jpg

BUILD       := Build
SOURCES     := Source Source/fatfs Source/acc Source/dump Source/err Source/es Source/fs Source/hos Source/ncm Source/net Source/ns Source/nsp Source/set Source/nfp Source/ui Source/usb
INCLUDES    := ../libnx/nx/external/bsd/include Include Include/fatfs Include/acc Include/dump Include/err Include/es Include/fs Include/hos Include/ncm Include/net Include/nfp Include/ns Include/nsp Include/set Include/ui Include/usb
ROMFS       := RomFs

ARCH	:=	-march=armv8-a -mtune=cortex-a57 -mtp=soft -fPIE

CFLAGS	:=	-g -Wall -O2 -ffunction-sections \
			$(ARCH) $(DEFINES) $(CFLAGS)

CFLAGS	+=	$(INCLUDE) -D__SWITCH__ -DGOLDLEAF_VERSION=\"$(APP_VERSION)\" -DGOLDLEAF_APPID=0x$(APP_TITLEID)

CXXFLAGS	:= $(CFLAGS) -std=gnu++17 $(CXXFLAGS)

ASFLAGS	:=	-g $(ARCH)
LDFLAGS	=	-specs=$(DEVKITPRO)/libnx/switch.specs -g $(ARCH) -Wl,-Map,$(notdir $*.map)

LIBS	:=  -lcurl -lz -lmbedtls -lmbedcrypto -lmbedx509 -lnx -lpu -lfreetype -lSDL2_mixer -lopusfile -lopus -lmodplug -lmpg123 -lvorbisidec -lSDL2 -lc -logg -lSDL2_ttf -lSDL2_gfx -lSDL2_image -lwebp -lpng -ljpeg `sdl2-config --libs` `freetype-config --libs`
LIBDIRS	:= $(PORTLIBS) $(CURDIR)/../Plutonium/Plutonium/Output $(CURDIR)/../libnx/nx

ifneq ($(BUILD),$(notdir $(CURDIR)))

export OUTPUT	:=	$(CURDIR)/Output/$(TARGET)
export TOPDIR	:=	$(CURDIR)

export VPATH	:=	$(foreach dir,$(SOURCES),$(CURDIR)/$(dir)) \
			$(foreach dir,$(DATA),$(CURDIR)/$(dir))

export DEPSDIR	:=	$(CURDIR)/$(BUILD)

CFILES		:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.c)))
CPPFILES	:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.cpp)))
SFILES		:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.s)))
BINFILES	:=	$(foreach dir,$(DATA),$(notdir $(wildcard $(dir)/*.*)))

ifeq ($(strip $(CPPFILES)),)
	export LD	:=	$(CC)
else
	export LD	:=	$(CXX)
endif

export OFILES_BIN	:=	$(addsuffix .o,$(BINFILES))
export OFILES_SRC	:=	$(CPPFILES:.cpp=.o) $(CFILES:.c=.o) $(SFILES:.s=.o)
export OFILES 	:=	$(OFILES_BIN) $(OFILES_SRC)
export HFILES_BIN	:=	$(addsuffix .h,$(subst .,_,$(BINFILES)))

export INCLUDE	:=	$(foreach dir,$(INCLUDES),-I$(CURDIR)/$(dir)) \
			$(foreach dir,$(LIBDIRS),-I$(dir)/include) \
			-I$(CURDIR)/$(BUILD)

export LIBPATHS	:=	$(foreach dir,$(LIBDIRS),-L$(dir)/lib)

export BUILD_EXEFS_SRC := $(TOPDIR)/$(EXEFS_SRC)

ifeq ($(strip $(ICON)),)
	icons := $(wildcard *.jpg)
	ifneq (,$(findstring $(TARGET).jpg,$(icons)))
		export APP_ICON := $(TOPDIR)/$(TARGET).jpg
	else
		ifneq (,$(findstring icon.jpg,$(icons)))
			export APP_ICON := $(TOPDIR)/icon.jpg
		endif
	endif
else
	export APP_ICON := $(TOPDIR)/$(ICON)
endif

ifeq ($(strip $(NO_ICON)),)
	export NROFLAGS += --icon=$(APP_ICON)
endif

ifeq ($(strip $(NO_NACP)),)
	export NROFLAGS += --nacp=$(OUTPUT).nacp
endif

ifneq ($(APP_TITLEID),)
	export NACPFLAGS += --titleid=$(APP_TITLEID)
endif

ifneq ($(ROMFS),)
	export NROFLAGS += --romfsdir=$(CURDIR)/$(ROMFS)
endif

.PHONY: $(BUILD) clean

$(BUILD):
	@[ -d $@ ] || mkdir -p $@
	@[ -d $(CURDIR)/Output ] || mkdir -p $(CURDIR)/Output
	@$(MAKE) -C ../libnx/
	@$(MAKE) -C ../Plutonium/Plutonium/
	@$(MAKE) --no-print-directory -C $(BUILD) -f $(CURDIR)/Makefile

clean:
	@echo Goldleaf: Cleaning...
	@rm -fr $(BUILD)
	@rm -fr $(CURDIR)/Output

else

.PHONY:	all

DEPENDS	:=	$(OFILES:.o=.d)

all: $(OUTPUT).nro

ifeq ($(strip $(NO_NACP)),)
$(OUTPUT).nro	:	$(OUTPUT).elf $(OUTPUT).nacp
else
$(OUTPUT).nro	:	$(OUTPUT).elf
endif

$(OUTPUT).elf	:	$(OFILES)

$(OFILES_SRC)	: $(HFILES_BIN)

-include $(DEPENDS)

endif