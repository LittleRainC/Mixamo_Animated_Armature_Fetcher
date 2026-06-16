# Mixamo Model Fetcher

Batch download **character models** and **animations** from [mixamo.com](https://www.mixamo.com).

**Table of Contents**

[Installation](#install-instructions) ·
[Environment](#env-file-configuration) ·
[Usage](#use-instructions) ·
[Examples](#examples) ·
[Output layout](#output-layout) ·
[Notes](#notes) ·
[References](#references)

---

## Install Instructions

1. Install [Node.js](https://nodejs.org/) (LTS recommended) and npm
2. In the project directory, run:

```bash
npm install
```

3. Create a `.env` file — see [Environment](#env-file-configuration). **Do not share your `.env`; each user needs their own Mixamo login token.**

Optional: `character_ids.txt` lists Mixamo character UUIDs you can pass to `-c`.

---

## .env File Configuration

```env
AUTHKEY="Bearer YOUR_ACCESS_TOKEN_HERE"
```

**How to get the token**

1. Go to [mixamo.com](https://www.mixamo.com/) and log in
2. Open the browser developer console
3. Run `localStorage.access_token` and copy the string (without quotes)
4. Set `AUTHKEY="Bearer <token>"` — there must be a **space** between `Bearer` and the token

The token **expires** after some time. If downloads start failing with auth errors, repeat the steps above.

---

## Use Instructions

```
Usage: app [options] <type> [misc...]

Arguments:
  type        character | animation | search [names...]

Options:
  -o, --out <directory>       output directory (default: "./results")
  -d, --dry-run               list only, do not download
  -c, --characters <ids...>   character UUID(s) for animation export
  --start <index>             catalog start index (default: 0)
  --count <count>             number of catalog entries (-1 = all, default)
  -h, --help                  help
```

Run with:

```bash
node src/app.mjs <type> [options]
```

---

## Examples

**Search character ID by name**

```bash
node src/app.mjs search Maynard
```

**Download all character FBX models**

```bash
node src/app.mjs character -o ./characters
```

**Download all animations for one character** (skeleton-only FBX + pack zips)

```bash
node src/app.mjs animation \
  -c 6f2a5541-27eb-4c2a-a3f5-7fd1a54f5081 \
  -o ./animations
```

Output goes to `./animations/06f/` (see [Output layout](#output-layout)).

**Batch / resume: first 100 catalog entries**

```bash
node src/app.mjs animation \
  -c 6f2a5541-27eb-4c2a-a3f5-7fd1a54f5081 \
  -o ./animations \
  --start 0 \
  --count 100
```

**Re-download a single pack or motion by index** (verify name in the log)

```bash
node src/app.mjs animation \
  -c 6f2a5541-27eb-4c2a-a3f5-7fd1a54f5081 \
  -o ./animations \
  --start 11 \
  --count 1
```

Check `[STEP] exporting <name>` in the terminal — catalog order can change over time, so index numbers are not permanent IDs.

**Multiple characters**

```bash
node src/app.mjs animation -c <uuid-1> <uuid-2> -o ./animations
```

**Search + download in one line (bash/zsh)**

```bash
node src/app.mjs animation -c $(node src/app.mjs search Maynard Alex) -o ./animations
```

---

## Output layout

When `-c` is used, files are placed in a subfolder to avoid thousands of files in one directory:

```
<out>/
└── 06f/                          ← "0" + first 2 chars of character UUID
    ├── Idle.fbx                  ← single motion (~0.4–2 MB)
    ├── Walking.fbx
    └── Action_Adventure_Pack.zip ← motion pack (~100 MB, many FBX inside)
```

Without `-c`, animations use the default Mixamo character and write directly to `-o`.

---

## Notes

### Skeleton-only animations (`skin=false`)

This build exports animations with **`skin=false` by default**:

| Type | File | Typical size |
|------|------|----------------|
| Single motion | `.fbx` | ~0.4–2 MB (armature + animation only) |
| Motion pack | `.zip` | ~50–100 MB (many FBX inside) |

This is intended for retargeting / skeleton animation libraries, not mesh-included downloads.

### Catalog size

The animation catalog is roughly **2500 entries** (motions + packs). One pack counts as one entry but contains many animations inside the zip.

### Download retries

Large pack downloads use retries with progress logs (`[STATUS] downloading XX MB`). If a pack fails with `HTTP_ERROR` or timeout, retry that index with `--start N --count 1`. Do not interrupt pack downloads mid-way if possible.

### Failed entries

Failed motions are logged as `[WARNING] motion N failed (...)` and the script continues. Re-run failed ranges with `--start` / `--count`.

---

## References

- [mixamo_anims_downloader](https://github.com/gnuton/mixamo_anims_downloader) — `getGmsHash` implementation basis
- [mixamo_model_fetcher](https://github.com/Sapiens-wx/mixamo_model_fetcher) - by [Sapiens-wx](https://github.com/Sapiens-wx)
