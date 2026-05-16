# 6 DOF Robotic Gripper Bluetooth Control App
## MIT App Inventor Complete Setup Guide

---

## 1. PROJECT OVERVIEW

This MIT App Inventor application provides full wireless control of a 6-DOF robotic gripper arm via Bluetooth. The app communicates with an Arduino/microcontroller that drives 7 servo motors (6 joints + 1 gripper).

**Features:**
- Real-time slider control for all 6 joints
- Gripper open/close buttons
- Bluetooth device connection management
- Command logging and status display
- Quick preset positions
- Live position feedback

---

## 2. COMPONENTS TO ADD IN MIT APP INVENTOR

### 2.1 Non-Visible Components
1. **BluetoothClient** (Palette → Connectivity)
   - Name: `BluetoothClient1`
   - This handles all wireless communication

2. **Clock** (Palette → Sensors)
   - Name: `Clock1`
   - TimerInterval: 100 ms
   - Enabled: checked
   - Purpose: Update status displays

3. **Notifier** (Palette → User Interface)
   - Name: `Notifier1`
   - Purpose: Show alerts and errors

### 2.2 Visible Components

#### Screen Properties
- Title: "6 DOF Gripper Control"
- AlignHorizontal: Center
- ScrollableArrangement: true (to handle small screens)

#### Top Section: Status Card
**VerticalArrangement** (StatusPanel)
- Width: Fill Parent
- Height: Auto
- Background Color: Light gray

Components inside:
- **Label** (StatusTitle)
  - Text: "Bluetooth Status"
  - FontSize: 14
  - TextColor: Dark gray

- **Label** (StatusLabel)
  - Name: `StatusDisplay`
  - Text: "Not Connected"
  - FontSize: 20
  - TextBold: true

- **HorizontalArrangement** (ButtonPanel)
  - **Button** (ConnectBtn)
    - Text: "Connect"
    - BackgroundColor: Green
  
  - **Button** (DisconnectBtn)
    - Text: "Disconnect"
    - BackgroundColor: Red

#### Middle Section: Joint Controls
**VerticalArrangement** (JointsPanel)

For each of 6 joints, create this pattern:

**Horizontal Arrangement** (Joint_N_Header)
- **Label** (Joint_N_Label)
  - Text: "Joint N: [Joint Description]"
  - FontSize: 16
  - TextBold: true

- **Label** (Joint_N_Value)
  - Name: `Label_J{N}_Value`
  - Text: "0°"
  - FontSize: 14

**Slider** (Slider_Joint_N)
- Name: `Slider_Joint{N}`
- Min: 0
- Max: 180
- CurrentValue: 0
- Width: Fill Parent
- Height: 50 pixels

---

### 2.3 Gripper Control Section
**HorizontalArrangement** (GripperPanel)
- Width: Fill Parent
- Height: Auto

**Button** (OpenButton)
- Name: `Button_GripperOpen`
- Text: "Open Gripper ✋"
- BackgroundColor: Green
- Width: Fill Parent (set to 50% within horizontal arrangement)
- Height: 60 pixels
- FontSize: 14

**Button** (CloseButton)
- Name: `Button_GripperClose`
- Text: "Close Gripper ✊"
- BackgroundColor: Orange
- Width: Fill Parent
- Height: 60 pixels
- FontSize: 14

---

### 2.4 Command Log Section
**VerticalArrangement** (LogPanel)

**Label** (LogTitle)
- Text: "Command Log"
- FontSize: 16
- TextBold: true

**TextBox** (CommandLog)
- Name: `CommandLog`
- Hint: "Commands will appear here..."
- Enabled: false (read-only)
- MultiLine: true
- Width: Fill Parent
- Height: 200 pixels
- FontSize: 12
- FontTypeface: Monospace

---

### 2.5 Preset Positions Section
**VerticalArrangement** (PresetPanel)

**Label** (PresetTitle)
- Text: "Quick Presets"
- FontSize: 16
- TextBold: true

**Grid Arrangement** (PresetGrid)
- Columns: 2
- Rows: 2

Create 4 buttons:
- **Button** `Preset_Home` - Text: "Home"
- **Button** `Preset_Rest` - Text: "Rest"
- **Button** `Preset_Pickup` - Text: "Pick Up"
- **Button** `Preset_Place` - Text: "Place"

All buttons:
- Width: Fill Parent
- Height: 60 pixels
- FontSize: 12

---

## 3. BLOCK LOGIC (VISUAL PROGRAMMING)

### 3.1 Initialization - Screen.Initialize Block
```
When Screen1.Initialize:
  - Set StatusDisplay.Text to "Not Connected"
  - Set Clock1.TimerEnabled to true
  - Initialize all sliders to 0
```

### 3.2 Send Command Procedure
```
Procedure SendCommand (command):
  If BluetoothClient1.IsConnected:
    - Call BluetoothClient1.SendText (command)
    - Call UpdateLog (command)
  Else:
    - Call Notifier1.ShowAlert ("Please connect to Bluetooth first!")
```

