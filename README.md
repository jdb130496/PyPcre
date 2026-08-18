PyPcre — Meson Build Fork

Fork notice: This repository (jdb130496/PyPcre) diverges from upstream ModelCloud/PyPcre in one key way: the build system has been migrated from setuptools to Meson (meson.build + meson.options). All Python API, flags, benchmarks, and C extension functionality are identical to upstream and kept in sync by merging from ModelCloud/PyPcre.

Why Meson?
Modern, declarative build description
Better dependency discovery
Faster incremental builds with Ninja backend
Cleaner separation of build configuration from packaging
Better Windows cross-compiler support (tested with Windows Clang-cl)
Platform

Tested on:

Windows 11 with Clang-cl (D:\Programs\clang) + lld-link + standalone Ninja
Python 3.14 (CPython)
PCRE2 10.48-DEV (static, built separately)

Linux/macOS should work with appropriate PCRE2 paths but are not yet tested from this fork.

Prerequisites
1. PCRE2 (static library)

You need a separately built PCRE2 static library. On Windows with Clang-cl:

bash
# Example paths used in this fork (adjust to your own)
# Include: D:\Programs\pcre2-clang-win\include
# Library: D:\Programs\pcre2-clang-win\lib\pcre2-8-static.lib

Build PCRE2 from source or install via your package manager. On Linux:

bash
sudo apt install libpcre2-dev        # Debian/Ubuntu
sudo dnf install pcre2-devel         # Fedora
2. Meson and Ninja
bash
pip install meson ninja

Or use standalone builds (recommended on Windows to avoid PATH conflicts):

Meson: build from source as a .pyz zipapp, wrap in a meson.cmd launcher
Ninja: standalone binary at a fixed path
3. A C compiler
Windows: Clang-cl (clang-cl.exe + lld-link.exe) — tested and recommended
Linux/macOS: GCC or Clang
4. Python build tools
bash
pip install meson-python
Building and Installing
Editable install (development mode)

On Windows with Clang-cl and a static PCRE2:

bash
pip install -e . --no-build-isolation --no-cache-dir `
  -Csetup-args="-Dpcre2_include_dir=D:\Programs\pcre2-clang-win\include" `
  -Csetup-args="-Dpcre2_library=D:\Programs\pcre2-clang-win\lib\pcre2-8-static.lib"

On Linux/macOS (PCRE2 installed system-wide via pkg-config):

bash
pip install -e . --no-build-isolation --no-cache-dir
Clean rebuild (always do this after merging upstream or changing meson.build)
bash
Remove-Item -Recurse -Force _build    # PowerShell
# or
rm -rf _build                         # bash

pip uninstall pypcre -y
pip install -e . --no-build-isolation --no-cache-dir `
  -Csetup-args="-Dpcre2_include_dir=..." `
  -Csetup-args="-Dpcre2_library=..."

Important: Always nuke _build and reinstall from scratch rather than just reinstalling. The editable loader caches the meson invocation path, and a stale _build can cause meson.pyz vs meson.cmd conflicts on Windows resulting in %1 is not a valid Win32 application errors.

Verify the build
bash
python -c "import pcre_ext_c; print('escape' in dir(pcre_ext_c))"   # should print True
python -c "import pcre; print(pcre.__version__)"                      # should print 0.6.0
Running Tests
bash
pip install tabulate pytest
python -m pytest tests/ -v --tb=short --ignore=tests/test_benchmark.py

Expected result: 824 passed, 14 skipped (2 known harmless failures on Windows):

Test	Reason	Severity
test_cache_strategy_benchmark	Timing fluke — sub-millisecond margin sensitive to system load	Harmless, rerun to confirm
test_parallel_cross_over_lengths	Windows cp1252 terminal can't encode ≤ Unicode character in test output	Harmless, set $env:PYTHONUTF8 = "1" to fix

To fix the Unicode terminal issue permanently, add to your PowerShell profile:

powershell
$env:PYTHONUTF8 = "1"
Keeping in Sync with Upstream
bash
git fetch upstream
git merge upstream/main       # your meson.build is on main in this fork
git push origin main

If the merge touched meson.build, pyproject.toml, or any file under pcre_ext/:

bash
Remove-Item -Recurse -Force _build
pip uninstall pypcre -y
pip install -e . --no-build-isolation --no-cache-dir `
  -Csetup-args="-Dpcre2_include_dir=D:\Programs\pcre2-clang-win\include" `
  -Csetup-args="-Dpcre2_library=D:\Programs\pcre2-clang-win\lib\pcre2-8-static.lib"
python -m pytest tests/ --tb=short --ignore=tests/test_benchmark.py 2>&1 | Select-Object -Last 20

If meson.build conflicts during merge (upstream doesn't use Meson so this is unlikely but possible):

bash
git checkout --ours meson.build
git add meson.build
git commit -m "merge: upstream/main, retain meson build system"
Repository Structure (Meson-specific files)
File	Purpose
meson.build	Main build definition — declares C sources, Python install sources, PCRE2 dependency
meson.options	Build options — PCRE2 include/library path overrides for Windows static builds
pyproject.toml	Declares meson-python as the build backend
Upstream

All credit for the Python API, C extension, benchmarks, and PCRE2 integration goes to ModelCloud/PyPcre. This fork only changes the build system.
