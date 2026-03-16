# ungoogled-chromium

*Giải pháp nhẹ nhàng để loại bỏ sự phụ thuộc vào dịch vụ web của Google*

## Dự án này dùng để làm gì?

**ungoogled-chromium** là một phiên bản sửa đổi của trình duyệt Google Chromium, được tạo ra với mục đích **loại bỏ hoàn toàn sự phụ thuộc vào các dịch vụ web của Google** trong khi vẫn giữ nguyên trải nghiệm duyệt web của Chromium.

Nói đơn giản: đây là Chromium nhưng **không có Google**.

### Tại sao cần dự án này?

Khi sử dụng Google Chromium (ngay cả khi không đăng nhập tài khoản Google), trình duyệt vẫn âm thầm:

- Gửi yêu cầu nền (background requests) tới các máy chủ Google
- Sử dụng các file nhị phân (binary) được Google đóng gói sẵn
- Thu thập dữ liệu sử dụng qua các dịch vụ như Safe Browsing, Cloud Messaging, crash reporter
- Hạn chế quyền kiểm soát và tùy chỉnh của người dùng

**ungoogled-chromium giải quyết tất cả các vấn đề này.**

---

## Mục tiêu chính

Theo thứ tự ưu tiên từ cao đến thấp:

1. **Loại bỏ phụ thuộc Google** — Xóa tất cả kết nối tới dịch vụ web của Google
2. **Giữ nguyên trải nghiệm Chromium** — Không thay đổi giao diện hay tính năng cốt lõi, hoạt động như một bản thay thế trực tiếp cho Chromium
3. **Tăng cường quyền riêng tư** — Thêm các tùy chỉnh cho quyền riêng tư, kiểm soát và minh bạch (hầu hết cần kích hoạt thủ công)

---

## Dự án này làm những gì cụ thể?

### 🔒 Tính năng cốt lõi (Tự động)

| Tính năng | Mô tả |
|-----------|--------|
| **Vô hiệu hóa dịch vụ Google** | Tắt Google Host Detector, URL Tracker, Cloud Messaging, Hotwording, Safe Browsing, GAIA (tài khoản Google), crash reporter |
| **Chặn yêu cầu nền tới Google** | Thay thế domain Google trong mã nguồn bằng domain giả `.qjz9zk`, sau đó chặn mọi kết nối tới domain này — không có yêu cầu nào được gửi đi |
| **Xóa file nhị phân đóng gói sẵn** | Loại bỏ ~12.444 file nhị phân (executables, libraries) khỏi mã nguồn, đảm bảo mọi thứ được biên dịch từ source code |
| **Xóa API keys của Google** | Loại bỏ Google API key, OAuth client ID và secret |
| **Tắt telemetry và báo cáo** | Không gửi dữ liệu crash, usage, hoặc bất kỳ thông tin nào tới Google |
| **Tắt field trial testing** | Ngăn Google điều khiển tính năng từ xa qua A/B testing |
| **Hỗ trợ Manifest V2** | Giữ hỗ trợ extension Manifest V2 (Google đang loại bỏ trong Chrome) |

### 🛡️ Tính năng tăng cường quyền riêng tư (Kích hoạt thủ công)

Các tính năng này được cung cấp qua cờ dòng lệnh hoặc `chrome://flags`:

| Tính năng | Cờ (flag) | Mô tả |
|-----------|-----------|--------|
| **Chống fingerprinting Canvas** | `--fingerprinting-canvas-image-data-noise` | Thêm nhiễu nhỏ vào dữ liệu Canvas để ngăn website nhận dạng |
| **Chống fingerprinting WebGL** | `--enable-features=SpoofWebGLInfo` | Trả về thông tin WebGL chung chung thay vì thông tin thật |
| **Giảm thông tin hệ thống** | `--enable-features=ReducedSystemInfo` | Giảm lượng thông tin hệ thống mà website có thể thu thập |
| **Xóa Client Hints** | `--enable-features=RemoveClientHints` | Loại bỏ thông tin client hints gửi tới server |
| **Tùy chỉnh referrer** | `--enable-features=NoReferrers` | Xóa referrer để website không biết bạn đến từ đâu |
| **Kiểm soát WebRTC IP** | `--webrtc-ip-handling-policy` | Ngăn rò rỉ địa chỉ IP qua WebRTC |
| **Tắt TLS GREASE** | `--disable-grease-tls` | Kết hợp với `--http-accept-header` để trình duyệt giống Tor Browser hơn |
| **Trang chủ tùy chỉnh** | `--custom-ntp` | Đặt trang chủ tùy chỉnh (VD: `about:blank`, URL bất kỳ) |
| **Xóa dữ liệu khi thoát** | `--enable-features=ClearDataOnExit` | Tự động xóa dữ liệu duyệt web khi đóng trình duyệt |
| **Popups thành tab** | `--popups-to-tabs` | Mở popup trong tab mới thay vì cửa sổ riêng |

