# EmbeddedSystemsLab3
/**********************************
 * ADC Controlled LED
 *
 * Example which controls the LED Brightness via PWM which is varied based on an ADC Reading
 *
 * Author: Russell Trafford
 * Revision: 1.1
 * Last Revision Date: 12OCT22
 *
 * Change Log:
 *  - Changed LED Pin to 6.0 to allow for control by Timer3B module
 *  - Using Pin 1.1 for being connected to Grove Connector Analog Channel 1
 *
 *
 * Pin Map:
 *               MSP430FR2355
 *            -----------------
 *        /|\.  |                             |
 *         |.    |                               |
 *         --   |RST                       |
 *               |                               |
 *       >---|P1.1/A1      P6.0|--> LED --> Wired to LED 2 through external wire to P6.6
 *
 *       ****WARNING****
 *       Please disconnect J11 and connect to Pin 6.0 to the lower part of the jumper
 *
 *       ____
 *      | NC | <-- Keep this Disconnected
 *      |----  |
 *      | **    | <-- Connect to this part of J11
 *      |____. |
 *
 *
 * Peripherals Used:
 *      ADC12 - Channel 1 (Pin 1.1)
 *      GPIO - Pin 6.0
 *      TimerB0 - UP Mode, CCR1 for PWM Control, controls Pin 6.0
 */

#include <msp430.h>
/**
 * main.c
 */
void gpioInit();        // Setup the GPIO Pins for controlling the LEDS through PWM (no input arguments)
void timerInit();       // Setup Timer to control the LEDS (no input arguments)
void adcInit();         // Setup ADC to do single, on demand samples (no input arguments)
int readADCSingle();    // Single Read ADC, no input arguments, returns current ADC reading
int main(void)
{
WDTCTL = WDTPW | WDTHOLD; // stop watchdog timer


gpioInit();
adcInit();
timerInit();
while(1){
    readADCSingle();
    setLEDBrightness();
}
return 0;
}
/**
 * GPIO INIT
 * Initialize all GPIO Pins. Current Pin configuration:
 * Pin 1.0: OUTPUT, GPIO
 * Pin 1.1: INPUT,  ADC Channel 1
 */
void gpioInit(){
    // Configure ADC Pin
    P1SEL0 |= BIT1;
    P1SEL1 |= BIT1;

    // P6.0 output
    P6DIR |= BIT0;


    P6SEL0 |= BIT0;
    P6SEL1 &= ~(BIT0);

   // P6OUT |= BIT6;
    // Configure GPIO Pin

}
/**
 * Timer INIT
 * Initialize the Timer Peripheral to control a GPIO Pin with PWM in up mode
 * Timer Mode: Up Mode
 */
void timerInit(){
           TB0CCR0 = 1000-1;                         // PWM Period
           TB0CCTL1 = OUTMOD_7;                      // CCR1 reset/set
           TB0CCR1 = 750;                            // CCR1 PWM duty cycle

           TB3CTL = TBSSEL__SMCLK | MC__UP | TBCLR;  // SMCLK, up mode, clear TBR
}
/**
 * ADC INIT
 * Initialize the ADC Peripheral to read Single measurements on demand.
 * ADC MODE: Single Sample, Single Conversion
 */
void adcInit(){
           ADCCTL0 |= ADCSHT_2 | ADCON;                             // ADCON, S&H=16 ADC clks
           ADCCTL1 |= ADCSHP;                                       // ADCCLK = MODOSC; sampling timer
           ADCCTL2 &= ~ADCRES;                                      // clear ADCRES in ADCCTL
           ADCCTL2 |= ADCRES_2;                                     // 12-bit conversion results
           ADCMCTL0 |= ADCINCH_1;                                   // A1 ADC input select; Vref=AVCC
}
/**
 * Read Single ADC Sample from Channel 0 of the ADC12 Peripheral
 * Returns an ADC Sample without interrupts.
 */
int readADCSingle(){
    ADCCTL0 |= ADCENC | ADCSC;     // Sampling and conversion start
    while (!(ADCIFG & ADCIFG0));   // Wait for sample to be sampled and converted
    return ADCMEM0;
}
/**

* Set LED Brightness on the LED configured from gpioInit. This modifies the

* CCR register based on the input.

* int DutyCycle: Value between 0-100

*/
void setLEDBrightness(int DutyCycle) {
    TB3CCR1 =(int) DutyCycle*9.9;
}
