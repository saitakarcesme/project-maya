# Maya-S24 on 2x RTX 3090 with 32 GB of RAM (2026-10-21)

v1.1.20 (`35bb2b2`), Maya-S24, 32K context, MTP, vision on. 2x RTX 3090 24 GB (GPU0 PCIe 4.0 x16, GPU1 4.0 x4,
no NVLink), Core i5-14400F, 32 GB RAM (~24 GB free), Kingston NV3 2 TB (DRAM-less). Built and run in a CUDA 13.0
container. `./maya.sh --bench`:

| settings | decode | prefill 2K | prefill 8K |
| --- | ---: | ---: | ---: |
| stock | 11.7 tokens/s | 145 | 555 |
| `PROMOTE_MIN=6 KV_INT8=1 MTP_KEEP=2 SPEC_SAMPLE=1 SPEC_CANCEL=1` | 8.0 | 24 | 70 |
| `DISK_READERS=8 RAM_HEADROOM_GB=4 MTP_KEEP=2 SPEC_SAMPLE=1 SPEC_CANCEL=1` | 12.2 | 194 | 657 |
| the same + `DISK_SKIP_W=0.12 MISS_SKIP_W=0.14 MISS_SKIP_FROM=23` | **29.1** | 197 | 523 |

RAM tier 11-16 GB: decode waits ~80-100 ms a token on ~25 disk reads (stock); `DISK_SKIP_W=0.12` cuts that to
5-7 reads and ~25 ms. `PROMOTE_MIN=6` (good with `RAM_RESIDENT`) costs a third here: promotions 35 -> 6 a token,
so the next token reads the same experts from the SSD again. GPU1's x4 link: a PCIe expert fetch 1.1 ms against
0.32 ms on GPU0. Thinking-on chat (t=0.6) with the skip settings: 35 tokens/s; three 2000-token answers 28.8 / 28.2 /
18.0. A functional battery (tools, multi-turn, SSE, vision, 17K needle, greedy) 8/8.

Configs and engine lines: [saitakarcesme/maya-s24-turbo `lowram-32gb/`](https://github.com/saitakarcesme/maya-s24-turbo/tree/main/lowram-32gb).
