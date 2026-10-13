# AGENTS.md — Dự án kênh YouTube "đời sống nước Mỹ thế kỷ 19"

File này để Codex (VS Code) nắm bối cảnh. Trả lời chủ dự án bằng **tiếng Việt**; mọi nội dung kênh (kịch bản, tiêu đề, prompt) bằng **tiếng Anh**.
Cập nhật mục "Trạng thái hiện tại" mỗi khi có kết quả mới.

## 1. Mục tiêu

- Kênh YouTube faceless, làm bằng AI, kiếm tiền từ khán giả **Mỹ, nói tiếng Anh, 45 tuổi trở lên**.
- Ngách: **lịch sử — đời sống thường ngày ở Mỹ thế kỷ 19** (không phải chiến tranh, chính trị).
- Dạng video: long-form trên 8 phút (để có mid-roll), kiểu **phim AI điện ảnh**, KHÔNG phải slide ảnh tĩnh.
- Video mẫu: **"How Did Americans Survive Winter Before Electricity?"** — khoảng 13 phút, ~1.850 từ, 99 clip.
- Chủ dự án muốn tự động tối đa: tự làm, ít hỏi lại. Chỉ dừng để hỏi ở các việc ghi trong mục 3.

## 2. Nội dung video mẫu

- Bối cảnh: gia đình Miller, nông trại ở **Vermont, mùa đông 1855**.
- Nhân vật (giữ nhất quán giữa các clip): **Thomas** (cha), **Sarah** (mẹ), **Eli** (con trai), **Anna** (con gái nhỏ). Mô tả ngoại hình chi tiết nằm trong `sample/winter_1800s_script.md`.
- Phong cách hình: period drama, photorealistic, ánh sáng tự nhiên (lửa, nến, đèn dầu, ánh ngày), hạt phim 35mm, 16:9.
- Cấm trong hình: đèn điện, đồ vật hiện đại, chữ, watermark, đồ vật sai thời kỳ.
- Kịch bản đã được kiểm chứng nguồn (bảng fact-check ở cuối file kịch bản). Không thêm số liệu lịch sử mới nếu chưa có nguồn.

## 3. Quy tắc bắt buộc

**API key**
- Key nằm trong `api-key-*.txt` và `api-comfy-cloud.txt`. Script tự đọc tại máy này.
- Không in key ra console, log, hay file kết quả; không đưa key lên kho HF; không commit key.

**Chi phí GPU (Vast.ai)**
- Giá thuê dưới **0,60 USD/giờ**, mỗi lần chạy có giới hạn thời gian (`MAX_MIN`).
- Chỉ thuê GPU đời mới, VRAM từ 24 GB (`compute_cap>=860`) — từng thuê nhầm Tesla P40.
- **Luôn xoá máy thuê** khi xong hoặc lỗi, rồi kiểm tra lại bằng `vastai show instances`.

**YouTube**
- Không upload, đăng, hay đổi cài đặt kênh khi chưa được chủ dự án đồng ý cho từng lần.
- Tránh dấu hiệu "inauthentic content": thumbnail/tiêu đề gần giống hệt nhau, slideshow ảnh tĩnh dài, upload hàng loạt, nhân vật chuyên gia AI giả.
- Mốc cần nhớ: từ **1/2/2027** điều kiện YPP tăng gấp đôi (1.000 sub + 8.000 giờ xem/365 ngày, hoặc 20 triệu view Shorts/90 ngày).

## 4. Thư mục

