# Understanding Artifacts

## 1. What Is an Artifact?

Basically, an **artifact** is a file or collection of files produced by a process and stored so that it can be used later.

In software development, artifacts can be things such as:

* Build files
* Binary files
* Compiled code
* `dist` folders
* Test reports
* Other files generated during a workflow

---

# 2. Why Do We Need Artifacts?

In a typical build and deployment process, we have:

1. **Build the project**
2. **Store the build output**
3. **Deploy the project using the stored build output**

For example:

```text
Source Code
    ↓
Build
    ↓
dist/
    ↓
Store Artifact
    ↓
Deploy
```

The important thing is that **build and deploy can be separate jobs**.

For example:

```text
Build Job
    ↓
creates dist/
    ↓
Upload Artifact
    ↓
Deploy Job
    ↓
Download Artifact
    ↓
Deploy
```

---

# 3. Why Do We Need to Store the Artifact?

Each GitHub Actions job runs on its own runner environment.

For example:

```text
build job
    ↓
Runner A
    ↓
creates dist/


deploy job
    ↓
Runner B
```

The `dist/` folder created inside **Runner A** does not automatically exist inside **Runner B**.

Therefore, we need to store the build output as an artifact:

```text
Runner A
   │
   ├── build project
   │
   ├── dist/
   │
   └── upload artifact
            │
            ↓
     GitHub Artifact Storage
            │
            ↓
Runner B
   │
   └── download artifact
```

---

# 4. Uploading an Artifact

GitHub Actions provides an action called:

```yaml
actions/upload-artifact@v4
```

Example:

```yaml
build:
  runs-on: ubuntu-latest

  steps:
    - name: Get Code
      uses: actions/checkout@v4

    - name: Install Dependency
      working-directory: 4. Job Artifacts And Output/starting-project
      run: npm ci

    - name: Build Project
      working-directory: 4. Job Artifacts And Output/starting-project
      run: npm run build

    - name: Upload Artifacts
      uses: actions/upload-artifact@v4
      with:
        name: dist-files
        path: |
          4. Job Artifacts And Output/starting-project/dist
```

The important part is:

```yaml
with:
  name: dist-files
  path: |
    4. Job Artifacts And Output/starting-project/dist
```

### `name`

```yaml
name: dist-files
```

This is the **unique identifier/name of the artifact**.

We will use this name later when downloading the artifact.

### `path`

```yaml
path: |
  4. Job Artifacts And Output/starting-project/dist
```

This specifies **which file or directory should be uploaded and stored as the artifact**.

---

# 5. Downloading an Artifact

After the artifact has been uploaded, another job can retrieve it using:

```yaml
actions/download-artifact@v4
```

Example:

```yaml
deploy:
  needs: build
  runs-on: ubuntu-latest

  steps:
    - name: Get Build Artifacts
      uses: actions/download-artifact@v4
      with:
        name: dist-files

    - name: List Files
      run: ls -a
```

The important part is:

```yaml
with:
  name: dist-files
```

This tells GitHub Actions:

> Download the artifact whose name is `dist-files`.

The name must correspond to the name used during upload:

```yaml
# Upload
name: dist-files
```

and:

```yaml
# Download
name: dist-files
```

---

# 6. Complete Build → Artifact → Deploy Flow

The overall process is:

```text
                    GitHub Repository
                           │
                           ↓
                       Build Job
                           │
                    npm run build
                           │
                           ↓
                         dist/
                           │
                           ↓
                  upload-artifact@v4
                           │
                           ↓
                GitHub Artifact Storage
                           │
                           ↓
                 download-artifact@v4
                           │
                           ↓
                     Deploy Job
                           │
                           ↓
                       Deploy
```

In short:

```text
Build
  ↓
Create build output
  ↓
Upload Artifact
  ↓
Store Artifact
  ↓
Download Artifact
  ↓
Deploy
```

---

# 7. Important Concept

An artifact is essentially a way to **persist files between jobs or make generated workflow files available for later use**.

Without artifacts:

```text
Build Job
  ↓
Runner A
  ↓
dist/

        ❌ Runner A is gone


Deploy Job
  ↓
Runner B
  ↓
No dist/
```

With artifacts:

```text
Build Job
  ↓
Runner A
  ↓
dist/
  ↓
Upload Artifact
  ↓
GitHub Artifact Storage
  ↓
Download Artifact
  ↓
Deploy Job
  ↓
dist/
```

Therefore, artifacts are especially useful when one job **produces files that another job needs**.
