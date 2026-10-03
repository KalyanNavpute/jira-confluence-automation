# Module 15 Completion Report

## Script Metadata
- Filename: `bulk_validate_walkthroughs.py`
- Language: Python
- Purpose: Finds Markdown walkthroughs and checks that each has exact `Summary` and `Quiz` headings. By default it searches `modules/**/walkthrough.md`; it can also validate explicitly supplied Markdown files. It reports issues without modifying files.

## Script Contents
```python
#!/usr/bin/env python3
import argparse
import re
import sys
from pathlib import Path


REQUIRED_SECTIONS = ("Summary", "Quiz")
SECTION_PATTERN = re.compile(r"^#{1,6}\s+(Summary|Quiz)\s*#*\s*$", re.IGNORECASE | re.MULTILINE)


def validate_file(path):
    if path.suffix.lower() != ".md":
        return ["expected a Markdown (.md) file"]
    if not path.is_file():
        return ["file does not exist"]

    try:
        contents = path.read_text(encoding="utf-8")
    except (OSError, UnicodeError) as error:
        return [f"could not read file: {error}"]

    found_sections = {match.group(1).lower() for match in SECTION_PATTERN.finditer(contents)}
    return [
        f"missing {section} section"
        for section in REQUIRED_SECTIONS
        if section.lower() not in found_sections
    ]


def collect_files(args):
    if args.files:
        return args.files

    root = args.root
    if not root.is_dir():
        return []
    return sorted(root.glob(args.pattern))


def main(argv=None):
    parser = argparse.ArgumentParser(
        description="Check Markdown walkthroughs for required Summary and Quiz sections."
    )
    parser.add_argument(
        "--root",
        type=Path,
        default=Path(__file__).resolve().parent / "modules",
        help="Directory to search when no file paths are provided.",
    )
    parser.add_argument(
        "--pattern",
        default="**/walkthrough.md",
        help="Glob pattern relative to --root (default: **/walkthrough.md).",
    )
    parser.add_argument(
        "files",
        nargs="*",
        type=Path,
        help="Optional explicit Markdown files to validate instead of discovering files.",
    )
    args = parser.parse_args(argv)
    files = collect_files(args)

    if not files:
        print(f"No files found for pattern '{args.pattern}' under '{args.root}'.")
        return 1

    passed = 0
    failed = 0
    for path in files:
        issues = validate_file(path)
        if issues:
            failed += 1
            print(f"FAIL {path}: {'; '.join(issues)}")
        else:
            passed += 1
            print(f"PASS {path}")

    print(f"Summary: {len(files)} file(s), {passed} passed, {failed} failed.")
    return 1 if failed else 0


if __name__ == "__main__":
    sys.exit(main())
```

## Parameters
| Parameter | Description | Default |
|-----------|-------------|---------|
| `--root PATH` | Directory to search when no explicit files are provided. | `modules/` next to the script |
| `--pattern GLOB` | Glob pattern searched relative to `--root`. | `**/walkthrough.md` |
| `files...` | Optional explicit Markdown file paths; when supplied, discovery is skipped. | No paths; discover files under `--root` |

## Test Run Output
Command:
```text
python3 bulk_validate_walkthroughs.py tests/fixtures/walkthroughs/complete.md tests/fixtures/walkthroughs/missing-summary.md tests/fixtures/walkthroughs/missing-quiz.md
```

Output:
```text
PASS tests/fixtures/walkthroughs/complete.md
FAIL tests/fixtures/walkthroughs/missing-summary.md: missing Summary section
FAIL tests/fixtures/walkthroughs/missing-quiz.md: missing Quiz section
Summary: 3 file(s), 1 passed, 2 failed.
Exit code: 1
```