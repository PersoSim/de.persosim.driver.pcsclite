CC       = gcc
CFLAGS   = -g -std=gnu99 -fpic

UNAME := $(shell uname -s)
ifeq ($(UNAME),Linux)
	OS=LINUX
endif
ifeq ($(UNAME),Darwin)
	OS=OSX
endif

DRIVERNAME = ifdhPersoSim

ifeq ($(OS),LINUX)
	INCLUDE  = -I. -I/usr/include/PCSC
	DEFS     = -DPCSC_DEBUG=1 #-DATR_DEBUG=1
	LIBNAME = $(DRIVERNAME).so
else ifeq ($(OS),OSX)
	INCLUDE  = -I. -IMacOSX
	DEFS     = -DPCSC_DEBUG=1 -DUSE_SYSLOG -DRESPONSECODE_DEFINED_IN_WINTYPES_H
	VPATH    = MacOSX
	LIBNAME = $(DRIVERNAME).dylib
endif


DEPENDFILE = .depend

#LDFLAGS  = -lm
#LDLIBS  = -lm

ifeq ($(OS),LINUX)
	OBJ     := $(filter-out %test.o %Test.o, $(patsubst %.c,%.o,$(wildcard *.c)))
else ifeq ($(OS),OSX)
	OBJ     := $(filter-out %test.o %Test.o, $(patsubst %.c,%.o,$(wildcard *.c) $(wildcard MacOSX/*.c)))
endif



all: $(LIBNAME) tests

# generate and pull in dependency info
$(DEPENDFILE): *.c *.h
	$(CC) $(INCLUDE) -MM $+ > $(DEPENDFILE)
-include $(DEPENDFILE)

clean:
	rm -f *.o $(LIBNAME) $(DEPENDFILE) *Test
	rm -f MacOSX/*.o
	

	
install: install_$(OS)

install_LINUX: $(LIBNAME)
	sudo cp Linux/reader.conf /etc/reader.conf.d/$(DRIVERNAME).conf
	sudo cp $(LIBNAME) /usr/lib/pcsc/drivers/serial/

install_OSX: $(LIBNAME)
	sudo cp MacOSX/reader.conf /etc/reader.conf
	mkdir -p /usr/local/libexec/SmartCardServices/drivers/$(DRIVERNAME).bundle/Contents/MacOS
	sudo cp $(LIBNAME) /usr/local/libexec/SmartCardServices/drivers/$(DRIVERNAME).bundle/Contents/MacOS
	sudo cp MacOSX/Info.plist /usr/local/libexec/SmartCardServices/drivers/$(DRIVERNAME).bundle/Contents/Info.plist

uninstall: uninstall_$(OS)

uninstall_OSX:
	sudo rm -f /etc/$(DRIVERNAME).conf
	sudo rm -rf /usr/local/libexec/SmartCardServices/drivers/$(DRIVERNAME).bundle
	
uninstall_LINUX:
	sudo rm -f /etc/reader.conf.d/$(DRIVERNAME).conf
	sudo rm -f /usr/lib/pcsc/drivers/serial/$(LIBNAME)
	
tests: hexStringTest

runtests: tests
	./hexStringTest


MacOSX/%.o : MacOSX/%.c
	$(CC) $(CFLAGS) -c $< $(INCLUDE) $(DEFS) -o $@

MacOSX/% : MacOSX/%.c
	$(CC) $(CFLAGS) $^ -o $@ $(INCLUDE) 
	
%.so: $(OBJ)
	$(CC) $(CFLAGS) -shared $(OBJ) -o $@
	
%.dylib: $(OBJ)
	$(CC) $(CFLAGS) -shared $(OBJ) -o $@

%.o : %.c
	$(CC) $(CFLAGS) -c $< $(INCLUDE) $(DEFS)

% : %.o hexString.o 
	#TODO remove the hardcoded dependecy above
	$(CC) $(CFLAGS) $^ -o $@ $(INCLUDE) 

.PHONY: clean test

print-%  : ; @echo $* = $($*)
