# FT232H-FPGA-DDS-interface
FPGA controls a DDS, AD9959 for fast and precisely timed generation of frequencies upto 250 MHz.

To control the precisely timed generation of RF, an FPGA is utilized to control a DDS. The FPGA communicates using 4-lane SPI bus to the AD9959 in order to drive the DDS in various modes
available, eg. linear sweep, single tone as described here [https://www.analog.com/media/en/technical-documentation/data-sheets/AD9959.pdf].

In order to synchronize the DDS clock with the SPI clock, we generate the clock from the FPGA that is fed to the DDS ref clock. The SPI clock runs at 125 MHz which takes 2 clocks (8ns) 
for transfer of 1 Byte of data in the 4-lane SPI mode.

The FPGA used for this purspose is CMOD A7 from Digilent [https://reference.digilentinc.com/reference/programmable-logic/cmod-a7/start]. Any other FPGA can be used, I picked this one mainly because it was cheap + it has a lot block RAM.

The frequency profiles and mode settings are stored in the BRAM of the FPGA which is transferred on demand at a very high speed with hardware timing which is the key requirement for
generation of precisely timed RF.

These frequency profiles and the mode data are transferred from a computer. To match with the timings, we need to transfer the data at a very high speed to the FPGA. For this, I used
an FT232H board to obtain full speed USB 2.0 communication speed (480Mbps). The FT232H chip is configured to run in a synchronous parallel data transfer mode. In this mode 8 pins are toggled at 60 MHz giving a maximum of 480 Mbps theoretical speed. I obtain a speed around 30-40 MBps (MBytes/s) depending on the payload which is good enough. The FPGA is configured to 
read the FT232H data synchronously using the clock from the FT board. The data is then transferred to the BRAM (which again, runs with the FT clock). BRAM is used in dual port mode which allows for reading of the data at a different clock (125MHz, this is the clock used to transfer the data to DDS). Since, the data is transferred only in the beginning, that leaves no conflict between when the DDS is controlled. The depth of the BRAM is above 28000 words (each word containing 64bits). In such setting, data for about 56000 new frequencies
can be transfered in about 11ms.

The schematic of the fT232H + FPGA + DDS is depicted [here](https://github.com/mdnoaman/FT232H-FPGA-DDS-interface/blob/80740cd9f6ae0a7318a6581928db9e9b108dc234/AD9959_FPGA_FT232H_schematics.png)

I set up two different modes of operation for the driving of the DDS:

Mode 1. The idea is to generate fast jumping of frequency between multiple values. In such mode, after detecting a TTL trigger, the FPGA starts to drive the DDS to drive different 
frequencies for a predefined time. In particular, we begin with a single frequency which starts to "split" between two by jumping up and down about the center frequency. The initial splitting gap is very small which is linearly increased so that the splitting becomes very smooth. After certain predefined time, the splitting rate can be changed. These steps woudl make the final splitting profile look pretty much non-linear in a piecemeal linear fashion. After certain time or TTL event, further splitting can be started. The splitting can reversibly merg back or even cross over. Such frequency splitting has immediate usage in cold atom experiments where atoms trapped in a so called dipole trap can be split and merged (or even collided). This [file](https://github.com/mdnoaman/FT232H-FPGA-DDS-interface/blob/main/Scope_24.png) depicts how the splitting occurs from two to 4 frequency values. The overall splitting profile is measured in this [graph](https://github.com/mdnoaman/FT232H-FPGA-DDS-interface/blob/a74002547db240fff604762b1c38133e18d42f81/FPGA-DDS-output-20200708.png).

Mode2. In this mode the RF is swept up and down rapidly in a general shape, called as sawtooth sweep. After generation of certain number of such sawtooth sweeps, an entirely new sawtooth profile can be initiated. The sequence can be extended for generation of 9000 completely new sawtooth profiles. The sweitching from one profile to the next is also achieved by external trigger. However, the external trigger only comes in effect when the onging sweep is completed. This is deliberately done so that the unwanted jump in the frequency is avoided. These images [here](https://github.com/mdnoaman/FT232H-FPGA-DDS-interface/blob/00bc1a16752d3964d5ff7854c399dfa53ca81b0a/IMG_20210429_164436.jpg), [here](https://github.com/mdnoaman/FT232H-FPGA-DDS-interface/blob/00bc1a16752d3964d5ff7854c399dfa53ca81b0a/LeCroy--00030.jpg) show how the frequency sweeps for different durations/external triggers.






