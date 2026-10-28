# Reflection – Lab 5: Static Code Analysis

**Name:** Aditya Alur  **SRN:** PES2UG24CS052

**1. Which issues were the easiest to fix, and which were the hardest?**
The style issues were the easiest: renaming functions, adding docstrings, removing unused imports/variables, wrapping long lines and adding a final newline are mechanical, and Pylint/Flake8 tell you the exact line. The hardest was the `eval()` call. There was no one-line substitution; I had to change the function's contract (it now takes a numeric `discount_rate` instead of an expression string), validate the range, and update the caller. Replacing `os.system` and the hard-coded secret also needed a small design decision (print instead of shell; environment variable instead of a literal).

**2. Did the tools report any false positives or issues that weren't really problems?**
Mostly no, but a few needed judgement. B311 (`random` for the locker PIN) is rated Low, but a locker PIN is a credential, so it was worth fixing. B105 (hard-coded password) was "Medium confidence": the string is a portal secret, so it was a true positive, and the real problem was also that the code printed it. On the other hand Bandit rated `eval` only *Medium* even though it allows arbitrary code execution, so severity labels should not be followed blindly. Pylint's C0116 (missing docstring) and the E501 line-length checks are style preferences rather than defects. Conversely, the tools missed the fee-ignores-plan logic problem and `main()` running on import, so a clean report does not mean correct code.

**3. How would you integrate static analysis into a real development workflow?**
Run Flake8 and Pylint as a pre-commit hook so style and obvious bugs never reach the repo, and run Pylint, Flake8 and Bandit in CI (e.g. GitHub Actions) on every pull request, failing the build on any High/Medium Bandit finding or a Pylint score below a threshold such as 9.5. Config files (`.pylintrc`, `setup.cfg`, `bandit.yaml`) would hold agreed rules and any justified suppressions (`# nosec`, `# pylint: disable`) with a comment explaining why. Reports would be kept as CI artifacts so trends are visible over time.

**4. What tangible improvements did I see in code quality, security and readability?**
Pylint went from 4.81/10 to 10.00/10, Bandit from 6 findings (2 High, 1 Medium, 3 Low) to none, and Flake8 from 9 warnings to none, while the program output (e.g. renewal fee 510.0, active count 3) stayed the same. Security improved the most: no code execution through `eval`/`os.system`, a modern hash, a cryptographically secure PIN, and no secret in the source. Reliability improved by fixing the shared mutable default, the silent bare `except`, and the unclosed-file risk. Readability improved through consistent snake_case names, docstrings, f-strings, and simpler boolean returns, and the module can now be imported without side effects.
