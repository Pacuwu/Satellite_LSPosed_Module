# SatelliteIconHook — LSPosed Module

![Android](https://img.shields.io/badge/Android-16--green)
![MistOS](https://img.shields.io/badge/Compatible-version-4.5-grenn)

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

---

## ENGLISH:

# SatelliteIconHook — LSPosed Module[cite: 7]

LSPosed module to force the satellite connection icon (NTN) 
in the status bar of **MistOS 4.5 "Velour"** (Android 16 QPR2).[cite: 7]

---

## Requirements[cite: 7]

| Component | Version |
|---|---|
| Android Studio | Hedgehog 2023.1.1+[cite: 7] |
| Android Gradle Plugin | 8.3.0[cite: 7] |
| compileSdk | 36 (Android 16)[cite: 7] |
| LSPosed | Latest stable version[cite: 7] |
| Root | Magisk / KernelSU-Next / APatch[cite: 7] |
| SIM | Deactivated (required to initialize the telephony stack)[cite: 7] |

---

## Compilation[cite: 7]

1. Open the project in Android Studio.[cite: 7]
2. `Build → Build APK(s)` (or `./gradlew assembleRelease`).[cite: 7]
3. The APK is generated in `app/build/outputs/apk/`.[cite: 7]

---

## Installation[cite: 7]

```bash
# Option A — ADB
adb install app/build/outputs/apk/release/app-release-unsigned.apk

# Option B — Copy to device and open with a file manager
```[cite: 7]

Then, in **LSPosed Manager**:[cite: 7]
1. Enable the "SatelliteIconHook" module.[cite: 7]
2. Confirm the scope includes **System Framework** and **System UI**.[cite: 7]
3. Restart the device (or kill SystemUI using `adb shell killall com.android.systemui`).[cite: 7]

---

## Debugging[cite: 7]

If the icon does not appear, check the logcat:[cite: 7]

```bash
adb logcat -s "SatelliteIconHook" "LSPosed" -v time
```[cite: 7]

The module logs:[cite: 7]
- Which hooks were successfully applied.[cite: 7]
- Which `MobileState` fields were found.[cite: 7]
- If the NTN field was not found → lists all available fields so you can identify the correct name in your build.[cite: 7]

### Adjusting the NTN field manually[cite: 7]

If the log says "NTN field not found," search for the actual name:[cite: 7]

```bash
# Pull SystemUI from the device
adb pull /system/priv-app/SystemUI/SystemUI.apk

# Open with jadx-gui and search for:
# "isNonTerrestrialNetwork" | "mIsNtn" | "satellite" | "ntn"
# within the MobileState or MobileIconState classes
```[cite: 7]

Then, add the found name to the `NTN_FIELD_CANDIDATES` array in `SystemUIHook.java` and recompile.[cite: 7]

---

## Hook Architecture[cite: 7]

```
system_server ("android" process)
└── TelephonyHook
    ├── SatelliteManager.requestIsSatelliteEnabled()  → true
    ├── SatelliteManager.requestIsSatelliteSupported() → true
    ├── SatelliteController.isSatelliteEnabled()       → true
    ├── SatelliteController.isSatelliteSupported()     → true
    └── SatelliteController.*  (alternative methods)  → true

com.android.systemui
└── SystemUIHook
    ├── MobileSignalController.updateTelephony()
    │   └── MobileState.isNonTerrestrialNetwork = true
    ├── SatelliteViewModel / SatelliteInteractor (modern pipeline)
    └── MobileIconInteractor (modern pipeline)
```[cite: 7]

---

## Notes[cite: 7]

- The effect is **cosmetic**: the icon is displayed, but there is no actual connectivity without active satellite hardware.[cite: 7]
- A **deactivated SIM is required** for the `SatelliteController` process to initialize correctly.[cite: 7]
- Tested against the MistOS 4.5 source tree (LineageOS branch 16.2).[cite: 7]
- If you update MistOS, some method names might change; check the logcat after every update.[cite: 7]