> Xem [danh sách đầy đủ hơn 50 cờ tùy chỉnh](docs/flags.md) để biết thêm chi tiết.

### 🎨 Tính năng giao diện (Tùy chọn)

| Tính năng | Cờ (flag) | Mô tả |
|-----------|-----------|--------|
| Ẩn nút avatar | `--show-avatar-button=never` | Ẩn nút hình đại diện trên thanh công cụ |
| Ẩn nút đóng tab | `--hide-tab-close-buttons` | Ẩn nút X trên mỗi tab |
| Ẩn menu extensions | `--hide-extensions-menu` | Ẩn biểu tượng puzzle piece |
| Ẩn giao diện fullscreen | `--hide-fullscreen-exit-ui` | Ẩn nút X khi xem fullscreen |
| Tab cuộn | `--scroll-tabs=always` | Cuộn giữa các tab bằng chuột |
| Tab hover cards | `--tab-hover-cards=none` | Tắt hoặc thay đổi tooltip khi hover tab |
| Đóng cửa sổ với tab cuối | `--close-window-with-last-tab=never` | Giữ cửa sổ mở khi đóng tab cuối |

---

## Cách hoạt động (Kỹ thuật)

Repository này chứa **cấu hình và công cụ** để biến đổi mã nguồn Chromium. Quá trình gồm 3 bước chính:

### 1. Binary Pruning (Xóa file nhị phân)
```
pruning.list → utils/prune_binaries.py → Xóa ~12.444 file binary
```
Loại bỏ tất cả file thực thi, thư viện, WASM đã biên dịch sẵn. Đảm bảo mọi thứ được build từ source code.

### 2. Patch Application (Áp dụng bản vá)
```
patches/series → utils/patches.py → Áp dụng 110 patches
```
110 bản vá được chia thành:
- **Core** (35 patches): Bắt buộc — loại bỏ dịch vụ Google, chặn kết nối
- **Extra** (74 patches): Tùy chọn — thêm tính năng quyền riêng tư, giao diện
- **Upstream fixes** (1 patch): Sửa lỗi từ Chromium upstream

### 3. Domain Substitution (Thay thế domain)
```
domain_regex.list → utils/domain_substitution.py → Thay thế domain trong ~17.714 files
```
Thay thế domain Google (google.com, youtube.com, v.v.) bằng domain giả `.qjz9zk`. Mọi kết nối tới domain `.qjz9zk` đều bị chặn — đây là lớp bảo vệ bổ sung phòng khi patch bỏ sót.

---

## Cài đặt nhanh

### Từ package manager (không cần build)

```bash
# macOS
brew install --cask ungoogled-chromium

# Arch Linux (AUR)
yay -S ungoogled-chromium

# Flatpak (mọi Linux distro)
flatpak install flathub io.github.ungoogled_software.ungoogled_chromium

# NixOS
nix-env -iA nixpkgs.ungoogled-chromium
```

### Từ repository phần mềm

