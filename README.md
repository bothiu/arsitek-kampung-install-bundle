# Arsitek Kampung — Install Bundle

Companion repo untuk distribusi **single-file** skill `arsitek-kampung`.

> File di repo ini di-generate otomatis dari repo sumber utama.

Tujuan repo ini: mengatasi installer yang saat ini hanya menyalin `SKILL.md`, sehingga file pendukung seperti `references/` dan `examples/` tidak ikut kebawa. Di bundle ini, konten pendukung sudah **di-embed langsung** ke dalam `SKILL.md`.

## Canonical source

Repo sumber utama:
- https://github.com/bothiu/arsitek-kampung

## Install ke Hermes

### Raw URL
```bash
hermes skills install https://raw.githubusercontent.com/bothiu/arsitek-kampung-install-bundle/main/SKILL.md
```

### npx skills
```bash
npx -y skills add bothiu/arsitek-kampung-install-bundle --skill arsitek-kampung --agent hermes-agent --global --yes
```

## Regenerate bundle

Dari repo source:
```bash
python3 scripts/generate_bundle.py
```

Default output target:
- `/home/runner/work/arsitek-kampung/arsitek-kampung-install-bundle`

## Catatan

- Nama skill tetap `arsitek-kampung`.
- Repo ini adalah **bundle distribution**, bukan source of truth.
- Kalau skill dengan nama yang sama sudah terpasang, installer mungkin butuh `--force` untuk replace install lama.
