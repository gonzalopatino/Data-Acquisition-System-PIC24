/***********************PROJECT MICROCONTROLLERS 2 by Ing. Gonzalo Patino*******************************************************/
/*****************************************************************************************************************************/
/************************************* All rights reserved ********************************************************************/
/************************************** Student ID: 300740701 *****************************************************************/


#include "pic24_all.h"      // PIC24 header
#include <stdint.h>         // Standard integer types
#include <xc.h>             // XC PIC24 compiler functions
#include <libpic30.h>       // PIC30 library functions
#include <math.h>           // Math functions
#include "main.h"           // Project-specific main header
#include "glcd.h"           // GLCD driver
#include "fonts/font5x7.h"  // Font for GLCD
#include "fonts/Liberation_Sans15x21_Numbers.h" // Font for GLCD

/* Constants */
#define VREF 3.3                    // Reference voltage for ADC
#define LSb 104                     // Conversion factor from Voltage to °C
#define MAX_SAVE_LOCATIONS 10        // Maximum save locations for ADC values
#define INVALID_KEY 'E'              // Invalid key error code
#define DEBOUNCE_DELAY_MS 175        // Debounce delay in milliseconds

/* GPIO Configuration for LED */
#define LED_PIN _LATA0
#define LED_TRIS_PIN _TRISA0
#define CONFIG_LED1() CONFIG_RB6_AS_DIG_OUTPUT()
#define LED1 (_LATB6)    // LED state

/* Function to Blink LED */
void LED_Blink_1T(void) {
    LED1 = 1;
    DELAY_MS(1000);
    LED1 = 0;
    DELAY_MS(1000);
}

/* Structure to Hold ADC Values */
typedef struct {
    float values[MAX_SAVE_LOCATIONS];
} ADCStorage;

/* Global Variables */
volatile uint8_t u8_newKey = 0;       // Key pressed indicator
uint16_t u16_adcVal;                  // ADC raw value
float f_adcVal_temp;                  // Temporary ADC value
ADCStorage adcStorage;                // ADC Storage to hold saved values

/* Enum for State Machine */
typedef enum {
    STATE_MENU,
    STATE_OPTIONS,
    STATE_INVALID,
    STATE_READ,
    STATE_SAVE,
    STATE_SAVE_LOCATION,
    STATE_SAVE_INVALID_LOCATION,
    STATE_RECALL,
    STATE_RECALL_LOCATION,
    STATE_RECALL_INVALID_LOCATION,
    STATE_CLEAR,
    STATE_CLEAR_LOCATION,
    STATE_DOWNLOAD,
} state_t;

/* Function Prototypes */
void display_menu(void);
void handle_keypad_input(void);
void display_invalid_option(void);
void save_adc_value(uint8_t location);
void recall_adc_value(uint8_t location);
void clear_adc_value(uint8_t location);
void download_adc_values(void);

