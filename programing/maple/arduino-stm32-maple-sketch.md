# arduino-stm32-maple库结构分析

顶层目录结构：

```shell
.
├── cores #内含maple重新实现的STM32硬件操作库和中断向量表，以及部分二次包装的库函数
├── libraries #内含实例，写应用时可以参考
├── system #内含头文件，以及调试和测试用的一些脚本
└── variants #各个板子特定的文件，连接脚本，启动文件，板级初始化函数
```

core目录和variants目录比较重要

variants目录（generic_stm32f103r）：

```shell
.
├── board
│   └── board.h
├── board.cpp
├── ld
│   ├── bootloader.ld
│   ├── common.inc
│   ├── extra_libs.inc
│   ├── flash.ld
│   ├── jtag.ld
│   ├── mem-flash.inc
│   ├── mem-jtag.inc
│   ├── mem-ram.inc
│   ├── ram.ld
│   ├── stm32f103rb_bootloader.ld
│   ├── stm32f103rb.ld
│   ├── stm32f103rc_bootloader.ld
│   ├── stm32f103rc.ld
│   ├── stm32f103re.ld
│   └── vector_symbols.inc
├── pins_arduino.h
├── variant.h
└── wirish
    ├── boards.cpp
    ├── boards_setup.cpp
    ├── start_c.c
    ├── start.S
    └── syscalls.c
```



core目录：

```shell
.
└── maple
    ├── Arduino.h
    ├── avr
    │   ├── dtostrf.c
    │   ├── dtostrf.h
    │   ├── interrupt.h
    │   └── pgmspace.h
    ├── bit_constants.h
    ├── bits.h
    ├── boards.h
    ├── boards_private.h
    ├── Client.h
    ├── cxxabi-compat.cpp
    ├── ext_interrupts.cpp
    ├── ext_interrupts.h
    ├── HardwareSerial.cpp
    ├── HardwareSerial.h
    ├── HardwareTimer.cpp
    ├── HardwareTimer.h
    ├── hooks.c
    ├── io.h
    ├── IPAddress.cpp
    ├── IPAddress.h
    ├── itoa.c
    ├── itoa.h
    ├── libmaple
    │   ├── adc.c
    │   ├── adc_f1.c
    │   ├── bkp_f1.c
    │   ├── dac.c
    │   ├── dma.c
    │   ├── dma_f1.c
    │   ├── exc.S
    │   ├── exti.c
    │   ├── exti_f1.c
    │   ├── flash.c
    │   ├── fsmc_f1.c
    │   ├── gpio.c
    │   ├── gpio_f1.c
    │   ├── i2c.c
    │   ├── i2c_f1.c
    │   ├── iwdg.c
    │   ├── nvic.c
    │   ├── pwr.c
    │   ├── rcc.c
    │   ├── rcc_f1.c
    │   ├── spi.c
    │   ├── spi_f1.c
    │   ├── stm32f1
    │   │   └── performance
    │   │       ├── isrs.S
    │   │       └── vector_table.S
    │   ├── systick.c
    │   ├── timer.c
    │   ├── timer_f1.c
    │   ├── usart.c
    │   ├── usart_f1.c
    │   ├── usart_private.c
    │   ├── usb
    │   │   ├── README
    │   │   ├── rules.mk
    │   │   ├── stm32f1
    │   │   │   ├── usb.c
    │   │   │   ├── usb_cdcacm.c
    │   │   │   └── usb_reg_map.c
    │   │   └── usb_lib
    │   │       ├── usb_core.c
    │   │       ├── usb_init.c
    │   │       ├── usb_mem.c
    │   │       └── usb_regs.c
    │   └── util.c
    ├── main.cpp
    ├── new.cpp
    ├── Printable.h
    ├── Print.cpp
    ├── Print.h
    ├── pwm.cpp
    ├── pwm.h
    ├── rules.mk
    ├── sdio.cpp
    ├── Server.h
    ├── stm32f1
    │   ├── util_hooks.c
    │   ├── wiring_pulse_f1.cpp
    │   ├── wirish_debug.cpp
    │   └── wirish_digital_f1.cpp
    ├── Stream.cpp
    ├── Stream.h
    ├── tone.cpp
    ├── tone.h
    ├── Udp.h
    ├── usb_serial.cpp
    ├── usb_serial.h
    ├── WCharacter.h
    ├── wiring_private.h
    ├── wiring_pulse.h
    ├── wirish_analog.cpp
    ├── wirish_constants.h
    ├── wirish_debug.h
    ├── wirish_digital.cpp
    ├── wirish.h
    ├── wirish_math.cpp
    ├── wirish_math.h
    ├── wirish_shift.cpp
    ├── wirish_time.cpp
    ├── wirish_time.h
    ├── wirish_types.h
    ├── WProgram.h
    ├── WString.cpp
    └── WString.h
```

