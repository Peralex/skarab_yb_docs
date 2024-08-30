# Tutorial 5: SKARAB ADC Synchronous Data Acquisition

## Introduction ##
This tutorial explains how a Simulink design can be created and controlled to perform synchronised ADC sampling among the channels of one or more SKARAB ADC boards that are installed in one or more SKARAB systems. The Simulink design is created using the CASPER Toolflow and the software control of the synchronised sampling is performed using the casperfpga Python library.

## SKARAB ADC Board Overview ##
The SKARAB ADC board has four analogue channels which can each be sampled at a maximum rate of 3 GSPS. The sampling is either performed by two dual-channel TI ADC32RF80 ADC ICs or two dual-channel TI ADC32RF45 ADC ICs. The board also contains a PLL which locks to a reference clock to derive and provide sampling clocks to the ADC ICs. Both the ADC32RF45 and ADC32RF80 include DDCs, but only the ADC32RF45 allows the DDCs to be bypassed to provide the full bandwidth.

<p align="center">
<img src="figures/SkarabAdcBoard.png" width="300" />
</p>

<p align="center">
<img src="figures/AdcICs.png" width="650" />
</p>

There are two different bandwidth modes in which the SKARAB ADC board can be used: "Bypass Mode" and "DDC Mode". The differences between these modes are summarised in the following table:
Bandwidth Mode | Sampling Frequency | Decimation Factors | SKARAB ADC Board Support
--- | --- | --- | --- 
DDC Mode | 3 GSPS | 4, 8, 16, 32 | All SKARAB ADC Board Types
Bypass Mode | 2.8 GSPS | None (bypassed) | Only SKARAB ADC Boards with ADC32RF45

Note that other sampling frequencies are available using the PLL of the SKARAB ADC board and that other decimation factors are provided by the ADC ICs. Please contact Peralex Electronics for more information.

An update to SKARAB ADC firmware version 2.3 enables support for operating at 2.56 GSPS in both DDC Mode and Bypass Mode. In conjunction with SKARAB ADC firmware version 2.3, a Yellow Block firmware update adds support for decimate by 64 and 128. A 4/5 resampler (interpolate by 4, decimate by 5) converts a 2.56 GSPS sample rate to 2.048 GSPS. The updated table is as follows:

Bandwidth Mode | Sampling Frequency | Decimation Factors | SKARAB ADC Board Support
--- | --- | --- | --- 
DDC Mode | 3 GSPS, 2.56 GSPS, 2.048 GSPS | 4, 8, 16, 32, 64, 128 | All SKARAB ADC Board Types with SKARAB ADC firmware version 2.3 or later
Bypass Mode | 2.8 GSPS, 2.56 GSPS, 2.048 GSPS | None (bypassed) | Only SKARAB ADC Boards with ADC32RF45 with SKARAB ADC firmware version 2.3 or later

The SKARAB ADC board is available in three ordering variants. These ordering variants are summarised in the table below:
Ordering Variant | ADC ICs | PGA Support
--- | --- | ---
12533.02C | Two ADC32RF45 ICs | No 
12533.02E | Two ADC32RF80 ICs | Yes 
12533.02G | Two ADC32RF45 ICs | Yes 

Please contact Peralex Electronics for more information.

## SKARAB ADC Hardware Setup ##

It is possible to, not only perform synchronised sampling among the four channels of a single SKARAB ADC board, but also to perform synchronised sampling among all the channels of multiple SKARAB ADC boards.

In cases where synchronised sampling among only four channels are required, a single SKARAB ADC board will be installed in a SKARAB system. During operation, a synchronisation signal is sent from the Virtex-7 FPGA to a SKARAB ADC board to allow synchronised sampling among the four channels.

<p align="center">
<img src="figures/TestSetupBasic.png" width="550" />
</p>

In cases where synchronised sampling among more than four channels is required, one option is to install more SKARAB ADC boards in a SKARAB system. In this case, the synchronisation signal is daisy chained. Up to three SKARAB ADC boards can be installed in a system.  The first SKARAB ADC board in the chain is referred to as the "Master" while the rest are referred to as the "Slaves".

