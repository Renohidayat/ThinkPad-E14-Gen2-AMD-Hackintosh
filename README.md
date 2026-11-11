# Lenovo ThinkPad E14 Gen 2 (AMD) — OpenCore EFI

[ **English** ](#english) • [ **Bahasa Indonesia** ](#bahasa-indonesia)

---

<a name="english"></a>
## English

A verified OpenCore configuration for running macOS Ventura on the **Lenovo ThinkPad E14 Gen 2 (Type 20T9)** equipped with an **AMD Ryzen 5 4500U (Renoir)** processor.

### Hardware Specifications

| Component | Specification |
| :--- | :--- |
| **Model** | Lenovo ThinkPad E14 Gen 2 (Type 20T9) |
| **Processor** | AMD Ryzen 5 4500U (6 Cores, 6 Threads, Zen 2 Renoir) |
| **Graphics** | AMD Radeon Vega 6 (Integrated iGPU) |
| **Display** | 14.0" Full HD (1920x1080) IPS |
| **Audio** | Realtek ALC257 (Layout ID `96` or `11`) |
| **Touchpad / TrackPoint** | Synaptics / ELAN PS/2 Keyboard & TrackPoint |
| **Bootloader** | OpenCore 1.0.8 |
| **Target OS** | macOS Ventura (13.x) |

### Feature & Hardware Status

- **CPU Power Management**: Working (`AMDRyzenCPUPowerManagement.kext` + `SMCAMDProcessor.kext`)
- **Graphics Acceleration**: Working with full Metal 2/3 hardware acceleration via `NootedRed.kext`
- **Display Brightness Control**: Working via `SSDT-PNLF.aml` and `BrightnessKeys.kext`
- **Keyboard & TrackPoint**: Working via `VoodooPS2Controller.kext`
- **Battery Readout**: Working via `ECEnabler.kext` + `SMCBatteryManager.kext`
- **USB Power & Sleep**: Fully configured via `SSDT-USBX.aml` and `SSDT-USB-Reset.aml`
- **Audio Output & Mic**: Working via `AppleALC.kext`

### Key Technical Solution: Bypassing `[EB|LOG:EXITBS:START]`

On AMD Renoir ThinkPads, standard OpenCore quirks often freeze indefinitely at `#[EB|LOG:EXITBS:START]`. This configuration solves that through:

1. **UEFI Memory Map Handling**:
   - `EnableWriteUnprotector` = `True`
   - `RebuildAppleMemoryMap` = `False`
   - `SyncRuntimePermissions` = `False`
   - `SetupVirtualMap` = `True`
   *Explanation:* Lenovo ThinkPad AMD firmware conflicts with OpenCore runtime memory rebuilds. Disabling runtime rebuild avoids the transition stall.

2. **AMD Vanilla Core Topology**:
   - `ProvideCurrentCpuInfo` = `True`
   *Explanation:* Required by universal AMD Vanilla kernel patches to feed the correct CPU core layout to macOS.

3. **Kernel Panic Prevention**:
   - `AppleMCEReporterDisabler.kext` prevents the immediate panic caused by Apple's machine-check reporter on non-Intel CPUs.

4. **Embedded Controller**:
   - `SSDT-EC.aml` targets `\_SB.PCI0.LPC0.EC` matching the ThinkPad DSDT LPC bus hierarchy.

### Setup Instructions

1. **Generate Unique Serial Numbers**:
   - Open `EFI/OC/config.plist` using ProperTree or OpenCore Auxiliary Tools.
   - Go to `PlatformInfo` -> `Generic`.
   - Set `SystemProductName` to `MacBookPro16,3`.
   - Generate `SystemSerialNumber`, `MLB`, and `SystemUUID` using `macserial`.
2. **BIOS Settings**:
   - **Disable**: Secure Boot, Fast Boot.
   - **Enable**: AMD V(TM) Technology (SVM), UEFI Only Mode.
3. **Booting**:
   - Copy the `EFI` directory to your USB FAT32 EFI partition.
   - In the OpenCore menu, run **Reset NVRAM** once.
   - Select the macOS installer to proceed.

---

<a name="bahasa-indonesia"></a>
## Bahasa Indonesia

Konfigurasi OpenCore yang telah diuji untuk menjalankan macOS Ventura pada laptop **Lenovo ThinkPad E14 Gen 2 (Tipe 20T9)** dengan prosesor **AMD Ryzen 5 4500U (Renoir)**.

### Spesifikasi Perangkat Keras

| Komponen | Spesifikasi |
| :--- | :--- |
| **Model Laptop** | Lenovo ThinkPad E14 Gen 2 (Type 20T9) |
| **Prosesor** | AMD Ryzen 5 4500U (6 Core, 6 Thread, Zen 2 Renoir) |
| **Kartu Grafis** | AMD Radeon Vega 6 (iGPU Terintegrasi) |
| **Layar** | 14.0" Full HD (1920x1080) IPS |
| **Audio** | Realtek ALC257 (Layout ID `96` atau `11`) |
| **Keyboard & Touchpad** | Keyboard PS/2, TrackPoint & Touchpad Synaptics |
| **Bootloader** | OpenCore 1.0.8 |
| **Target Sistem Operasi** | macOS Ventura (13.x) |

### Status Fungsi Perangkat Keras

- **Manajemen Daya CPU**: Berfungsi normal (`AMDRyzenCPUPowerManagement.kext` + `SMCAMDProcessor.kext`)
- **Akselerasi Grafis**: Berfungsi penuh dengan dukungan Metal 2/3 melalui `NootedRed.kext`
- **Pengatur Kecerahan Layar**: Berfungsi melalui `SSDT-PNLF.aml` dan `BrightnessKeys.kext`
- **Keyboard & TrackPoint**: Berfungsi melalui `VoodooPS2Controller.kext`
- **Indikator Persentase Baterai**: Berfungsi melalui `ECEnabler.kext` + `SMCBatteryManager.kext`
- **Port & Manajemen Daya USB**: Berfungsi melalui `SSDT-USBX.aml` dan `SSDT-USB-Reset.aml`
- **Audio & Mikrofon**: Berfungsi melalui `AppleALC.kext`

### Penjelasan Teknis: Solusi Masalah `[EB|LOG:EXITBS:START]`

Kebanyakan laptop ThinkPad AMD Ryzen 4000 mengalami macet total (*freeze*) pada baris log `#[EB|LOG:EXITBS:START]`. Konfigurasi ini menyelesaikannya dengan kombinasi parameter berikut:

1. **Penyesuaian Peta Memori UEFI**:
   - `EnableWriteUnprotector` = `True`
   - `RebuildAppleMemoryMap` = `False`
   - `SyncRuntimePermissions` = `False`
   - `SetupVirtualMap` = `True`
   *Alasan:* Firmware BIOS Lenovo AMD mengalami bentrok jika OpenCore membangun ulang tabel memori saat runtime. Mematikan `RebuildAppleMemoryMap` memungkinkan kernel masuk dengan mulus.

2. **Topologi Core AMD Vanilla**:
   - `ProvideCurrentCpuInfo` = `True`
   *Alasan:* Wajib aktif agar kumpulan patch kernel AMD Vanilla dapat memberikan informasi core CPU yang valid ke kernel macOS.

3. **Pencegahan Kernel Panic**:
   - `AppleMCEReporterDisabler.kext` digunakan untuk mencegah panic bawaan sistem macOS pada CPU non-Intel.

4. **Embedded Controller**:
   - `SSDT-EC.aml` mengarah langsung ke jalur perangkat `\_SB.PCI0.LPC0.EC` sesuai struktur DSDT ThinkPad.

### Panduan Penggunaan

1. **Mengisi Nomor Seri Apple (SMBIOS)**:
   - Buka `EFI/OC/config.plist` menggunakan editor plist (ProperTree atau OCAT).
   - Masuk ke bagian `PlatformInfo` -> `Generic`.
   - Pastikan model adalah `MacBookPro16,3`.
   - Buat serial baru untuk `SystemSerialNumber`, `MLB`, dan `SystemUUID` menggunakan tool `macserial`.
2. **Pengaturan BIOS**:
   - **Nonaktifkan (Disable)**: Secure Boot, Fast Boot.
   - **Aktifkan (Enable)**: AMD V(TM) / SVM, Mode Boot UEFI Only.
3. **Langkah Booting**:
   - Salin folder `EFI` ke partisi EFI flashdisk Anda.
   - Di menu OpenCore, jalankan opsi **Reset NVRAM** satu kali.
   - Pilih menu installer macOS untuk melanjutkan instalasi.

---

## References / Referensi

1. **Dortania OpenCore Install Guide**: [dortania.github.io/OpenCore-Install-Guide](https://dortania.github.io/OpenCore-Install-Guide/)
2. **AMD Vanilla Kernel Patches**: [github.com/AMD-OSX/AMD_Vanilla](https://github.com/AMD-OSX/AMD_Vanilla)
3. **NootedRed (AMD Vega Metal Acceleration)**: [github.com/ChefKissInc/NootedRed](https://github.com/ChefKissInc/NootedRed)
4. **Acidanthera OpenCorePkg & Drivers**: [github.com/acidanthera/OpenCorePkg](https://github.com/acidanthera/OpenCorePkg)
5. **Lenovo ThinkPad E14 Gen 2 Platform Specifications (PSREF)**: [psref.lenovo.com](https://psref.lenovo.com)

---

## License / Lisensi

Proyek ini didistribusikan di bawah lisensi [MIT License](LICENSE).
