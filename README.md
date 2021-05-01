# FT232H-FPGA-DDS-interface
FPGA controls a DDS, AD9959 for fast and precisely timed generation of frequencies upto 250 MHz.

To control the precisely timed generation of RF, an FPGA is utilized to control a DDS. The FPGA communicates using 4-lane SPI bus to the AD9959 in order to drive the DDS in various modes
available, eg. linear sweep, single tone as described here [https://www.analog.com/media/en/technical-documentation/data-sheets/AD9959.pdf].

In order to synchronize the DDS clock with the SPI clock, we generate the clock from the FPGA that is fed to the DDS ref clock. The SPI clock runs at 125 MHz which takes 2 clocks (8ns) 
for transfer of 1 Byte of data in the 4-lane SPI mode.


