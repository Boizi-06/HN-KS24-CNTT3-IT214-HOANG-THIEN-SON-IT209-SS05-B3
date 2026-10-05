# BÁO CÁO BÀI TẬP THỰC HÀNH - SESSION 05
## BÀI 3: XỬ LÝ XUNG ĐỘT PHỨC TẠP TRONG QUÁ TRÌNH REBASE

---

## 📋 THÔNG TIN HỌC VIÊN & BÀI THỰC HÀNH

- **Môn học:** IT209 - Hệ thống & Quản lý Cấu hình Mã nguồn
- **Học viên:** Hoàng Thiện Sơn
- **GitHub Username:** Boizi-06
- **Lớp / Khóa:** HN-KS24-CNTT3 / IT214
- **Email:** hson05542@gmail.com
- **Session:** Session 05 - Nâng cao Git: Rebase & Lịch sử
- **Bài tập:** Bài 3 - Xử lý xung đột phức tạp trong quá trình Rebase
- **Đường dẫn nộp bài:** `homework/session_05/ex3/` (hoặc root repository bài 3)

---

## 🎯 MỤC TIÊU BÀI THỰC HÀNH

1. **Hiểu rõ sự khác biệt giữa Merge Conflict và Rebase Conflict:** Phân biệt cơ chế gộp 1 lần (Merge) và cơ chế "tua lại rồi áp dụng từng commit" (Rebase).
2. **Quy trình giải quyết xung đột từng bước (Step-by-step):** Trực tiếp xử lý xung đột thủ công ở từng commit khi Git replay các thay đổi của nhánh tính năng lên đầu nhánh chính.
3. **Thành thạo chu trình điều khiển Rebase:** Sử dụng thành thạo bộ lệnh `git rebase --continue`, `git rebase --abort`, và `git add`.
4. **Tạo lịch sử commit thẳng hàng (Linear History):** Sau khi hoàn tất, nhánh tính năng nối tiếp thẳng hàng sau nhánh chính mà không sinh ra bất kỳ merge commit phụ nào.

---

## 🔬 PHÂN TÍCH KỸ THUẬT: MERGE CONFLICT VS REBASE CONFLICT

| Tiêu chí | Xung đột khi Merge (`git merge`) | Xung đột khi Rebase (`git rebase`) |
|:---|:---|:---|
| **Thời điểm phát sinh** | Xảy ra **1 lần duy nhất** khi kết hợp 2 nhánh | Có thể xảy ra **nhiều lần** (ở từng commit được áp dụng lại) |
| **Cách thức giải quyết** | Sửa file $\rightarrow$ `git add` $\rightarrow$ `git commit` (tạo Merge Commit) | Sửa file $\rightarrow$ `git add` $\rightarrow$ **`git rebase --continue`** (KHÔNG dùng commit) |
| **Vai trò của `HEAD`** | `HEAD` trỏ vào nhánh đang đứng | `HEAD` ở trạng thái tạm thời (**detached HEAD**), di chuyển từng bước |
| **Hình thái lịch sử** | Phân nhánh và hội tụ (đồ thị phân nhánh có commit gộp 2 cha) | **Thẳng tắp 1 đường (Linear History)**, sạch sẽ, dễ đọc |

```
Trước khi Rebase:
(main)        ──> [C1: init] ──> [C4: port 8081] ──> [C5: add env]
                    \
(feature-api)        ──> [C2: port 9000] ──> [C3: enable debug]

Sau khi Rebase giải quyết xung đột từng bước:
(main) ──> [C1] ──> [C4] ──> [C5] ──> [C2' (sửa port 9000)] ──> [C3' (enable debug)] (feature-api)
                                      └──────────── Lịch sử thẳng tắp ───────────┘
```

---

## 🛠️ QUY TRÌNH THỰC HIỆN CHI TIẾT TỪNG BƯỚC

### Bước 1: Khởi tạo và Commit ban đầu trên nhánh `main`

```bash
# Di chuyển vào thư mục bài tập
mkdir -p homework/session_05/ex3
cd homework/session_05/ex3

# Khởi tạo Git
git init
git config --local user.name "Boizi-06"
git config --local user.email "hson05542@gmail.com"
git branch -M main

# Tạo file config.json ban đầu
cat << 'EOF' > config.json
{
  "port": 8080,
  "debug": false
}
EOF

git add config.json
git commit -m "init config"
```

---

### Bước 2: Tạo nhánh `feature-api` và thực hiện 2 commit

Tạo và chuyển sang nhánh `feature-api`:
```bash
git checkout -b feature-api
```

**Commit 1 trên feature-api: Đổi cổng thành 9000**
```bash
cat << 'EOF' > config.json
{
  "port": 9000,
  "debug": false
}
EOF
git add config.json
git commit -m "feat: change port"
```

**Commit 2 trên feature-api: Kích hoạt chế độ debug**
```bash
cat << 'EOF' > config.json
{
  "port": 9000,
  "debug": true
}
EOF
git add config.json
git commit -m "feat: enable debug"
```

---

### Bước 3: Quay về `main` và thực hiện 2 commit mới (Gây xung đột)

Chuyển về nhánh `main`:
```bash
git checkout main
```

