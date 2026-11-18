# Chạy model trên Google Colab bằng PowerShell

Tài liệu này hướng dẫn dùng Colab để chạy model của repo Day 08 trên GPU T4. Bạn chạy lệnh trong **PowerShell trên Windows**; model chạy trên máy Linux tạm thời của Colab.

## Hiểu nhanh trước khi bắt đầu

- **PowerShell** là cửa sổ dòng lệnh trên Windows. Các khối lệnh bên dưới được dán vào PowerShell, không phải vào ô code của trang Colab.
- **`.venv`** là môi trường Python riêng của repo trên máy bạn. Nó giúp lệnh `python` tìm được Colab CLI. Nếu đầu dòng đã có `(.venv)`, môi trường đang bật.
- **SSH key** là cặp file dùng để CLI kết nối tới máy Colab. File không có đuôi `.pub` là private key; không gửi file này cho ai.
- **Runtime/session** là máy Linux có GPU do Colab cấp. Dữ liệu ở đó là tạm thời; hãy tải kết quả về máy trước khi phiên kết thúc.
- Máy local và runtime Colab là hai máy riêng. Colab không tự thấy file trong repo Windows; hướng dẫn này clone repo lên runtime.

> Các lệnh dưới đây đã được chạy thành công trong PowerShell trên máy này.

## A. Chuẩn bị trên máy Windows

### Bước 1. Mở PowerShell tại thư mục repo

Dán lệnh sau:

```powershell
cd 'C:\Personal\AI_In_Action\L2\K5-L23-Day08-2A202603268_VuHoangNam-AI-Assisted-Labeling'
```

### Bước 2. Bật môi trường Python

Nếu đầu dòng PowerShell đã hiện `(.venv)`, bỏ qua bước này. Nếu chưa, chạy:

```powershell
.\.venv\Scripts\Activate.ps1
```

Đây là file kích hoạt đã được tạo sẵn khi môi trường `.venv` được tạo; bạn không cần tự viết file này. Lệnh chỉ áp dụng cho cửa sổ PowerShell hiện tại, không bật GPU Colab.

Nếu PowerShell báo script bị chặn, chạy lệnh sau rồi thử kích hoạt lại. Thiết lập này chỉ áp dụng cho cửa sổ PowerShell hiện tại:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Nếu PowerShell báo không tìm thấy file `Activate.ps1`, tạo môi trường rồi bật nó:

```powershell
py -m venv .venv
```

```powershell
.\.venv\Scripts\Activate.ps1
```

Cài Colab CLI vào môi trường này một lần:

```powershell
python -m pip install google-colab-cli
```

### Bước 3. Kiểm tra SSH key

Kiểm tra key mặc định có sẵn chưa:

```powershell
Test-Path "$env:USERPROFILE\.ssh\id_ed25519"
```