<p align="center">
<img src="figures/TestSetupTwelveCh.png" width="700" />
</p>

Another option to allow synchronised sampling among more than four channels is to use more than one SKARAB system. This allows the number of SKARAB ADC boards in the hardware setup to extend beyond three. In this case, the synchronisation signal is daisy chained from the SKARAB ADC board Master to the rest of the SKARAB ADC board Slaves among the SKARAB systems. Similarly, the SKARAB system containing the Master SKARAB ADC board is referred to as the "Master", while the remaining SKARAB systems are referred to as the "Slaves". The connections inside the SKARAB systems are made by Peralex Electronics while the connections outside will be made by the user of the hardware setup. The user will also terminate the synchronisation signal at the last system with 50 Ohm.

<p align="center">
<img src="figures/TestSetupAdvanced.png" width="800" />
</p>

As the number of channels that need to be synchronised increases (for example 80), the length of the daisy chained synchronisation signal increases as well, and may become too long. In these cases, the SKARAB SYNC multiplier is used.

<p align="center">
<img src="figures/SkarabSyncMultiplier.png" width="600" />
</p>

Contact Peralex Electronics for more details of these, and potentially other, hardware setups.



## SKARAB ADC Yellow Block ##
### Overview ###
Each SKARAB ADC board installed in a SKARAB system needs a corresponding Yellow Block in a Simulink design so that synchronised ADC sample data can be captured. There are two types of Yellow Blocks, namely: the DDC Mode SKARAB ADC Yellow Block and the Bypass Mode SKARAB ADC Yellow Block. The DDC Mode SKARAB ADC Yellow Block (named "skarab_adc4x3g_14" in the CASPER Toolflow) can be used along with any SKARAB ADC board variant since they all support the DDC bandwidth mode. The Bypass Mode SKARAB ADC Yellow Block (named "skarab_adc4x3g_14_byp" in the CASPER Toolflow) can only be used along with a SKARAB ADC board that supports the Bypass bandwidth mode.

### Master and Slaves ###
The clocking and synchronisation functions can only be performed by one of the SKARAB ADC Yellow Blocks in a Simulink design. The SKARAB ADC Yellow Block that performs these functions are known as the "Master" while the others are known as the "Slaves". If only one SKARAB ADC Yellow Block is used in the Simulink design, it needs to be set as the "Master". If there are more than one SKARAB ADC Yellow Blocks in the design, one needs to be set as the Master while the remaining ones are set as the Slaves. Also, if a SKARAB ADC board needs to be used as a Master in a hardware setup, its corresponding SKARAB ADC Yellow Block needs to be set as the Master too.

### Description of DDC Mode SKARAB ADC Yellow Block (skarab_adc4x3g_14) ###
This Yellow Block provides four 16-bit I/Q sample pairs from each of the four channels of the SKARAB ADC board as output (in parallel). For a 3 GSPS sampling frequency that is decimated by 4, the four sample pairs are provided at a rate of 187.5 MSPS:

[3 GSPS]/[decimate by 4 × 4 sample pairs per output clock cycle] = 187.5 MSPS

When decimated by 8, 16, 32, 64 or 128, the four sample pairs are provided at rates of 93.75, 46.875, 23.4375, 11.71875 or 5.859375 MSPS, respectively.

[2.56 GSPS]/[decimate by 4 × 4 sample pairs per output clock cycle] = 160 MSPS

When decimated by 8, 16, 32, 64 or 128, the four sample pairs are provided at rates of 80, 40, 20, 10 or 5 MSPS, respectively.

[2.048 GSPS]/[decimate by 4 × 4 sample pairs per output clock cycle] = 128 MSPS

When decimated by 8, 16, 32, 64 or 128, the four sample pairs are provided at rates of 64, 32, 16, 8 or 4 MSPS, respectively.