system/libmaple目录：

```shell
.
├── dma_private.h
├── exti_private.h
├── i2c_private.h
├── include
│   ├── libmaple
│   │   ├── adc.h
│   │   ├── bitband.h
│   │   ├── bkp.h
│   │   ├── dac.h
│   │   ├── delay.h
│   │   ├── dma_common.h
│   │   ├── dma.h
│   │   ├── exti.h
│   │   ├── flash.h
│   │   ├── fsmc.h
│   │   ├── gpio.h
│   │   ├── i2c_common.h
│   │   ├── i2c.h
│   │   ├── iwdg.h
│   │   ├── libmaple.h
│   │   ├── libmaple_types.h
│   │   ├── nvic.h
│   │   ├── pwr.h
│   │   ├── rcc.h
│   │   ├── ring_buffer.h
│   │   ├── scb.h
│   │   ├── sdio.h
│   │   ├── spi.h
│   │   ├── stm32.h
│   │   ├── syscfg.h
│   │   ├── systick.h
│   │   ├── timer.h
│   │   ├── usart.h
│   │   ├── usb_cdcacm.h
│   │   ├── usb.h
│   │   └── util.h
│   └── util
│       └── atomic.h
├── rcc_private.h
├── rules.mk
├── spi_private.h
├── stm32f1
│   ├── include
│   │   └── series
│   │       ├── adc.h
│   │       ├── dac.h
│   │       ├── dma.h
│   │       ├── exti.h
│   │       ├── flash.h
│   │       ├── gpio.h
│   │       ├── i2c.h
│   │       ├── nvic.h
│   │       ├── pwr.h
│   │       ├── rcc.h
│   │       ├── spi.h
│   │       ├── stm32.h
│   │       ├── timer.h
│   │       └── usart.h
│   └── rules.mk
├── stm32f2
│   ├── include
│   │   └── series
│   │       ├── adc.h
│   │       ├── dac.h
│   │       ├── dma.h
│   │       ├── exti.h
│   │       ├── flash.h
│   │       ├── gpio.h
│   │       ├── nvic.h
│   │       ├── pwr.h
│   │       ├── rcc.h
│   │       ├── spi.h
│   │       ├── stm32.h
│   │       ├── timer.h
│   │       └── usart.h
│   └── rules.mk
├── stm32_private.h
├── timer_private.h
├── usart_private.h
└── usb
    ├── README
    ├── rules.mk
    ├── stm32f1
    │   ├── usb_lib_globals.h
    │   └── usb_reg_map.h
    └── usb_lib
        ├── usb_core.h
        ├── usb_def.h
        ├── usb_init.h
        ├── usb_lib.h
        ├── usb_mem.h
        ├── usb_regs.h
        └── usb_type.h
```

system/maple的抽象层级为：

**system/libmaple/include/libmaple：**这里的头文件提供对外的libmaple 硬件操作API接口

**system/libmaple/stm32f1/include/series：**这里的头文件主要是对硬件寄存器的结构体组织