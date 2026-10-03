# Validation Rules

Apply these checks to every file you create or modify. Choose tools appropriate to the file type; do not assume every file is plain text.

1. **Confirm scope and location.** Verify the file belongs at the intended path, uses the expected format, and contains only changes needed for the task.
2. **Check structural validity.** Parse or validate the file with a format-aware tool, such as a compiler, linter, schema validator, or Markdown checker. For binary files, use a tool that understands the format.
3. **Verify required content and constraints.** Check required fields, sections, keys, types, ranges, naming, and format-specific conventions against the applicable specification.
4. **Resolve references and dependencies.** Confirm referenced files, links, imports, assets, configuration keys, and external identifiers exist and agree with their consumers.
5. **Review security and sensitive data.** Look for credentials, tokens, private information, unsafe defaults, unintended permissions, and risky executable content; remove or protect anything inappropriate.
6. **Run focused verification and inspect the result.** Execute the narrowest relevant test, build, rendering, or round-trip check, then review the diff to catch unintended changes or generated artifacts.