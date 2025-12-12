# SimpleBlockWhitelist

Plugin Minecraft cho phép **whitelist (cho phép theo danh sách)** các loại block mà người chơi có thể đặt/phá, cấu hình riêng theo từng world và/hoặc từng WorldGuard region.

## Tính năng chính
- Whitelist block theo `whitelist.worlds` (theo world) và `whitelist.regions` (theo WorldGuard region). **Không có whitelist global**.
- Tự động xác định “managed world” và chỉ áp dụng rule ở các world/region được khai báo.
- Có thể bypass theo từng block bằng permission `blocks.<block_name_lowercase>`.
- Tùy chọn cho OP bypass toàn bộ whitelist bằng `opsCanBypass`.
- Thông báo khi bị chặn có thể gửi qua chat hoặc actionbar, và có thể tắt.
- Tương thích Folia (thread‑safe, không dùng Bukkit scheduler).
- Reload cấu hình nhanh bằng lệnh `/simpleblockwhitelist`.

## Yêu cầu / tương thích
- Minecraft **1.20 → 1.21.11** trên Spigot / Paper / Folia.
- **WorldGuard 7+** (tùy chọn):
  - Cần nếu bạn muốn dùng rule theo region (`whitelist.regions`).
  - **Nếu không có WorldGuard, phần `whitelist.regions` sẽ không áp dụng** (plugin chỉ đọc `whitelist.worlds`).

## Cài đặt
1. Copy file `.jar` của plugin vào thư mục `/plugins`.
2. (Tùy chọn) Cài **WorldGuard 7+** nếu muốn dùng region rules.
3. Khởi động lại server. Plugin sẽ tạo `plugins/SimpleBlockWhitelist/config.yml`.

## Cấu hình (`config.yml`)

### Các tùy chọn chung
- `opsCanBypass`
  - `true`: người chơi OP bỏ qua whitelist (đặt/phá bình thường).
  - `false`: OP bị kiểm soát như người thường.
- `sendTo`
  - Nơi gửi thông báo khi bị chặn: `chat` hoặc `actionbar`.
- `noPermission`
  - Tin nhắn khi người chơi cố đặt/phá block không được phép.
  - Hỗ trợ màu `&` (ví dụ `&c`).
  - Đặt `""` để tắt thông báo.

### Whitelist
Plugin chỉ đọc hai mục sau. Tên world/region và block **không phân biệt hoa thường**, nhưng nên dùng đúng tên Material của Minecraft (thường là UPPERCASE).

```yml
# whitelist:
#   worlds:
#     <world1>:
#       - <BLOCK1>
#       - <BLOCK2>
#       ...
#     <world2>:
#       - <BLOCK1>
#       - <BLOCK2>
#       ...
#
#   regions:
#     <world>:
#       <region1>:
#         - <BLOCK1>
#         - <BLOCK2>
#         ...
#       <region2>:
#         - <BLOCK1>
#         - <BLOCK2>
#         ...
#     <world2>:
#       <region1>:
#         - <BLOCK1>
#         - <BLOCK2>
#         ...
#       <region2>:
#         - <BLOCK1>
#         - <BLOCK2>
#         ...
```

#### `whitelist.worlds`
Danh sách block được phép **trên toàn bộ world** đó.

```yml
whitelist:
  worlds:
    world_nether:
      - NETHERRACK
      - SOUL_SAND
      - NETHER_BRICKS
```

#### `whitelist.regions`
Danh sách block được phép **trong từng WorldGuard region**.

```yml
whitelist:
  regions:
    world:
      spawn:
        - GLASS
        - GLOWSTONE
        - STONE
```

#### Managed world behavior (Lựa chọn 2)
- Một world được coi là **managed** nếu nó xuất hiện trong `whitelist.worlds.<world>` **hoặc** `whitelist.regions.<world>`.
- World **không managed** (không có trong cả hai mục) thì plugin **không ảnh hưởng gì**: người chơi đặt/phá như bình thường.
- Nếu world có entry trong `whitelist.worlds`:
  - Plugin enforce whitelist **toàn world** (world‑level).
  - Nếu đồng thời có rule region, thì **bên trong region được khai báo** sẽ cộng thêm rule region (region‑level).
- Nếu world **không có** entry trong `whitelist.worlds` nhưng có trong `whitelist.regions`:
  - Plugin **chỉ enforce bên trong các region được khai báo**.
  - Ngoài các region đó, người chơi vẫn đặt/phá tự do.

### Ví dụ cấu hình thực tế

**A) Enforce toàn world `world_nether`**

```yml
opsCanBypass: false
sendTo: actionbar
noPermission: "&cBạn không thể đặt/phá block này!"

whitelist:
  worlds:
    world_nether:
      - NETHERRACK
      - SOUL_SAND
      - NETHER_BRICKS
  regions: {}
```

**B) Chỉ enforce trong region `spawn` của world `world` (region‑only)**

```yml
whitelist:
  worlds: {}
  regions:
    world:
      spawn:
        - GLASS
        - GLOWSTONE
```

Trong ví dụ này, world `world` được managed do có `whitelist.regions.world`. Plugin chỉ chặn/cho phép **bên trong region `spawn`**.

**C) World không managed**

```yml
whitelist:
  worlds:
    world_nether:
      - NETHERRACK
  regions: {}
```

World như `world`, `world_the_end`... không xuất hiện trong whitelist ⇒ plugin không tác động.

## Tương tác với WorldGuard (case đặc biệt)
- Nếu tại vị trí đó WorldGuard **deny build** (flag `build: deny` hoặc player không có quyền build):
  - WorldGuard sẽ chặn sự kiện từ gốc; SimpleBlockWhitelist **không can thiệp** và **không gửi message**.
- Nếu muốn SimpleBlockWhitelist kiểm soát block trong một region:
  - Hãy đảm bảo WorldGuard **allow build** trong region đó.
  - Sau đó plugin mới lọc theo whitelist của region/world.

## Lệnh
- `/simpleblockwhitelist` — reload `config.yml`. Cần permission `simpleblockwhitelist.admin`.

## Quyền (Permissions)
- `simpleblockwhitelist.admin` — dùng `/simpleblockwhitelist`.
- `blocks.<block_name_lowercase>` — cho phép người chơi đặt/phá block đó dù không nằm trong whitelist.  
  Ví dụ: `blocks.tnt`, `blocks.obsidian`.  
  *Lưu ý:* quyền này vẫn tôn trọng WorldGuard; nếu WG deny build thì vẫn không đặt/phá được.
- `opsCanBypass` **không phải permission**; chỉ là tùy chọn trong config.

## Troubleshooting
- “Plugin không hoạt động trong region” → kiểm tra WorldGuard flag `build` có đang `deny` không.
- “Region whitelist không áp dụng” → thiếu WorldGuard 7+ hoặc tên region trong config sai (không khớp `region id`).
- “World bị chặn hết” → world managed nhưng whitelist rỗng hoặc config sai indentation YAML.

## Hiệu năng & Folia
- Lookup whitelist dùng `Map/Set` O(1), không chạy scheduler.
- Dữ liệu config được load thành snapshot immutable và swap atomically (phù hợp Folia và server đông người).

## Credits
- Dựa trên plugin gốc **Simple Block Whitelist** của `mfnalex`: https://github.com/mfnalex/simple-block-whitelist  
  Đây là modified fork để bổ sung/điều chỉnh hành vi theo nhu cầu server.
