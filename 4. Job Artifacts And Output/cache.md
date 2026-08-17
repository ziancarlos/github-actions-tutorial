# Cache

## 1. What Is Cache?

**Cache** means remembering or storing something so that it does not have to be downloaded or generated again and again.

In GitHub Actions, we can use caching to speed up workflows by reusing files that were previously downloaded or generated.

---

# 2. Caching Dependencies

Installing npm dependencies can take time and consume resources.

For example:

```bash
npm ci
```

may need to download many packages.

Therefore, it is a good practice to **cache npm's package cache** so that dependencies can be reused across jobs or workflow runs.

GitHub Actions provides a cache action:

```yaml
actions/cache
```

---

# 3. Rule of Thumb

The cache step should generally be placed **before installing dependencies**.

Example:

```text
Checkout Code
      ↓
Restore Cache
      ↓
Install Dependencies
      ↓
Run Tests
```

The reason is that `npm ci` can reuse packages available in the npm cache.

---

# 4. Cache Syntax

Example:

```yaml
- name: Cache Dependencies
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
```

There are three important parts:

```yaml
uses: actions/cache@v3
path: ~/.npm
key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
```

---

# 5. `uses: actions/cache`

```yaml
uses: actions/cache@v3
```

This tells GitHub Actions to use the **cache action**.

For current workflows, `v4` should be preferred:

```yaml
uses: actions/cache@v4
```

---

# 6. `path`

```yaml
path: ~/.npm
```

`path` specifies **which files or directories should be cached**.

For npm:

```text
~/.npm
```

is npm's local package cache directory.

It contains packages that npm has previously downloaded.

It is important to distinguish this from:

```text
node_modules/
```

The cache in this example is:

```text
~/.npm
```

not:

```text
node_modules/
```

So the general flow is:

```text
~/.npm
   ↓
npm ci
   ↓
node_modules/
```

npm can reuse packages from its cache when installing dependencies.

---

# 7. `key`

```yaml
key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
```

The `key` is the **identifier for the cache**.

Think of it as the name/version of a particular cache.

For example, it might result in something like:

```text
deps-node-modules-a8f31c...
```

The:

```text
deps-node-modules-
```

part is simply a descriptive string chosen by us.

It does not have any special meaning to GitHub Actions.

You could technically use:

```yaml
key: npm-cache-${{ hashFiles('**/package-lock.json') }}
```

as well.

---

# 8. `hashFiles()`

This part:

```yaml
${{ hashFiles('**/package-lock.json') }}
```

creates a **hash based on the contents of the matching file(s)**.

For example:

```text
package-lock.json
       ↓
   hashFiles()
       ↓
ABC123...
```

Then the cache key becomes:

```text
deps-node-modules-ABC123...
```

---

# 9. Why Use `package-lock.json`?

The `package-lock.json` contains information about the project's dependency tree and resolved versions.

If the dependencies change, the `package-lock.json` will normally change as well.

For example:

```text
Before:

package-lock.json
      ↓
hash = ABC123
      ↓
cache key = deps-node-modules-ABC123
```

After adding a dependency:

```text
package-lock.json
      ↓
hash = XYZ789
      ↓
cache key = deps-node-modules-XYZ789
```

Because the hash changed, the cache key also changes.

This allows GitHub Actions to distinguish between different dependency states.

---

# 10. What Does `**/package-lock.json` Mean?

```yaml
hashFiles('**/package-lock.json')
```

The:

```text
**/
```

means that GitHub can look for `package-lock.json` files in the repository and its subdirectories.

For example:

```text
repository/
├── package-lock.json
├── frontend/
│   └── package-lock.json
└── backend/
    └── package-lock.json
```

The pattern:

```text
**/package-lock.json
```

can match these files.

This is useful when the project is not located directly in the repository root.

---

# 11. Cache in the Test Job

Example:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Get Code
        uses: actions/checkout@v3

      - name: Cache Dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}

      - name: Install Dependency
        working-directory: 4. Job Artifacts And Output/starting-project
        run: npm ci

      - name: Test Code
        working-directory: 4. Job Artifacts And Output/starting-project
        run: npm run test
```

The flow is:

```text
Checkout
   ↓
Restore npm cache
   ↓
npm ci
   ↓
Run tests
```

---

# 12. Cache in the Build Job

The build job can use the same cache strategy:

```yaml
build:
  needs: [test]
  runs-on: ubuntu-latest

  outputs:
    script-file: ${{ steps.publish.outputs.script-file }}

  steps:
    - name: Get Code
      uses: actions/checkout@v3

    - name: Cache Dependencies
      uses: actions/cache@v3
      with:
        path: ~/.npm
        key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}

    - name: Install Dependency
      working-directory: 4. Job Artifacts And Output/starting-project
      run: npm ci

    - name: Build Code
      working-directory: 4. Job Artifacts And Output/starting-project
      run: npm run build

    - name: Publish Javascript File Name
      id: publish
      working-directory: 4. Job Artifacts And Output/starting-project
      run: find dist/assets/*js -type f -execdir echo 'script-file={}' >> $GITHUB_OUTPUT ';'

    - name: Upload Artifacts
      uses: actions/upload-artifact@v4
      with:
        name: dist-files
        path: |
          4. Job Artifacts And Output/starting-project/dist
```

---

# 13. Cache vs. Artifact

Do not confuse **cache** with **artifact**.

### Cache

Used to speed up future work by reusing files.

```text
Cache
  ↓
Reuse files
  ↓
Speed up workflow
```

Example:

```text
~/.npm
```

### Artifact

Used to store files produced by a job so that they can be used later.

```text
Build
  ↓
dist/
  ↓
Upload Artifact
  ↓
Deploy
```

Example:

```text
dist/
```

So:

```text
Cache
→ "I don't want to download/generate this again."

Artifact
→ "I need to preserve this file so another job can use it."
```

---

# 14. Complete Mental Model

Your workflow can be understood like this:

```text
                         PUSH
                           │
                           ↓
                         TEST
                           │
                    ┌──────┴──────┐
                    ↓             ↓
              Restore Cache    npm ci
                    │             │
                    └──────┬──────┘
                           ↓
                        npm test
                           │
                           ↓
                         BUILD
                           │
                    Restore Cache
                           ↓
                         npm ci
                           ↓
                       npm build
                           │
                           ↓
                         dist/
                           │
                           ↓
                    Upload Artifact
                           │
                           ↓
                        DEPLOY
                           │
                           ↓
                    Download Artifact
```

The key idea to remember:

> **`path` tells GitHub what to cache, while `key` tells GitHub which cache to use. `hashFiles('**/package-lock.json')` makes the cache key change when the dependency lockfile changes.**
