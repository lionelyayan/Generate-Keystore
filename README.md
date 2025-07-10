# Generate-Keystore

1. Buat File .bat baru
```
@echo off
setlocal EnableDelayedExpansion

echo.
echo ========== Membuat Keystore Flutter + key.properties ==========
echo.

:: ===== Lokasi penyimpanan otomatis ke Desktop
set "TARGET_DIR=%USERPROFILE%\Desktop"
echo Keystore dan key.properties akan disimpan di: %TARGET_DIR%

:: ===== Input nama file keystore
echo.
set /p KEYSTORE_NAME=Nama file keystore (contoh: my-release-key.jks): 
if "%KEYSTORE_NAME%"=="" (
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
set /p DNAME=Distinguished Name (contoh: CN=Nama, OU=Unit, O=Org, L=Kota, S=Provinsi, C=ID): 

:: ===== Buat folder jika belum ada
if not exist "%TARGET_DIR%" (
    echo Membuat folder: %TARGET_DIR%
    mkdir "%TARGET_DIR%"
)

:: ===== Jalankan keytool
echo.
echo Membuat keystore di: %KEYSTORE_PATH%
keytool -genkey -v -keystore "%KEYSTORE_PATH%" -alias "%ALIAS_NAME%" ^
 -keyalg RSA -keysize 2048 -validity 10000 ^
 -storepass "%STORE_PASS%" -keypass "%KEY_PASS%" ^
 -dname "%DNAME%"

if %ERRORLEVEL% NEQ 0 (
    echo.
    echo ❌ Gagal membuat keystore. Silakan periksa kembali input atau keytool Anda.
    pause
    exit /b
)

:: ===== Buat key.properties di Desktop
set "KEY_PROP_PATH=%TARGET_DIR%\key.properties"

(
    echo storePassword=%STORE_PASS%
    echo keyPassword=%KEY_PASS%
    echo keyAlias=%ALIAS_NAME%
    echo storeFile=%KEYSTORE_NAME%
) > "%KEY_PROP_PATH%"

if exist "%KEY_PROP_PATH%" (
    echo.
    echo ✔ key.properties berhasil dibuat di: %KEY_PROP_PATH%
) else (
    echo ❌ Gagal membuat file key.properties.
    pause
    exit /b
)

echo.
echo ✅ Proses selesai. File disimpan di: %TARGET_DIR%
pause
endlocal

```

2. Simpan kode tersebut dengan ekstensi .bat
3. Jalankan file tersebut
4. Hasil output berada di Desktop (keystore dan key.properties)
---
### Berikutnya
1. Pindahkan file .jks ke <project-root>/android/app/my-release-key.jks
2. Pindahkan file key.properties ke <project-root>/android/key.properties
3. Tambahkan kode pada .gitignote ```# Android keystore & config
android/key.properties
android/app/*.jks```
4. Di android/app/build.gradle, pastikan bagian ini ada (biasanya sudah ada secara default):
```
def keystorePropertiesFile = rootProject.file("key.properties")
def keystoreProperties = new Properties()
keystoreProperties.load(new FileInputStream(keystorePropertiesFile))

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
            ...
        }
    }
}
```
