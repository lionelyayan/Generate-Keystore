# Generate Android Keystore — Windows & macOS

Panduan ini menjelaskan cara membuat **Android Keystore (`.jks`)** dan file **`key.properties`** untuk signing aplikasi Android pada project Flutter.

Keystore digunakan untuk menandatangani aplikasi sebelum melakukan build release seperti:

```text
.apk
.aab
```

> ⚠️ **Penting:** Simpan file `.jks`, alias, dan password di tempat yang aman. Jangan pernah commit file tersebut ke repository Git publik.

---

# Prasyarat

Pastikan Java / JDK sudah tersedia karena proses pembuatan keystore menggunakan:

```bash
keytool
```

Cek apakah `keytool` tersedia:

```bash
keytool -help
```

Jika perintah tersebut dapat dijalankan, proses generate keystore dapat dilanjutkan.

---

# Windows

Pada Windows kita dapat menggunakan file:

```text
.bat
```

agar proses generate keystore dan `key.properties` dapat dilakukan secara otomatis.

## 1. Buat File `.bat`

Buat file baru, misalnya:

```text
generate-keystore.bat
```

Isi dengan kode berikut:

```bat
@echo off
setlocal EnableDelayedExpansion

echo.
echo ========== Membuat Keystore Flutter + key.properties ==========
echo.

:: ===== Lokasi penyimpanan otomatis ke Desktop
set "TARGET_DIR=%USERPROFILE%\Desktop"

echo Keystore dan key.properties akan disimpan di:
echo %TARGET_DIR%

:: ===== Input nama file keystore
echo.
set /p KEYSTORE_NAME=Nama file keystore (contoh: my-release-key.jks): 

if "%KEYSTORE_NAME%"=="" (
    echo.
    echo ❌ Nama keystore tidak boleh kosong!
    pause
    exit /b
)

set "KEYSTORE_PATH=%TARGET_DIR%\%KEYSTORE_NAME%"

:: ===== Input lainnya
echo.
set /p ALIAS_NAME=Alias key (contoh: release): 
set /p STORE_PASS=Password keystore: 
set /p KEY_PASS=Password key: 
set /p DNAME=Distinguished Name (contoh: CN=Nama, OU=Unit, O=Organisasi, L=Kota, S=Provinsi, C=ID): 

:: ===== Buat folder jika belum ada
if not exist "%TARGET_DIR%" (
    echo Membuat folder: %TARGET_DIR%
    mkdir "%TARGET_DIR%"
)

:: ===== Jalankan keytool
echo.
echo Membuat keystore di:
echo %KEYSTORE_PATH%

keytool -genkey -v ^
 -keystore "%KEYSTORE_PATH%" ^
 -alias "%ALIAS_NAME%" ^
 -keyalg RSA ^
 -keysize 2048 ^
 -validity 10000 ^
 -storepass "%STORE_PASS%" ^
 -keypass "%KEY_PASS%" ^
 -dname "%DNAME%"

if %ERRORLEVEL% NEQ 0 (
    echo.
    echo ❌ Gagal membuat keystore.
    echo Silakan periksa kembali input atau konfigurasi keytool.
    pause
    exit /b
)

:: ===== Buat key.properties
set "KEY_PROP_PATH=%TARGET_DIR%\key.properties"

(
    echo storePassword=%STORE_PASS%
    echo keyPassword=%KEY_PASS%
    echo keyAlias=%ALIAS_NAME%
    echo storeFile=%KEYSTORE_NAME%
) > "%KEY_PROP_PATH%"

if exist "%KEY_PROP_PATH%" (
    echo.
    echo ✅ key.properties berhasil dibuat di:
    echo %KEY_PROP_PATH%
) else (
    echo.
    echo ❌ Gagal membuat file key.properties.
    pause
    exit /b
)

echo.
echo ==========================================
echo ✅ Proses selesai
echo ==========================================
echo.
echo File disimpan di:
echo %TARGET_DIR%
echo.

pause
endlocal
```

---

## 2. Simpan File

Simpan file dengan ekstensi:

```text
.bat
```

Contoh:

```text
generate-keystore.bat
```

---

## 3. Jalankan

Double-click:

```text
generate-keystore.bat
```

Kemudian isi informasi yang diminta.

Contoh:

```text
Nama file keystore:
my-release-key.jks

Alias:
release

Password keystore:
********

Password key:
********

Distinguished Name:
CN=Nama Anda, OU=Development, O=Nama Organisasi, L=Kota, S=Provinsi, C=ID
```

---

## 4. Hasil

