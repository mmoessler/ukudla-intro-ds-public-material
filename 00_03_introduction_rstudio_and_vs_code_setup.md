# 0.3) R, RStudio, and Visual Studio Code Guide

---

- Source: [00_03_introduction_rstudio_and_vs_code_setup.md](https://github.com/mmoessler/ukudla-intro-ds-public-material/blob/main/00_03_introduction_rstudio_and_vs_code_setup.md)
- History: [Commit History](https://github.com/mmoessler/ukudla-intro-ds-public-material/commits/main/00_03_introduction_rstudio_and_vs_code_setup.md)
- Feedback: [Topic 00: Introduction](https://github.com/mmoessler/ukudla-intro-ds-public-material/discussions/1)

---

## Outline

- [Outline](#outline)
- [Food Systems and Data Science Certificate](#food-systems-and-data-science-certificate)
- [1. Software Overview](#1-software-overview)
- [2. Installing R](#2-installing-r)
  - [Windows](#windows)
  - [macOS](#macos)
  - [Linux](#linux)
- [3. Installing RStudio](#3-installing-rstudio)
  - [Windows](#windows-1)
  - [macOS](#macos-1)
  - [Linux](#linux-1)
- [4. Installing Visual Studio Code](#4-installing-visual-studio-code)
  - [Windows](#windows-2)
  - [macOS](#macos-2)
  - [Linux](#linux-2)
- [5. Recommended VS Code Extensions](#5-recommended-vs-code-extensions)
- [6. Installing Course Packages](#6-installing-course-packages)
- [7. Using RStudio](#7-using-rstudio)
  - [Running Code](#running-code)
  - [Using the Terminal](#using-the-terminal)
- [8. Working with Projects](#8-working-with-projects)
- [9. Using Visual Studio Code](#9-using-visual-studio-code)
- [10. Troubleshooting](#10-troubleshooting)
  - [Problem 01](#problem-01)
  - [Problem 02](#problem-02)
  - [Problem 03](#problem-03)
- [11. Best Practices](#11-best-practices)
- [Summary](#summary)

---

## Food Systems and Data Science Certificate

**Scope**

This guide explains how to install and use **R**, **RStudio**, and **Visual Studio Code (VS Code)**. Git installation, GitHub, and repository setup are covered in a separate guide.

---

## 1. Software Overview

The course primarily uses **R** for programming and data analysis.

We recommend using:

| Software | Purpose |
|-----------|---------|
| **R** | Programming language for statistics and data science |
| **RStudio** | Primary integrated development environment (IDE) for writing and running R code |
| **Visual Studio Code** | General-purpose editor for Markdown, YAML, documentation, and project files |

Most analyses will be completed in **RStudio**, while **VS Code** is useful for editing text files, browsing project folders, and working with Quarto documents.

---

## 2. Installing R

Always install **R before RStudio**.

Download R from:

https://cran.r-project.org/

---

### Windows

1. Select **Download R for Windows**.
2. Download the latest installer.
3. Run the installer.
4. Accept the default installation options.

Verify the installation by opening **R** from the Start Menu.

---

### macOS

1. Select **Download R for macOS**.
2. Download the installer appropriate for your version of macOS (Apple Silicon or Intel if applicable).
3. Open the downloaded package.
4. Follow the installation wizard.

Launch R from the Applications folder to verify the installation.

---

### Linux

Most Linux distributions provide R through their package manager.

For more information, please consult the web or your preferred LLM.

---

## 3. Installing RStudio

Download the free **RStudio Desktop** from:

https://posit.co/download/rstudio-desktop/

Choose the installer for your operating system.

---

### Windows

Run the installer and follow the default options.


---

### macOS

Open the downloaded `.dmg` file and drag **RStudio** into the Applications folder.

---

### Linux

Install the package provided by Posit or use your distribution's package manager if available.

After installation, open RStudio. It should automatically detect your R installation.

---

## 4. Installing Visual Studio Code

Download VS Code from:

https://code.visualstudio.com/

---

### Windows

Run the installer and accept the default options.

---

### macOS

Download the macOS version and move VS Code into the Applications folder.

---

### Linux

Install using your preferred package manager.

Example (Ubuntu/Debian):

```bash
sudo snap install code --classic
```

or use the package provided on the VS Code website.

---

## 5. Recommended VS Code Extensions

Install the following extensions from the **Extensions** panel in VS Code.

| Extension                        | Purpose                                      |
| -------------------------------- | -------------------------------------------- |
| Markdown All in One              | Markdown editing and productivity features   |
| Markdown Preview Mermaid Support | Mermaid diagram support in Markdown previews |
| Python                           | Python language support                      |

Additional VS Code extensions are available in the [Visual Studio Marketplace](https://marketplace.visualstudio.com/).

---

## 6. Installing Course Packages

Open **RStudio**.

Install **renv** (only once):

```r
install.packages("renv")
```

Restore the project environment:

```r
renv::restore()
```

Install additional packages only when instructed by the course.

---

## 7. Using RStudio

The RStudio interface consists of four main panes:

- **Source** – edit scripts and Quarto documents
- **Console** – execute R commands
- **Environment/History** – inspect objects
- **Files/Plots/Packages/Help** – manage files and view outputs

### Running Code

Run the current line:

- Windows/Linux: **Ctrl + Enter**
- macOS: **Cmd + Enter**

Run an entire script:

```
Source → Run All
```

### Using the Terminal

The Terminal tab provides access to your operating system shell without leaving RStudio.

---

## 8. Working with Projects

Always open the supplied `.Rproj` file.

Working inside an RStudio Project ensures:

- consistent working directories
- reproducible file paths
- easier collaboration

Do not move files outside the project directory.

---

## 9. Using Visual Studio Code

VS Code complements RStudio rather than replacing it.

Recommended uses include:

- editing Markdown
- browsing project files
- reading documentation
- comparing files
- searching across the project

Open the **project folder**, not individual files.

---

## 10. Troubleshooting

### Problem 01

Problem: RStudio cannot find R

Reinstall R first, then restart RStudio.

---

### Problem 02

Problem: Missing packages

```r
renv::restore()
```

---

### Problem 03

Problem: "Package is not available"

Check your internet connection and ensure you are using the latest version of R.

---

## 11. Best Practices

- Work inside the supplied RStudio Project.
- Keep raw data unchanged.
- Store generated files separately.
- Use Quarto for reports.
- Keep scripts organized.
- Restart R occasionally to ensure your scripts run from a clean session.
- Write clear comments and meaningful variable names.

---

## Summary

For this course:

- **R** is the programming language used for analysis.
- **RStudio** is the primary environment for programming and data analysis.
- **VS Code** is a complementary editor for Markdown, Quarto, and project files.
- **renv** keeps package versions consistent across all students.
