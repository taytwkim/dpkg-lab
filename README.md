# Debian Packaging Workflow

Learning packaging workflow in Debian/Ubuntu.

A Linux distribution is largely a collection of packages. This includes commonly used tools like `git` and `nginx`, but also core parts of the system such as `bash`, the package manager, the service manager, and even the Linux kernel itself. Users usually do not install these core packages manually, but they are still managed through the package system and can be updated with commands like `sudo apt upgrade`.

So a new distro release, such as Ubuntu 26.04, is essentially a tested combination of specific package versions that are known to work together.

## Getting Started

1. Start and log into an Ubuntu machine.

2. Install packaging tools.

```bash
sudo apt update
sudo apt install -y build-essential devscripts debhelper autopkgtest
```

3. `git clone` this repo.

4. `hello-tay` is a small script we want to package. It should be an executable.

```bash
cd dpkg-lab
chmod +x hello-tay
```

5. Review the packaging metadata under `debian/`.

The overall structure is:

```text
dpkg-lab/
├── hello-tay
└── debian/
    ├── changelog
    ├── control
    ├── install
    ├── rules
    ├── source/
    │   └── format
    └── tests/
        ├── control
        └── smoke-test
```

* `control`: package name, dependencies, description, and other metadata.
* `changelog`: package version and version history.
* `install`: which files to install and where to place them.
* `rules`: how to build the package. `rules` should be an executable (`chmod +x debian/rules`).
* `source/format`: tells Debian’s packaging tools how the source package is structured. `native` means the source and Debian packaging are treated as one project. In many real packages, the upstream source and Debian packaging are kept separate, with Debian-specific changes layered on top.
* `tests/control`: metadata that tells `autopkgtest` which test to run and what dependencies it needs.
* `tests/smoke-test`: actual test script that `autopkgtest` runs.

6. From the repo root, build the package.

```bash
dpkg-buildpackage -us -uc
```

7. Check the generated `.deb` package. You should see a file like `hello-tay_1.0_all.deb`.

```bash
cd ..
ls
```

8. Install the package.

```bash
sudo apt install ./hello-tay_1.0_all.deb
```

* Note on the naming convention: since this is a native package, the version does not include a separate Debian revision, so the package name looks like `pkg-name_version`. For a non-native package based on an upstream source, the version typically includes both the upstream version and the Debian revision, for example `hello-tay_1.0-1_all.deb`.

9. Check where the package was installed.

```bash
which hello-tay
dpkg -L hello-tay
```

10. Check that the command works.

```bash
hello-tay
```

11. Test package.

```bash
autopkgtest . -- null
```

* `autopkgtest` runs package-level tests to check that a new version works as expected. In Ubuntu’s testing infrastructure, a package change can also trigger `autopkgtest` runs for affected reverse dependencies to catch regressions in other packages.
* `-- null` runs the tests directly on the current machine. `autopkgtest` can also run tests in isolated environments such as containers or VMs.