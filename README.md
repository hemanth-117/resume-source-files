# Resume CI/CD Pipeline

Automated workflow that triggers whenever changes are merged into `master`, compiles `resume_source.tex` into a PDF, and updates the latest PDF in the repository.

---

## 🔗 Live PDF Links

Once the workflow runs, the updated resume PDF is always accessible at:

- **Browser View (GitHub PDF Viewer):**  
  [https://github.com/hemanth-117/resume-source-files/blob/master/resume.pdf](https://github.com/hemanth-117/resume-source-files/blob/master/resume.pdf)

- **Direct Download (Raw PDF):**  
  [https://raw.githubusercontent.com/hemanth-117/resume-source-files/master/resume.pdf](https://raw.githubusercontent.com/hemanth-117/resume-source-files/master/resume.pdf)

---

## 📋 Requirements

Before the automation can work, the following requirements must be met:

### 1. GitHub Repository Permission (Mandatory)
GitHub Actions requires explicit write permissions to commit the compiled PDF back to your repository.
- **Where to configure:**  
  Go to your repo: **Settings** → **Actions** → **General**  
  Direct URL: [https://github.com/hemanth-117/resume-source-files/settings/actions](https://github.com/hemanth-117/resume-source-files/settings/actions)
- **Setting to change:**  
  Under **Workflow permissions**, change from *"Read repository contents permission"* to **"Read and write permissions"**.
- Click **Save**.

*(If this is not done, GitHub Actions will fail with `HTTP 403: Resource not accessible by integration`).*

### 2. Workflow Permissions (In YAML)
The workflow file requests write scope:
```yaml
permissions:
  contents: write
```

### 3. LaTeX Engine & Dependencies
- Source file: `resume_source.tex`
- Engine: `pdflatex` with standard TeX Live packages (`fontawesome5`, `titlesec`, `tabularx`, `enumitem`, `hyperref`).
- Handled automatically in CI via `xu-cheng/latex-action@v3`.

---

## ✅ What All We Should Do (Step-by-Step Checklist)

- [ ] **1. Enable GitHub Settings Permission**
  - Open [https://github.com/hemanth-117/resume-source-files/settings/actions](https://github.com/hemanth-117/resume-source-files/settings/actions)
  - Select **"Read and write permissions"** and save.

- [x] **2. Create Workflow Directory & File**
  - File created at [`.github/workflows/compile-resume.yml`](./.github/workflows/compile-resume.yml).

- [ ] **3. Push & Merge into `master`**
  - Commit changes to branch `workflow` and push to remote.
  - Merge the `workflow` branch into `master`.
  - The workflow will automatically trigger, build the PDF, and commit `resume.pdf` to the root of `master`.

- [ ] **4. Verify Output**
  - Check the **Actions** tab on GitHub to see the build finish.
  - Access your resume using the direct links above.

---

## 🛠️ Workflow File Configuration

File location: [`.github/workflows/compile-resume.yml`](./.github/workflows/compile-resume.yml)

```yaml
name: Build and Update Resume PDF

on:
  push:
    branches:
      - master
    paths-ignore:
      - 'README.md'
      - '.gitignore'
  workflow_dispatch:

permissions:
  contents: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Compile LaTeX to PDF
        uses: xu-cheng/latex-action@v3
        with:
          root_file: resume_source.tex

      - name: Rename PDF to resume.pdf
        run: |
          mv resume_source.pdf resume.pdf

      - name: Upload PDF artifact
        uses: actions/upload-artifact@v4
        with:
          name: resume-pdf
          path: resume.pdf

      - name: Commit and push updated PDF
        run: |
          git config --local user.name "github-actions[bot]"
          git config --local user.email "github-actions[bot]@users.noreply.github.com"
          git add -f resume.pdf
          if git diff --staged --quiet; then
            echo "No changes in resume.pdf"
          else
            git commit -m "chore: auto-update resume.pdf [skip ci]"
            git push origin HEAD:master
          fi
```
