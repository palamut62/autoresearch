# autoresearch

This is an experiment to have the LLM do its own research.

## Setup

To set up a new experiment, work with the user to:

1. **Agree on a run tag**: propose a tag based on today's date (e.g. `apr7`). The branch `autoresearch/<tag>` must not already exist — this is a fresh run.
2. **Create the branch**: `git checkout -b autoresearch/<tag>` from current master.
3. **Read the in-scope files**: The repo is small. Read these files for full context:
   - `README.md` — repository context.
   - `prepare.py` — fixed constants, data prep, tokenizer, dataloader, evaluation. Do not modify.
   - `train.py` — the file you modify. Model architecture, optimizer, training loop.
4. **Verify data exists**: Check that `~/.cache/autoresearch/` contains data shards and a tokenizer. If not, tell the human to run `uv run prepare.py`.
5. **Initialize results.tsv**: Create `results.tsv` with just the header row. The baseline will be recorded after the first run.
6. **Confirm and go**: Confirm setup looks good.

Once you get confirmation, kick off the experimentation.

---

## Experimentation

Each experiment runs on a single GPU. The training script runs for a **fixed time budget of 5 minutes** (wall clock training time, excluding startup/compilation). You launch it simply as: `uv run train.py`.

**What you CAN do:**
- Modify `train.py` — this is the only file you edit. Everything is fair game: model architecture, optimizer, hyperparameters, training loop, batch size, model size, etc.

**What you CANNOT do:**
- Modify `prepare.py`. It is read-only.
- Install new packages or add dependencies.
- Modify the evaluation harness (`evaluate_bpb` in `prepare.py`).

**The goal:** get the lowest val_bpb. Lower is better.

**Simplicity criterion**: All else being equal, simpler is better. A small improvement that adds ugly complexity is not worth it. Removing something and getting equal or better results is a win.

**The first run**: Always establish the baseline first — run train.py as-is.

---

## Output format

```
---
val_bpb:          0.997900
training_seconds: 300.1
total_seconds:    325.9
peak_vram_mb:     45060.2
mfu_percent:      39.80
total_tokens_M:   499.6
num_steps:        953
num_params_M:     50.3
depth:            8
```

Extract the key metric:
```bash
grep "^val_bpb:" run.log
```

---

## Logging results

Log to `results.tsv` (tab-separated, NOT comma-separated):

```
commit	val_bpb	memory_gb	status	description
```

- commit: short hash (7 chars)
- val_bpb: e.g. 1.234567 (use 0.000000 for crashes)
- memory_gb: peak_vram_mb / 1024, rounded to .1f (use 0.0 for crashes)
- status: `keep`, `discard`, or `crash`
- description: short text

Do NOT commit results.tsv — leave it untracked.

---

## The experiment loop

**Maksimum 10 deney** yapıp dur. Her deneyin ardından kullanıcıya kısa bir durum mesajı ver.

1. Git durumuna bak (branch, commit)
2. `train.py`'de bir değişiklik yap
3. `git commit`
4. `uv run train.py > run.log 2>&1`
5. `grep "^val_bpb:\|^peak_vram_mb:" run.log`
6. Boşsa crash: `tail -n 50 run.log` oku, basit hata ise düzelt, değilse atla
7. `results.tsv`'e kaydet
8. val_bpb iyileştiyse → commit'i tut
9. Kötüleştiyse → `git reset --soft HEAD~1`

**Timeout**: 10 dakikayı aşan run'ı öldür, discard say.

**Crashes**: Basit hata (typo, import) ise düzelt ve tekrar çalıştır. Temel fikir kırıksa atla.

---

## Bitişte (10 deney sonrası)

Dur ve kullanıcıya şunu göster:
- En iyi val_bpb ve hangi değişiklik sağladı
- results.tsv tam içeriği
- Bir sonraki deney için öneri
