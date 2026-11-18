<div align="center">

# PanSocks Desktop

**SOCKS5 Tunnel Client** - Windows - Linux - macOS

Route semua trafik (TCP, UDP, DNS) melalui server SOCKS5 kamu.
Anti DNS leak, satu GUI Gio untuk tiga desktop.

[![Windows](https://img.shields.io/badge/Windows-Installer-0078d4?style=for-the-badge&logo=windows&logoColor=white)](../../releases/latest)
[![Linux](https://img.shields.io/badge/Linux-.deb-fcc624?style=for-the-badge&logo=linux&logoColor=black)](../../releases/latest)
[![macOS](https://img.shields.io/badge/macOS-Universal-000000?style=for-the-badge&logo=apple&logoColor=white)](../../releases/latest)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

</div>

---

Fork dari [jhopan/SocksClient](https://github.com/jhopan/SocksClient) - hanya bagian desktop
(Go + Gio + sing-box core). Versi aplikasi: lihat `appVersion` di
`cmd/socksgui-gio/main.go`.

## Fitur

| Fitur | Detail |
|---|---|
| **Anti DNS Leak** | Semua query DNS di-tunnel melalui SOCKS (`hijack-dns` + WFP blocking di Windows) |
| **TUN gvisor** | Userspace network stack - tidak bergantung driver/kernel perangkat |
| **MTU 1400** | Dioptimalkan untuk hotspot mobile (mencegah PMTU blackhole) |
| **No console popup** | Core sing-box di-spawn dengan `CREATE_NO_WINDOW` (Windows) |
| **HTTP ping** | Cek koneksi tiap 30 detik via `generate_204` (opsional, checkbox) |
| **Preflight check** | Validasi IP -> TCP connect -> SOCKS5 auth sebelum tunnel naik |
| **Cross-platform GUI** | Satu kode Gio untuk Windows, Linux, macOS |
| **Password terenkripsi** | Windows DPAPI - Linux file 0600 |

## Download

Aset dibuat otomatis oleh `.github/workflows/build-desktop-release.yml` saat tag
`desktop-v*` di-push.

| Platform | File |
|---|---|
| **Windows** | `SocksClientDesktop_Setup_v*.exe` (installer + core) |
| **Linux x86_64** | `socksclient_*_amd64.deb`, `SocksClient-linux-amd64-*.tar.gz` |
| **Linux arm64** | `socksclient_*_arm64.deb`, `SocksClient-linux-arm64-*.tar.gz` |
| **macOS** | `SocksClient-macos-universal-*.tar.gz` (Intel + Apple Silicon) |

## Quick Start

### Windows
1. Download installer `.exe` -> install
2. Buka aplikasi (UAC muncul - butuh admin untuk TUN)
3. Isi **Host** dan **Port** -> klik **Connect**

### Linux
```bash
sudo dpkg -i socksclient_*_amd64.deb
sudo socksgui                          # GUI (Gio)
# atau headless:
sudo socksctl up -host 10.0.0.1 -port 1080
```

### macOS
```bash
tar -xzf SocksClient-macos-universal-*.tar.gz
cd SocksClient && ./install.sh
sudo socksctl up -host 10.0.0.1 -port 1080
```

> **Host wajib IP literal** (contoh `10.0.0.1`), bukan hostname.
> Hostname akan bocor DNS di luar tunnel.

## Arsitektur

```
Device -> TUN (gvisor, MTU 1400) -> sing-box -> SOCKS5 -> Server -> Internet
              |
              +-- DNS hijack: semua query dijawab lewat tunnel
              +-- Anti-loop: trafik ke server sendiri bypass tunnel
              +-- strict_route: tidak ada app yang bisa bypass
```

## Struktur repo

```
├── cmd/
│   ├── socksgui-gio/   # GUI cross-platform (Gio, cgo)
│   ├── socksctl/       # CLI headless + UI browser
│   └── dumpconfig/     # cetak config sing-box (dipakai CI)
├── internal/           # engine, settings, ping, boxcfg, winutil
├── embed/              # sing-box.exe (dari release `core`, bukan git)
├── scripts/            # fetch-core, package-linux, package-macos, tun-selftest
└── .github/workflows/  # CI desktop + build & release
```

## Build dari source

Butuh Go 1.25+ dan GCC (cgo untuk Gio).

```bash
bash scripts/fetch-core.sh              # unduh sing-box core ke embed/
go test -count=1 ./internal/...         # unit test
go build -trimpath -ldflags="-s -w -H windowsgui" \
  -o SocksClient.exe ./cmd/socksgui-gio # GUI Windows
go build -trimpath -o socksctl ./cmd/socksctl
```

Installer Windows: `ISCC.exe setup.iss` (Inno Setup) setelah `SocksClient.exe` ada.

Release manual (tanpa CI):

```bash
bash scripts/package-linux.sh   # -> dist/*.deb + tarball
bash scripts/package-macos.sh    # -> dist/SocksClient-macos-universal-*.tar.gz
```

> Core sing-box TIDAK di-track di git; selalu diambil dari release `core` di
> `jhopan/SocksClient` (dipakai bersama dengan build Android upstream).

## Spec

| | Windows | Linux | macOS |
|---|---|---|---|
| GUI | Gio (cgo) | Gio (cgo) | Gio (cgo) |
| TUN | wintun | gvisor | gvisor |
| Binary GUI | ~12,4 MB | ~12 MB | ~12 MB |
| CLI `socksctl` | ~6,6 MB | ~6,6 MB | ~6,6 MB |
| Core sing-box | 30 MB (v1.14.1-minimal, `with_gvisor`) | sama | sama |

## QA

```bash
go test -count=1 ./internal/...
bash scripts/tun-selftest.sh    # butuh admin
```

DNS leak test manual: buka <https://dnsleaktest.com> -> Extended test, semua
resolver harus dari SOCKS server.

## Versi

| Perubahan | Versi |
|---|---|
| Fitur baru | naik minor (`1.7.0` -> `1.8.0`) |
| Bug fix / rebuild | tambah segmen 4 (`1.7.0` -> `1.7.0.1`) |

Tag rilis desktop: `desktop-v<versi>`.

## License

MIT - lihat [LICENSE](LICENSE). Kode asli oleh **JhopanStore**
(<https://github.com/jhopan/SocksClient>).