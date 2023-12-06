
---
tags: 📥/📽️
aliases:
  - 
cssclass:
type: video
status: 🟩
---

# Title: [[How to Safely Update Npm Packages]]

## Metadata

- `Tags:` 
- `Title:` How to Safely Update Npm Packages
- `Type:` [[+]]
- `Channel/Host:` Coding in Public
- `Reference:` 
- `Publish Date:` 
- `Creation Date:` `$= dv.current().file.ctime`
- `Last Modified Date:` `$= dv.current().file.mtime`

---
## Embedded Video

![YouTube Video](https://www.youtube.com/watch?v=0XQXGx3lLaU)

---

## Steps

### 1. Install NPM Check Updates

```bash
# Will install globally
npm install -g npm-check-updates
```

### 2. Run NPM Check Updates

```bash
# Will list packages that need updating
npx ncu
```

### 3. Update Patches

Only updates patches (e.g. `0.0.1` -> `0.0.23`)
```bash
npx ncu -u -t patch

# Install to make sure everything is still working - will update package-lock.json
npm i
```

### 4. Update Minor Versions

Updates minor versions (e.g. `0.3.0` -> `0.5.0`)
```bash
npx ncu -u -t minor

# Install to make sure everything is still working - will update package-lock.json
npm i
```

### 5. Update Major Versions

> These should be updated with caution as there may be breaking changes. Read release note docs to check how the changes will affect your project.

Updates major versions (e.g. `1.0.0` -> `2.0.0`)
```bash
# Run on individual packages by using the `-f` or `-filter` flags
npx ncu -u -f axios

# Install to make sure everything is still working - will update package-lock.json
npm i
```



🔗 Links to this page:
[[Javascript]]
[[npm]]
