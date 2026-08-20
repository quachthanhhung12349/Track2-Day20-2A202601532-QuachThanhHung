# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** _<Họ Tên>_
**Cohort:** _<A20-K1 / A20-K2 / ...>_
**Ngày submit:** _<YYYY-MM-DD>_

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** Linux 6.8.0-138-generic (x86_64)
- **CPU:** Intel(R) Core(TM) i5-8300H CPU @ 2.30GHz
- **Cores:** 4 physical / 8 logical
- **CPU extensions:** AVX2 (no AVX-512 or NEON)
- **RAM:** 15.5 GB
- **Accelerator:** NVIDIA GeForce GTX 1050 (4 GB VRAM), CUDA and Vulkan available
- **llama.cpp asset đã tải:** source build b10488 with CUDA (`-DGGML_CUDA=ON`)
- **Model đã dùng:** Gemma 4 E2B (`LAB_MODEL=gemma4-e2b`)
- **Quantization:** UD-Q4_K_XL + UD-Q2_K_XL (từ `models/active.json`)

**Chạy ở đâu:** laptop của tôi (local)

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Máy local có 15.5 GB RAM, đủ cho Gemma 4 E2B. GTX 1050 4 GB hỗ trợ CUDA và Vulkan; tôi dùng llama.cpp build b10488 với CUDA (`-DGGML_CUDA=ON`) để offload model lên GPU. Không cần cloud fallback.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 10375 | 380 / 1642 | 33.8 / 34.1 | 2513 / 3761 / 3761 | 29.6 |
| UD-Q2_K_XL | 2.24 | 5074 | 354 / 2152 | 43.2 / 46.1 | 3066 / 4866 / 4866 | 23.1 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

2 bit nhanh hơn 4 bit khoảng 2 lần khi load model, tuy nhiên khi decode model thì model 4 bit lại cho tốc độ nhanh hơn. Vì máy tôi đủ VRAM -> không nên chạy model 2 bit ở đây.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 13 | 0.23 | 31000 | 55000 | 55000 | 8.0 | 0.0% |
| 50 | 39 | 0.69 | 31000 | 56000 | 56000 | 20.3 | 0.0% |
- **Offered load tăng 5×, throughput thực tăng:** _<3.01>×_
- **P95 tăng:** _<1.02>×_
- **Effective concurrency ở 50 users:** _<20.3>_ so với `--parallel` = _<4>_ slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): _<số>_ / _<slots>_ slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

_Server mới bắt đầu saturate, nhưng ta chưa chỉ được server bão hoà ở đâu. Tôi sẽ chạy lâu hơn và thêm các load point >50 đến khi RPS dừng tăng và P95 tăng. Tôi sẽ ddoorii knob --parallel trước nếu tôi có đủ RAM/VRAM, vì vấn đề hiện tại có vẻ là concurrency/queueing với 4 request slots._

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | | stub |
| N17 Data pipeline | | stub |
| N18 Lakehouse | | stub |
| N19 Vector + features | keyword-overlap retrieval | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: _<0.0>_
- retrieve: _<0.2>_
- llm: _<1745.5>_
- **stage chiếm nhiều nhất:** _<llm>_ (_<100%>_ của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

_Bottleneck ở LLM, hợp với kỳ vọng. Nếu phải giảm latency thì ta phải tập trung vào LLM và cách tối ưu mô hình LLM_

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** _<vd: hạ -t từ 16 xuống 8; vd: đổi sang UD-Q2_K_XL; vd: --parallel 4 → 8>_

```
before:  <-t = 16 -> tg128 = 26.6>
after:   <-t = 1 -> tg128 = 29,6>
speedup: <1.11>×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

_Giải thích như đang nói với bạn ngồi cạnh. Bám vào **cơ chế**, không phải "vibes":
memory bandwidth? vector width? cache residency? scheduling? queueing? Nếu kết quả
**khác** với kỳ vọng từ deck — nói rõ, và giải thích vì sao. Grader thưởng điểm cho
lập luận đúng về một kết quả bất ngờ, hơn là một con số đẹp không được giải thích._

Tôi nghĩ nguyên nhân mà mô hình lại chạy tốt nhất ở 1 luòng là vì: khi chạy trên GPU, cho nên tốc độ decode bị giới hạn bởi GPU và băng thông RAM của GPU. Các nhân CPu khi đó chỉ tăng scheduling overhead, và áp lực lên cache/memory bandwidth của CPU khi điều phối lệnh GPU.
---

## 6. Bonus  *(optional — tối đa 20 điểm)*

> Bỏ trống nếu không làm. Xem `bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _<B1 build-compare / B2 sweep nào / B4 challenge nào / B5 lựa chọn nào>_

**Numbers:**

```
before:  <số>
after:   <số>
speedup: <X.Y>×
```

**Điều này nói lên gì mà deck chưa nói:**

_(để trống nếu bạn không làm phần này)_

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

_(để trống nếu bạn không làm phần này)_

---

## 8. Self-check trước khi push

- [ ] `hardware.json` committed
- [ ] `models/active.json` committed
- [ ] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [ ] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [ ] `benchmarks/02-server-results.md` committed (`make load-report`)
- [ ] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [ ] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [ ] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [ ] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [ ] 5 screenshots trong `submission/screenshots/`
- [ ] `make verify` → **exit 0**
- [ ] Repo GitHub ở chế độ **public**
- [ ] Đã paste public URL vào VinUni LMS
- [ ] **Không** commit `models/*.gguf` hay `runtime/` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.
