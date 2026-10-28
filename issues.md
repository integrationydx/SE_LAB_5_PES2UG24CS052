# Issues Found – `gym_membership_system.py`

**Name:** Aditya Alur  **SRN:** PES2UG24CS052

Line numbers refer to the **original** file. Ordered by severity (High → Low).
Tool scores: Pylint 4.81 → **10.00**, Bandit 6 findings → **0**, Flake8 9 findings → **0**.

| # | Issue | Type | Line(s) | Description | Fix Approach |
|---|-------|------|---------|-------------|--------------|
| 1 | Shell command built from a string (B605) | Security – **High** | 58 | `os.system("echo ... " + filename)` passes a variable into a shell, so a filename like `x; rm -rf ~` would run arbitrary commands. | Removed the shell call and used `print("Roster exported to:", filename)`. |
| 2 | Weak MD5 hash (B324) | Security – **High** | 47 | `hashlib.md5` is broken/collision-prone and should not be used for identifiers or anything security related. | Switched to `hashlib.sha256`. |
| 3 | Use of `eval()` on caller input (B307 / W0123) | Security – **Medium** (arbitrary code execution) | 40-43 | `eval(discount_expr)` executes whatever string is passed, e.g. `"__import__('os').system(...)"`. The `base_fee` variable was also only used "by magic" inside the string (Pylint W0612). | Removed `eval`. `calculate_renewal_rate(m_id, discount_rate)` now takes a number (0-1), validates it, and returns `base_fee * (1 - discount_rate)`. |
| 4 | Mutable default argument (W0102) | Bug | 15 | `member_log=[]` is created once and shared across every call, so the log grows across unrelated registrations. | Default is `None`; a fresh list is created inside the function. |
| 5 | Bare `except` + `pass` (B110 / E722 / W0702) | Bug / Low | 23-27 | Catches *everything* (even `KeyboardInterrupt`) and silently hides failures; the caller can't tell whether the cancel worked. | Replaced with `members.pop(m_id, None) is not None`, which returns `True`/`False` explicitly. |
| 6 | Hard-coded secret, then printed (B105) | Security – Low (real risk) | 70, 79 | The portal secret sits in source code (ends up in Git) and is also printed to the console. | Read from env var `GYM_API_SECRET`; output now only says whether it is configured. |
| 7 | Non-cryptographic RNG for a PIN (B311) | Security – Low | 50-51 | `random.randint` is predictable; locker PINs are access credentials. | Used `secrets.randbelow(9000) + 1000` (still a 4-digit PIN). |
| 8 | File opened without `with` / encoding (R1732, W1514) | Bug (resource leak) | 55-57 | If `write()` raises, `close()` is never reached; encoding depends on the OS. | `with open(filename, "w", encoding="utf-8") as roster_file:`. |
| 9 | Function names not snake_case (C0103) | Style | 15, 23, 30, 40 | `registerMember`, `cancelMembership`, `checkMembershipStatus`, `calculateRenewalRate` violate PEP 8. | Renamed to `register_member`, `cancel_membership`, `check_membership_status`, `calculate_renewal_rate`; updated callers. |
| 10 | `== None` comparison (E711 / C0121) | Style | 32 | Identity should be tested with `is`. | Rewrote as `member is not None and member["months_left"] > 0`. |
| 11 | Redundant if/else returning booleans (R1703, R1705) | Refactor | 34-37 | Four lines to return the result of a comparison. | Collapsed into one boolean `return` expression (combined with #10). |
| 12 | Unused variables / import (W0612, F841, W0611, F401) | Dead code | 6, 31, 41, 63 | `math` import, `locker_deposit`, and loop variable `m` are never used. | Deleted them; `get_active_summary` now uses `sum(... for details in roster.values() ...)`. |
| 13 | Parameter shadows global (W0621) | Style / bug risk | 61 | `get_active_summary(members)` hides the module-level `members`. | Parameter renamed to `roster`. |
| 14 | `%` string formatting (C0209) | Style | 19 | Old-style formatting. | Converted to an f-string. |
| 15 | Missing docstrings (C0116) | Documentation | 15-69 | Every function lacked a docstring. | Added a one-line docstring to every function. |
| 16 | Lines > 79 chars (E501) | Style (PEP 8) | 10, 11, 18 | Long dict literals. | Wrapped the dict literals across multiple lines. |
| 17 | No newline at EOF (W292 / C0304) | Style | 82 | File ended without a newline. | Added trailing newline. |
| 18 | `main()` runs on import (not flagged by tools) | Design | 82 | Importing the module executes the demo (and writes a file). | Wrapped in `if __name__ == "__main__":`. |

## Observation not fixed (not tool-flagged)

* **Fee ignores the plan** (line 18): `registerMember` always charges `50.0 * months`, whatever the `plan` is, yet the seed data uses different rates per tier (Gold 100/month, Platinum ≈83/month). Correct pricing is a business decision, so I left the behaviour unchanged and am flagging it here.