### 3.3 Each Slider Changed Event
```
When Slider_Joint1.Changed (thumbPosition):
  - Set Label_J1_Value.Text to thumbPosition + "°"
  - Call SendCommand ("J1:" + thumbPosition + ";")
  - Call UpdateLog ("J1:" + thumbPosition + ";")

(Repeat for Slider_Joint2 through Slider_Joint6)
```

### 3.4 Gripper Control
```
When Button_GripperOpen.Click:
  - Call SendCommand ("GRIP:OPEN;")
  - Call UpdateLog ("GRIP:OPEN;")

When Button_GripperClose.Click:
  - Call SendCommand ("GRIP:CLOSE;")
  - Call UpdateLog ("GRIP:CLOSE;")
```

### 3.5 Bluetooth Connection
```
When Button_Connect.Click:
  - Open system Bluetooth list picker
  - When device selected:
    - Call BluetoothClient1.SendText ("CONNECT;")
    - Set StatusDisplay.Text to "Connected"
    - Set StatusDisplay.TextColor to Green

When Button_Disconnect.Click:
  - Call BluetoothClient1.Close ()
  - Set StatusDisplay.Text to "Disconnected"
  - Set StatusDisplay.TextColor to Red
```

### 3.6 Update Log Procedure
```
Procedure UpdateLog (message):
  - Get current time
  - Append "[HH:MM:SS] " + message to CommandLog.Text
  - Scroll to bottom of log
```

### 3.7 Preset Positions
```
Procedure LoadPreset (presetName):
  Create dictionary of positions based on presetName:
  
  HOME:     J1=90, J2=90, J3=90, J4=90, J5=90, J6=90
  REST:     J1=0,  J2=45, J3=45, J4=90, J5=90, J6=0
  PICKUP:   J1=90, J2=45, J3=60, J4=90, J5=120,J6=0
  PLACE:    J1=90, J2=120,J3=45, J4=90, J5=90, J6=0
  
  For each joint:
    - Set slider value
    - Send command
```

### 3.8 Clock Timer Event
```
When Clock1.Timer:
  - Update StatusDisplay based on BluetoothClient1.IsConnected
  - Refresh all position labels from slider values
```

---

## 4. COMMUNICATION PROTOCOL

### Command Format
All commands end with semicolon (`;`) for message delimitation.

**Joint Control:**
```
J1:angle;   where angle is 0-180
J2:angle;
J3:angle;
J4:angle;
J5:angle;
J6:angle;
```

**Gripper Control:**
```
GRIP:OPEN;
GRIP:CLOSE;
```

**Connection:**
```
CONNECT;
DISCONNECT;
```

### Example Sequence
```
J1:45;          (Set joint 1 to 45°)
J2:90;          (Set joint 2 to 90°)
J3:120;         (Set joint 3 to 120°)
GRIP:OPEN;      (Open gripper)
[user moves slider]
J1:67;          (Joint 1 updated to 67°)
GRIP:CLOSE;     (Close gripper)
```

---

## 5. INSTALLING AND DEPLOYING

### 5.1 In MIT App Inventor
1. Go to http://appinventor.mit.edu
2. Sign in with Google account
3. Click "Create apps!"
4. Click "Start new project"
5. Name it: `RoboticGripperControl`
6. Build all components as described above
7. Add all block logic

### 5.2 Testing
1. Connect Android device via USB or WiFi
2. In MIT App Inventor: Connect → AI Companion
3. Scan QR code with phone
4. App installs on device
5. Test all sliders and buttons

### 5.3 Building APK
1. Build → App (provide .apk)
2. Install on Android device
3. Grant Bluetooth permissions when prompted

---

## 6. TROUBLESHOOTING

### App won't connect to Bluetooth
- Ensure HC-05 module is powered and in pairing mode (LED blinking)
- Check that Bluetooth is enabled on Android device
- Clear app data and reinstall
- Try "forget" device in Android settings and re-pair

### Sliders don't send commands
- Check StatusDisplay shows "Connected"
- Verify BluetoothClient1 serial transmission
- Add debug messages to command log
- Check baud rate matches: 9600

### Gripper doesn't respond
- Verify servo is connected to correct Arduino pin
- Test servo separately with Arduino IDE
- Check power supply to servo (should be 5V)
- Measure voltage at servo pin with multimeter

### App crashes on startup
- Check all component names match exactly
- Ensure all blocks reference correct components
- Test on emulator first
- Check manifest permissions (Bluetooth granted)

---

## 7. NEXT STEPS

1. **Real-time Feedback**: Modify Arduino code to send joint angles back to app for display
2. **Motion Recording**: Add ability to record and playback movement sequences
3. **Multi-gripper Support**: Extend to control multiple grippers simultaneously
4. **Graphical Position Display**: Add 2D/3D visualization of arm position
5. **PID Tuning Interface**: Add sliders to adjust PID parameters on servo controller

