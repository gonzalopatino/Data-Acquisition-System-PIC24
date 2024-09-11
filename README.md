# 📜 Project Title: **Microcontrollers 2 Project**

## Description

This project demonstrates the use of **PIC24 microcontrollers** to read, save, recall, and manipulate sensor data through an **ADC (Analog-to-Digital Converter)**, displaying results on a **GLCD (Graphical LCD)**. It features a **state machine** for handling various user inputs from a keypad and performs tasks such as reading temperature data, storing values, and recalling them on demand.

The project showcases efficient embedded system development practices, focusing on modular design and state management. Additionally, it supports saving data to multiple locations, recalling saved data, clearing stored values, and exporting data via **UART**.

---

## 🧩 Features

- **State Machine Implementation**: Efficient handling of user input via keypad and executing different operations such as read, save, recall, and clear.
- **ADC Temperature Reading**: Reads data from an analog sensor, converts it to temperature, and displays it on the GLCD.
- **Data Storage and Recall**: Supports saving up to 10 different temperature readings in memory locations.
- **GLCD Display**: Outputs messages and sensor data on the graphical LCD.
- **UART Communication**: Downloads all saved data to a serial terminal.
- **Keypad Input**: User interaction through a 4x3 keypad for selecting options.
- **LED Indicator**: Provides visual feedback for successful operations.

---

## ⚙️ Code Structure

The code is organized into several key components:

### 1. **State Machine**
   The state machine is the heart of the application. It manages the different user interactions and transitions between states like `STATE_READ`, `STATE_SAVE`, `STATE_RECALL`, and more. Each state is designed to perform a specific task based on the user’s input.

### 2. **ADC (Analog-to-Digital Conversion)**
   The project reads temperature data from an analog sensor using the **PIC24 ADC module**. The raw ADC value is then converted to a temperature using reference voltage (VREF) and LSb, and displayed on the GLCD.

### 3. **GLCD Display**
   - The **GLCD** is used to provide feedback to the user.
   - The display shows the menu options, results of temperature readings, save confirmations, and error messages.
   - A modular approach is used for updating the display.

### 4. **Keypad Input**
   - The **4x3 keypad** allows the user to interact with the system by selecting various options.
   - The keypad is scanned for key presses, which are then used to transition between states in the state machine.

### 5. **UART Communication**
   - The project includes a feature to download saved data to a serial terminal via UART.
   - This allows for external monitoring and data logging.

### 6. **LED Feedback**
   - A simple LED blink function provides feedback to the user during important operations, such as saving data or completing a transaction.

### 7. **Data Management (ADC Storage)**
   - An array-based structure is used to store up to 10 different temperature readings.
   - The system supports saving and recalling specific data locations, clearing values, and more.

---

## 📁 Project Structure

```plaintext
├── main.c                  # Main code file, containing the state machine and functionality
├── pic24_all.h             # PIC24 configuration file
├── glcd.h                  # GLCD driver and display functions
├── fonts/                  # Font files for displaying text on the GLCD
└── README.md               # Project documentation

## Prerequisites

- **PIC24 Microcontroller**: This project is designed for **PIC24** microcontrollers.
- **MPLAB X IDE**: Used for compiling and uploading the code.
- **MPLAB XC16 Compiler**: To compile the C code for the microcontroller.
- **Keypad (4x3)**: For user input.
- **GLCD (Graphical LCD)**: For visual output.
- **ADC-Compatible Sensor**: To measure temperature or other analog values.
- **UART Terminal**: For downloading data to a PC.

## Installation

1. **Clone the repository**:

   ```bash
   git clone https://github.com/your-username/microcontrollers2-project.git
## Installation

2. **Open in MPLAB X**:
   - Launch **MPLAB X IDE**.
   - Import the cloned project into MPLAB X by selecting **File -> Open Project**, then navigate to the cloned repository folder.

3. **Compile and Upload**:
   - Build the project using the **XC16** compiler.
   - Upload the code to your **PIC24** microcontroller.

---

## 🛠️ How It Works

### Key Features:

- **Menu Options**: After booting, the system presents a menu on the GLCD, allowing users to:
  - Read sensor data.
  - Save the sensor data.
  - Recall previously saved data.
  - Clear stored data.
  - Download all stored data via UART.

- **Keypad Interaction**: The user interacts with the system via a 4x3 keypad. Each key corresponds to a specific action (e.g., '1' to read data, '2' to save data, etc.).

- **Data Storage**: The system can store up to 10 different temperature readings. Each reading can be saved in a specific location (0–9) and later recalled or cleared.

- **UART Download**: Users can export all saved data to a connected serial terminal by selecting the download option from the menu.

---

## 🖼️ Displaying Data on the GLCD

The project uses a **graphical LCD (GLCD)** to visually display important information such as:

- **Menu options** for the user to choose from.
- **Temperature readings** after conversion from the ADC sensor data.
- **Save and recall confirmations** to indicate whether operations were successful.

---

## 🔧 Future Enhancements

- **Add More Sensors**: The project can be expanded to support additional sensors (e.g., humidity, pressure).
- **Support for More Save Locations**: Extend the storage capability to handle more than 10 values.
- **Enhanced UI**: Improve the graphical interface on the GLCD for a more intuitive user experience.

---



---

## 🤝 Contributing

Contributions are welcome! Feel free to open issues or submit pull requests for improvements, bug fixes, or new features.
