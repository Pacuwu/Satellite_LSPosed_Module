# SatelliteIconHook — LSPosed Module

![Android](https://img.shields.io/badge/Android-16-green)
![LSPosed](https://img.shields.io/badge/LSPosed-Compatible-green)
![Status](https://img.shields.io/badge/Status-Beta-blue)

## ESPAÑOL:

Módulo LSPosed para forzar el ícono de conexión satelital (NTN)
en la status bar de **MistOS 4.5 "Velour"** (Android 16 QPR2).

---

## Requisitos

| Componente | Versión |
|---|---|
| Android Studio | Hedgehog 2023.1.1+ |
| Android Gradle Plugin | 8.3.0 |
| compileSdk | 36 (Android 16) |
| LSPosed | Última versión estable |
| Root | Magisk / KernelSU-Next / APatch |
| SIM | Dada de baja (necesaria para inicializar el stack de telefonía) |

---

## Compilar

1. Abrir el proyecto en Android Studio
2. `Build → Build APK(s)` (o `./gradlew assembleRelease`)
3. El APK se genera en `app/build/outputs/apk/`

---

## Instalar

```bash
# Opción A — ADB
adb install app/build/outputs/apk/release/app-release-unsigned.apk

# Opción B — Copiar al dispositivo y abrir con gestor de archivos
```

Luego en **LSPosed Manager**:
1. Activar el módulo "SatelliteIconHook"
2. Confirmar que el scope incluye **System Framework** y **System UI**
3. Reiniciar el dispositivo (o matar SystemUI con `adb shell killall com.android.systemui`)

---

## Depuración

Si el ícono no aparece, revisá el logcat:

```bash
adb logcat -s "SatelliteIconHook" "LSPosed" -v time
```

El módulo loguea:
- Qué hooks se aplicaron correctamente
- Qué campos de `MobileState` encontró
- Si no encontró el campo NTN → lista todos los campos disponibles
  para que puedas identificar el nombre correcto en tu build

### Ajustar campo NTN manualmente

Si el log dice "no se encontró campo NTN", buscá el nombre real:

```bash
# Extraer SystemUI del dispositivo
adb pull /system/priv-app/SystemUI/SystemUI.apk

# Abrir con jadx-gui y buscar:
# "isNonTerrestrialNetwork" | "mIsNtn" | "satellite" | "ntn"
# en la clase MobileState o MobileIconState
```

Luego agregá el nombre encontrado al array `NTN_FIELD_CANDIDATES`
en `SystemUIHook.java` y recompilá.

---

## Arquitectura del hook

```
system_server (proceso "android")
└── TelephonyHook
    ├── SatelliteManager.requestIsSatelliteEnabled()  → true
    ├── SatelliteManager.requestIsSatelliteSupported() → true
    ├── SatelliteController.isSatelliteEnabled()       → true
    ├── SatelliteController.isSatelliteSupported()     → true
    └── SatelliteController.*  (métodos alternativos)  → true

com.android.systemui
└── SystemUIHook
    ├── MobileSignalController.updateTelephony()
    │   └── MobileState.isNonTerrestrialNetwork = true
    ├── SatelliteViewModel / SatelliteInteractor (pipeline moderno)
    └── MobileIconInteractor (pipeline moderno)
```

---

## Notas

- El efecto es **cosmético**: el ícono se muestra pero no hay
  conectividad real sin hardware satelital activo.
- La SIM dada de baja **sí es necesaria** para que el proceso
  `SatelliteController` se inicialice correctamente.
- Probado contra el árbol fuente de MistOS 4.5 (branch 16.2 de LineageOS).
- Si actualizás MistOS, puede que algunos nombres de método cambien;
  revisá el logcat después de cada actualización.

ENGLISH:

---

LSPosed module to force the satellite connection icon (NTN)
in the status bar of MistOS 4.5 "Velour" (Android 16 QPR2).

---

| Component             | Version                                                  |
| --------------------- | -------------------------------------------------------- |
| Android Studio        | Hedgehog 2023.1.1+                                       |
| Android Gradle Plugin | 8.3.0                                                    |
| compileSdk            | 36 (Android 16)                                          |
| LSPosed               | Latest stable version                                    |
| Root                  | Magisk / KernelSU-Next / APatch                          |
| SIM                   | Deactivated (required to initialize the telephony stack) |

---

Build
Open the project in Android Studio
Build → Build APK(s) (or ./gradlew assembleRelease)
The APK will be generated at app/build/outputs/apk/

---

# Option A — ADB
adb install app/build/outputs/apk/release/app-release-unsigned.apk

# Option B — Copy to device and open with a file manager

Then in LSPosed Manager:

1. Enable the module "SatelliteIconHook"
2. Make sure the scope includes System Framework and System UI
3. Reboot the device (or kill SystemUI with adb shell killall com.android.systemui)

---

Debugging

If the icon does not appear, check logcat:
adb logcat -s "SatelliteIconHook" "LSPosed" -v time

The module logs:

- Which hooks were successfully applied
- Which MobileState fields were found
- If the NTN field was not found → it lists all available fields
so you can identify the correct name for your build

Manually adjust NTN field

If the log says "NTN field not found", look for the real name:

# Extract SystemUI from the device
adb pull /system/priv-app/SystemUI/SystemUI.apk

# Open with jadx-gui and search:
# "isNonTerrestrialNetwork" | "mIsNtn" | "satellite" | "ntn"
# in the MobileState or MobileIconState class
Then add the discovered name to the NTN_FIELD_CANDIDATES array
in SystemUIHook.java and rebuild.

---

system_server (process "android")
└── TelephonyHook
    ├── SatelliteManager.requestIsSatelliteEnabled()  → true
    ├── SatelliteManager.requestIsSatelliteSupported() → true
    ├── SatelliteController.isSatelliteEnabled()       → true
    ├── SatelliteController.isSatelliteSupported()     → true
    └── SatelliteController.*  (alternative methods)   → true

com.android.systemui
└── SystemUIHook
    ├── MobileSignalController.updateTelephony()
    │   └── MobileState.isNonTerrestrialNetwork = true
    ├── SatelliteViewModel / SatelliteInteractor (modern pipeline)
    └── MobileIconInteractor (modern pipeline)

---

Notes
The effect is cosmetic: the icon is shown but there is no real connectivity without active satellite hardware.
A deactivated SIM is still required for the SatelliteController process to initialize correctly.
Tested against MistOS 4.5 source tree (LineageOS branch 16.2).
If you update MistOS, some method names may change; check logcat after each update.