File akan dibuat di Desktop:

```text
Desktop/
├── my-release-key.jks
└── key.properties
```

---

# macOS

Pada macOS konsepnya sama dengan Windows.

Perbedaannya hanya pada jenis script yang digunakan:

```text
Windows → .bat
macOS   → .command / .sh
```

Untuk penggunaan dengan double-click melalui Finder, `.command` lebih praktis.

---

## 1. Buat File `.command`

Buat file:

```text
generate-keystore.command
```

Kemudian isi dengan kode berikut:

```bash
#!/bin/bash

clear

echo ""
echo "========== Membuat Keystore Flutter + key.properties =========="
echo ""

# ========================================
# Lokasi penyimpanan otomatis ke Desktop
# ========================================
TARGET_DIR="$HOME/Desktop"

echo "Keystore dan key.properties akan disimpan di:"
echo "$TARGET_DIR"

# ========================================
# Cek keytool
# ========================================
if ! command -v keytool >/dev/null 2>&1; then
    echo ""
    echo "❌ keytool tidak ditemukan."
    echo "Pastikan Java/JDK sudah terinstall."
    echo ""
    read -p "Tekan Enter untuk keluar..."
    exit 1
fi

# ========================================
# Input nama keystore
# ========================================
echo ""

read -p "Nama file keystore (contoh: my-release-key.jks): " KEYSTORE_NAME

if [ -z "$KEYSTORE_NAME" ]; then
    echo ""
    echo "❌ Nama keystore tidak boleh kosong!"
    read -p "Tekan Enter untuk keluar..."
    exit 1
fi

# Tambahkan ekstensi .jks jika belum ada
if [[ "$KEYSTORE_NAME" != *.jks ]]; then
    KEYSTORE_NAME="${KEYSTORE_NAME}.jks"
fi

KEYSTORE_PATH="$TARGET_DIR/$KEYSTORE_NAME"

# ========================================
# Input Alias
# ========================================
echo ""

read -p "Alias key (contoh: release): " ALIAS_NAME

if [ -z "$ALIAS_NAME" ]; then
    echo ""
    echo "❌ Alias tidak boleh kosong!"
    read -p "Tekan Enter untuk keluar..."
    exit 1
fi

# ========================================
# Input Password
# ========================================
echo ""

read -s -p "Password keystore: " STORE_PASS
echo ""

if [ -z "$STORE_PASS" ]; then
    echo ""
    echo "❌ Password keystore tidak boleh kosong!"
    read -p "Tekan Enter untuk keluar..."
    exit 1
fi

read -s -p "Password key: " KEY_PASS
echo ""

if [ -z "$KEY_PASS" ]; then
    echo ""
    echo "❌ Password key tidak boleh kosong!"
    read -p "Tekan Enter untuk keluar..."
    exit 1
fi

# ========================================
# Distinguished Name
# ========================================
echo ""
echo "Contoh:"
echo "CN=Nama Anda, OU=Development, O=Nama Organisasi, L=Kota, ST=Provinsi, C=ID"
echo ""

read -p "Distinguished Name: " DNAME

if [ -z "$DNAME" ]; then
    echo ""
    echo "❌ Distinguished Name tidak boleh kosong!"
    read -p "Tekan Enter untuk keluar..."
    exit 1
fi

# ========================================
# Buat folder jika belum ada
# ========================================
mkdir -p "$TARGET_DIR"

# ========================================
# Cek file existing
# ========================================
if [ -f "$KEYSTORE_PATH" ]; then
    echo ""
    echo "⚠️ File sudah ada:"
    echo "$KEYSTORE_PATH"
    echo ""

    read -p "Timpa file tersebut? (y/n): " OVERWRITE

    if [[ "$OVERWRITE" != "y" && "$OVERWRITE" != "Y" ]]; then
        echo ""
        echo "Proses dibatalkan."
        read -p "Tekan Enter untuk keluar..."
        exit 0
    fi

    rm "$KEYSTORE_PATH"
fi

# ========================================
# Generate Keystore
# ========================================
echo ""
echo "Membuat keystore..."
echo ""

keytool \
    -genkeypair \
    -v \
    -keystore "$KEYSTORE_PATH" \
    -alias "$ALIAS_NAME" \
    -keyalg RSA \
    -keysize 2048 \
    -validity 10000 \
    -storepass "$STORE_PASS" \
    -keypass "$KEY_PASS" \
    -dname "$DNAME"

if [ $? -ne 0 ]; then
    echo ""
    echo "❌ Gagal membuat keystore."
    echo "Silakan periksa input atau konfigurasi Java/keytool."
    echo ""
    read -p "Tekan Enter untuk keluar..."
    exit 1
fi

# ========================================
# Generate key.properties
# ========================================
KEY_PROP_PATH="$TARGET_DIR/key.properties"

cat > "$KEY_PROP_PATH" <<EOF
storePassword=$STORE_PASS
keyPassword=$KEY_PASS
keyAlias=$ALIAS_NAME
storeFile=$KEYSTORE_NAME
EOF

# ========================================
# Validasi
# ========================================
if [ -f "$KEY_PROP_PATH" ]; then
    echo ""
    echo "✅ key.properties berhasil dibuat."
else
    echo ""
    echo "❌ Gagal membuat key.properties."
    read -p "Tekan Enter untuk keluar..."
    exit 1
fi

# ========================================
# Finish
# ========================================
echo ""
echo "=========================================="
echo "✅ Proses selesai"
echo "=========================================="
echo ""

echo "Keystore:"
echo "$KEYSTORE_PATH"

echo ""
echo "key.properties:"
echo "$KEY_PROP_PATH"

echo ""
echo "Selanjutnya pindahkan:"
echo ""

echo "$KEYSTORE_NAME"
echo "→ <project-root>/android/app/$KEYSTORE_NAME"

echo ""
echo "key.properties"
echo "→ <project-root>/android/key.properties"

echo ""

read -p "Tekan Enter untuk selesai..."
```

