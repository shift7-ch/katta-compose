FROM nginx:1.31.5
# Only a linux-amd64 build of katta-admin-cli is published; arm64 hosts are not supported yet.
ARG KATTA_CLI_VERSION=1.0.3
RUN curl -fsSL -o /usr/local/bin/katta \
      https://github.com/shift7-ch/katta-admin-cli/releases/download/${KATTA_CLI_VERSION}/katta-linux-amd64 \
 && chmod +x /usr/local/bin/katta
