# WinKeyPress


## Overview
WinKeyPress is a Windows-based application designed to send automated key presses to a targeted window. It features a graphical user interface that allows users to configure key press sequences and define the target window by its title. This tool is particularly useful for automating repetitive tasks in other applications while not requiring the application to be focussed in the foreground.

---

## Features
- **Customisable Key Sequences**: Define a sequence of key presses, intervals, and delays in a simple input format.
- **Target Window Selection**: Specify the target window using a partial match of its title.
- **Does not require window focus**: Allowing the sequence to be played out while the user completes tasks on other applications.
- **Start/Stop Button**: Easily control the execution of key press automation.

---

## Requirements
- Windows OS
- Visual Studio or compatible compiler for building the source code

---

## Installation
1. Clone or download the repository containing the source code.
2. Open the project in Visual Studio or another C++ IDE.
3. Build the project to create the executable file.
4. Run the executable file.

---

## Usage

### Input Format
The sequence of key presses should be entered in the following format:
```
<key> <intervalMs> <delayMs>
```
Where:
- `<key>` is the key to press (e.g., `A`, `B`, `C`).
- `<intervalMs>` is the time in milliseconds the key is held down.
- `<delayMs>` is the delay in milliseconds before the next key press.

**Example Input**:
```
C 600 1200
D 300 1500
```
- Presses the `C` key for 600 ms, waits for 1200 ms, then presses the `D` key for 300 ms and waits for 1500 ms.

### Steps to Use
1. Launch the application.
2. Enter the key sequence in the sequence edit box.
3. Enter the title or partial title of the target window in the target window edit box.
4. Click the `Start` button to begin automation.
5. Click the `Stop` button to end automation.

---

## How It Works
1. **Parsing Input**:
   - The key sequences are parsed from the input box into a vector of `KeyPressSequence` structures.
2. **Finding Target Window**:
   - The application searches for a window whose title starts with the user-specified string.
3. **Sending Key Presses**:
   - Key presses are simulated using `WM_KEYDOWN` and `WM_KEYUP` messages sent to the target window.

---

## GUI Components
1. **Start/Stop Button**:
   - Toggles the key press automation.
2. **Sequence Edit Box**:
   - Multi-line input box for defining key press sequences.
3. **Target Window Edit Box**:
   - Input box for specifying the target window by partial title.

---

## Known Limitations
- Only works on Windows systems.
- Requires the target window to have a title.
- Key presses may not work with applications that block simulated input or require administrator privileges.
- Sequence parsing does not handle invalid or malformed input gracefully.
- Testing has not been done for special keys such as Return and Backspace.

---

## Future Improvements
- Enhance input validation for key press sequences.
- Support additional input methods such as mouse clicks.
- Improve compatibility with applications that use advanced input handling.

---

## Contributing
If you would like to contribute, please submit a pull request or raise an issue on the project repository.
