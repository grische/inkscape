# syntax=docker/dockerfile:1
#
# Patched Inkscape 1.4.2 (GTK3) for headless / command-line use.
#
# Builds Inkscape from the upstream release tag with two small downstream patches
# that stop the spurious "Failed to wrap object of type ..." warnings printed on
# every headless export (no display => gtkmm/pangomm wrappers never registered):
#
#   glibmm-WARNING **: Failed to wrap object of type 'GtkRecentManager'.
#   ... WARNING **: Failed to wrap object of type 'PangoFT2FontMap'.
#
# 0001 (GtkRecentManager): https://gitlab.com/inkscape/inkscape/-/work_items/5419
# 0002 (PangoFT2FontMap): drops an unused Glib::wrap() of the FT2 font map.
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

# The ubuntu-2204 dependency image omits Boost.Stacktrace, which Inkscape 1.4.2
# requires (used for crash backtraces). The upstream /master CI image — the one
# that historically built 1.4.2 — does install it, so add it here.
RUN set -eux; \
    export DEBIAN_FRONTEND=noninteractive; \
    apt-get update; \
    apt-get install -y --no-install-recommends libboost-stacktrace-dev; \
    rm -rf /var/lib/apt/lists/*

COPY 0001-headless-recentmanager-5419.patch 0002-headless-pangoft2fontmap.patch /tmp/

# Clone the exact release (incl. submodules: 2geom and libcroco are required to
# build), apply the patches, build, install, then drop the source tree so it does
# not bloat the image. Kept in one layer so the cleanup actually saves space.
RUN set -eux; \
    git clone --depth 1 --branch "${INKSCAPE_TAG}" \
        --recurse-submodules --shallow-submodules \
        https://gitlab.com/inkscape/inkscape.git /src; \
    cd /src; \
    git apply -v /tmp/0001-headless-recentmanager-5419.patch \
                 /tmp/0002-headless-pangoft2fontmap.patch; \
    cmake -S . -B build \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/usr/local \
        -DBUILD_TESTING=OFF; \
    cmake --build build --parallel "$(nproc)"; \
    cmake --install build; \
    ldconfig; \
    cd /; \
    rm -rf /src /tmp/0001-headless-recentmanager-5419.patch /tmp/0002-headless-pangoft2fontmap.patch

# Smoke test: must run and must not print a "Failed to wrap object" warning.
RUN inkscape --version
