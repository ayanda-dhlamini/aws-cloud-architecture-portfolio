# Per-Account Malware Cleanup and Verification Procedure

Generalised from a real remediation. Indicators and names have been removed. Run read-only checks first, act only on findings, and re-verify to zero.

## Decide: clean or rebuild

| Condition | Route |
|---|---|
| Only a PHP loader and redirect injector files, structure otherwise intact | **Clean in place** |
| Injected code inside core or plugin PHP (including `wp-config.php`) | **Rebuild** |
| Hidden toolkit or webshell files in nested directories inside WordPress core | **Rebuild** |
| Many fake directories with `index.php` payloads under core and content | **Rebuild** |

Never try to disinfect a deep infection. Replace the code from official sources.

## Six-step routine

1. **Loader by hash and pattern.** Find PHP files matching known hashes, then search by pattern too (self-decoding `gzuncompress`, `eval`, `file_get_contents(__FILE__)`), because variants change the hash.
2. **Redirect injector by content.** Search HTML files for the known redirect domains. Confirm by content, not by file size, because legitimate placeholder pages can match the size.
3. **`.htaccess` review.** Flag any file that puts `index.html` before `index.php` in `DirectoryIndex`, or that rewrites all requests or error pages to `/index.html`. Replace with the standard WordPress block. Scanners may miss this.
4. **Ownership and structure.** Find PHP files owned by `root` under user directories. Look for `index.php` files inside fake directories under `wp-includes` and `wp-content`.
5. **Core integrity.** Check the installed WordPress version, then diff `wp-admin` and `wp-includes` against the official release **of that exact version**. A version mismatch produces thousands of false differences. A clean result has no modified core files and no foreign files.
6. **Re-verify.** Re-run steps 1–5. All must come back empty before marking the account clean.

## Beyond the web root
Sweep the whole home directory, not just `public_html`: mail, stats, temp and `.well-known` directories can hold injector copies.

## Go-live gate
An account goes live only when all four are done:
1. Cleaned and verified
2. Credentials rotated (WordPress, database, cPanel, SSH, FTP, email)
3. Mail routing set to local
4. DNS TTL lowered, then cut over

## Notes
- Quarantine before deleting, and keep evidence of what was removed.
- Take a clean backup after remediation.
- Update plugins and themes after cleanup, since old versions are a likely way in.