The Yellow Block parameters are as follows:
- Mezzanine slot: Mezzanine slot (0 to 3) of the SKARAB where the SKARAB ADC board is installed.
- Master/Slave: The Master performs clocking (generates adc_clk) and synchronisation functions. There should only be one Master SKARAB ADC Yellow Block in a Simulink design while the remaining (if any) need to be Slaves.
- Trigger: The two available options are "Output" or "Input". Specifies whether the trigger port on the SKARAB ADC Board is an output or an input. Depending on the build of the SKARAB ADC Board, trigger can either be an input to the mezzanine or an output from the mezzanine. When Trigger is configured as "Output", drive the "trigger_out" port of yellow block with the desired signal that you want output by the SKARAB ADC Board. Note that "trigger_out" is an INPUT to the yellow block. When Trigger is configured as "Input", use the "trigger_in" port of the yellow block to monitor the current state of the SKARAB ADC Board trigger port. Note that "trigger_in" is an OUTPUT of the yellow block.
- Decimation modes: The four available options are "4, 8, 16", "4, 8, 16, 32", "4, 8, 16, 32, 64" or "4, 8, 16, 32, 64, 128". The decimate-by-32, decimate-by-64 and decimate-by-128 modes require additional processing inside the FPGA which would consume unnecessary logic resources if they were not required. Thus, options are added to exclude them.
- 4/5 Resampler: The two available options are "Enable" or "Disable". Specifies whether to include the 4/5 Resampler in the FPGA firmware. The 4/5 Resampler is required to convert 2.56 GSPS to 2.048 GSPS. The 4/5 resampler requires additional processing inside the FPGA which would consume unnecessary logic resources if it was not required. Thus, an option was added to exclude it. 

The Yellow Block outputs are as follows:
- adc0_data_i_out&lt;X&gt; (where X: 0 to 3): Four successive 16-bit I samples from channel 0
- adc0_data_q_out&lt;X&gt; (where X: 0 to 3): Four successive 16-bit Q samples from channel 0
- adc1_data_i_out&lt;X&gt; (where X: 0 to 3): Four successive 16-bit I samples from channel 1
- adc1_data_q_out&lt;X&gt; (where X: 0 to 3): Four successive 16-bit Q samples from channel 1
- adc2_data_i_out&lt;X&gt; (where X: 0 to 3): Four successive 16-bit I samples from channel 2
- adc2_data_q_out&lt;X&gt; (where X: 0 to 3): Four successive 16-bit Q samples from channel 2
- adc3_data_i_out&lt;X&gt; (where X: 0 to 3): Four successive 16-bit I samples from channel 3
- adc3_data_q_out&lt;X&gt; (where X: 0 to 3): Four successive 16-bit Q samples from channel 3
- adc0_data_val_out: Channel 0 sample data valid signal
- adc1_data_val_out: Channel 1 sample data valid signal
- adc2_data_val_out: Channel 2 sample data valid signal
- adc3_data_val_out: Channel 3 sample data valid signal
- trigger_in: Current state of SKARAB ADC Board trigger input (when configured as an input)
- pps_in: Current state of SKARAB ADC Board PPS input

The Yellow Block inputs are as follows:
- trigger_out: Signal to drive on SKARAB ADC Board trigger output (when configured as an output)

### Description of Bypass Mode SKARAB ADC Yellow Block (skarab_adc4x3g_14_byp) ###
This Yellow Block provides sixteen 12-bit samples from each of the four channels of the SKARAB ADC board as output (in parallel). For a 2.8 GSPS sampling frequency, the sixteen samples are provided at a rate of 175 MSPS:

[2.8 GSPS]/[16 samples per output clock cycle] = 175 MSPS

[2.56 GSPS]/[16 samples per output clock cycle] = 160 MSPS

[2.048 GSPS]/[16 samples per output clock cycle] = 128 MSPS

