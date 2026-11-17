# Research Methodology and Preservation Rules

## Operating principles
1. Prefer read-only inspection.
2. Copy artifacts to the Mac before analysis.
3. Preserve original artifacts and record hashes.
4. Keep raw command output alongside interpretations.
5. Label statements as confirmed, preliminary, hypothesized, or unknown.
6. Reproduce findings with exact commands and tool versions where possible.
7. Avoid unnecessary device-side changes.

## Device preservation requirements

The existing iOS 9.3.5 installation must be preserved.

Do not restore, update, erase, or overwrite the iPad as an exploratory step. Do not modify NVRAM, boot arguments, bootloader components, kernelcache files, or system files without first explaining the consequences and obtaining explicit approval.

No experimental boot has been started.

## Command presentation

Commands must be labeled by execution location:
- **Mac:** commands run in macOS Terminal.
- **iPad:** commands run over SSH on the iPad.

Explain each command. Prefer batches that are efficient but manageable. Analyze actual output rather than assuming success.

## Evidence and confidence labels
- **Confirmed:** Directly supported by command output, an artifact, or an authoritative source.
- **Preliminary:** Strong indication, but structure or interpretation is not fully validated.
- **Hypothesis:** Plausible explanation requiring a test.
- **Unknown:** No sufficient evidence yet.
- **Not started:** No experiment has been performed.

Keep exact output and note date and context. If a finding changes, preserve the earlier conclusion in the change history rather than silently rewriting the record.

## Firmware and reverse-engineering work

For kernelcache, IMG3, Mach-O, device-tree, and firmware analysis:
- Work on local copies.
- Never overwrite the original downloaded kernelcache.
- Record SHA-256 hashes before and after transformations.
- Verify container lengths and tag boundaries before extraction.
- Identify payload compression from evidence, not guesswork.
- Verify extracted file type before parsing it.
- Treat a kernel from a different device as a comparison artifact, not a compatible replacement.
- Separate static analysis from any future boot experiment.

## Future automation requirements

A future app or workflow may maintain this repository when new discoveries occur. It should:
- Update the relevant topic file and root research.md.
- Add a dated entry to a changelog or research log.
- Preserve command output and artifact hashes.
- Track source/provenance for claims.
- Distinguish verified facts from hypotheses.
- Produce reviewable diffs before committing.
- Never perform device-side actions or destructive operations automatically.

**No automatic documentation updater is currently configured.**