---

## 2. Berikan Permission

File `.command` perlu mendapatkan permission agar dapat dijalankan.

Jika file berada di Desktop:

```bash
chmod +x ~/Desktop/generate-keystore.command
```

Jika berada di folder lain, sesuaikan path.

Contoh:

```bash
chmod +x ~/Downloads/generate-keystore.command
```

Permission ini cukup diberikan satu kali.

---

## 3. Jalankan

Setelah permission diberikan, double-click:

```text
generate-keystore.command
```

Terminal akan terbuka dan meminta informasi keystore.

---

## 4. Hasil

File akan dibuat di Desktop:

```text
Desktop/
├── my-release-key.jks
└── key.properties
```

---

# Konfigurasi Project Flutter

Setelah keystore berhasil dibuat, langkah berikutnya sama pada Windows maupun macOS.

## 1. Pindahkan Keystore

Pindahkan:

```text
my-release-key.jks
```

ke:

```text
<project-root>/android/app/my-release-key.jks
```

---

## 2. Pindahkan `key.properties`

Pindahkan:

```text
key.properties
```

ke:

```text
<project-root>/android/key.properties
```

Struktur akhirnya:

```text
project-root/
└── android/
    ├── key.properties
    └── app/
        └── my-release-key.jks
```

---

# Isi `key.properties`

Contoh:

```properties
storePassword=YOUR_STORE_PASSWORD
keyPassword=YOUR_KEY_PASSWORD
keyAlias=release
storeFile=my-release-key.jks
```

Nilai tersebut otomatis dibuat oleh script berdasarkan data yang dimasukkan.

---

# `.gitignore`

Tambahkan konfigurasi berikut:

```gitignore
# Android keystore & signing configuration
android/key.properties
android/app/*.jks
android/app/*.keystore
```

Dengan demikian file sensitif seperti:

```text
.jks
key.properties
```

tidak ikut masuk ke repository Git.

---

# Konfigurasi `build.gradle`

Untuk project yang masih menggunakan Groovy:

```text
android/app/build.gradle
```

Tambahkan:

```gradle
def keystorePropertiesFile = rootProject.file("key.properties")
def keystoreProperties = new Properties()

if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(
        new FileInputStream(keystorePropertiesFile)
    )
}

android {

    ...

    signingConfigs {
        release {
            storeFile file(keystoreProperties['storeFile'])
            storePassword keystoreProperties['storePassword']
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

---

# Konfigurasi `build.gradle.kts`

Untuk project yang menggunakan Kotlin DSL:

```text
android/app/build.gradle.kts
```

Tambahkan di bagian atas:

```kotlin
import java.util.Properties
import java.io.FileInputStream
```

Kemudian:

```kotlin
val keystoreProperties = Properties()
val keystorePropertiesFile = rootProject.file("key.properties")

