# Issues found in `pet_shelter.py`

Before: Pylint 3.95/10, Bandit 3 issues (1 Medium, 2 Low), Flake8 5 issues.
After: Pylint 10.00/10, Bandit 0 issues, Flake8 0 issues.
Reports are in `reports/before/` and `reports/after/`.

| Issue | Type | Line(s) | Description | Fix Approach |
| ----- | ---- | ------- | ----------- | ------------ |
| SQL injection (B608) | Security (Medium) | 10 | `registerPet` built the INSERT with `%` string formatting, so a name like `x'); DROP TABLE pets;--` is interpreted as SQL. | Use `?` placeholders and pass values as a tuple, as `adopt` already did. (Bandit commit) |
| Weak random for codes (B311) | Security (Low) | 22 | `random.randint` is predictable, so adoption codes can be guessed. | Use `secrets.randbelow(9000) + 1000`. (Bandit commit) |
| Try/except/pass (B110) and broad `except Exception` (W0718) | Security (Low) / Bug | 34-37 | `clearUnadopted` swallowed every error silently, so a failed delete looked like success. | Catch only `sqlite3.Error` and log it with `logger.exception`. (Bandit commit) |
| Bare `except` (E722, W0702) | Bug | 26-30 | `oldestAvailable` caught everything, including `KeyboardInterrupt`, mainly to hide `row[0]` failing on an empty result. | Remove the try block and check `row is None` explicitly. (Flake8 commit) |
| Lines too long (E501, C0301) | Style | 5, 10, 27 | Three lines exceeded 79 characters. | Split SQL strings and calls across lines. (Flake8 commit) |
| Multiple statements on one line (E701, C0321) | Style | 41 | `if ...: return "active"` on one line. | Put the return on its own line. (Flake8 commit) |
| Mutable default argument (W0102) | Bug | 9 | `tags=[]` is created once and shared, so tags leaked between calls. | Default to `None` and create a new list inside the function. (Pylint commit) |
| Global statement (W0603) and constant naming (C0103) | Design | 6, 16 | `adoption_count` was module-level mutable state changed through `global`. It also counted adopting ids that do not exist. | Remove the counter. `shelter_status` now counts adopted rows in the database. (Pylint commit) |
| Non-snake_case function names (C0103) | Style | 9, 21, 25, 33, 40 | `registerPet`, `adoptionCode`, `oldestAvailable`, `clearUnadopted`, `shelterStatus`. | Renamed to `register_pet`, `adoption_code`, `oldest_available`, `clear_unadopted`, `shelter_status`. (Pylint commit) |
| Missing docstrings (C0114, C0116) | Documentation | 1, 9, 15, 21, 25, 33, 40, 45 | Module and all functions lacked docstrings. | Added a docstring to each. (Pylint commit) |
| Old-style string formatting (C0209) | Style | 10 | `%` formatting instead of an f-string. | Gone with the parameterized query fix. (Bandit commit) |
| `main()` runs on import | Design | 54 | Importing the module ran the demo and created output. | Wrapped in `if __name__ == "__main__":`. (Pylint commit) |

## Behaviour check

`python pet_shelter.py` prints `Kiwi PET-xxxx active` before and after. The code number varies on every run.
