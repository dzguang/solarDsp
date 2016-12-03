David M. Alter
Texas Instruments, Inc.
September 29, 2008
----------------------------------


This code accompanies the Texas Instruments application report "Running an Application from Internal Flash Memory on the TMS320F28xxx DSP," literature #SPRA958.  The application report can be downloaded from the TI website, www.ti.com.

Please read and understand the disclaimer contained in the file disclaimer.txt.

----------------------------------

These are example code projects for the eZdspF2808 board.  They are complete with all needed files.  They have been built and tested using Code Composer Studio v3.3.81.5, C28x C-Compiler v5.1.0, and DSP/BIOS v5.33.  Test hardware was an eZdspF2808 development board with revision C silicon.

There are four projects:

   F2808_example_BIOS_ram.pjt - DSP/BIOS project that runs from on-chip RAM
   F2808_example_BIOS_flash.pjt - DSP/BIOS project that runs from on-chip FLASH
   F2808_example_nonBIOS_ram.pjt - non-DSP/BIOS project that runs from on-chip RAM
   F2808_example_nonBIOS_flash.pjt - non-DSP/BIOS project that runs from on-chip FLASH

These are just examples of DSP/BIOS and non-DSP/BIOS projects.  They have only been given brief tests, and no guarantees are made about their suitability or performance for application useage.


The projects all do the following:
----------------------------------

1) Illustrates F2808 DSP initialization (in function main()).  The PLL is configured for x5 mode.
2) Toggles the GPIOF34 pin to blink the LED on the eZdspF2808 board.  In the non-DSP/BIOS projects, this is done in the ADCINT ISR.  In the DSP/BIOS projects, a periodic function is used.
3) Configures the ADC to sample on ADCINA0 channel at a 50 kHz rate.
4) Services the ADC interrupt.
5) Sends out 2 kHz symmetric PWM on EPWM1A (muxed to GPIO0 pin).
6) Configures the enhanced capture unit ECAP1 (muxed to GPIO5 pin).
7) Services the capture unit #1 interrupt.


Things to know:
---------------
1) The .pjt project files can be found in the \projects directory.  After compiling a project, the .out file will be located in the \projects\Debug directory.

2) The \src directory contains code source files common to all projects.  Note: not every source file is used by all projects.

3) The \DSP280x_headers\include directory contains include files from the DSP280x Peripherals Structures header file download.  All four projects use these structures to program the peripherals in C.  The header file download is available from the TI website, literature #SPRC191.  Specifically, v1.60 of the header files has been used.  The contents of this directory are IDENTICAL to the same directory from the header file download v1.60.  In addition, the file DSP280x_GlobalVariableDefs.c has been moved from the \DSP280x_headers\src directory of the header file download to the \src directory of this project.

4a) If using the RAM examples, the F2808 on the eZdsp board needs to be configured for "Jump to M0" bootmode.  Check that switches 1, 2, and 3 on dip-switch bank SW1 are configured as:

   SW1.1 = ON  (CLOSED)
   SW1.2 = OFF (OPEN)
   SW1.3 = ON  (CLOSED)

If this does not seem to be working, check the reference manual for your eZdsp board to confirm the dip=switch settings.  Dip settings may have changed if the eZdsp board was revised.

4b) If using the FLASH examples, the F2808 on the eZdsp board needs to be configured for "Jump to Flash" bootmode.    Check that switches 1, 2, and 3 on dip-switch bank SW1 are configured as:

   SW1.1 = OFF (OPEN)
   SW1.2 = OFF (OPEN)
   SW1.3 = OFF (OPEN)

If this does not seem to be working, check the reference manual for your eZdsp board to confirm the dip=switch settings.  Dip settings may have changed if the eZdsp board was revised.

5) The ram example is linking sections in various places that may look unnecessary (e.g., the section ramfuncs is loaded to one ram area, and copied to and run from another ram area.  On the surface, this look rather pointless.  However, these things were really done in preparation to build the flash project.  In reality, a real embedded system cannot run on ram alone.  It must have non-volatile memory somewhere.  Hence, in the flash system, you will see the same sections being loaded to flash, but copied to and run from ram.

6) The user will reach a point where he will need to modify the linker command file for the project.  The linking is actually controlled by three different files: user .cmd file, DSP/BIOS generated .cmd file (for DSP/BIOS projects only), and the DSP280x header file .cmd.  The user .cmd file is named f2808_BIOS_ram.cmd, f2808_BIOS_flash.cmd, f2808_nonBIOS_ram.cmd, or f2808_nonBIOS_flash.cmd, depending on which project is used.  The DSP/BIOS generated .cmd file is similarly named f2808_BIOS_ramcfg.cmd, f2808_BIOS_flashcfg.cmd, f2808_nonBIOS_ramcfg.cmd, or f2808_nonBIOS_flashcfg.cmd.  Finally, the DSP280x header file .cmd file is named either DSP280x_Headers_BIOS.cmd or DSP280x_Headers_nonBIOS.cmd.

Be careful modifying the user .cmd file. For DSP/BIOS projects, the RAM and FLASH memory has been defined in the DSP BIOS configuration tool (i.e., in the DSP/BIOS generated .cmd file).  The peripheral structure memory and other DSP280x header file related sections is defined in the DSP280x header file .cmd file.  There has not been too much attention given to where everything is linked.  The goal in writing this example code was simply to get it to work correctly.  The linking may need to be tuned to get better performance (e.g., to avoid memory block access contention, or to better manage memory block utilization).

7) For non-DSP/BIOS projects, a complete set of interrupt service routines are defined in the file defaultISR_nonBIOS.c.  Each interrupt is executed directly in its hardware ISR.  However, with the exception of the ADCINT and CAPINT1, each ISR actually executes an ESTOP0 instruction (emulation stop) to trap spurious interrupts during debug.  Note that each ISR is using the "interrupt" keyword which tells the compiler to perform a context save/restore upon function entry/exit.

8) For DSP/BIOS projects, a complete set of (hardware) interrupt service routines are defined in the file DefaultISR_BIOS.c.  Each ISR can be hooked to the desired interrupt using the HWI manager in the DSP/BIOS configuration tool.  Also, the DSP/BIOS Interrupt Dispatcher can be used to handle the context save/restore,  which is why the ISRs are not using the "interrupt" keyword (as in the non-DSP/BIOS case).  In these examples, the ECAP1_INT ISR is performed directly in the DefaultIsr.c file (as an example of reducing latency), whereas the ADC interrupt function in DefaultIsr_BIOS.c posts a SWI to perform the ADC routine.  These are just examples.  Note that the ECAP1_INT is still using the DSP/BIOS dispatcher to perform context save/restore (as selected in the HWI manager of the configuration tool).  If absolute minimum latency is required (for some time critical ISR), one could disable the interrupt dispatcher for that interrupt, and add the "interrupt" keyword to the ISR function declaration.  Note that doing so will preclude the user for utilizing any DSP/BIOS functionality in that ISR.


------------ END OF FILE -------------------