The Yellow Block parameters are as follows:
- Mezzanine slot: Mezzanine slot (0 to 3) of the SKARAB where the SKARAB ADC board is installed.
- Master/Slave: The Master performs clocking (generates adc_clk) and synchronisation functions. There should only be one Master SKARAB ADC Yellow Block in a Simulink design while the remaining (if any) need to be Slaves.
- Trigger: The two available options are "Output" or "Input". Specifies whether the trigger port on the SKARAB ADC Board is an output or an input. Depending on the build of the SKARAB ADC Board, trigger can either be an input to the mezzanine or an output from the mezzanine. When Trigger is configured as "Output", drive the "trigger_out" port of yellow block with the desired signal that you want output by the SKARAB ADC Board. Note that "trigger_out" is an INPUT to the yellow block. When Trigger is configured as "Input", use the "trigger_in" port of the yellow block to monitor the current state of the SKARAB ADC Board trigger port. Note that "trigger_in" is an OUTPUT of the yellow block.
- 4/5 Resampler: The two available options are "Enable" or "Disable". Specifies whether to include the 4/5 Resampler in the FPGA firmware. The 4/5 Resampler is required to convert 2.56 GSPS to 2.048 GSPS. The 4/5 resampler requires additional processing inside the FPGA which would consume unnecessary logic resources if it was not required. Thus, an option was added to exclude it.

The Yellow Block outputs are as follows:
- adc0_data_out&lt;X&gt; (where X: 0 to 15): Sixteen successive 12-bit samples from channel 0
- adc1_data_out&lt;X&gt; (where X: 0 to 15): Sixteen successive 12-bit samples from channel 1
- adc2_data_out&lt;X&gt; (where X: 0 to 15): Sixteen successive 12-bit samples from channel 2
- adc3_data_out&lt;X&gt; (where X: 0 to 15): Sixteen successive 12-bit samples from channel 3
- adc0_data_val_out: Channel 0 sample data valid signal
- adc1_data_val_out: Channel 1 sample data valid signal
- adc2_data_val_out: Channel 2 sample data valid signal
- adc3_data_val_out: Channel 3 sample data valid signal
- trigger_in: Current state of SKARAB ADC Board trigger input (when configured as an input)
- pps_in: Current state of SKARAB ADC Board PPS input
  
The Yellow Block inputs are as follows:
- trigger_out: Signal to drive on SKARAB ADC Board trigger output (when configured as an output)

## Simulink Design Clocking Considerations ##

Please note the User IP Clock source parameter of the SKARAB Yellow Block. Either sys_clk or adc_clk can be selected to clock the User IP. The implications of choosing either clock source follows:

- adc_clk is a clock generated by a SKARAB ADC Yellow Block with a frequency that matches its output clock rate. Its frequency depends on the sample rate of the ADC and the operating mode. When using adc_clk, the Simulink Design User IP will not be clocked until initialisation of the SKARAB ADC board along with its corresponding SKARAB ADC Yellow Block. Depending on the SKARAB ADC Yellow Block mode, sample rate and decimation rate, the behaviour of the data valid signal changes.
  - Bypass mode:
    - Full sample rate (2.8GSPS) and 2.56GSPS: Data valid is constantly high and therefore does not need to be monitored after initial assertion.
    - 2.048GSPS: Since the 4/5 resampler is enabled, data valid is high for four clock cycles out of five and so must be respected.
  - DDC mode:
    - Full sample rate (3GSPS) and 2.56GSPS:
      - Decimate by 4: Data valid is contantly high and therefore does not need to be monitored after initial assertion.
      - Decimate by 8: Data valid is high in blocks of 21 clocks out of every 42 clocks and so must be respected.
      - Decimate by 16: Data valid is high in blocks of 14 clocks out of every 56 clocks and so must be respected.
      - Decimate by 32: Data valid is high for a single clock cycle out of every 8 clocks and so must be respected.
      - Decimate by 64: Data valid is high for a single clock cycle out of every 16 clocks and so must be respected.
      - Decimate by 128: Data valid is high for a single clock cycle out of every 32 clocks and so must be respected.
    - 2.048GSPS:
      - Decimate by 4: Since the 4/5 resampler is enabled, data valid is high for four clock cycles out of five and so must be respected.
      - Decimate by 8: Data valid is high in blocks of 17 clocks out of every 42 clocks and so must be respected.
      - Decimate by 16: Data valid is high in blocks of 11 clocks out of every 55 clocks and so must be respected.
      - Decimate by 32: Data valid is high for a single clock cycle at a rate of four clock pulses out of every 40 clocks and so must be respected.
      - Decimate by 64: Data valid is high for a single clock cycle at a rate of four clock pulses out of every 80 clocks and so must be respected.
      - Decimate by 128: Data valid is high for a single clock cycle at a rate of four clock pulses out of every 160 clocks and so must be respected.
