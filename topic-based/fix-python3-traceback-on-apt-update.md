# 🐛 Fix: `sudo apt update` Python Traceback on Ubuntu

## Problem

Running `sudo apt update` throws a Python traceback related to `command-not-found`:

```
Current thread 0x0000756241cfa080 (most recent call first):
  File "/usr/lib/python3/dist-packages/CommandNotFound/db/creator.py", line 201 in _parse_single_commands_file
  File "/usr/lib/python3/dist-packages/CommandNotFound/db/creator.py", line 143 in _fill_commands
  File "/usr/lib/python3/dist-packages/CommandNotFound/db/creator.py", line 96 in create
  File "/usr/lib/cnf-update-db", line 32 in <module>

Extension modules: apt_pkg (total: 1)
```

## Cause

On Ubuntu, `/usr/bin/python3` is expected to point directly to the installed Python binary (e.g. `/usr/bin/python3.10`), but in this case it was incorrectly symlinked to `/etc/alternatives/python3`.

This non-standard setup breaks Python scripts that rely on the correct interpreter.

## Solution

### 1. Check the current symlink

```bash
ls -l /usr/bin/python3
```

If you see:

```
/usr/bin/python3 -> /etc/alternatives/python3
```

… then proceed to fix it.

---

### 2. Find the actual Python 3 binary

```bash
ls /usr/bin/python3*
```

Look for something like `/usr/bin/python3.10` or `/usr/bin/python3.11`.

---

### 3. Update the `python3` symlink

Replace `python3.10` with the correct version you found:

```bash
sudo ln -sf /usr/bin/python3.10 /usr/bin/python3
```

---

### 4. (Optional) Remove the broken `alternatives` symlink

```bash
sudo rm /etc/alternatives/python3
```

---

### 5. Retry the update command

```bash
sudo apt update
```

The traceback should now be resolved.

---

## Optional: Reinstall command-not-found (if needed)

```bash
sudo apt install --reinstall command-not-found command-not-found-data python3-apt
```

To test the database manually:

```bash
sudo /usr/lib/cnf-update-db
```

---

## Notes

- Ubuntu **does not** manage `python3` via `update-alternatives` by default.
- For managing multiple Python versions, consider using [`pyenv`](https://github.com/pyenv/pyenv).