| Hệ điều hành | Nguồn |
|---------------|-------|
| Arch Linux | [AUR](https://github.com/ungoogled-software/ungoogled-chromium-archlinux) |
| Debian/Ubuntu | [OBS](https://github.com/ungoogled-software/ungoogled-chromium-debian), [XtraDeb PPA](https://xtradeb.net/apps/ungoogled-chromium/) |
| Fedora | [COPR](https://copr.fedorainfracloud.org/coprs/wojnilowicz/ungoogled-chromium/) |
| Gentoo | [`::pf4public` overlay](https://github.com/PF4Public/gentoo-overlay) |
| openSUSE | [OBS](https://software.opensuse.org//download.html?project=network%3Achromium&package=ungoogled-chromium) |
| macOS | [Homebrew](https://formulae.brew.sh/cask/ungoogled-chromium) |
| FreeBSD | [pkg](https://www.freshports.org/www/ungoogled-chromium/) |
| Flatpak | [Flathub](https://flathub.org/apps/details/io.github.ungoogled_software.ungoogled_chromium) |

### Tải binary trực tiếp

[**Tải tại đây**](https://ungoogled-software.github.io/ungoogled-chromium-binaries/)

> ⚠️ Các binary này do cộng đồng cung cấp và không đảm bảo tính xác thực 100%.

---

## Tự build từ mã nguồn

Xem [hướng dẫn build tiếng Việt](docs/building_vi.md) hoặc [hướng dẫn build tiếng Anh](docs/building.md).

Tóm tắt quy trình:

```
Tải source → Xóa binary → Áp dụng patch → Thay thế domain → Build GN → Build với Ninja
```

Mỗi nền tảng có repository riêng:

| Nền tảng | Repository |
|----------|-----------|
| Android | [ungoogled-chromium-android](https://github.com/ungoogled-software/ungoogled-chromium-android) |
| Arch Linux | [ungoogled-chromium-archlinux](https://github.com/ungoogled-software/ungoogled-chromium-archlinux) |
| Debian/Ubuntu | [ungoogled-chromium-debian](https://github.com/ungoogled-software/ungoogled-chromium-debian) |
| Fedora/CentOS | [ungoogled-chromium-fedora](https://github.com/ungoogled-software/ungoogled-chromium-fedora) |
| Portable Linux | [ungoogled-chromium-portablelinux](https://github.com/ungoogled-software/ungoogled-chromium-portablelinux) |
| Windows | [ungoogled-chromium-windows](https://github.com/ungoogled-software/ungoogled-chromium-windows) |
| macOS | [ungoogled-chromium-macos](https://github.com/ungoogled-software/ungoogled-chromium-macos) |

---

## Cấu trúc Repository

```
ungoogled-chromium/
├── chromium_version.txt      # Phiên bản Chromium hiện tại (146.0.7680.80)
├── revision.txt              # Số revision của ungoogled-chromium
├── flags.gn                  # Cờ cấu hình build (tắt dịch vụ Google)
├── downloads.ini             # URL tải mã nguồn Chromium
├── pruning.list              # Danh sách ~12.444 file binary cần xóa
├── domain_regex.list         # 21 regex thay thế domain
├── domain_substitution.list  # ~17.714 file cần thay thế domain
├── patches/                  # 110 bản vá mã nguồn
│   ├── series                # Thứ tự áp dụng patch
│   ├── core/                 # Patch cốt lõi (bắt buộc)
│   └── extra/                # Patch mở rộng (tùy chọn)
├── utils/                    # Scripts Python xử lý mã nguồn
│   ├── downloads.py          # Tải mã nguồn Chromium
│   ├── prune_binaries.py     # Xóa file binary
│   ├── patches.py            # Áp dụng patch
│   └── domain_substitution.py # Thay thế domain
├── devutils/                 # Công cụ phát triển và kiểm tra
└── docs/                     # Tài liệu
```

---

## So sánh với các trình duyệt khác

| Tính năng | Chrome | Chromium | ungoogled-chromium | Firefox |
|-----------|--------|----------|--------------------|---------|
| Mã nguồn mở | ❌ | ✅ | ✅ | ✅ |
| Không phụ thuộc Google | ❌ | ❌ | ✅ | ✅ |
| Không gửi dữ liệu tới Google | ❌ | ❌ | ✅ | ✅ |
| Tương thích extension Chrome | ✅ | ✅ | ✅ | ❌ |
| Hỗ trợ Manifest V2 | ❌ | ❌ | ✅ | N/A |
| Chống fingerprinting tích hợp | ❌ | ❌ | ✅ (thủ công) | ✅ |
| Không binary đóng gói sẵn | ❌ | ❌ | ✅ | ❌ |
| DRM (Widevine) | ✅ | ❌* | ✅ | ✅ |

*Chromium không bao gồm Widevine theo mặc định; ungoogled-chromium bật cờ hỗ trợ.

---

## Câu hỏi thường gặp

### Đây có phải là một trình duyệt hoàn toàn mới không?
**Không.** Đây vẫn là Chromium — cùng engine, cùng giao diện, cùng extension store. Chỉ khác là không có sự phụ thuộc vào Google.

### Có an toàn không khi tắt Safe Browsing?
Safe Browsing hoạt động bằng cách gửi URL bạn truy cập tới máy chủ Google để kiểm tra. Tắt nó loại bỏ một kênh thu thập dữ liệu. Bạn có thể sử dụng các giải pháp thay thế như [uBlock Origin](https://github.com/gorhill/uBlock).

### Extension Chrome có hoạt động không?
**Có**, nhưng bạn cần cài đặt thủ công (tải file `.crx` hoặc dùng extension từ nguồn khác), vì Chrome Web Store bị vô hiệu hóa theo mặc định.

### Dự án này khác gì Brave, Vivaldi?
Brave và Vivaldi là các trình duyệt **fork riêng** với giao diện và tính năng riêng. ungoogled-chromium **giữ nguyên Chromium** — chỉ loại bỏ Google.

---

## Phiên bản hiện tại

- **Chromium**: 146.0.7680.80
- **Revision**: 1
- **Giấy phép**: BSD-3-Clause

## Tài liệu thêm

- [Hướng dẫn build tiếng Việt](docs/building_vi.md)
- [Hướng dẫn build tiếng Anh](docs/building.md)
- [Tài liệu thiết kế kỹ thuật](docs/design.md)
- [Danh sách cờ tùy chỉnh đầy đủ](docs/flags.md)
- [Nền tảng hỗ trợ](docs/platforms.md)
- [FAQ chính thức](https://ungoogled-software.github.io/ungoogled-chromium-wiki/faq)
- [Hướng dẫn đóng góp](docs/contributing.md)