| Đường dẫn | Nội dung |
|---|---|
| `make_frames.py` | Tạo 5 ảnh frame đầu bằng OpenAI `gpt-image-1` (1536x1024) vào `frames\` |
| `run_all.py` | Chạy 1 lệnh: tạo ảnh nếu thiếu → đẩy job lên kho HF → thuê GPU Vast.ai → chờ `DONE` → xoá máy → tải kết quả → đo chuyển động. Ghi log vào `run_all_log.txt` |
| `job_ltx.py` | Chạy trên máy thuê: ảnh → video bằng LTX. Đọc `job_config.json` |
| `job_config.json` | `mode` (`test` = thử 1 ảnh với từng config rồi chọn; `full` = chạy cả 5 ảnh), `config`, `image_cond_noise_scale`, `crf`, ngưỡng chuyển động |
| `motion.json` | Prompt chuyển động cho từng clip (4–6 câu, mô tả hành động theo trình tự + chuyển động máy quay) |
| `frames\` | 5 ảnh frame: C005 lò sưởi, C014 bổ củi, C029 warming pan, C046 nhà hun khói, C087 bão tuyết ở trường |
| `ltx_result\out\` | 92 clip 1024x576 mới nhất (config b); `out_v1\` bản trước khi sửa 16 clip |
| `final\` | `winter_1800s_v1.mp4` bản dựng đầy đủ + `edl.csv` + `seg\` (clip 1080p trung gian) |
| `stitch.py`, `clips.json`, `frames_prompts.json`, `redo_list.json` | Ghép video; thứ tự/thời lượng clip; prompt ảnh; danh sách clip cần tạo lại |
| `ltx_result_configA\out\` | Kết quả lần test A/B + 5 clip config a, có `decision.json` |
| `ltx_result_old\out\` | Lần chạy đầu, clip gần như tĩnh (để so sánh) |
| `sample\winter_1800s_script.md` | Kịch bản đầy đủ + 60 cảnh chính + bảng fact-check |
| `sample\winter_1800s_clips.csv` | Bảng 99 clip: `clip, section, seconds, type, characters, image_prompt, video_prompt, negative_prompt, archive_note` |

Truyền file với máy thuê qua kho dataset riêng tư trên Hugging Face `<HF user>/ltx-test-job` (vì `vastai copy` không chạy trên Windows).

## 5. Trạng thái hiện tại (cập nhật 8/10/2026, 11:30)

**Đang làm lại toàn bộ theo yêu cầu 13–14/10/2026:** nguồn lời đọc vẫn là `blocks.json` (34 đoạn, 1.584 từ). Đã tạo đủ giọng đọc trong `final/voice/`, tải đủ 7 tư liệu và lưu nguồn/quyền trong `rebuild_20261013/archive/sources.json`. GPU RTX 5000 Ada 32 GB, instance `55778831`, giá thực tế gồm 150 GB disk ~0,4683 USD/giờ (offer ~0,4281), MAX_MIN=240; controller `rebuild_full.py` có watchdog xóa đúng instance của lần chạy, không xóa máy khác. Toàn bộ 104 ảnh/clip AI tạo mới trong `rebuild_20261013/frames/` và `out/`; không tái dùng clip v1. Upload input một lần vào dataset riêng tư dành cho rebuild; tải kết quả liên tục qua SSH để tránh 128 commit/giờ. Kế hoạch/thời lượng trong `rebuild_20261013/plan.json`; script dựng cuối `finish_rebuild.py`; đích `final/rebuild_20261013/winter_1800s_full.mp4`. Chưa hoàn thành bản tổng; không coi file pilot là kết quả của yêu cầu này. Lưu ý: Vast CLI hiện trả `cur_state` thay vì `actual_status` trên máy này. Đã tạo đủ 104 ảnh và sửa 17 cảnh qua kiểm tra, thêm lượt sửa riêng C066 (chuyển sang ngựa uống nước để bỏ thẻ tai) và C073 (bỏ bếp cửa kính). Prompt/motion thực tế trong rebuild_20261013/*_repaired.json; lịch sử trong frame_repairs_applied.json. 34 đoạn giọng đọc mới có mốc từng từ trong rebuild_20261013/voice/; finish_rebuild.py dùng thư mục này và sửa phụ đề theo mốc TTS. Score gốc Winter Hearth và âm gió/lửa tổng hợp được mix qua OpenMontage AudioMixer. Khóa ngoại hình bằng mô tả chung trong prompt; chưa có LoRA/reference-conditioned image generation.

**Kết quả mới 13/10/2026 — thử OpenMontage:** đã tạo `final/openmontage_pilot_42s.mp4` (41,8 giây, 1080p, 24 fps, lời đọc B03–B04 + âm nền thử). Dùng clip v1 hiện có; không thuê GPU hoặc gọi API trả phí. OpenMontage thực hiện mix và dựng qua FFmpeg; script `openmontage_pilot.py` chuyển về 24 fps và kiểm tra file giao. Báo cáo: `final/openmontage_pilot_report.md`. Bản giao giải mã không lỗi, khoảng −14,84 LUFS / −1,47 dBTP. Chưa chuyển toàn bộ pipeline; đang chờ chủ dự án xem/nghe đoạn thử. Nhân vật chưa nhất quán và hình về luồng nhiệt lên ống khói chưa đủ cụ thể. Chi tiết/tool checkout trong `tools/OpenMontage/`, môi trường riêng `.venv-openmontage/`. Bản v2 đầy đủ chưa được render bởi thử nghiệm này.

**Đã có bản dựng đầy đủ v1:** `final\winter_1800s_v1.mp4` (1920x1080, 24 fps, H.264 + AAC im lặng, 92 clip AI nối thẳng theo thứ tự CSV, ~8,7 phút). Bảng thời gian từng clip trong `final\edl.csv`. 7 clip `archive` (C022, C051, C059, C075, C081, C088, C091) CHƯA có, cần ảnh tư liệu thật rồi chèn vào vị trí ghi trong EDL. Chưa có lời đọc, nhạc, tiêu đề.

Pipeline hiện tại (một lần thuê, ~2,5 giờ, ~0,7 USD): `python run_all.py --mode full --config b --gen-frames --max-min 240`
1. Ảnh frame: **Z-Image-Turbo** (Apache-2.0) trên máy thuê, 1536x864, 8 bước, ~12 giây/ảnh, prompt từ `frames_prompts.json` (sinh từ CSV; 15 ảnh đã viết lại prompt "interior" trong `redo_list.json`). Không dùng OpenAI nữa (chủ dự án yêu cầu miễn phí).
2. Clip: LTX-Video-0.9.7-distilled (config b), 1024x576, số frame theo `clips.json` (5 s = 121, 6 s = 145, ...), ~55-95 giây/clip. Ảnh điều kiện được cắt 16:9 + nén H.264 một vòng.
3. Ghép: `python stitch.py` → 1080p.

Chất lượng: chuyển động mạch lạc ở hầu hết clip (chênh frame đầu-cuối 10-70/255). Lỗi còn lại cần xem khi duyệt: vòi nước nhỏ còn sót sau bồn rửa (C002, C003); bếp lò vẽ có cửa kính (C023, C067, C078); một số clip máy quay trôi khỏi bố cục (C009 đã sửa, C015, C029 cũ). Nhân vật chưa nhất quán giữa các clip (chưa có LoRA).

## 6. Việc tiếp theo

1. Chủ dự án xem `final\winter_1800s_v1.mp4`, ghi lại clip cần làm lại (thêm mã clip vào `redo_list.json`, sửa prompt trong `frames_prompts.json`/`motion.json`, chạy lại với `--resume --gen-frames` sau khi xóa clip đó trên kho hoặc trên máy thuê).
2. Tải 7 ảnh tư liệu public-domain (Library of Congress, Chronicling America) cho các clip `archive`, làm Ken Burns bằng ffmpeg, chèn theo `final\edl.csv`.
3. Lời đọc tiếng Anh (TTS), nhạc nền, tiêu đề/thumbnail; ghép lại với âm thanh.
4. LoRA nhân vật khi cần nhất quán hơn.
5. Upload: chỉ khi chủ dự án duyệt.

## 7. Lỗi đã gặp — đừng lặp lại

- `vastai destroy instance` treo nếu không trả lời xác nhận → truyền `input="y\n"` và đặt timeout.
- `huggingface_hub` cần `sys.stdout.isatty()` → lớp `Tee` ghi log phải có `isatty` và `__getattr__`.
- Model 13B bản đầy đủ (không distilled) chạy quá chậm trên 24 GB, hết giờ mà không ra kết quả.
- Ảnh PNG sạch + prompt ngắn → LTX sinh video đứng im.
- Lọc offer Vast thiếu `compute_cap` → thuê phải GPU cũ.
- Hugging Face giới hạn **128 commit/giờ** cho một kho: đẩy từng ảnh/clip một làm job treo (429). Giờ job KHÔNG đẩy từng clip; kết quả lấy về bằng `scp` qua SSH (`vastai attach ssh`), hoặc dùng `upload_folder` một lần.
- Vast tính **phí băng thông theo GB** (`inet_down_cost`): model 13B + Z-Image ≈ 81 GB mỗi lần thuê mới. Lọc `inet_down_cost<=0.005`; host 141499 băng thông rẻ nhưng tải 0,5 GB/phút (loại bằng `--exclude-machines`).
- `vastai destroy instance` cần `-y`, nếu không sẽ treo chờ xác nhận.
- Z-Image với prompt chung "deep winter, snow" hay đặt cảnh trong nhà ra ngoài trời → prompt cảnh nội thất phải mở đầu bằng "Interior of ...".
- `pkill -f job_ltx.py` qua SSH tự giết phiên SSH (dòng lệnh chứa tên file) → dùng mẫu neo `"^python job/job_ltx"`.

## 8. Bản v2 (9/10/2026) — đọc `V2_HUONG_DAN.md` trước khi làm tiếp

- Kịch bản v2 và bước dựng khớp lời đọc đã chuẩn bị xong: `blocks.json` (nguồn chuẩn lời đọc + clip từng đoạn), `voice.py`, `stitch_v2.py`, `prepare_redo.py`, `redo_v2.json`.
- `frames_prompts.json`, `motion.json`, `clips.json`, `redo_list.json` đã được thay; bản cũ trong `backup_v1/`.
- Cần render 55 clip (40 sửa, 12 mới C100–C111, 3 chỉ sửa motion). Thứ tự lệnh và bảng lỗi clip v1 nằm trong `V2_HUONG_DAN.md`.
- `stitch.py` và `final/winter_1800s_v1.mp4` là bản cũ, giữ để so sánh.
