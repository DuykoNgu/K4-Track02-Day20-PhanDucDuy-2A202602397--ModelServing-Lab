# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Phan Duc Duy
**MSSV:** 2A202602397
**Cohort:** A20-K4
**Ngày submit:** 2026-10-06

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** macOS Darwin 25.6.0 (arm64)
- **CPU:** Apple M1 Pro
- **Cores:** 10 physical / 10 logical
- **CPU extensions:** NEON
- **RAM:** 16 GB
- **Accelerator:** Apple Metal
- **llama.cpp asset đã tải:** `llama-b10488-bin-macos-arm64.tar.gz`
- **Model đã dùng:** Gemma 4 E2B (`LAB_MODEL=gemma4-e2b`)
- **Quantization:** UD-Q4_K_XL + UD-Q2_K_XL (từ `models/active.json`)

**Chạy ở đâu:** laptop của tôi (Apple M1 Pro)
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Setup hoàn tất trên máy local bằng runtime prebuilt; không cần build source. Hai quantization tải thành công, không cần workaround.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 3062 | 108 / 120 | 18.6 / 21.2 | 1275 / 1446 / 1446 | 53.7 |
| UD-Q2_K_XL | 2.24 | 3056 | 108 / 112 | 17.5 / 17.9 | 1205 / 1233 / 1233 | 57.1 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

Q2 nhanh hơn 1.06× theo decode và nhỏ hơn 0.73 GB (~25%). Với prompt ngắn đã thử, cả hai cùng chọn Q4 để ưu tiên chất lượng; câu trả lời Q2 cụ thể hơn đôi chút. Một prompt chưa đủ kết luận chất lượng tổng quát. Chọn Q4 khi ưu tiên chất lượng, Q2 khi cần giảm dung lượng.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 1.44 | 5900 | 8200 | 8900 | 8.3 | 0 |
| 50 | 1.37 | 29000 | 38000 | 40000 | 34.0 | 0 |

- **Offered load tăng 5×, throughput thực tăng:** 0.95×
- **P95 tăng:** 4.63×
- **Effective concurrency ở 50 users:** 34.0 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): 3.98 / 4 slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

Server đã bão hòa ở mức không quá 10 users: concurrency hiệu dụng 8.3 đã vượt 4 slots. Từ 10 lên 50 users, RPS giảm nhẹ 0.95× nhưng P95 tăng 4.63×; metrics ghi nhận 3.98/4 slots bận và 46 request deferred. Tôi sẽ thử giảm `max_tokens` trước để rút ngắn decode và hàng đợi, đổi lại câu trả lời ngắn hơn.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | localhost only | stub |
| N17 Data pipeline | no pipeline job | stub |
| N18 Lakehouse | toy in-memory docs | stub |
| N19 Vector + features | keyword overlap | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: 0.0 ms
- retrieve: 0.0 ms
- llm: 859.2 ms
- **stage chiếm nhiều nhất:** llm (100% of total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

LLM là bottleneck như dự đoán, chiếm gần toàn bộ 859.2 ms; embedding không bật và keyword retrieval trên bộ dữ liệu rất nhỏ gần như không tốn thời gian. Muốn giảm latency 2×, trước tiên tôi sẽ giảm số token đầu ra rồi đo lại.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** giảm số thread từ 10 xuống 5

```
before:  49.6 tok/s (`-t 10`)
after:   53.7 tok/s (`-t 5`)
speedup: 1.08×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

Sweep cho thấy 5 thread đạt 53.7 tok/s, cao hơn 10 thread (49.6 tok/s) 8%. Đỉnh nằm dưới số lõi vật lý. Với `ngl=99`, model được offload lên Metal nên số thread CPU không ánh xạ trực tiếp thành số luồng decode; nhiều thread hơn có thể thêm chi phí điều phối mà không tăng phần việc GPU. Kết quả này chỉ nói 5 thread là tốt nhất trong các mức đã thử trên cấu hình này.

---

## 6. Bonus  *(optional — tối đa 10 điểm)*

> Bỏ trống nếu không làm. Xem `docs/bonus/README.md`. Đừng làm hết — **một** finding sâu
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
- [ ] Repo tên đúng mẫu `K4-L3-DAY20-HoVaTen-MSSV-ModelServing` (xem `docs/SUBMISSION.md`)
- [ ] Repo GitHub ở chế độ **public**
- [ ] Đã push và paste public URL vào VinUni LMS **trước 23:59 (UTC+7) ngày làm lab**
- [ ] **Không** commit `models/*.gguf`, `runtime/` hay `.env` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.

---

## 9. Khai báo sử dụng AI  *(xem `docs/RULES.md` §3)*

Đã dùng ChatGPT để giải thích lab, đọc số liệu benchmark/load test và hỗ trợ soạn nhận xét dựa trên kết quả đo của máy tôi.
