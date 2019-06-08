## Setup the Arduino IDE for picorv32

This hardware and code was build and tested using icestudio (icestorm) and 
an Alhambra II board. It should work out of the box or be fairly easy to
adopt to other similar boards (e.g. TinyFPGA, etc.).

### 1.) unpack (clone) this archive (repo) to ~/sketchbook/hardware/

    $ cd ~/sketchbook/hardware/
    $ wget https://github.com/drtrigon/fpgarduino-icestorm/archive/master.zip
    $ unzip master.zip
    $ mv fpgarduino-icestorm-master/AlhambraII .
    $ rm -rf fpgarduino-icestorm-master

or clone

    $ cd ~/sketchbook/hardware/
    $ git clone https://github.com/drtrigon/fpgarduino-icestorm
    $ mv fpgarduino-icestorm/AlhambraII .
    $ rm -rf fpgarduino-icestorm


### 2.) follow https://github.com/cliffordwolf/picorv32#building-a-pure-rv32i-toolchain

    # Ubuntu packages needed:
    sudo apt-get install autoconf automake autotools-dev curl libmpc-dev \
        libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo \
        gperf libtool patchutils bc zlib1g-dev git libexpat1-dev

    sudo mkdir ~/sketchbook/hardware/AlhambraII/picorv32/tools/riscv32i
    sudo chown $USER ~/sketchbook/hardware/AlhambraII/picorv32/tools/riscv32i

    git clone https://github.com/riscv/riscv-gnu-toolchain riscv-gnu-toolchain-rv32i
    cd riscv-gnu-toolchain-rv32i
    git checkout c3ad555
    git submodule update --init --recursive

    mkdir build; cd build
    ../configure --with-arch=rv32i --prefix=$HOME/sketchbook/hardware/AlhambraII/picorv32/tools/riscv32i
    make -j$(nproc)

Now the Arduino IDE replaces the Makefile used before to compile and upload
the firmware.

This is like uploading a "bootloader" to Arduino/AVR. Uploading a "sketch"
via bootloader as the Arduino IDE usualy does is done within the FPGArduino
project e.g.

In Arduino IDE "Tools" pulldown menu, select:

    Board: Alhambra II Generic picorv32
    Port: (of the 2 available it's typically the second one, e.g. /dev/ttyUSB1)
    Programmer: iceprog (Icestudio/Icestorm)

Note: The port needs to be set if using the Serial Montor only. For the programming it has no effect.
Note: Forgetting to set the correct programmer results in a java error.


### 3.) OPTIONAL: use iceprog directly from installed icestudio and its toolchain:

    cd ~/sketchbook/hardware/AlhambraII/picorv32/tools/iceprog/
    rm iceprog
    ln -s ~/.icestudio/apio/packages/toolchain-icestorm/bin/iceprog iceprog

or alternatively you can set the following line in platform.txt:

    tools.iceprog.path={runtime.platform.path}/../../../../.icestudio/apio/packages/toolchain-icestorm/bin


### 4.) INFORMATION: The bitstream to use is stored in $HOME/sketchbook/hardware/AlhambraII/picorv32/variants/AlhambraII/

The bitstream can be programmed using: $ iceprog hardware.bin

This bitstream was derived from GPIO_ADC-recent-picorv32.zip using icestudio 0.3.3 (linux64.AppImage).

What does work:
* Verify (building)
* Upload
* Serial Monitor using any baudrate, default is 115200 (needs setting the correct port)
* Arduino IDE examples:
  * 01.Basics
    * BareMinimum: (compiles)
    * Blink: pinMode dummy for leds, digitalWrite on leds also using full 8-bit uint8_t, delay
    * DigitalReadSerial: Serial.println, Serial.begin all baudrates, pinMode (0-7), digitalRead (0-15)
    * AnalogReadSerial: analogRead (A0, 4Hz, bits increased from 8 to 10 by multiplying with 4)
    * ReadAnalogVoltage: (voltage calculation wrong 5.0->3.3)
    * (TODO: Fade - needs PWM)
  * 02.Digital
    * BlinkWithoutDelay: millis, micros, (delay refactored to use micros)
    * Button: (works after changing ledPin 13->LED_BUILTIN)
    * Debounce: (works after changing ledPin 13->LED_BUILTIN)
    * StateChangeDetection: (works after changing ledPin 13->LED_BUILTIN)
    * (TODO: DigitalInputPullup, tone...)
  * 03.Analog
    * AnalogInput: (works after changing ledPin 13->LED_BUILTIN)
    * Smoothing: (works)
    * (TODO: AnalogInOutSerial - PWM, Calibration - PWM, Fading)
  * 04.Communication
    * ASCIITable: (works)
    * Graph: (works)
    * VirtualColorMixer: (works with A0 only, A1 and A2 read garbage)
    * (TODO: Dimmer - PWM+serialread, PhysicalPixel - serialread, ReadASCIIString - serialread, SerialCallResponse - serialread, SerialCallResponseASCII - serialread, SerialEvent)
  * 05.Control
    * IfStatementConditional: (works after changing ledPin 13->LED_BUILTIN)
    * (TODO: Array - digitalWrite, ForLoopIteration - digitalWrite, switchCase - map, switchCase2 - serialread, WhileStatementConditional - PWM)
  * 08.Strings
    * (TODO: add support for String class)
  * 11.ArduinoISP
    * (TODO: BitBanged/software SPI, USE_OLD_STYLE_WIRING, RESET/PIN_MOSI/PIN_MISO/PIN_SCK/etc. to use gpio (0-7), delayMicroseconds)
  * (TODO: PJON ???)


### TODO:
    TODO: serial, PWM, gpio with pull-up, ArduinoISP example (software bit-bang SPI?)
    TODO: implement/test gpio output/write mode
    TODO: assert warning on non supported stuff

    (TODO: serial input (serial is not expected to work fully, e.g. 'available' funcs etc.))
    (TODO: add travis integration in order to automatically check whether all examples compile and link properly - functional check has to be done manually)
    (TODO: virtual functions around Print(::write) do not work...)
    (TODO: remove "picorv32: work-a-round" and adopt to arduino template where possible)


### Further info:
* https://github.com/riscv/riscv-tools
* https://github.com/riscv/riscv-wiki/wiki/RISC-V-Software-Status
* https://github.com/FPGAwars/Alhambra-II-FPGA/tree/master/examples/picorv32/picosoc (old, including icestudio project)
* https://github.com/cliffordwolf/picorv32 (recent)
* http://www.nxlab.fer.hr/fpgarduino/
* https://github.com/arduino/Arduino/wiki/Arduino-IDE-1.5-3rd-party-Hardware-specification
* https://github.com/f32c/arduino/issues/32
* https://github.com/FPGAwars/icestudio/issues/321