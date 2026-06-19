# syntax=docker/dockerfile:1
#
# Patched Inkscape 1.4.2 (GTK3) for headless / command-line use.
#
# Builds Inkscape from the upstream release tag with a small downstream patch
# that stops the spurious warning emitted on every headless export:
#
#   glibmm-WARNING **: Failed to wrap object of type 'GtkRecentManager'.
#   Hint: this error is commonly caused by failing to call a library
#   init() function.
#
# Root cause and fix: https://gitlab.com/inkscape/inkscape/-/work_items/5419
#
# The build base is Inkscape's own CI dependency image for Ubuntu 22.04, which
# already ships the full GTK3 build dependency set (libgtkmm-3.0-dev, cmake,
# build-essential, poppler, ...). Single stage on purpose: it guarantees the
# runtime libraries match what Inkscape was linked against, with no dependency
# guesswork. The image is therefore large (~2 GB); slimming via a separate
# runtime stage is a possible follow-up.

ARG INKSCAPE_CI_IMAGE=registry.gitlab.com/inkscape/inkscape-ci-docker/ubuntu-2204
ARG INKSCAPE_TAG=INKSCAPE_1_4_2

FROM ${INKSCAPE_CI_IMAGE}

ARG INKSCAPE_TAG

COPY 0001-headless-recentmanager-5419.patch /tmp/inkscape.patch

# Clone the exact release (incl. submodules: 2geom and libcroco are required to
# build), apply the patch, build, install, then drop the source tree so it does
# not bloat the image. Kept in one layer so the cleanup actually saves space.
RUN set -eux; \
    git clone --depth 1 --branch "${INKSCAPE_TAG}" \
        --recurse-submodules --shallow-submodules \
        https://gitlab.com/inkscape/inkscape.git /src; \
    cd /src; \
    git apply -v /tmp/inkscape.patch; \
    cmake -S . -B build \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/usr/local \
        -DBUILD_TESTING=OFF; \
    cmake --build build --parallel "$(nproc)"; \
    cmake --install build; \
    ldconfig; \
    cd /; \
    rm -rf /src /tmp/inkscape.patch

# Smoke test: must run and must NOT print the GtkRecentManager warning.
RUN inkscape --version
