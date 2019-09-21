# ungoogled-chromium-windows

Windows packaging for [ungoogled-chromium](//github.com/Eloston/ungoogled-chromium).

## Downloads

[Download binaries from the Contributor Binaries website](//ungoogled-software.github.io/ungoogled-chromium-binaries/).

**Source Code**: It is recommended to use a tag via `git checkout` (see building instructions below). You may also use `master`, but it is for development and may not be stable.

## Building

Google only supports [Windows 7 x64 or newer](https://chromium.googlesource.com/chromium/src/+/refs/tags/77.0.3865.90/docs/windows_build_instructions.md#system-requirements). These instructions are tested on Windows 7 Professional x64.

NOTE: The default configuration will build 64-bit binaries for maximum security (TODO: Link some explanation). This can be changed to 32-bit by following the instructions in `build.py`

### Setting up the build environment

#### Setting up Visual Studio

[Follow the "Visual Studio" section of the official Windows build instructions](https://chromium.googlesource.com/chromium/src/+/refs/tags/77.0.3865.90/docs/windows_build_instructions.md#visual-studio).

* Make sure to read through the entire section and install/configure all the required components.

#### Other build requirements

**IMPORTANT**: Currently, the `MAX_PATH` path length restriction (which is 260 characters by default) must be lifted in order for buildkit to function properly. One such setup that works is Windows 10 (which added this option since Anniversary) with Python 3.6 or newer from the official installer (which contains the manifest files that allow use of long file paths). Other possible setups are being discussed in [Issue #345](https://github.com/Eloston/ungoogled-chromium/issues/345).

1. Setup the following:

    * 7-zip
    * Python 3.6+ with pypiwin32 module (`pip install pypiwin32`)

2. Make sure Python 3.6+ is set in the user or system environment variable `PATH` as `python`.

### Setup and build

NOTE: The commands below assume the `py` command was installed by Python 3 into `PATH`. If this is not the case, then substitute it with `python`.

Run in `cmd.exe`:

```cmd
git clone --recurse-submodules https://github.com/ungoogled-software/ungoogled-chromium-windows.git
# Replace TAG_OR_BRANCH_HERE with a tag or branch name
git checkout --recurse-submodules TAG_OR_BRANCH_HERE
py build.py
py package.py
```

A zip archive will be created under `build`

**NOTE**: If the build fails, you must take additional steps before re-running the build:

* If the build fails while downloading the Chromium source code (which is during `build.py`), it can be fixed by removing `build\download_cache` and re-running the build instructions.
* If the build fails at any other point during `build.py`, it can be fixed by removing `build\src` and re-running the build instructions. This will clear out all the code used by the build, and any files generated by the build.

## Developer info

### First-time setup

1. [Setup MSYS2](http://www.msys2.org/)
2. Run the following in a "MSYS2 MSYS" shell:

```sh
pacman -S quilt python3 vim
# By default, there doesn't seem to be a vi command for less, quilt edit, etc.
ln -s /usr/bin/vim /usr/bin/vi
```
### Updating patches

Run the following in a "MSYS2 MSYS" shell:

```sh
# You can use Git Bash to determine the path to this repo
# Or, you can find it yourself via /<drive letter>/<path with forward slashes>
cd /path/to/repo/ungoogled-chromium-windows

# Setup patches and shell to update patches
./devutils/update_patches.sh merge
source devutils/set_quilt_vars.sh

# Setup Chromium source
mkdir -p build/{src,download_cache}
./ungoogled-chromium/utils/downloads.py retrieve -i ungoogled-chromium/downloads.ini -c build/download_cache
./ungoogled-chromium/utils/downloads.py unpack -i ungoogled-chromium/downloads.ini -c build/download_cache build/src

cd build/src
# Use quilt to refresh patches. See ungoogled-chromium's docs/developing.md section "Updating patches" for more details
quilt pop -a

cd ../../
# Remove all patches introduced by ungoogled-chromium
./devutils/update_patches.sh unmerge
# Ensure patches/series is formatted correctly, e.g. blank lines

# Sanity checking for consistency in series file
./devutils/check_patch_files.sh

# Use git to add changes and commit
```

## License

See [LICENSE](LICENSE)