- sys_clk is a free running clock which has no relation to the sampling rate of the SKARAB ADC board. By necessity, the selected sys_clk frequency must be greater than the SKARAB ADC Yellow Block sample data output rate. Thus, the user needs to respect the data valid signals of the SKARAB ADC Yellow Block on subsequent processing blocks. This clocking scheme allows subsequent signal processing to run at a higher clock rate than the SKARAB ADC Yellow Block data output rate. However, not all Yellow Blocks support clock gating (data valid) signals. Also, an advantage of sys_clk is that the clocking of the Simulink Design User IP is independent of the initialisation of any of the SKARAB ADC boards or their corresponding SKARAB ADC Yellow Blocks.

<p align="center">
<img src="figures/UserIpClock.png" width="600" />
</p>

## Example Designs ##

The following example designs are provided to demonstrate synchronised ADC sampling in a hardware setup that contains one or more SKARAB ADC boards (installed in a set of one or more SKARAB systems):
- Four Simulink design models, one of which will be used to create an fpg file to upload to each of the SKARAB systems. The design model to use depends on the number of SKARAB ADC boards per SKARAB (one or two) as well as the bandwidth mode in which they need to be used (DDC or Bypass).
- A Python test script (using the casperfpga library) which uploads the fpg file (created from any of the four Simulink design models) to the SKARABs and then controls them to perform synchronised sampling. The usage of this script is described in its comments.
- Two ADC data plotting Matlab scripts which overlays the captured ADC data from all sampled channels. The usage of these scripts are described in their comments.

Each of the four Simulink design models consists of the following components:
- One or two SKARAB ADC Yellow Blocks used to perform synchronised sampling among several SKARAB ADC boards (in DDC or Bypass mode).
  
<p align="center"><img src="figures/SkarabAdcYbs.png" width="450" /></p>

- Multiple Snapshot Yellow Blocks which are used to capture the ADC sample data recovered by the SKARAB ADC Yellow Blocks.

<p align="center"><img src="figures/SnapshotYbs.png" width="300" /></p>

- Several components which are used to provide a GbE and a 40 GbE interface to the design so that the design can be controlled from the Python test script. 

<p align="center"><img src="figures/FgbeAndGbeYbs.png" width="750" /></p>

- The Xilinx System Generator, which is a plug-in to Simulink that enables designers to develop high-performance DSP systems for Xilinx FPGAs such as the Virtex-7 FPGA on the SKARAB.
- The SKARAB Yellow Block, which is used to create the Simulink design for the SKARAB platform. 

<p align="center"><img src="figures/SkarabAndXilinxYbs.png" width="400" /></p>

The user of the Simulink design needs to do the following before creating an fpg file from it:
- Set the Mezzanine Slot of each SKARAB ADC Yellow Block to indicate where its corresponding SKARAB ADC board is installed.
- Set the User IP Clock Source of the design.
- If a DDC Mode SKARAB ADC Yellow Block is used in the design, set the decimation modes that should be supported.
- Set whether trigger is an input or output.
- Set whether the 4/5 resampler is needed in the design or not.
- Ensure that there is only one Master SKARAB ADC Yellow Block in the design.

