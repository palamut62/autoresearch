# autoresearch — Minimal Program

## Setup

1. Run etiketi belirle (örn: `apr7`). Branch oluştur: `git checkout -b autoresearch/apr7`
2. Şu dosyaları oku: `prepare.py`, `train.py`
3. `~/.cache/autoresearch/` klasöründe veri var mı kontrol et. Yoksa: `uv run prepare.py` çalıştır.
4. `results.tsv` oluştur (sadece header).
5. Onayla ve başla.

---

## Kurallar

- Sadece `train.py` düzenlenir. `prepare.py` dokunulmaz.
- Hedef: val_bpb'yi düşür (düşük = iyi).
- **Maksimum 5 deney.** Bitince dur, kullanıcıya özet sun.
- Her deneyde mutlaka kullanıcıya sor: "Devam edeyim mi?"

---

## Deney Sırası

1. **Baseline** — train.py olduğu gibi çalıştır
2. **Learning Rate** — LR'yi %20 artır
3. **Batch Size** — TOTAL_BATCH_SIZE'ı 2 katına çıkar
4. **Depth** — DEPTH'i 6'ya düşür
5. **En iyi kombinasyon** — 2-4 arası en iyi ikisini birleştir

---

## Her Deney İçin

```bash
git commit -m "deney: açıklama"
uv run train.py > run.log 2>&1
grep "^val_bpb:\|^peak_vram_mb:" run.log
```

- İyileştiyse: tut
- Kötüleştiyse: `git reset --soft HEAD~1`
- Crash: `tail -n 30 run.log` oku, basit hata ise düzelt, değilse atla

---

## Logging

`results.tsv` (tab-separated):
```
commit	val_bpb	memory_gb	status	description
```

---

## Bitişte

5 deney tamamlanınca dur. Kullanıcıya göster:
- En iyi val_bpb ve hangi değişiklik
- results.tsv özeti
