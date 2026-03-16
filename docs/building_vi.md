# Phân tích dự án và Hướng dẫn Build ungoogled-chromium

## Mục lục

* [Tổng quan dự án](#tổng-quan-dự-án)
* [Kiến trúc và cấu trúc thư mục](#kiến-trúc-và-cấu-trúc-thư-mục)
* [Các thành phần chính](#các-thành-phần-chính)
* [Yêu cầu hệ thống](#yêu-cầu-hệ-thống)
* [Hướng dẫn Build từng bước](#hướng-dẫn-build-từng-bước)
* [Build trên các nền tảng cụ thể](#build-trên-các-nền-tảng-cụ-thể)
* [Giải thích các GN flags](#giải-thích-các-gn-flags)
* [Hệ thống Patch](#hệ-thống-patch)
* [Xử lý sự cố](#xử-lý-sự-cố)

---

## Tổng quan dự án

**ungoogled-chromium** là một phiên bản Chromium được loại bỏ hoàn toàn sự phụ thuộc vào các dịch vụ web của Google. Dự án này hoạt động như một giải pháp thay thế trực tiếp cho Google Chromium.

### Mục tiêu chính (theo thứ tự ưu tiên)

1. **Loại bỏ phụ thuộc Google**: Xóa tất cả các yêu cầu nền (background requests) tới dịch vụ Google
2. **Giữ nguyên trải nghiệm Chromium**: Không thay đổi giao diện hay tính năng cốt lõi
3. **Tăng cường quyền riêng tư**: Thêm các tùy chỉnh cho quyền riêng tư, kiểm soát và minh bạch

### Các tính năng chính

- Vô hiệu hóa Google Safe Browsing, Google Cloud Messaging (GCM), GAIA
- Chặn các yêu cầu nội bộ tới Google khi chạy (thay thế domain bằng `.qjz9zk`)
- Loại bỏ các file nhị phân (binary) đã được biên dịch sẵn từ mã nguồn
- Thêm hơn 50 cờ dòng lệnh (command-line flags) và mục `chrome://flags`
- Hỗ trợ Manifest V2 cho extensions

### Phiên bản hiện tại

- **Chromium**: 146.0.7680.80
- **Revision**: 1

---

## Kiến trúc và cấu trúc thư mục

```
ungoogled-chromium/
├── chromium_version.txt          # Phiên bản Chromium (146.0.7680.80)
├── revision.txt                  # Số revision của ungoogled-chromium
├── flags.gn                      # Cờ cấu hình build GN (18 flags)
├── downloads.ini                 # Cấu hình tải mã nguồn Chromium
├── pruning.list                  # Danh sách file nhị phân cần xóa (~12.444 mục)
├── domain_regex.list             # Biểu thức regex thay thế domain (21 mẫu)
├── domain_substitution.list      # Danh sách file cần thay thế domain (~17.714 mục)
├── patches/                      # 110 patch files
│   ├── series                    # Thứ tự áp dụng patch (112 dòng)
│   ├── core/                     # Patch cốt lõi (bắt buộc)
│   │   ├── bromite/
│   │   ├── inox-patchset/
│   │   ├── iridium-browser/
│   │   └── ungoogled-chromium/   # 26 patch chính
│   ├── extra/                    # Patch mở rộng (tùy chọn)
│   │   ├── bromite/
│   │   ├── debian/
│   │   ├── inox-patchset/
│   │   ├── iridium-browser/
│   │   └── ungoogled-chromium/   # Hơn 60 patch bổ sung
│   └── upstream-fixes/           # Sửa lỗi từ upstream
├── utils/                        # Scripts tiện ích chính
│   ├── downloads.py              # Tải và giải nén mã nguồn
│   ├── patches.py                # Áp dụng patch
│   ├── prune_binaries.py         # Xóa file nhị phân
│   ├── domain_substitution.py    # Thay thế domain
│   ├── clone.py                  # Clone mã nguồn từ Git
│   └── filescfg.py               # Cấu hình đóng gói
├── devutils/                     # Công cụ phát triển
│   ├── validate_config.py        # Kiểm tra cấu hình
│   ├── validate_patches.py       # Kiểm tra patch
│   ├── update_lists.py           # Cập nhật danh sách
│   └── check_*.py                # Các script kiểm tra khác
└── docs/                         # Tài liệu
    ├── building.md               # Hướng dẫn build (tiếng Anh)
    ├── design.md                 # Thiết kế kỹ thuật
    ├── flags.md                  # Danh sách cờ đầy đủ
    └── platforms.md              # Nền tảng hỗ trợ
```

---

## Các thành phần chính

### 1. Source File Processors (Bộ xử lý mã nguồn)

Dự án sử dụng 3 bộ xử lý trước khi build:

| Bộ xử lý | Mô tả | File cấu hình |
|-----------|--------|----------------|
| **Binary Pruning** | Xóa các file nhị phân đã build sẵn khỏi mã nguồn | `pruning.list` |
| **Domain Substitution** | Thay thế domain Google bằng domain giả `.qjz9zk` | `domain_regex.list`, `domain_substitution.list` |
| **Patch Application** | Áp dụng 110 patch để sửa đổi chức năng | `patches/series` |

### 2. Hệ thống Build

- **Build system**: GN (Generate Ninja) + Ninja
- **Ngôn ngữ**: C++ (Chromium), Python 3 (scripts tiện ích)
- **CI/CD**: Cirrus CI + GitHub Actions

### 3. Công cụ phát triển (`devutils/`)

| Script | Chức năng |
|--------|-----------|
| `validate_config.py` | Kiểm tra tính nhất quán của cấu hình |
| `validate_patches.py` | Kiểm tra syntax và khả năng áp dụng của patch |
| `check_gn_flags.py` | Đảm bảo GN flags được sắp xếp và không trùng lặp |
| `update_lists.py` | Tự động cập nhật `pruning.list` và `domain_substitution.list` |
| `run_utils_pylint.py` | Lint code trong `utils/` |

---

## Yêu cầu hệ thống

### Phần cứng tối thiểu

| Thành phần | Tối thiểu | Khuyến nghị |
|------------|-----------|-------------|
| RAM | 8 GB | 16 GB trở lên |
| Dung lượng ổ cứng | 50 GB | 100 GB |
| CPU | 4 cores | 8 cores trở lên |

### Phần mềm cần thiết

- **Python**: 3.10 trở lên
- **Git**: phiên bản mới nhất
- **Ninja**: build system
- **Công cụ biên dịch**: GCC/Clang (Linux), Xcode (macOS), Visual Studio (Windows)
- **Các gói Python**: `httplib2`, `six`, `requests` (cho development)

### Cài đặt trên Ubuntu/Debian

```bash
sudo apt update
sudo apt install -y python3 python3-pip git ninja-build clang lld \
    build-essential libglib2.0-dev libgtk-3-dev libdrm-dev \
    libnss3-dev libxss-dev libpci-dev libcups2-dev libasound2-dev \
    libpulse-dev libxtst-dev libxkbcommon-dev
```

### Cài đặt trên Fedora/CentOS

```bash
sudo dnf install -y python3 python3-pip git ninja-build clang lld \
    gcc-c++ glib2-devel gtk3-devel libdrm-devel nss-devel \
    libXScrnSaver-devel pciutils-devel cups-devel alsa-lib-devel \
    pulseaudio-libs-devel libXtst-devel libxkbcommon-devel
```

---

## Hướng dẫn Build từng bước

### Quy trình tổng quan

```
Tải mã nguồn → Xóa binary → Áp dụng patch → Thay thế domain → Build GN → Build Chromium → Đóng gói
```

### Bước 1: Clone repository ungoogled-chromium

```bash
git clone https://github.com/ungoogled-software/ungoogled-chromium.git
cd ungoogled-chromium
```

### Bước 2: Tải mã nguồn Chromium

Tải và giải nén mã nguồn Chromium chính thức:

```bash
# Tạo thư mục cache và build
mkdir -p build/download_cache

# Tải mã nguồn (khoảng 1.5 GB)
./utils/downloads.py retrieve -c build/download_cache -i downloads.ini

# Giải nén vào thư mục build/src
./utils/downloads.py unpack -c build/download_cache -i downloads.ini -- build/src
```

> **Ghi chú**: Bước này sẽ tải file `chromium-146.0.7680.80-lite.tar.xz` từ server của Google và giải nén.

### Bước 3: Xóa file nhị phân (Binary Pruning)

Loại bỏ khoảng 12.444 file nhị phân đã biên dịch sẵn:

```bash
./utils/prune_binaries.py build/src pruning.list
```

> **Mục đích**: Đảm bảo tất cả mã được biên dịch từ source, không sử dụng binary đóng gói sẵn.

### Bước 4: Áp dụng Patch

Áp dụng 110 patch theo thứ tự được định nghĩa trong `patches/series`:

```bash
./utils/patches.py apply build/src patches
```

> **Ghi chú**: Các patch này sẽ:
> - Vô hiệu hóa các dịch vụ Google (crash reporter, GCM, GAIA, v.v.)
> - Chặn các URL theo dõi
> - Thêm các cờ tùy chỉnh mới
> - Sửa lỗi build khi không có Safe Browsing

### Bước 5: Thay thế Domain (Domain Substitution)

Thay thế các domain Google bằng domain giả `.qjz9zk`:

```bash
./utils/domain_substitution.py apply \
    -r domain_regex.list \
    -f domain_substitution.list \
    -c build/domsubcache.tar.gz \
    build/src
```

> **Cơ chế**: 21 biểu thức regex được áp dụng lên ~17.714 file. Cache được lưu tại `build/domsubcache.tar.gz` để có thể hoàn tác nếu cần.

### Bước 6: Build GN (nếu cần)

Nếu bạn không sử dụng `depot_tools` hoặc chưa có binary GN:

```bash
mkdir -p build/src/out/Default
cd build/src
./tools/gn/bootstrap/bootstrap.py --skip-generate-buildfiles -j4 -o out/Default/
```

> **Ghi chú**: Nếu đã có GN binary (ví dụ thông qua `depot_tools`), bỏ qua bước này.

### Bước 7: Cấu hình và Build Chromium

```bash
# Tạo thư mục output
mkdir -p build/src/out/Default

# Sao chép cấu hình GN flags
cp flags.gn build/src/out/Default/args.gn

# Vào thư mục source
cd build/src

# (Tùy chọn) Thêm GN flags bổ sung vào args.gn tại đây
# Ví dụ: echo 'is_debug=false' >> out/Default/args.gn

# Tạo build files
./out/Default/gn gen out/Default --fail-on-unused-args

# Build Chromium (có thể mất vài giờ)
ninja -C out/Default chrome chromedriver chrome_sandbox
```

> **Thời gian build**: Tùy thuộc vào cấu hình máy, quá trình build có thể mất từ 2 đến 8 giờ.

### Bước 8 (Tùy chọn): Đóng gói

Sau khi build thành công, kết quả nằm trong `build/src/out/Default/`. Cách đóng gói phụ thuộc vào nền tảng cụ thể — tham khảo các repo nền tảng tương ứng.

---

## Build trên các nền tảng cụ thể

Mỗi nền tảng có một repository riêng chứa cấu hình và scripts bổ sung:

| Nền tảng | Repository | Ghi chú |
|----------|-----------|---------|
| **Arch Linux** | [ungoogled-chromium-archlinux](https://github.com/ungoogled-software/ungoogled-chromium-archlinux) | Có sẵn trong AUR |
| **Debian/Ubuntu** | [ungoogled-chromium-debian](https://github.com/ungoogled-software/ungoogled-chromium-debian) | Có sẵn trong OBS |
| **Fedora/CentOS** | [ungoogled-chromium-fedora](https://github.com/ungoogled-software/ungoogled-chromium-fedora) | Có sẵn trong COPR |
| **Portable Linux** | [ungoogled-chromium-portablelinux](https://github.com/ungoogled-software/ungoogled-chromium-portablelinux) | Cho mọi distro Linux |
| **Windows** | [ungoogled-chromium-windows](https://github.com/ungoogled-software/ungoogled-chromium-windows) | Cần Visual Studio |
| **macOS** | [ungoogled-chromium-macos](https://github.com/ungoogled-software/ungoogled-chromium-macos) | Cần Xcode |
| **Android** | [ungoogled-chromium-android](https://github.com/ungoogled-software/ungoogled-chromium-android) | Cần Android NDK |

### Cài đặt nhanh (không cần build)

Nếu bạn không muốn tự build, có thể cài đặt từ các nguồn sau:

```bash
# macOS (qua Homebrew)
brew install --cask ungoogled-chromium

# Flatpak
flatpak install flathub io.github.ungoogled_software.ungoogled_chromium

# NixOS
nix-env -iA nixpkgs.ungoogled-chromium

# Arch Linux (qua AUR)
yay -S ungoogled-chromium
```

---

## Giải thích các GN flags

File `flags.gn` chứa các cờ cấu hình build. Dưới đây là giải thích từng flag:

| Flag | Giá trị | Mô tả |
|------|---------|--------|
| `build_with_tflite_lib` | `false` | Vô hiệu hóa TensorFlow Lite |
| `chrome_pgo_phase` | `0` | Tắt Profile-Guided Optimization |
| `clang_use_chrome_plugins` | `false` | Không dùng Clang plugins của Chrome |
| `disable_fieldtrial_testing_config` | `true` | Tắt cấu hình thử nghiệm field trial |
| `enable_hangout_services_extension` | `false` | Tắt extension Hangouts |
| `enable_mdns` | `false` | Tắt mDNS (multicast DNS) |
| `enable_remoting` | `false` | Tắt Chrome Remote Desktop |
| `enable_reporting` | `false` | Tắt báo cáo crash/usage |
| `enable_service_discovery` | `false` | Tắt service discovery |
| `enable_widevine` | `true` | Giữ Widevine (cần cho DRM/Netflix) |
| `exclude_unwind_tables` | `true` | Giảm kích thước binary |
| `google_api_key` | `""` | Xóa Google API key |
| `google_default_client_id` | `""` | Xóa OAuth client ID |
| `google_default_client_secret` | `""` | Xóa OAuth secret |
| `safe_browsing_mode` | `0` | Tắt hoàn toàn Safe Browsing |
| `treat_warnings_as_errors` | `false` | Không dừng build khi có warning |
| `use_official_google_api_keys` | `false` | Không dùng Google API keys chính thức |
| `use_unofficial_version_number` | `false` | Dùng số phiên bản tùy chỉnh |

### Thêm GN flags bổ sung

Bạn có thể thêm flags vào `args.gn` trước khi build:

```bash
# Ví dụ: Build release tối ưu
cat >> build/src/out/Default/args.gn << 'EOF'
is_debug=false
is_official_build=true
symbol_level=0
EOF
```

---

## Hệ thống Patch

### Phân loại Patch

Tổng cộng có **110 patch** được chia thành 3 nhóm:

#### 1. Core Patches (Bắt buộc)
Các patch cốt lõi để loại bỏ phụ thuộc Google:

- `disable-crash-reporter.patch` — Tắt báo cáo crash
- `disable-google-host-detection.patch` — Tắt phát hiện host Google
- `disable-gaia.patch` — Tắt Google Account (GAIA)
- `disable-gcm.patch` — Tắt Google Cloud Messaging
- `block-trk-and-subdomains.patch` — Chặn domain `trk:` và `qjz9zk`
- `block-requests.patch` — Chặn yêu cầu tới domain đã thay thế
- `disable-privacy-sandbox.patch` — Tắt Privacy Sandbox
- `extensions-manifestv2.patch` — Hỗ trợ Manifest V2

#### 2. Extra Patches (Tùy chọn)
Các patch tăng cường quyền riêng tư và kiểm soát:

- `add-flag-*` — Thêm nhiều cờ tùy chỉnh mới
- `fingerprinting-flags-*` — Cờ chống fingerprinting
- `disable-formatting-in-omnibox.patch` — Tắt format URL tự động
- `default-webrtc-ip-handling-policy.patch` — Chính sách WebRTC IP
- `add-flag-to-spoof-webgl-renderer-info.patch` — Giả mạo thông tin WebGL

#### 3. Upstream Fixes
Sửa lỗi từ Chromium upstream chưa được merge.

### Nguồn gốc Patch

Patch được vay mượn từ các dự án:
- **Inox patchset** — Tập patch quyền riêng tư
- **Bromite** — Trình duyệt Android chống fingerprinting
- **Debian** — Patch từ Chromium Debian
- **Iridium Browser** — Trình duyệt quyền riêng tư

---

## Xử lý sự cố

### 1. Hết RAM khi build

```bash
# Giảm số thread song song (ví dụ: chỉ dùng 2 thread)
ninja -j2 -C out/Default chrome

# Hoặc thêm swap space
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### 2. Patch không áp dụng được

```bash
# Kiểm tra patch nào bị lỗi
./utils/patches.py apply build/src patches 2>&1 | grep -i "fail\|error"

# Áp dụng patch thủ công để debug
cd build/src
patch -p1 --dry-run < ../../patches/core/ungoogled-chromium/disable-crash-reporter.patch
```

### 3. Lỗi download mã nguồn

```bash
# Xóa cache và tải lại
rm -rf build/download_cache
mkdir -p build/download_cache
./utils/downloads.py retrieve -c build/download_cache -i downloads.ini
```

### 4. Kiểm tra cấu hình trước khi build

```bash
# Chạy validation
python3 devutils/validate_config.py

# Kiểm tra GN flags
python3 devutils/check_gn_flags.py flags.gn

# Kiểm tra patch files
python3 devutils/check_patch_files.py patches
```

### 5. Lỗi thiếu dependencies khi build

Đảm bảo đã cài đầy đủ các dependencies. Trên Ubuntu/Debian, chạy script cài đặt của Chromium:

```bash
cd build/src
./build/install-build-deps.sh
```

---

## Tài liệu tham khảo

- [Hướng dẫn build (tiếng Anh)](building.md)
- [Tài liệu thiết kế](design.md)
- [Danh sách flags đầy đủ](flags.md)
- [Nền tảng hỗ trợ](platforms.md)
- [Hướng dẫn đóng góp](contributing.md)
- [FAQ chính thức](https://ungoogled-software.github.io/ungoogled-chromium-wiki/faq)
