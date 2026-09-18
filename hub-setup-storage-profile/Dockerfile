FROM nginx:1.31.5
# Set by BuildKit to the architecture of the build target, amd64 or arm64
ARG TARGETARCH
ARG KATTA_CLI_VERSION=1.0.3
RUN curl -fsSL -o /usr/local/bin/katta \
      https://github.com/shift7-ch/katta-admin-cli/releases/download/${KATTA_CLI_VERSION}/katta-linux-${TARGETARCH} \
 && chmod +x /usr/local/bin/katta