Nếu kết quả là `True`, dùng key này và sang bước 4. Nếu là `False`, tạo key mới bằng hai lệnh sau:

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.ssh" | Out-Null
```

```powershell
ssh-keygen -t ed25519 -f "$env:USERPROFILE\.ssh\id_ed25519" -C 'colab-day08'
```

Khi được hỏi `Enter passphrase`, nhấn Enter hai lần để không đặt mật khẩu cho key; cách này đơn giản hơn khi chạy CLI. Nếu báo file đã tồn tại, không ghi đè key cũ. Hãy kiểm tra lại đường dẫn hoặc dùng tên file khác.

### Bước 4. Lưu đường dẫn key để dùng trong lệnh

Chạy hai dòng này trong PowerShell:

```powershell
$key = "$env:USERPROFILE\.ssh\id_ed25519"
```

```powershell
Test-Path $key
```

Kết quả phải là `True`. `$key` chỉ là tên biến PowerShell chứa đường dẫn tới file private key. Giữ nguyên cửa sổ này để dùng tiếp; đóng cửa sổ thì cần chạy lại dòng gán `$key`.

## B. Tạo hoặc kết nối runtime Colab

### Bước 5. Tạo runtime T4 (chỉ tạo khi chưa có phiên `day08`)

```powershell
python -m colab_cli.cli new -s day08 --gpu T4
```

Ở lần chạy đầu, CLI có thể yêu cầu đăng nhập Google hoặc mở trình duyệt. Hoàn tất đăng nhập rồi đợi đến khi thấy:

```text
[colab] Session READY.
```

Nếu đã có phiên `day08` đang hoạt động, bỏ qua lệnh tạo phiên. Tạo lại phiên không cần thiết có thể cấp một runtime khác.

### Bước 6. Kiểm tra kết nối và GPU

```powershell
python -m colab_cli.cli ssh -s day08 -i $key -- nvidia-smi -L
```

Kết quả cần có tên GPU, ví dụ:

```text
GPU 0: Tesla T4
```

Khi thấy tên GPU, kết nối đã thành công. Tiếp tục chạy lệnh tiếp theo trong chính cửa sổ PowerShell này.

## C. Chuẩn bị repo trên runtime Colab

Các lệnh trong mục này chạy trên runtime Linux thông qua CLI. Chúng vẫn được dán vào PowerShell.

### Bước 7. Chép mã nguồn repo lên Colab

Lệnh `git clone` tải một bản sao repo công khai từ GitHub vào runtime Colab. Nó không chép các file riêng trên máy Windows.

Chạy một lần cho mỗi runtime mới:

```powershell
python -m colab_cli.cli ssh -s day08 -i $key -- bash -lc "git clone https://github.com/hoangnam312/K5-L23-Day08-AI-Assisted-Labeling /content/lab"
```

Nếu runtime hiện tại đã có thư mục `/content/lab`, bỏ qua lệnh clone. Runtime mới thường cần clone lại vì filesystem Colab là tạm thời.

### Bước 8. Cài thư viện trên Colab

```powershell
python -m colab_cli.cli ssh -s day08 -i $key -- bash -lc "cd /content/lab && pip -q install 'ultralytics>=8.4.0' opencv-python-headless pyyaml matplotlib 'transformers>=5.0' huggingface_hub"
```

Lệnh này cài thư viện **trên Colab**, không cài vào máy Windows. Chạy lại nếu tạo runtime mới.

### Bước 9. Tải model lên Colab

```powershell
python -m colab_cli.cli ssh -s day08 -i $key -- bash -lc "cd /content/lab && python scripts/download_models.py --set core"
```

Đợi lệnh chạy xong. Các model cần dùng phải báo tải thành công (`OK`). Chạy lại nếu tạo runtime mới.

## D. Chạy thử trên tập pilot

`Pilot` là lượt chạy thử trên một phần ảnh để xem model hoạt động ra sao. Chạy từng lệnh một và đợi lệnh trước hoàn tất. Các kết quả pilot được lưu trên Colab trong `work/prelabel/`; ảnh xem trước nằm trong thư mục `viz/`.

### YOLO26, ngưỡng 0.25

```powershell
python -m colab_cli.cli ssh -s day08 -i $key -- bash -lc "cd /content/lab && python scripts/prelabel.py --model yolo26 --size m --split pilot --conf 0.25 --out work/prelabel/t_y26"
```

### YOLO26, ngưỡng 0.5

```powershell
python -m colab_cli.cli ssh -s day08 -i $key -- bash -lc "cd /content/lab && python scripts/prelabel.py --model yolo26 --size m --split pilot --conf 0.5 --out work/prelabel/t_y26_c05"
```

### YOLOE cho lớp xe ba bánh

```powershell
python -m colab_cli.cli ssh -s day08 -i $key -- bash -lc "cd /content/lab && python scripts/prelabel.py --model yoloe --classes three_wheeler --split pilot --conf 0.25 --out work/prelabel/t_yoloe"
```

### YOLO26 kết hợp SAM 2.1 để tạo mask

```powershell
python -m colab_cli.cli ssh -s day08 -i $key -- bash -lc "cd /content/lab && python scripts/prelabel.py --model yolo26 --size m --classes 'person,motorbike,car' --images img_004.jpg img_013.jpg --conf 0.3 --segmenter sam --sam sam2.1_b.pt --out work/prelabel/t_sam"
```

Mỗi lệnh tạo một thư mục kết quả riêng. Mở các ảnh trong thư mục `viz/` để kiểm tra box (khung quanh vật thể), mask (vùng vật thể), nhãn sai và vật thể bị bỏ sót. Ghi nhận xét vào `submission/model_log.csv`. `Gold` là các ảnh được gán nhãn thủ công để làm chuẩn đối chiếu. Chỉ chạy toàn bộ ảnh sau khi pilot đã được review và hoàn tất gán tay các ảnh gold.

### Chạy YOLO26 trên toàn bộ ảnh (chỉ sau khi pilot và gold đã hoàn tất)

```powershell
python -m colab_cli.cli ssh -s day08 -i $key -- bash -lc "cd /content/lab && python scripts/prelabel.py --model yolo26 --size m --split all --conf 0.05 --out work/prelabel/y26_all --no-viz"
```

Kết quả được lưu trên Colab tại `/content/lab/work/prelabel/y26_all`. Đây là bước chạy toàn bộ, không phải bước chạy thử. Nếu cần giữ kết quả này, hãy nén và tải riêng thư mục đó trước khi Colab kết thúc:

```powershell
python -m colab_cli.cli ssh -s day08 -i $key -- bash -lc "cd /content/lab && zip -rq /content/lab/y26_all.zip work/prelabel/y26_all"
```

```powershell
python -m colab_cli.cli download -s day08 /content/lab/y26_all.zip .\y26_all.zip
```

## E. Tải kết quả về máy Windows

Sau khi đã chạy đủ bốn lệnh pilot ở mục D, đóng gói kết quả trên Colab:

```powershell
python -m colab_cli.cli ssh -s day08 -i $key -- bash -lc "cd /content/lab && zip -rq /content/lab/day08_pilot.zip work/prelabel/t_y26 work/prelabel/t_y26_c05 work/prelabel/t_yoloe work/prelabel/t_sam"
```

Tải file ZIP từ Colab về thư mục repo trên Windows:

```powershell
python -m colab_cli.cli download -s day08 /content/lab/day08_pilot.zip .\day08_pilot.zip
```

Giải nén kết quả:

```powershell
Expand-Archive .\day08_pilot.zip -DestinationPath . -Force
```

Mở `work/prelabel/` trong repo để xem kết quả. Hãy tải các file cần giữ về máy trước khi runtime Colab kết thúc.

## Lỗi thường gặp

| Thông báo hoặc tình huống | Cách xử lý |
|---|---|
| `(.venv)` đã hiện ở đầu dòng | Môi trường Python đã bật; bỏ qua lệnh `Activate.ps1`. |
| `Activate.ps1` không tìm thấy | Đang ở sai thư mục hoặc `.venv` chưa được tạo. Quay lại bước 1 hoặc chạy `py -m venv .venv`. |
| PowerShell báo không cho chạy `Activate.ps1` | Chạy `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`, rồi thử kích hoạt lại. |
| `Test-Path $key` trả về `False` | Key chưa có ở đường dẫn đó. Làm theo bước 3 để tạo key, rồi gán lại `$key` ở bước 4. |
| `Identity file -- not accessible` | `$key` chưa được gán trong cửa sổ PowerShell hiện tại. Chạy lại dòng `$key = ...` ở bước 4. |
| `PermissionError` với `settings.json.lock` | Không đổi `USERPROFILE` sang thư mục trong repo. Mở PowerShell mới, làm lại bước 1 và giữ `USERPROFILE` mặc định của Windows. |
| `HTTP 429` khi SSH | Có thể một kết nối SSH khác vẫn đang mở. Nhấn `Ctrl+C` ở kết nối cũ rồi thử lại; không cần tạo runtime mới. |
| Đã tạo runtime mới nhưng không thấy repo/model/output cũ | Runtime mới có ổ đĩa tạm riêng. Làm lại bước 7 đến bước 9 và chạy lại model cần thiết. |

## Dùng CLI với repo khác

Các bước trên dành cho repo Day 08. Với repo khác, giữ quy trình chung: tạo runtime → clone repo → cài thư viện → chạy thử trên ít dữ liệu → review → chạy toàn bộ → tải kết quả về máy. Thay URL repo, lệnh cài đặt, lệnh inference và đường dẫn output theo dự án đó.