/* Update the state machine based on input */
void update_state(void) {
    static state_t e_state = STATE_MENU;

    switch (e_state) {
        case STATE_MENU:
            display_menu();
            e_state = STATE_OPTIONS;
            break;

        case STATE_OPTIONS:
            handle_keypad_input();
            break;

        case STATE_INVALID:
            display_invalid_option();
            e_state = STATE_MENU;
            break;

        case STATE_READ:
            u16_adcVal = convertADC1(); // Read ADC value
            f_adcVal_temp = u16_adcVal / 4096.0 * VREF * LSb; // Convert ADC to temperature

            // Display ADC value on GLCD
            char tempString[8];
            sprintf(tempString, "%4.3f", f_adcVal_temp);
            glcd_tiny_set_font(Font5x7, 5, 7, 32, 127);
            glcd_clear_buffer();
            glcd_tiny_draw_string(0, 3, "Temperature:");
            glcd_tiny_draw_string(0, 5, tempString);
            glcd_write();

            DELAY_MS(3000);
            outString("Read successful...\n\r");
            LED_Blink_1T();
            e_state = STATE_MENU;
            break;

        case STATE_SAVE:
            u8_newKey = 0;
            glcd_tiny_set_font(Font5x7, 5, 7, 32, 127);
            glcd_clear_buffer();
            glcd_tiny_draw_string(0, 3, "Save: 0-9");
            glcd_write();
            DELAY_MS(175);
            e_state = STATE_SAVE_LOCATION;
            break;

        case STATE_SAVE_LOCATION:
            if (u8_newKey >= '0' && u8_newKey <= '9') {
                save_adc_value(u8_newKey - '0');
                e_state = STATE_MENU;
            } else {
                e_state = STATE_SAVE_INVALID_LOCATION;
            }
            break;

        case STATE_SAVE_INVALID_LOCATION:
            glcd_tiny_set_font(Font5x7, 5, 7, 32, 127);
            glcd_clear_buffer();
            glcd_tiny_draw_string(0, 3, "Invalid Option");
            glcd_tiny_draw_string(0, 5, "Select 0 to 9");
            glcd_write();
            DELAY_MS(175);
            e_state = STATE_SAVE;
            break;

        case STATE_RECALL:
            u8_newKey = 0;
            glcd_tiny_set_font(Font5x7, 5, 7, 32, 127);
            glcd_clear_buffer();
            glcd_tiny_draw_string(0, 3, "Recall: 0-9");
            glcd_write();
            DELAY_MS(175);
            e_state = STATE_RECALL_LOCATION;
            break;

        case STATE_RECALL_LOCATION:
            if (u8_newKey >= '0' && u8_newKey <= '9') {
                recall_adc_value(u8_newKey - '0');
                e_state = STATE_MENU;
            } else {
                e_state = STATE_RECALL_INVALID_LOCATION;
            }
            break;

        case STATE_RECALL_INVALID_LOCATION:
            glcd_tiny_set_font(Font5x7, 5, 7, 32, 127);
            glcd_clear_buffer();
            glcd_tiny_draw_string(0, 3, "Invalid Option");
            glcd_tiny_draw_string(0, 5, "Select 0 to 9");
            glcd_write();
            DELAY_MS(175);
            e_state = STATE_RECALL;
            break;

        case STATE_CLEAR:
            u8_newKey = 0;
            glcd_tiny_set_font(Font5x7, 5, 7, 32, 127);
            glcd_clear_buffer();
            glcd_tiny_draw_string(0, 3, "Clear: 0-9");
            glcd_write();
            DELAY_MS(175);
            e_state = STATE_CLEAR_LOCATION;
            break;

        case STATE_CLEAR_LOCATION:
            if (u8_newKey >= '0' && u8_newKey <= '9') {
                clear_adc_value(u8_newKey - '0');
                e_state = STATE_MENU;
            } else {
                e_state = STATE_MENU;
            }
            break;

        case STATE_DOWNLOAD:
            download_adc_values();
            e_state = STATE_MENU;
            break;

        default:
            ASSERT(0); // Catch-all for invalid states
            break;
    }
}

/* Function to display menu on GLCD */
void display_menu(void) {
    outString("Hello! This is Gonzalo's Project for MICRO 2.\n\r");
    outString("Waiting for instruction...\n\r");

    glcd_tiny_set_font(Font5x7, 5, 7, 32, 127);
    glcd_clear_buffer();
    glcd_tiny_draw_string(0, 0, "1-Read       2-Save");
    glcd_tiny_draw_string(0, 3, "3-Recall     4-Clear");
    glcd_tiny_draw_string(0, 6, "     5-Download     ");
    glcd_write();
}

/* Function to handle keypad input */
void handle_keypad_input(void) {
    switch (u8_newKey) {
        case '1':
            outString("Reading value. Please wait...\n\r");
            u8_newKey = 0;
            update_state();
            break;

        case '2':
            outString("Saving value. Please wait...\n\r");
            u8_newKey = 0;
            update_state();
            break;

        case '3':
            outString("Recalling value. Please wait...\n\r");
            u8_newKey = 0;
            update_state();
            break;

        case '4':
            outString("Clearing value. Please wait...\n\r");
            u8_newKey = 0;
            update_state();
            break;

        case '5':
            outString("Downloading value. Please wait...\n\r");
            u8_newKey = 0;
            update_state();
            break;

        default:
            update_state();
            break;
    }
}

