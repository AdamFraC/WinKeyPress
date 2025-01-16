#include <windows.h>
#include <iostream>
#include <string>
#include <thread>
#include <atomic>
#include <chrono>
#include <sstream>
#include <vector>

// Globals
std::atomic<bool> isRunning(false);
std::thread keyPressThread;

// Structure to store key press sequence
typedef struct {
    char key;
    int intervalMs;
    int delayMs;
} KeyPressSequence;

// Function to parse sequences from the input string
std::vector<KeyPressSequence> ParseSequences(const std::wstring& input) {
    std::vector<KeyPressSequence> sequences;
    std::wistringstream stream(input);
    std::wstring line;

    while (std::getline(stream, line)) {
        std::wistringstream linestream(line);
        KeyPressSequence sequence;
        wchar_t key;
        if (linestream >> key >> sequence.intervalMs >> sequence.delayMs) {
            sequence.key = static_cast<char>(key);
            sequences.push_back(sequence);
        }
    }

    return sequences;
}

// Function to send key presses
void SendKeyPresses(HWND hWnd, const std::vector<KeyPressSequence>& sequences) {
    while (isRunning) {
        for (const auto& sequence : sequences) {
            if (!isRunning) return;

            // Simulate key press (key down)
            PostMessage(hWnd, WM_KEYDOWN, sequence.key, 0x00000001);

            // Small delay to simulate the key being held down briefly
            std::this_thread::sleep_for(std::chrono::milliseconds(sequence.intervalMs));

            // Simulate key release (key up)
            PostMessage(hWnd, WM_KEYUP, sequence.key, 0xC0000001);

            // Delay between key presses
            std::this_thread::sleep_for(std::chrono::milliseconds(sequence.delayMs));
        }
    }
}


// Structure to pass data to EnumWindowsProc
struct FindWindowData {
    std::wstring partialTitle;
    HWND foundWindow = NULL;
};

// Callback function for EnumWindows
BOOL CALLBACK EnumWindowsProc(HWND hWnd, LPARAM lParam) {
    FindWindowData* data = reinterpret_cast<FindWindowData*>(lParam);

    wchar_t windowTitle[256];
    GetWindowText(hWnd, windowTitle, sizeof(windowTitle) / sizeof(wchar_t));

    if (std::wstring(windowTitle).find(data->partialTitle) == 0) { // Title starts with partialTitle
        data->foundWindow = hWnd;
        return FALSE; // Stop enumeration
    }
    return TRUE; // Continue enumeration
}

// Function to find a window with a title starting with the given string
HWND FindWindowByPartialTitle(const std::wstring& partialTitle) {
    FindWindowData data = { partialTitle };
    EnumWindows(EnumWindowsProc, reinterpret_cast<LPARAM>(&data));
    return data.foundWindow;
}

// GUI Callback Function
LRESULT CALLBACK WindowProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    static HWND hStartStopButton, hSequenceEdit, hTargetWindow;
    static HWND targetWindow = NULL;

    switch (uMsg) {
    case WM_CREATE:
        // Create Start/Stop Button
        hStartStopButton = CreateWindow(L"BUTTON", L"Start", WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
            10, 10, 100, 30, hWnd, (HMENU)1, (HINSTANCE)GetWindowLongPtr(hWnd, GWLP_HINSTANCE), NULL);

        // Create Sequence Edit Box
        hSequenceEdit = CreateWindow(L"EDIT", L"C 600 1200\r\nD 300 1500", WS_TABSTOP | WS_VISIBLE | WS_CHILD | WS_BORDER | ES_MULTILINE | ES_AUTOVSCROLL | ES_WANTRETURN,
            10, 50, 300, 100, hWnd, NULL, (HINSTANCE)GetWindowLongPtr(hWnd, GWLP_HINSTANCE), NULL);

		// Create Target Window Edit Box
        hTargetWindow = CreateWindow(L"EDIT", L"Targeted Window", WS_TABSTOP | WS_VISIBLE | WS_CHILD | WS_BORDER,
            10, 160, 300, 25, hWnd, NULL, (HINSTANCE)GetWindowLongPtr(hWnd, GWLP_HINSTANCE), NULL);
        break;

    case WM_COMMAND:
        if (LOWORD(wParam) == 1) { // Start/Stop Button clicked
            wchar_t buffer[1024];

            // Get name of targetted window
            GetWindowText(hTargetWindow, buffer, sizeof(buffer) / sizeof(buffer[0]));
            std::wstring TargettedWindow = (buffer);

            // Get sequences
            GetWindowText(hSequenceEdit, buffer, sizeof(buffer) / sizeof(buffer[0]));
            std::wstring input(buffer);
            auto sequences = ParseSequences(input);

            if (sequences.empty()) {
                MessageBox(hWnd, L"Please enter valid sequences (key intervalMs delayMs).", L"Invalid Input", MB_OK | MB_ICONWARNING);
                return 0;
            }

            if (!isRunning) {
                // Find target window by partial title
                targetWindow = FindWindowByPartialTitle(TargettedWindow);
                if (!targetWindow) {
                    MessageBox(hWnd, L"Target window not found!", L"Error", MB_OK | MB_ICONERROR);
                    return 0;
                }

                // Start sending key presses
                isRunning = true;
                keyPressThread = std::thread(SendKeyPresses, targetWindow, sequences);
                SetWindowText(hStartStopButton, L"Stop");
            }
            else {
                // Stop sending key presses
                isRunning = false;
                if (keyPressThread.joinable()) keyPressThread.join();
                SetWindowText(hStartStopButton, L"Start");
            }
        }
        break;

    case WM_SIZE:
        if (wParam != SIZE_MINIMIZED) {
            int width = LOWORD(lParam);
            int height = HIWORD(lParam);

            // Resize and reposition controls
            MoveWindow(hStartStopButton, 10, 10, 100, 30, TRUE);
            MoveWindow(hSequenceEdit, 10, 50, width - 20, height - 150, TRUE);
            MoveWindow(hTargetWindow, 10, height - 90, width - 20, 25, TRUE);
        }
        break;

    case WM_DESTROY:
        // Ensure the thread stops when the program closes
        isRunning = false;
        if (keyPressThread.joinable()) keyPressThread.join();
        PostQuitMessage(0);
        break;

    default:
        return DefWindowProc(hWnd, uMsg, wParam, lParam);
    }
    return 0;
}

// WinMain: Entry point for the Win32 GUI application
int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow) {
    const wchar_t CLASS_NAME[] = L"KeyPressApp";

    // Register window class
    WNDCLASS wc = {};
    wc.lpfnWndProc = WindowProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = CLASS_NAME;
    wc.hCursor = LoadCursor(NULL, IDC_ARROW); // Set default cursor
    wc.hbrBackground = (HBRUSH)(COLOR_WINDOW + 1); // Set background color

    RegisterClass(&wc);

    // Create window
    HWND hWnd = CreateWindowEx(
        0,                              // Optional window styles
        CLASS_NAME,                     // Window class
        L"Key Press Simulator",         // Window text
        WS_OVERLAPPEDWINDOW | WS_SIZEBOX | WS_MAXIMIZEBOX, // Window style

        // Size and position
        CW_USEDEFAULT, CW_USEDEFAULT, 400, 250,

        NULL,       // Parent window    
        NULL,       // Menu
        hInstance,  // Instance handle
        NULL        // Additional application data
    );

    if (!hWnd) {
        return 0;
    }

    ShowWindow(hWnd, nCmdShow);
    UpdateWindow(hWnd);

    // Run the message loop
    MSG msg = {};
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return 0;
}
