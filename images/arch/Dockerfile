# SPDX-FileCopyrightText: 2026 Phosphor Works Team
# SPDX-License-Identifier: CC0-1.0
#
# The Arch CI image: archlinux:latest with every package that a C++ repository
# of the suite lists in .github/ci/arch-packages.txt (packages.txt, collected by
# .github/workflows/ci-images.yml). A job still installs its own list, which
# finds nothing missing when the image is current.
FROM archlinux:latest
COPY packages.txt /etc/phosphor-works-ci/packages.txt
RUN set -eu; \
    pacman -Syu --noconfirm --needed git $(sed -e 's/#.*//' -e 's/[[:space:]]//g' -e '/^$/d' /etc/phosphor-works-ci/packages.txt); \
    pacman -Scc --noconfirm
