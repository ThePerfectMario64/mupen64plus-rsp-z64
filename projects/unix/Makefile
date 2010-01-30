#/* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *
# *   mupen64plus-rsp-z64 - Makefile                                        *
# *   http://bitbucket.org/wahrhaft/mupen64plus-rsp-z64/                    *
# *   Copyright (C) 2010 Jon Ring                                           *
# *   Copyright (C) 2008-2009 Richard Goedeken                              *
# *   Copyright (C) 2007-2008 DarkJeztr Tillin9                             *
# *                                                                         *
# *   This program is free software; you can redistribute it and/or modify  *
# *   it under the terms of the GNU General Public License as published by  *
# *   the Free Software Foundation; either version 2 of the License, or     *
# *   (at your option) any later version.                                   *
# *                                                                         *
# *   This program is distributed in the hope that it will be useful,       *
# *   but WITHOUT ANY WARRANTY; without even the implied warranty of        *
# *   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the         *
# *   GNU General Public License for more details.                          *
# *                                                                         *
# *   You should have received a copy of the GNU General Public License     *
# *   along with this program; if not, write to the                         *
# *   Free Software Foundation, Inc.,                                       *
# *   51 Franklin Street, Fifth Floor, Boston, MA 02110-1301, USA.          *
# * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * */
# Makefile for Z64 RSP plugin in Mupen64plus.

# detect operating system
UNAME = $(shell uname -s)
OS := NONE
ifeq ("$(UNAME)","Linux")
  OS = LINUX
  SO_EXTENSION = so
  SHARED = -shared
endif
ifeq ("$(UNAME)","linux")
  OS = LINUX
  SO_EXTENSION = so
  SHARED = -shared
endif
ifeq ("$(UNAME)","Darwin")
  OS = OSX
  SO_EXTENSION = dylib
  SHARED = -bundle
endif
ifeq ("$(UNAME)","FreeBSD")
  OS = FREEBSD
  SO_EXTENSION = so
  SHARED = -shared
endif
ifeq ("$(OS)","NONE")
  $(error OS type "$(UNAME)" not supported.  Please file bug report at 'http://code.google.com/p/mupen64plus/issues')
endif

# detect system architecture
HOST_CPU ?= $(shell uname -m)
NO_ASM ?= 1
CPU := NONE
ifneq ("$(filter x86_64 amd64,$(HOST_CPU))","")
  CPU := X86
  ifeq ("$(BITS)", "32")
    ARCH_DETECTED := 64BITS_32
  else
    ARCH_DETECTED := 64BITS
  endif
endif
ifneq ("$(filter pentium i%86,$(HOST_CPU))","")
  CPU := X86
  ARCH_DETECTED := 32BITS
endif
# PPC doesn't work yet
#ifneq ("$(filter ppc powerpc,$(HOST_CPU))","")
#  CPU := PPC
#  ARCH_DETECTED := 32BITS
#endif
#ifneq ("$(filter ppc64 powerpc64,$(HOST_CPU))","")
#  CPU := PPC
#  ARCH_DETECTED := 64BITS
#endif
ifeq ("$(CPU)","NONE")
  $(error CPU type "$(HOST_CPU)" not supported.  Please file bug report at 'http://code.google.com/p/mupen64plus/issues')
endif

# base CFLAGS, LIBS, and LDFLAGS
CFLAGS = -ffast-math -funroll-loops -fexpensive-optimizations -fno-strict-aliasing -I../../src
LDFLAGS = -ldl

# Since we are building a shared library, we must compile with -fPIC for x86_64 CPUs.
# On 32-bit systems we do not want to use -fPIC because we don't have to and it has a big performance penalty on this arch
ifeq ($(ARCH_DETECTED), 64BITS)
  CFLAGS += -fPIC -DPIC -fvisibility=hidden
endif
# tweak flags for 32-bit build on 64-bit system
ifeq ($(ARCH_DETECTED), 64BITS_32)
  ifeq ($(OS), FREEBSD)
    $(error Do not use the BITS=32 option with FreeBSD, use -m32 and -m elf_i386)
  endif
  CFLAGS += -m32
  LDFLAGS += -m32 -m elf_i386
endif

# set special flags per-system
ifeq ($(OS), LINUX)
  ifeq ($(CPU), X86)
    ifeq ($(ARCH_DETECTED), 64BITS)
      CFLAGS += -pipe -O3 -march=athlon64
    else
      CFLAGS += -pipe -O3 -mmmx -msse -march=i686 -mtune=pentium-m -fomit-frame-pointer
    endif
  endif