The links to the Simulink design models are:
- [Simulink design model with one DDC mode SKARAB ADC Yellow Block](https://github.com/Peralex/mlib_devel/tree/master/jasper_library/test_models/test_skarab_adc.slx)

- [Simulink design model with one Bypass mode SKARAB ADC Yellow Block](https://github.com/Peralex/mlib_devel/tree/master/jasper_library/test_models/test_skarab_adc_byp.slx)

- [Simulink design model with two DDC mode SKARAB ADC Yellow Blocks](https://github.com/Peralex/mlib_devel/tree/master/jasper_library/test_models/test_skarab_adc_x2.slx)

- [Simulink design model with two Bypass mode SKARAB ADC Yellow Blocks](https://github.com/Peralex/mlib_devel/tree/master/jasper_library/test_models/test_skarab_adc_byp_x2.slx)

The link to the Python test script is:

- [Python test script](https://github.com/Peralex/mlib_devel/tree/master/jasper_library/test_models/scripts/skarab_adc_test/skarab_adc_test.py)

The links to the ADC data plotting MATLAB scripts are:

- [DDC mode ADC data plotting MATLAB script](https://github.com/Peralex/mlib_devel/tree/master/jasper_library/test_models/scripts/skarab_adc_test/plot_adc_data_ddc.m)

- [Bypass mode ADC data plotting MATLAB script](https://github.com/Peralex/mlib_devel/tree/master/jasper_library/test_models/scripts/skarab_adc_test/plot_adc_data_byp.m)


## SKARAB ADC Firmware Version Requirements ##
A SKARAB ADC board needs to be programmed with firmware version 2.3 to be compatible with the latest casperfgpa functions and the SKARAB ADC Yellow Blocks. The skarab_adc_reconfig.py Python script can be used to update the SKARAB ADC board firmware to this version. Please read the comments in the header of this script for an explanation of how to use it.

Two bin file variants of the SKARAB ADC firmware version 2.3 are provided: EMB124901U8R2_DDC.bin and EMB124901U8R2_BYP.bin. The only difference between them is the default bandwidth mode in which they configure the SKARAB ADC board on startup. EMB124901U8R2_DDC.bin configures it in DDC mode on startup, while EMB124901U8R2_BYP.bin configures it in Bypass mode on startup. When only using a SKARAB ADC board in a single bandwidth mode, the appropriate bin file can be chosen so that it is not required to configure the SKARAB ADC board after startup using a casperfpga function.

The skarab_adc_reconfig.py Python script and the two bin files can be found here:
[SKARAB ADC FW](https://github.com/Peralex/mlib_devel/tree/master/jasper_library/test_models/scripts/skarab_adc_reconfig)


WARNING: Please ensure that the power and network connection to the SKARAB is stable before using this script. There is a possibility that the SKARAB ADC board can be bricked if the firmware update process is interrupted by a power or network failure. In this case, the SKARAB ADC board will need to be returned to Peralex Electronics so that its firmware can be restored to a working state.

## CASPER Toolflow and casperfpga Library Requirements ##
Please install the following versions of the CASPER Toolflow and casperfpga library:

[Peralex's master branch of the CASPER Toolflow](https://github.com/Peralex/mlib_devel/tree/master)

[Peralex's main branch of the casperfpga library](https://github.com/Peralex/casperfpga/tree/main)

## Test Procedure ##
1. Open one of the provided Simulink design models and modify it as required (as described in the "Example Designs" section)
2. Build an fpg file from the modified Simulink design model
3. Set up the hardware test configuration according to the information described in the "SKARAB ADC Hardware Setup" section
4. Allow a DHCP or BOOTP server on the network to allocate IP addresses to the SKARABs.
An example of the IP address allocation is listed in the table below:

SKARAB MAC Address | IP Address
--- | ---
06:50:02:0D:03:00 | 10.0.7.2
06:50:02:0D:04:00 | 10.0.7.3
06:50:03:0D:01:00 | 10.0.7.4
06:50:02:0D:02:00 | 10.0.7.5

4.	Power on the SKARAB Systems and confirm that their IP address allocation was successful (e.g. PING)
5.	Read the Python test script comments and set the script configuration as required
6.	Run the Python test script which will upload the created fpg file to the SKARAB systems, perform synchronised ADC data sampling, and then write the sample data to text files
7.	Run one of the ADC data plotting Matlab scripts to plot the captured ADC sample data from the text files

The design files and documentation for this tutorial were supplied by Peralex Electronics. However, please feel free, as always, to contact the [CASPER email list](mailto:casper@lists.berkeley.edu) with questions/issues and we will do our best to help (or find someone who can). 
