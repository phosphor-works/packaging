# SPDX-FileCopyrightText: 2026 Phosphor Works Team
# SPDX-License-Identifier: CC0-1.0
#
# The Fedora CI image: fedora:<version> with every package that a C++
# repository of the suite lists in .github/ci/fedora-packages.txt (packages.txt,
# collected by .github/workflows/ci-images.yml). A job still installs its own
# list, which finds nothing missing when the image is current.
ARG FEDORA_VERSION=44
FROM fedora:${FEDORA_VERSION}
COPY packages.txt /etc/phosphor-works-ci/packages.txt
RUN set -eu; \
    dnf install -y git $(sed -e 's/#.*//' -e 's/[[:space:]]//g' -e '/^$/d' /etc/phosphor-works-ci/packages.txt); \
    dnf clean all