endif
ifeq ($(OS), OSX)
  ifeq ($(CPU), X86)
    ifeq ($(ARCH_DETECTED), 64BITS)
      CFLAGS += -pipe -O3 -arch x86_64 -mmacosx-version-min=10.5 -isysroot /Developer/SDKs/MacOSX10.5.sdk
      LDFLAGS += -arch x86_64
    else
      CFLAGS += -pipe -O3 -mmmx -msse -fomit-frame-pointer -arch i686 -mmacosx-version-min=10.5 -isysroot /Developer/SDKs/MacOSX10.5.sdk
      LDFLAGS += -arch i686
    endif
  endif
endif
ifeq ($(CPU), PPC)
  CFLAGS += -mcpu=powerpc
endif

# set mupen64plus core API header path
ifneq ("$(APIDIR)","")
  CFLAGS += "-I$(APIDIR)"
else
  TRYDIR = ../../../mupen64plus-core/src/api
  ifneq ("$(wildcard $(TRYDIR)/m64p_types.h)","")
    CFLAGS += -I$(TRYDIR)
  else
    TRYDIR = /usr/local/include/mupen64plus
    ifneq ("$(wildcard $(TRYDIR)/m64p_types.h)","")
      CFLAGS += -I$(TRYDIR)
    else
      TRYDIR = /usr/include/mupen64plus
      ifneq ("$(wildcard $(TRYDIR)/m64p_types.h)","")
        CFLAGS += -I$(TRYDIR)
      else
        $(error Mupen64Plus API header files not found! Use makefile parameter APIDIR to force a location.)
      endif
    endif
  endif
endif

# set shell function names
CC      ?= gcc
CXX     ?= g++
LD      ?= g++
INSTALL ?= install
ifeq ($(OS),OSX)
  STRIP	?= strip -x 
else
  STRIP	?= strip -s
endif

# set special flags for given Makefile parameters
ifeq ($(DEBUG),1)
  CFLAGS += -g
  STRIP = true # disable binary strip
endif

# set installation options
ifeq ($(PREFIX),)
  PREFIX := /usr/local
endif
ifeq ($(LIBDIR),)
  LIBDIR := $(PREFIX)/lib/mupen64plus
endif

SRCDIR = ../../src
OBJDIR = _obj

# list of source files to compile
SOURCE = \
	$(SRCDIR)/rsp.cpp \
	$(SRCDIR)/rsp_opinfo.cpp \
	$(SRCDIR)/rsp_recomp.cpp \
	$(SRCDIR)/rsp_dasm.cpp \
	$(SRCDIR)/main_rsp.cpp 

# generate a list of object files build, make a temporary directory for them
OBJECTS := $(patsubst $(SRCDIR)/%.c, $(OBJDIR)/%.o, $(filter %.c, $(SOURCE)))
OBJECTS += $(patsubst $(SRCDIR)/%.cpp, $(OBJDIR)/%.o, $(filter %.cpp, $(SOURCE)))
OBJDIRS = $(dir $(OBJECTS))
$(shell mkdir -p $(OBJDIRS))

# build targets
TARGET = mupen64plus-rsp-z64.$(SO_EXTENSION)

targets:
	@echo "Mupen64Plus-rsp-z64 makefile. "
	@echo "  Targets:"
	@echo "    all           == Build Mupen64Plus rsp-hle plugin"
	@echo "    clean         == remove object files"
	@echo "    rebuild       == clean and re-build all"
	@echo "    install       == Install Mupen64Plus rsp-hle plugin"
	@echo "    uninstall     == Uninstall Mupen64Plus rsp-hle plugin"
	@echo "  Options:"
	@echo "    BITS=32       == build 32-bit binaries on 64-bit machine"
	@echo "    APIDIR=path   == path to find Mupen64Plus Core headers"
	@echo "  Install Options:"
	@echo "    PREFIX=path   == install/uninstall prefix (default: /usr/local)"
	@echo "    LIBDIR=path   == path to install plugin libraries (default: PREFIX/lib/mupen64plus)"
	@echo "    DESTDIR=path  == path to prepend to all installation paths (only for packagers)"
	@echo "  Debugging Options:"
	@echo "    DEBUG=1       == add debugging symbols"

all: $(TARGET)

install: $(TARGET)
	$(INSTALL) -d -v "$(DESTDIR)$(LIBDIR)"
	$(INSTALL) -m 0644 $(TARGET) "$(DESTDIR)$(LIBDIR)"

uninstall:
	rm -f "$(DESTDIR)$(LIBDIR)/$(TARGET)"

clean:
	rm -rf ./_obj/* $(TARGET)

rebuild: clean all

# build rules

$(TARGET): $(OBJECTS)
	$(CXX) $(SHARED) $^ $(LDFLAGS) -o $@
	$(STRIP) $@

$(OBJDIR)/%.o: $(SRCDIR)/%.c
	$(CC) -o $@ $(CFLAGS) -c $<

$(OBJDIR)/%.o: $(SRCDIR)/%.cpp
	$(CXX) -o $@ $(CFLAGS) -c $<

