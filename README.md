# SoFM: Symmetry of Facial Makeup

SoFM dùng Siamese CNN để đánh giá tính đối xứng của trang điểm qua các cặp ảnh trái/phải ở mắt, má và môi. Dự án hỗ trợ huấn luyện với BCE hoặc contrastive loss.

## 1. Cài đặt

Thực hiện các lệnh từ **thư mục gốc repository**:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt PyYAML opencv-python dlib
```

Nếu chưa có `.env`, tạo từ mẫu:

```bash
cp .env.example .env
```

## 2. Chuẩn bị dữ liệu

Chuẩn bị ảnh đã cắt thành các cặp trái/phải, chia thành ba tập `train_data`, `val_data`, `test_data`. Ví dụ:

```text
DATA_ALL/data4train/downgrade_cropped/
├── train_data/
│   ├── label_0/
│   │   └── eye/
│   │       └── pair_001/
│   │           ├── left_eye.png
│   │           └── right_eye.png
│   └── label_1/              # Cùng cấu trúc như label_0
├── val_data/                 # Cùng cấu trúc như train_data
└── test_data/                # Cùng cấu trúc như train_data
```

- Nhãn được xác định bởi thư mục `label_0` hoặc `label_1`.
- Trong mỗi nhãn, chia theo vùng: `eye`, `cheek`, `lip`.
- Mỗi thư mục mẫu chứa đúng **hai ảnh** của cùng vùng, tương ứng bên trái và bên phải; tên ảnh có thể tùy chọn.
- Chia theo người/ảnh gốc để tránh trùng dữ liệu giữa các tập. Script hỗ trợ chuẩn bị dữ liệu nằm trong `scripts/data_preparation/`.

Đặt `DATASET_DIR=DATA_ALL/data4train` trong `.env`, hoặc đổi thành đường dẫn dữ liệu trên máy. Giữ `CONFIG_DIR=src/config`.

## 3. Huấn luyện

Trong `src/config/exp_config.yaml`, chỉnh các trường sau và giữ các cấu hình còn lại:

```yaml
MODEL_TYPE: BCE                # BCE, contrast_inter hoặc contrast_origin
DATASET:
  NAME: downgrade_cropped      # Thư mục con bên trong DATASET_DIR
  COMPONENT: all               # all, eye, cheek hoặc lip
is_pretrained: false           # Huấn luyện mới
```

Điều chỉnh `TRAIN_NUM_EPOCHS`, `TRAIN_BATCH_SIZE` và `LEARNING_RATE` trong `hyper_param`, sau đó chạy:

```bash
PYTHONPATH=src python examples/train_and_eval.py
```

Chương trình huấn luyện, lưu checkpoint và biểu đồ loss/accuracy, rồi đánh giá trên `test_data`. Tự dùng GPU nếu có CUDA, nếu không sẽ chạy CPU.

Theo `.env.example`, checkpoint nằm trong `CHECKPOINTS_ALL/`, biểu đồ trong `outputs/figures/`, log trong `outputs/logs/`. Dữ liệu, checkpoint và đầu ra cục bộ được Git bỏ qua.