if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(
        FileInputStream(keystorePropertiesFile)
    )
}
```

Pada:

```kotlin
android {
```

tambahkan:

```kotlin
signingConfigs {
    create("release") {
        keyAlias = keystoreProperties["keyAlias"] as String
        keyPassword = keystoreProperties["keyPassword"] as String
        storeFile = file(
            keystoreProperties["storeFile"] as String
        )
        storePassword = keystoreProperties["storePassword"] as String
    }
}
```

Kemudian pada `buildTypes`:

```kotlin
buildTypes {
    release {
        signingConfig = signingConfigs.getByName("release")
    }
}
```

Contoh:

```kotlin
import java.util.Properties
import java.io.FileInputStream

val keystoreProperties = Properties()
val keystorePropertiesFile = rootProject.file("key.properties")

if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(
        FileInputStream(keystorePropertiesFile)
    )
}

android {

    ...

    signingConfigs {
        create("release") {
            keyAlias = keystoreProperties["keyAlias"] as String
            keyPassword = keystoreProperties["keyPassword"] as String
            storeFile = file(
                keystoreProperties["storeFile"] as String
            )
            storePassword = keystoreProperties["storePassword"] as String
        }
    }

    buildTypes {
        release {
            signingConfig = signingConfigs.getByName("release")
        }
    }
}
```

---

# Build Android App Bundle

Setelah signing configuration selesai:

```bash
flutter clean
```

Kemudian:

```bash
flutter pub get
```

Build Android App Bundle:

```bash
flutter build appbundle --release
```

Hasil build:

```text
build/app/outputs/bundle/release/app-release.aab
```

File `.aab` dapat digunakan untuk distribusi aplikasi melalui Google Play atau distribution channel yang mendukung Android App Bundle.

---

# Build APK

Jika membutuhkan APK:

```bash
flutter build apk --release
```

Hasil:

```text
build/app/outputs/flutter-apk/app-release.apk
```

---

# Cek Informasi Keystore

Untuk melihat informasi keystore:

```bash
keytool -list -v -keystore my-release-key.jks
```

Kemudian masukkan password keystore.

Informasi yang akan ditampilkan antara lain:

```text
Alias name
Creation date
Entry type
Certificate fingerprints
SHA1
SHA256
```

---

# Mendapatkan SHA-1 dan SHA-256

Jalankan:

```bash
keytool -list -v \
-keystore my-release-key.jks \
-alias release
```

Contoh output:

```text
Certificate fingerprints:

SHA1:
AA:BB:CC:DD:...

SHA256:
AA:BB:CC:DD:...
```

Fingerprint tersebut sering dibutuhkan untuk integrasi layanan seperti:

* Firebase
* Google Sign-In
* Google Maps Platform
* OAuth
* API yang menggunakan certificate fingerprint

---

# Struktur Project

Contoh struktur setelah konfigurasi:

```text
project-root/
├── android/
│   ├── app/
│   │   ├── build.gradle.kts
│   │   └── my-release-key.jks
│   │
│   ├── key.properties
│   └── ...
│
├── lib/
├── pubspec.yaml
└── .gitignore
```

---

# Workflow Windows

```text
generate-keystore.bat
        ↓
Input informasi keystore
        ↓
Desktop
├── my-release-key.jks
└── key.properties
        ↓
Copy ke project
        ↓
android/app/my-release-key.jks
android/key.properties
        ↓
Konfigurasi Gradle
        ↓
flutter build appbundle --release
        ↓
app-release.aab
```

---

# Workflow macOS

```text
generate-keystore.command
        ↓
chmod +x
        ↓
Double-click
        ↓
Input informasi keystore
        ↓
Desktop
├── my-release-key.jks
└── key.properties
        ↓
Copy ke project
        ↓
android/app/my-release-key.jks
android/key.properties
        ↓
Konfigurasi Gradle
        ↓
flutter build appbundle --release
        ↓
app-release.aab
```

---

# Catatan Penting

Keystore merupakan bagian penting dari proses signing aplikasi Android.

Simpan dengan aman:

```text
*.jks
```

beserta informasi:

```text
storePassword
keyPassword
keyAlias
```

Disarankan memiliki backup di lokasi aman seperti:

* Encrypted cloud storage
* External storage terenkripsi
* Password manager
* Secure company/team storage

Jangan hanya menyimpan keystore di dalam folder project.

Jangan commit:

```text
android/key.properties
android/app/*.jks
android/app/*.keystore
```

ke repository Git.

Untuk aplikasi yang sudah dirilis, gunakan signing key yang sesuai untuk proses update aplikasi berikutnya.

> ⚠️ Jangan membuat keystore baru untuk setiap update aplikasi yang sama tanpa memahami konfigurasi signing dan mekanisme **Play App Signing** yang digunakan pada aplikasi tersebut.
