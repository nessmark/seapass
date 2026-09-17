# SeaPass - Maritime Ticketing & Passenger Manifest System

A modern maritime passenger ticketing, vessel scheduling, and manifest management system built with a **Laravel 12 REST API backend** and a cross-platform **Flutter Android client**.

> **Target Platform Constraints:**
> - **Host Operating System:** Windows 10 / Windows 11 (64-bit)
> - **Target Mobile Platform:** Android (Physical device or Android Emulator)

---

## Table of Contents
1. [System Overview & Tech Stack](#1-system-overview--tech-stack)
2. [Windows Prerequisites Setup Guide](#2-windows-prerequisites-setup-guide)
   - [Automated CLI Installation (winget)](#automated-cli-installation-winget)
   - [Manual Dependency Setup](#manual-dependency-setup)
   - [Windows System PATH Configuration](#windows-system-path-configuration)
   - [Verification Commands](#verification-commands)
3. [Backend Setup (Laravel 12)](#3-backend-setup-laravel-12)
   - [Quick Start via Batch Scripts](#quick-start-via-batch-scripts-recommended)
   - [Manual Backend Installation](#manual-backend-installation)
4. [Mobile Setup & Backend Connectivity Configuration](#4-mobile-setup--backend-connectivity-configuration)
   - [Detecting Windows Host IP](#detecting-windows-host-ip)
   - [Configuring API Endpoint in Flutter](#configuring-api-endpoint-in-flutter)
5. [Device Connection Procedures (Android)](#5-device-connection-procedures-android)
   - [Android Phone Preparation](#android-phone-preparation)
   - [Option A: Wired Connection (USB Cable + ADB Reverse)](#option-a-wired-connection-usb-cable--adb-reverse-fastest--most-reliable)
   - [Option B: Wireless Connection (Wi-Fi)](#option-b-wireless-connection-wi-fi)
     - [Android 11+ (Native Wireless Pairing)](#android-11-native-wireless-pairing--no-cable-needed)
     - [Android 10 & Older (Legacy TCP/IP Mode)](#android-10--older-legacy-tcpip-mode)
   - [Running Flutter App on Android](#running-flutter-app-on-android)
6. [Troubleshooting Checklist](#6-troubleshooting-checklist)

---

## 1. System Overview & Tech Stack

SeaPass consists of two decoupled sub-systems located within this repository:

| Component | Path | Technology | Description |
|---|---|---|---|
| **Backend API** | `./SeaPass` | Laravel 12, PHP 8.2+, MySQL 8+, Vite | RESTful JSON API handling authentication, bookings, schedules, QR validation, and admin web views. |
| **Mobile Client** | `./mApp` | Flutter 3.x, Dart 3.x, Android SDK | Android passenger app supporting QR ticketing, live schedule browsing, and manifest verification. |
| **Automation Scripts** | `./*.bat` | Windows Batch & PowerShell | Auto-detects network IP, opens firewall ports, sets up ADB reverse port forwarding, and starts servers. |

### Minimum System Requirements
- **OS:** Windows 10 (Version 2004 or higher) or Windows 11 (64-bit)
- **PHP:** 8.2 or 8.3 (with `pdo_mysql`, `curl`, `mbstring`, `openssl`, `fileinfo`, `sodium` extensions enabled)
- **Composer:** 2.6+
- **Database:** MySQL 8.0+ / MariaDB 10.5+
- **Node.js & NPM:** Node.js LTS (v18 or v20+)
- **Flutter SDK:** 3.19+ (Stable Channel)
- **Java JDK:** Eclipse Temurin / Oracle JDK 17 (Required by Android Gradle Plugin)
- **Android SDK & Tools:** Android SDK Command-line Tools, Android Platform-Tools (ADB API 34/35)

---

## 2. Windows Prerequisites Setup Guide

### Automated CLI Installation (winget)

Open **PowerShell as Administrator** and execute the following commands to install all core dependencies in one pass:

```powershell
# Core development runtime & tools
winget install -e --id Git.Git
winget install -e --id PHP.PHP.8.3
winget install -e --id Composer.Composer
winget install -e --id Oracle.MySQL
winget install -e --id OpenJS.NodeJS.LTS
winget install -e --id EclipseAdoptium.Temurin.17.JDK
winget install -e --id Flutter.Flutter
winget install -e --id Google.AndroidStudio
```

> *Tip: After installation, restart your terminal or log out and back into Windows so environment variables take effect.*

---

### Manual Dependency Setup

If you prefer installing tools manually:

1. **PHP 8.2 or 8.3:**
   - Download the **VS16 x64 Non-Thread Safe** or **Thread Safe** zip from [windows.php.net/download](https://windows.php.net/download/).
   - Extract to `C:\php`.
   - Rename `php.ini-development` to `php.ini`. Open it and enable the following lines (remove leading `;`):
     ```ini
     extension_dir = "ext"
     extension=curl
     extension=fileinfo
     extension=mbstring
     extension=mysqli
     extension=openssl
     extension=pdo_mysql
     ```
2. **Composer:**
   - Download and run the Windows installer from [getcomposer.org](https://getcomposer.org/Composer-Setup.exe).
   - Point the installer to `C:\php\php.exe`.
3. **Flutter SDK:**
   - Download the Windows bundle from [docs.flutter.dev](https://docs.flutter.dev/get-started/install/windows/mobile).
   - Extract to `C:\flutter` (do not install in `C:\Program Files` to prevent permission issues).
4. **Android Studio & Command-Line Tools:**
   - Download and install [Android Studio](https://developer.android.com/studio).
   - Open Android Studio > **More Actions** > **SDK Manager** > **SDK Tools**:
     - Check **Android SDK Build-Tools**
     - Check **Android SDK Command-line Tools (latest)**
     - Check **Android SDK Platform-Tools**
     - Click **Apply** and complete installation.

---

### Windows System PATH Configuration

Ensure your environment variables are configured correctly.

1. Press `Win + R`, type `sysdm.cpl`, and hit Enter.
2. Select the **Advanced** tab > click **Environment Variables...**.
3. Under **System variables** (or **User variables**), select `Path` and click **Edit...**.
4. Click **New** and add the following paths (adjust if your installation directories differ):

```text
C:\php
C:\ProgramData\ComposerSetup\bin
C:\flutter\bin
%LOCALAPPDATA%\Android\Sdk\platform-tools
%LOCALAPPDATA%\Android\Sdk\cmdline-tools\latest\bin
```

5. Under **User variables**, verify or create:
   - `JAVA_HOME` pointing to `C:\Program Files\Eclipse Adoptium\jdk-17...` (or your JDK 17 folder).
   - `ANDROID_HOME` pointing to `%LOCALAPPDATA%\Android\Sdk`.

---

### Verification Commands

Open a fresh PowerShell window and verify each tool:

```powershell
php -v
composer --version
node -v
git --version
adb version
flutter doctor -v
```

If `flutter doctor` indicates Android licenses are not accepted, run:
```powershell
flutter doctor --android-licenses
```
*(Press `y` to accept all licenses).*

---

## 3. Backend Setup (Laravel 12)

### Quick Start via Batch Scripts (Recommended)

This repository includes pre-built Windows batch scripts in the root directory:

1. **Open Port 8000 in Windows Defender Firewall:**
   Right-click `setup_firewall.bat` and click **Run as administrator**.
   *(Opens inbound TCP port 8000 so phones on the Wi-Fi network can connect).*

2. **Start Backend & Device Auto-Forwarding:**
   Double-click `start_server.bat`.
   This script will:
   - Auto-detect your PC's active Wi-Fi IPv4 address.
   - Detect connected USB Android devices and configure `adb reverse tcp:8000 tcp:8000`.
   - Free up port 8000 if occupied by lingering processes.
   - Start Laravel binding to all network interfaces (`0.0.0.0:8000`).

---

### Manual Backend Installation

If setting up the backend manually step-by-step:

1. Navigate to the backend directory:
   ```powershell
   cd d:\College_folder\BSIT3_CSU_2\IT_198\System\Admin\SeaPass
   ```

2. Install PHP and Node dependencies:
   ```powershell
   composer install
   npm install
   npm run build
   ```

3. Configure Environment file:
   ```powershell
   Copy-Item .env.example .env
   ```

4. Edit `.env` to match your local MySQL credentials:
   ```env
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=seapass_db
   DB_USERNAME=root
   DB_PASSWORD=your_mysql_password
   ```

5. Generate application encryption key:
   ```powershell
   php artisan key:generate
   ```

6. Run migrations and database seeders:
   ```powershell
   php artisan migrate --seed
   ```

7. Start the Laravel development server accessible to external devices:
   ```powershell
   php artisan serve --host=0.0.0.0 --port=8000
   ```

> **Note:** Binding to `--host=0.0.0.0` ensures the server listens on all network adapters, allowing your phone to communicate with the PC via Wi-Fi.

---

## 4. Mobile Setup & Backend Connectivity Configuration

### Detecting Windows Host IP

To connect your phone to the Laravel backend over Wi-Fi, determine your PC's local IP address:

```powershell
# Using ipconfig:
ipconfig

# Or directly extract your Wi-Fi IPv4 address with PowerShell:
(Get-NetIPAddress -AddressFamily IPv4 -InterfaceAlias "*Wi-Fi*").IPAddress
```
*Example IP:* `192.168.1.50`

---

### Configuring API Endpoint in Flutter

The mobile application's network configuration is located at:
`mApp/lib/config/api_config.dart`

Depending on your connection mode:

#### Approach A: USB ADB Reverse Mode (No IP changes needed)
When using a USB cable with `adb reverse tcp:8000 tcp:8000`, the phone communicates through `127.0.0.1:8000`. In `api_config.dart`:
```dart
static ConnectionMode currentMode = ConnectionMode.usbAdb; // 127.0.0.1:8000
```

#### Approach B: Local Wi-Fi Network
When running wirelessly over Wi-Fi, update `lanWifiHost` in `mApp/lib/config/api_config.dart`:
```dart
static String lanWifiHost = '192.168.1.50'; // Replace with your PC's Wi-Fi IP
static ConnectionMode currentMode = ConnectionMode.lanWifi;
```

#### Approach C: Build-Time Environment Override (No code edits)
You can inject the API base URL directly when launching the app:
```powershell
cd ..\mApp
flutter run --dart-define=SEAPASS_API_BASE_URL=http://192.168.1.50:8000/api
```

---

## 5. Device Connection Procedures (Android)

### Android Phone Preparation

1. Open **Settings** on your Android phone.
2. Navigate to **About phone** > tap **Build number** 7 times continuously until it displays *"You are now a developer!"*.
3. Go back to **Settings** > **System** (or **Additional settings**) > **Developer options**.
4. Enable **USB debugging**.

---

### Option A: Wired Connection (USB Cable + ADB Reverse) [Fastest & Most Reliable]

1. Connect your Android phone to the PC via a USB cable.
2. When prompted on the phone with *"Allow USB debugging?"*, check **Always allow from this computer** and tap **Allow**.
3. In PowerShell, forward port 8000 from the phone to your PC:
   ```powershell
   adb reverse tcp:8000 tcp:8000
   ```
   *(Or simply run `.\connect_usb.bat` from the project root).*
4. Verify device detection:
   ```powershell
   adb devices
   flutter devices
   ```
5. Run the mobile application:
   ```powershell
   cd mApp
   flutter pub get
   flutter run
   ```

---

### Option B: Wireless Connection (Wi-Fi)

Ensure your phone and PC are connected to the **same Wi-Fi network**.

#### Android 11+ (Native Wireless Pairing — No Cable Needed)

> **Important:** Android 11+ uses two different ports:
> 1. **Pairing Port:** Shown in the pairing popup (used only once with `adb pair`).
> 2. **Connection Port:** Shown on the main Wireless Debugging screen (used with `adb connect`).

1. On your phone: Open **Developer options** > Turn on **Wireless debugging**.
2. Tap the text **Wireless debugging** > Tap **Pair device with pairing code**.
   - Note the **6-digit pairing code**.
   - Note the **IP address & Pairing Port** (e.g., `192.168.1.45:38475`).
3. In your PC PowerShell terminal, run:
   ```powershell
   adb pair 192.168.1.45:38475
   ```
4. Enter the 6-digit Wi-Fi pairing code when prompted.
5. Close the pairing popup on your phone.
6. Check the main **Wireless debugging** screen under **IP address & Port** for your connection port (e.g., `192.168.1.45:41253`).
7. Connect to the device:
   ```powershell
   adb connect 192.168.1.45:41253
   ```
   *Expected output: `connected to 192.168.1.45:41253`*

---

#### Android 10 & Older (Legacy TCP/IP Mode)

Requires plugging in a USB cable once to switch the device's ADB daemon into TCP/IP mode.

1. Connect phone to PC with a USB cable.
2. Switch ADB on the phone to listen on TCP/IP port 5555:
   ```powershell
   adb tcpip 5555
   ```
   *Expected output: `restarting in TCP mode port: 5555`*
3. **Unplug the USB cable.**
4. Check your phone's Wi-Fi IP in **Settings** > **Wi-Fi** > Connected Network Details (e.g., `192.168.1.45`).
5. Connect wirelessly:
   ```powershell
   adb connect 192.168.1.45:5555
   ```
   *Expected output: `connected to 192.168.1.45:5555`*

---

### Running Flutter App on Android

Once connected (wired or wirelessly), target the device:

```powershell
# 1. List active devices and note the Device ID:
flutter devices

# 2. Run the application targeting your wireless or wired device:
cd mApp
flutter run -d <DEVICE_ID_OR_IP>

# Example:
flutter run -d 192.168.1.45:41253
```

---

## 6. Troubleshooting Checklist

### 1. Phone Cannot Reach Backend (`Connection timed out` / `SocketException`)
- **Windows Firewall Blocking:** Run `setup_firewall.bat` as Administrator, or manually open port 8000:
  ```powershell
  netsh advfirewall firewall add rule name="Laravel SeaPass (Port 8000)" dir=in action=allow protocol=TCP localport=8000 profile=any
  ```
- **Laravel Not Binding to All Hosts:** Make sure you launched with `--host=0.0.0.0` (not default `127.0.0.1`):
  ```powershell
  php artisan serve --host=0.0.0.0 --port=8000
  ```
- **AP / Client Isolation on Wi-Fi:** Many university, office, or public Wi-Fi networks block devices from communicating with each other.
  - *Fix:* Turn on **Mobile Hotspot** on your phone, connect your PC to the phone's hotspot, and re-check `ipconfig`.
  - *Alternative:* Use USB cable with `adb reverse tcp:8000 tcp:8000`.

### 2. ADB Device Shows as `unauthorized`
- Unlock your Android phone screen.
- Go to **Developer options** > tap **Revoke USB debugging authorizations**.
- Unplug and reconnect USB cable (or re-toggle Wireless Debugging).
- Tap **Allow** when the RSA key popup appears on your phone.
- Restart ADB server:
  ```powershell
  adb kill-server
  adb start-server
  adb devices
  ```

### 3. ADB Reverse Fails (`error: closed` or `device not found`)
- Ensure only one device is connected or specify device target using `-s`:
  ```powershell
  adb -s <DEVICE_ID> reverse tcp:8000 tcp:8000
  ```
- Check that ADB is running as Administrator if permission errors occur.

### 4. Wireless Debugging Disconnects After Inactivity / Reboot
- **Android 11+:** The connection port changes dynamically whenever Wi-Fi disconnects. Check the new port under **Wireless debugging** and run `adb connect <PHONE_IP>:<NEW_PORT>`.
- **Android 10:** The device resets TCP/IP mode on phone restart. Plug the USB cable back in once and re-run `adb tcpip 5555`.
