# Windows Registry Hives — Cheat Sheet

> Diurutkan dari yang **paling sering muncul/dipakai** di investigasi forensik (termasuk CTF) ke yang paling jarang.
> Format: **Hive/File | Path Windows | Penjelasan | Tools**

| # | Hive / File | Path Windows | Penjelasan | Tools |
| -: | --- | --- | --- | --- |
| 1 | **SYSTEM** | `C:\Windows\System32\config\SYSTEM` | Konfigurasi sistem: service, driver, ShimCache/AppCompatCache, USB history, timezone, computer name | Registry Explorer, RegRipper |
| 2 | **SOFTWARE** | `C:\Windows\System32\config\SOFTWARE` | Program terinstal, konfigurasi OS, network profile, App Paths | Registry Explorer, RegRipper |
| 3 | **NTUSER.DAT** | `C:\Users\<user>\NTUSER.DAT` | Aktivitas per-user: UserAssist, RunMRU, RecentDocs, TypedPaths, Run keys per-user | Registry Explorer, RegRipper |
| 4 | **SAM** | `C:\Windows\System32\config\SAM` | Akun user lokal & hash password | Registry Explorer, RegRipper |
| 5 | **Amcache.hve** | `C:\Windows\AppCompat\Programs\Amcache.hve` | Bukti keberadaan executable: hash SHA1, path, waktu compile/install | Registry Explorer, RegRipper |
| 6 | **UsrClass.dat** | `C:\Users\<user>\AppData\Local\Microsoft\Windows\UsrClass.dat` | ShellBags — folder/drive yang pernah dibuka user | Registry Explorer, RegRipper |
| 7 | **SECURITY** | `C:\Windows\System32\config\SECURITY` | Kebijakan keamanan lokal & cached domain credential (LSA secrets) | Registry Explorer, RegRipper |
| 8 | **DEFAULT** | `C:\Windows\System32\config\DEFAULT` | Profil registry default sebelum ada user yang login | Registry Explorer, RegRipper |
| 9 | **RegBack (backup hives)** | `C:\Windows\System32\config\RegBack\` | Salinan backup otomatis SAM/SYSTEM/SOFTWARE/SECURITY/DEFAULT (biasanya mingguan) — berguna kalau hive utama corrupt/dihapus atau mau bandingin versi lama | Registry Explorer, RegRipper |
| 10 | **Transaction Logs (.LOG1/.LOG2)** | Sebelahan tiap hive, misal `SYSTEM.LOG1`, `SYSTEM.LOG2` | Log transaksi registry — bisa menyimpan perubahan key yang belum ter-commit ke hive utama | Registry Explorer (auto-load kalau file `.LOG` ada di folder sama) |
| 11 | **COMPONENTS** | `C:\Windows\System32\config\COMPONENTS` | Info servicing/WinSxS — komponen & update OS terinstal | Registry Explorer |
| 12 | **BCD** | `\Boot\BCD` (partisi boot terpisah) | Boot Configuration Data — konfigurasi boot loader & entry OS | Registry Explorer (load sebagai hive) |
| 13 | **ELAM** | Bagian dari SYSTEM hive: `ControlSet00X\Control\EarlyLaunch` | Konfigurasi Early Launch Antimalware driver saat boot | Registry Explorer |
| 14 | **Syscache.hve** *(khusus Windows 7, jarang di sistem modern)* | `C:\System Volume Information\Syscache.hve` | Versi lama Amcache — bukti eksekusi file di Windows 7 | Registry Explorer |

## Catatan

- **Hive inti (baris 1–8)** ini yang paling sering kamu temui di soal CTF forensik Windows registry — cukup kuasai ini dulu.
- **RegBack & transaction logs (baris 9–10)** baru relevan kalau soal minta bandingin kondisi "sebelum vs sesudah", atau hive utamanya sengaja dikosongkan/dihapus attacker.
- **COMPONENTS, BCD, ELAM, Syscache (baris 11–14)** jarang jadi fokus soal, tapi kadang dipakai sebagai artefak pendukung/hint tambahan.
- Semua path di atas untuk hive yang **tersimpan di disk (offline)**. Kalau ambil dari disk image utuh, tarik dulu filenya pakai **KAPE** (target `RegistryHives`) atau `icat`/`tsk_recover` (TSK), baru dibuka di Registry Explorer/RegRipper.