/* Display invalid option message on GLCD */
void display_invalid_option(void) {
    outString("Invalid option...\n\r");

    glcd_tiny_set_font(Font5x7, 5, 7, 32, 127);
    glcd_clear_buffer();
    glcd_tiny_draw_string(0, 0, "Invalid Option");
    glcd_tiny_draw_string(0, 3, "Select 1 to 4 please");
    glcd_write();
    DELAY_MS(3000);
}

/* Save the current ADC value to a specific location */
void save_adc_value(uint8_t location) {
    adcStorage.values[location] = f_adcVal_temp;

    char msg[32];
    sprintf(msg, "Data saved in Location %d\n", location);
    outString(msg);

    glcd_tiny_set_font(Font5x7, 5, 7, 32, 127);
    glcd_clear_buffer();
    glcd_tiny_draw_string(0, 3, "Data saved in");
    sprintf(msg, "Location %d", location);
    glcd_tiny_draw_string(0, 5, msg);
    glcd_write();

    LED_Blink_1T();
    DELAY_MS(175);
}

/* Recall the saved ADC value from a specific location */
void recall_adc_value(uint8_t location) {
    if (adcStorage.values[location] != 0) {
        char tempString[8];
        sprintf(tempString, "%4.3f", adcStorage.values[location]);

        glcd_tiny_set_font(Font5x7, 5, 7, 32, 127);
        glcd_clear_buffer();
        glcd_tiny_draw_string(0, 3, "Temperature in C:");
        glcd_tiny_draw_string(0, 5, tempString);
        glcd_write();

        DELAY_MS(3000);
    } else {
        glcd_tiny_set_font(Font5x7, 5, 7, 32, 127);
        glcd_clear_buffer();
        glcd_tiny_draw_string(0, 3, "No data saved in");
        glcd_tiny_draw_string(0, 5, "Location ");
        glcd_tiny_draw_string(0, 6, "Choose valid location");
        glcd_write();

        DELAY_MS(3000);
    }
}

/* Clear the saved ADC value at a specific location */
void clear_adc_value(uint8_t location) {
    adcStorage.values[location] = 0;

    char msg[32];
    sprintf(msg, "Data cleared at Location %d\n", location);
    outString(msg);

    glcd_tiny_set_font(Font5x7, 5, 7, 32, 127);
    glcd_clear_buffer();
    glcd_tiny_draw_string(0, 3, "Data cleared at");
    sprintf(msg, "Location %d", location);
    glcd_tiny_draw_string(0, 5, msg);
    glcd_write();

    LED_Blink_1T();
    DELAY_MS(175);
}

/* Download the ADC values over UART */
void download_adc_values(void) {
    for (uint8_t i = 0; i < MAX_SAVE_LOCATIONS; i++) {
        char msg[32];
        sprintf(msg, "adcVal_%d = %4.3f\n", i, adcStorage.values[i]);
        outString(msg);
        DELAY_MS(500);
    }

    outString("Transfer successful...\n\r");
    outString("Data has been cleared at all locations...\n\r");

    LED_Blink_1T();
    DELAY_MS(3000);
}

/* Main function */
int main(void) {
    configBasic(HELLO_MSG);  // Basic configuration
    CONFIG_LED1();           // Configure LED
    configKeypad();          // Configure keypad
    configTimer3();          // Configure timer for keypad
    tick_init();             // Configure tick timer
    glcd_init();             // Initialize GLCD
    CONFIG_RA0_AS_ANALOG();  // Configure analog input for ADC
    configADC1_ManualCH0(RA0_AN, 31, 0);  // Configure ADC

    // Boot process
    outString("Project Micro 2, Gonzalo Patino\n\r");
    outString("***********BOOTING****************\n\r");

    // GLCD testing
    glcd_clear();
    glcd_test_glcdutils();
    DELAY_MS(1000);
    glcd_test_text_up_down();
    DELAY_MS(1000);
    glcd_test_text_up_down2();
    DELAY_MS(1000);
    glcd_test_scrolling_graph();
    glcd_test_circles();
    glcd_MY_INFO();  // Display project information
    DELAY_MS(10000);
    LED_Blink_1T();  // Blink LED once

    while (1) {
        update_state();  // Update state machine
        DELAY_MS(DEBOUNCE_DELAY_MS);  // Debounce delay
        doHeartbeat();  // Heartbeat function
    }

    return 0;
}
