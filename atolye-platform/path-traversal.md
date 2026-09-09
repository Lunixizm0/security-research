# Path Traversal in Upload Endpoint

## Overview

A path traversal vulnerability was identified in the upload functionality of Atolye.Platform.

An authenticated user could manipulate the `folderPath` parameter of the upload endpoint to write uploaded files outside of the intended uploads directory.

Additional path traversal protection was also missing from the file viewing and download endpoints.

## Affected Endpoints

| Endpoint | Method | Issue |
|---|---|---|
| `/api/uploads` | `POST` | Path traversal via `folderPath` |
| `/api/uploads/view/*` | `GET` | Missing path traversal validation |
| `/api/uploads/download/*` | `GET` | Missing path traversal validation |

The `/list/*` and `DELETE /*` endpoints already performed path boundary validation.

## Root Cause

The upload endpoint constructed filesystem paths using user-controlled input:

```js
const targetFolder = join(uploadsBase, folderPath);

if (!fs.existsSync(targetFolder)) {
  fs.mkdirSync(targetFolder, { recursive: true });
}

const targetPath = join(targetFolder, finalName);
fs.renameSync(req.file.path, targetPath);
````

The supplied `folderPath` was not validated against the intended upload directory before being used for filesystem operations.

The view and download endpoints had a similar issue by using `join()` without verifying that the resulting path remained inside `uploadsBase`.

## Proof of Concept

An authenticated request using a traversal sequence in `folderPath` could cause the uploaded file to be written outside the intended directory.

Example:

```text
folderPath=students/../../../etc
```

The vulnerable implementation resolved this path outside the intended upload directory.

The issue also affected the file view endpoint because its path handling did not enforce an upload-directory boundary.

## Impact

Successful exploitation could allow an authenticated attacker to:

* Write files to locations writable by the application process.
* Create directories outside the intended upload directory.
* Potentially overwrite or influence application files depending on filesystem permissions.
* Potentially achieve further impact, including code execution, if a writable location could be used to modify executable application/configuration files.

The actual impact depends on the privileges and filesystem layout of the backend process.

## Disclosure Timeline

* **2026-07-28** - Vulnerability reported
* **2026-08-18** - Issue closed as completed

## References

* [Original GitHub Issue](https://github.com/PolyOS-Team/Atolye.Platform/issues/20)

## Disclosure

This writeup documents a vulnerability that has been publicly disclosed.