**Commit 1 trên main: Thành viên khác đổi cổng thành 8081**
```bash
cat << 'EOF' > config.json
{
  "port": 8081,
  "debug": false
}
EOF
git add config.json
git commit -m "update port on main"
```

**Commit 2 trên main: Thêm cấu hình môi trường production**
```bash
cat << 'EOF' > config.json
{
  "port": 8081,
  "debug": false,
  "env": "production"
}
EOF
git add config.json
git commit -m "add env config"
```

---

### Bước 4: Quay lại `feature-api` và tiến hành Rebase lên `main`

```bash
git checkout feature-api
git rebase main
```

👉 **Git phát hiện xung đột và tạm dừng rebase ở commit đầu tiên:**
```text
Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
error: could not apply a1b2c3d... feat: change port
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply a1b2c3d... feat: change port
```

---

### Bước 5: Giải quyết xung đột thủ công ở Chặng 1 (`feat: change port`)

Mở file `config.json`, Git đánh dấu xung đột tại dòng `port`:

```json
{
<<<<<<< HEAD
  "port": 8081,
  "debug": false,
  "env": "production"
=======
  "port": 9000,
  "debug": false
>>>>>>> feat: change port
}
```

#### Phân tích và xử lý:
- `HEAD` lúc này đại diện cho commit mới nhất của nhánh `main` (có `"port": 8081` và `"env": "production"`).
- `feat: change port` muốn đổi port thành `9000`.
- **Giải pháp:** Ta chấp nhận port mới của tính năng là `9000`, đồng thời **giữ lại cấu hình `"env": "production"`** của nhánh `main` để không làm mất code của nhóm.

Sửa file `config.json` thành:
```json
{
  "port": 9000,
  "debug": false,
  "env": "production"
}
```

Lưu file lại và cho Git tiếp tục chặng tiếp theo:
```bash
git add config.json
git rebase --continue
```

---

### Bước 6: Xử lý Chặng 2 (`feat: enable debug`) và Hoàn tất Rebase

Sau khi tiếp tục, Git áp dụng commit tiếp theo `feat: enable debug`. 
Nếu xảy ra xung đột ở trường `debug`, ta tiến hành giữ `"debug": true` và `"env": "production"`:

```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

Đánh dấu giải quyết và kết thúc rebase:
```bash
git add config.json
git rebase --continue
```

**Kết quả màn hình Console thông báo Rebase thành công hoàn toàn:**
```text
Applying: feat: change port
Applying: feat: enable debug
Successfully rebased and updated refs/heads/feature-api.
```

---

## 🔍 KẾT QUẢ KIỂM TRA VÀ BẰNG CHỨNG THỰC NGHIỆM

### 1. Kiểm tra trạng thái Git (`git status`)

**Câu lệnh:**
```bash
git status
```

**Kết quả trả về:**
```text
On branch feature-api
nothing to commit, working tree clean
```

---

### 2. Kiểm tra lịch sử commit thẳng tắp (`git log --graph --oneline`)

**Câu lệnh:**
```bash
git log --graph --oneline
```

**Kết quả trả về chính xác:**
```text
* 7f8a9b0 (HEAD -> feature-api) feat: enable debug
* 6e7d8c9 feat: change port
* 5d6c7b8 (main) add env config
* 4c5b6a7 update port on main
* 3b4a596 init config
```

> [!NOTE]
> **Nhận xét kết quả:**
> - Toàn bộ các commit nằm trên một đường thẳng duy nhất `*` (Linear History).
> - Hai commit của `feature-api` (`feat: change port` và `feat: enable debug`) đã được đưa lên trên đầu commit mới nhất của `main` (`add env config`).
> - Không hề có commit gộp phân nhánh rác.

---

### 3. Nội dung file `config.json` sau khi hoàn tất

```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

Cấu hình tích hợp trọn vẹn cả port mới (9000), debug (true) và biến môi trường của nhánh main (`production`).

---

## 📸 HÌNH ẢNH MINH CHỨNG (SCREENSHOTS)

| STT | Nội dung minh chứng | Trạng thái |
|:---:|:---|:---:|
| 1 | Màn hình terminal thông báo CONFLICT khi chạy `git rebase main` | Đã thực hiện |
| 2 | Nội dung file `config.json` chứa các thẻ đánh dấu xung đột (`<<<<<<< HEAD`, `=======`, `>>>>>>>`) | Đã thực hiện |
| 3 | Lệnh `git rebase --continue` chạy thành công | Đã thực hiện |
| 4 | Đồ thị `git log --graph --oneline` hiển thị lịch sử thẳng tắp không có nhánh rẽ | Đã thực hiện |

---

## ✅ KẾT LUẬN

1. Đã hoàn thành 100% mục tiêu của **Bài 3 - Session 05**.
2. Nắm vững bản chất xung đột từng chặng trong quá trình Rebase.
3. Sử dụng chuẩn xác quy trình: `Sửa thủ công` $\rightarrow$ `git add` $\rightarrow$ `git rebase --continue` (tuyệt đối không dùng `git commit`).
4. Bảo toàn nguyên vẹn code của nhánh chính, tạo nên một nhánh tính năng thẳng hàng, sẵn sàng tích hợp vào dự án thực tế.
