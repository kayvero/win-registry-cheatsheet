# Windows Forensic Artifacts + EZ Tools — Cheat Sheet Lengkap

> Gabungan dari cheat sheet artefak & cheat sheet EZ Tools sebelumnya — satu tabel langsung nunjukin: **artefaknya apa, letaknya di mana, ini bagian dari hive apa (kalau ada), dan tool spesifik apa buat baca isinya.**
>
> Alur baca tabel: `Artefak → Lokasi/Hive Induk → Tool Parsing`

## 1. Registry Hives (file induk mentah)

| # | Hive | Path Windows | Penjelasan Singkat | Tool Parsing |
| -: | --- | --- | --- | --- |
| 1 | **SYSTEM** | `C:\Windows\System32\config\SYSTEM` | Konfigurasi sistem, service, driver — **induk** dari ShimCache & USB history | Registry Explorer / RECmd |
| 2 | **SOFTWARE** | `C:\Windows\System32\config\SOFTWARE` | Program terinstal, konfigurasi OS, network profile | Registry Explorer / RECmd |
| 3 | **NTUSER.DAT** | `C:\Users\<user>\NTUSER.DAT` | **Induk** dari UserAssist, RunMRU, RecentDocs, TypedPaths | Registry Explorer / RECmd |
| 4 | **SAM** | `C:\Windows\System32\config\SAM` | Akun user lokal & hash password | Registry Explorer / RECmd |
| 5 | **Amcache.hve** | `C:\Windows\AppCompat\Programs\Amcache.hve` | Bukti keberadaan executable (hash, path, compile time) | **AmcacheParser** |
| 6 | **UsrClass.dat** | `C:\Users\<user>\AppData\Local\Microsoft\Windows\UsrClass.dat` | **Induk** dari ShellBags | Registry Explorer / **SBECmd** |
| 7 | **SECURITY** | `C:\Windows\System32\config\SECURITY` | Kebijakan keamanan lokal & cached domain credential | Registry Explorer / RECmd |
| 8 | **DEFAULT** | `C:\Windows\System32\config\DEFAULT` | Profil registry default sebelum user login | Registry Explorer / RECmd |
| 9 | **RegBack** | `C:\Windows\System32\config\RegBack\` | Backup otomatis hive inti (mingguan) — buat bandingin versi lama vs baru | Registry Explorer / RECmd |
| 10 | **Transaction Logs** | Sebelahan hive, misal `SYSTEM.LOG1/.LOG2` | Perubahan key yang belum ter-commit ke hive utama | Registry Explorer (auto-load) |

## 2. Artefak di dalam Registry (sub-key spesifik)

| # | Artefak | Lokasi (Key Registry) | Hive Induk | Penjelasan Singkat | Tool Parsing |
| -: | --- | --- | --- | --- | --- |
| 1 | **ShimCache / AppCompatCache** | `ControlSet00X\Control\Session Manager\AppCompatCache` | SYSTEM | Path & waktu file executable yang pernah diakses | **AppCompatCacheParser** |
| 2 | **UserAssist** | `Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist` (key di-ROT13) | NTUSER.DAT | Program yang dijalankan via GUI Explorer + jumlah run | Registry Explorer (auto-decode ROT13) |
| 3 | **ShellBags** | `Shell\Bags` & `BagMRU` | UsrClass.dat (+ NTUSER.DAT) | Folder/drive/USB yang pernah dibuka | **ShellBags Explorer / SBECmd** |
| 4 | **RunMRU** | `Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU` | NTUSER.DAT | Command yang diketik di Run dialog (Win+R) | Registry Explorer |
| 5 | **RecentDocs** | `Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` | NTUSER.DAT | Dokumen terakhir dibuka lewat Explorer | Registry Explorer |
| 6 | **TypedPaths** | `Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths` | NTUSER.DAT | Path yang diketik manual di address bar | Registry Explorer |
| 7 | **Run / RunOnce** | `Software\Microsoft\Windows\CurrentVersion\Run(Once)` | SOFTWARE / NTUSER.DAT | Program auto-start saat boot/login (persistence) | Registry Explorer |
| 8 | **Services** | `ControlSet00X\Services` | SYSTEM | Daftar service & `ImagePath`-nya (persistence umum) | Registry Explorer |
| 9 | **USB History (USBSTOR)** | `ControlSet00X\Enum\USBSTOR` | SYSTEM | Riwayat device USB yang pernah dicolok | Registry Explorer |
| 10 | **Network Profiles** | `Software\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles` | SOFTWARE | Riwayat jaringan (SSID/nama network) yang pernah dikoneksi | Registry Explorer |

## 3. Artefak Filesystem & Non-Registry

| # | Artefak | Path Windows | Hive Induk | Penjelasan Singkat | Tool Parsing |
| -: | --- | --- | --- | --- | --- |
| 1 | **Prefetch (.pf)** | `C:\Windows\Prefetch\*.pf` | — (file terpisah) | Bukti eksekusi program: run count, waktu run pertama/terakhir | **PECmd** |
| 2 | **Windows Event Logs (.evtx)** | `C:\Windows\System32\winevt\Logs\*.evtx` | — (file terpisah) | Login/logout, service install, process creation, dll | **EvtxECmd** |
| 3 | **$MFT** | Metadata NTFS (root volume) | — (struktur filesystem) | Timeline file: create/modify/delete, termasuk file terhapus | **MFTECmd** |
| 4 | **$UsnJrnl / $J** | `$Extend\$UsnJrnl` (root volume) | — (struktur filesystem) | History perubahan file (rename/create/delete/overwrite) | **MFTECmd** |
| 5 | **$LogFile** | Metadata NTFS (root volume) | — (struktur filesystem) | Transaction log NTFS, rekonstruksi perubahan filesystem terbaru | **MFTECmd** |
| 6 | **Recycle Bin** | `C:\$Recycle.Bin\<SID>\` ($I & $R files) | — (file terpisah) | Nama asli file, path asal, & waktu dihapus | **RBCmd** |
| 7 | **LNK Files** | `C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\` | — (file terpisah) | Bukti file/target yang pernah dibuka, termasuk asal volume (USB) | **LECmd** |
| 8 | **Jump Lists** | `...\Recent\AutomaticDestinations\` & `CustomDestinations\` | — (file terpisah) | Riwayat file per-aplikasi via taskbar | **JLECmd** |
| 9 | **SRUM** | `C:\Windows\System32\sru\SRUDB.dat` | — (file terpisah) | Penggunaan network/resource per aplikasi, walau app sudah dihapus | **SrumECmd** |
| 10 | **Windows Timeline** | `...\ConnectedDevicesPlatform\...\ActivitiesCache.db` | — (file terpisah) | Riwayat aplikasi/dokumen yang dibuka (fitur Timeline Win10) | **WxTCmd** |
| 11 | **Browser History (SQLite)** | Path bervariasi per browser (`Chrome\...\History`, dll) | — (file terpisah) | URL, waktu akses, file yang didownload lewat browser | **SQLECmd** |
| 12 | **Volume Shadow Copies** | Tersembunyi, per-volume | — (snapshot filesystem) | Versi lama file/registry dari waktu snapshot dibuat | **VSCMount** (lalu parse isinya pakai tool sesuai jenis artefak) |
| 13 | **Raw strings di file apapun** | — (tergantung file yang dianalisis) | — | Cari string/URL/path tersembunyi di file besar | **bstrings** |

## 4. Tool Pendukung (Collector & Viewer)

| Tool | Fungsi | Kondisi Pemakaian |
| --- | --- | --- |
| **KAPE** | Collector — narik banyak artefak sekaligus dari disk image/live system | Dipakai **paling awal**, sebelum masuk ke tool parsing spesifik di atas |
| **Timeline Explorer** | Viewer — buka & filter output CSV dari semua tool EZ di atas | Dipakai **paling akhir**, buat korelasi semua hasil parsing jadi satu timeline |

## Alur pemakaian keseluruhan

```
KAPE (kalau sumbernya disk image/live system)
      ↓
Tarik file: hive registry + Prefetch + EventLogs + LNK + $MFT + dst
      ↓
Parsing per-jenis artefak pakai tool spesifik (tabel 1–3 di atas)
      ↓
Semua output CSV dibuka bareng di Timeline Explorer
      ↓
Sort by timestamp → korelasi antar artefak → rekonstruksi kejadian
```

> **Catatan:** kalau di CTF kamu langsung dikasih file artefak mentah (bukan disk image), tinggal skip KAPE dan langsung ke tool parsing yang sesuai jenis file-nya.
