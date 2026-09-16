# lto-enabled-list

A list of source packages to build with link time optimization
(LTO) by default.

deepin/UOS enables the dpkg vendor build feature `optimize/lto`
(`-flto=auto -ffat-lto-objects`) following an **opt-in model**:
only the source packages listed in this repository's
`lto-enabled-list` file are built with LTO by default. Every source
package absent from the list is built without LTO, unless its own
packaging opts in explicitly (`optimize=+lto` in
`DEB_BUILD_MAINT_OPTIONS`, or `-flto` in its build rules).

This is the inverse of Ubuntu's `lto-disabled-list` model (LTO on
for everything except a blacklist); the opt-in model keeps the
archive unchanged until a package is explicitly proven to build —
and work — with LTO.

## Mechanism

`Dpkg::Vendor::Debian` (dpkg >= 1.22.6deepin12) reads the list at
build-flag initialization time from the fixed path
`/usr/share/lto-enabled-list/lto-enabled-list`, where this package
installs it. `dpkg-dev` depends on this package, so the list is
present in every build environment. Supported architectures:
amd64, arm64, sw64 and loong64.

List format (one entry per line):

    <source> any | <arch> [<arch> ...]

- `any` — build the source package with LTO on every supported
  architecture
- `<arch> ...` — build it with LTO only on the listed architectures
- lines starting with `#` are comments

If the list file is missing, no package gets LTO by default (fail
closed); explicit opt-ins keep working.

## Current entries

The initial population is the update-lto-20260910 rebuild campaign
(deepin-community project 818): the 619 source packages with a
merged "Rebuild with LTO enabled" pull request — the 618-package
original batch (September 2026) plus a 12-package supplementary
batch (2026-09-12) — tagged `any`.

Not listed, although their campaign pull requests are merged:

- four packages later confirmed broken: boost1.83, kwin, lapack,
  openexr;
- binutils, cdrkit, gnustep-make, libnfs, x264: their debian/rules
  hardcode CFLAGS and never consume the dpkg build flags, so the
  injected LTO flags cannot reach the compiler.

Source packages outside deepin-community are not listed either;
add them once their LTO rebuild is proven and tracked.

Narrow an entry to specific architectures — or remove it entirely —
when a rebuild on an architecture fails; extend entries only with
evidence from an actual LTO rebuild.

The `qt6-*` entries are redundant by design: deepin builds the Qt6
stack with LTO by default (explicit `optimize=+lto` in
deepin-community `qt6-*` packaging). They are listed only because
the rebuild completed.

## Related

- deepin-community/dpkg — reads the list (vendor build features)
- deepin-community/lto-disabled-list — the former blacklist model,
  no longer consumed by dpkg
