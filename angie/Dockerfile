# ==============================================================================
# Angie with ModSecurity WAF - Multi-stage build
# ==============================================================================
# Stage 1: Download and prepare OWASP CRS rules
# Stage 2: Final image with entrypoint for auto-setup
# ==============================================================================

# ------------------------------------------------------------------------------
# Stage 1: Builder - download CRS rules
# ------------------------------------------------------------------------------
FROM alpine:3.21 AS crs-builder

# Install git for cloning CRS (pinned version for reproducibility)
# hadolint ignore=DL3018
RUN apk add --no-cache git~=2.47

# Clone OWASP CoreRuleSet (auto-updated on each build)
ARG CRS_VERSION=v4.18.0
RUN mkdir -p /crs && \
    git clone --depth 1 -b ${CRS_VERSION} \
        https://github.com/coreruleset/coreruleset /crs/coreruleset && \
    # Prepare configuration files
    cp /crs/coreruleset/crs-setup.conf.example \
       /crs/coreruleset/crs-setup.conf && \
    cp /crs/coreruleset/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example \
       /crs/coreruleset/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf && \
    cp /crs/coreruleset/rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf.example \
       /crs/coreruleset/rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf && \
    # Remove unnecessary files to reduce size
    rm -rf /crs/coreruleset/.git \
           /crs/coreruleset/.github \
           /crs/coreruleset/tests \
           /crs/coreruleset/util \
           /crs/coreruleset/CHANGES.md \
           /crs/coreruleset/CONTRIBUTING.md \
           /crs/coreruleset/CONTRIBUTORS.md \
           /crs/coreruleset/KNOWN_BUGS.md \
           /crs/coreruleset/docs

# ------------------------------------------------------------------------------
# Stage 2: Final image
# ------------------------------------------------------------------------------
# Pin to specific Angie version for reproducibility
# Check for updates: https://angie.software/en/install/
FROM docker.angie.software/angie:1.10.3

LABEL maintainer="your-email@example.com"
LABEL description="Angie web server with ModSecurity WAF and OWASP CRS"
LABEL version="1.0.0"
LABEL org.opencontainers.image.source="https://github.com/hvaclab/angie-modsecurity-docker"

# Install tools for auto-setup (curl for GeoIP, openssl for DH params)
# Versions pinned for reproducibility (Alpine 3.21 packages)
# hadolint ignore=DL3018
RUN apk add --no-cache curl~=8.14 openssl~=3.3

# Copy CRS rules from builder
COPY --from=crs-builder /crs/coreruleset /var/lib/angie/modsecurity/coreruleset

# Create necessary directories
RUN mkdir -p /var/lib/angie/acme \
             /var/log/angie \
             /var/www/html \
             /var/www/errors \
             /etc/angie/geoip \
             /etc/ssl/certs

# Copy entrypoint script
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Environment variables for auto-setup
ENV AUTO_UPDATE_GEOIP=false
ENV GEOIP_MAX_AGE_DAYS=30

# Set working directory
WORKDIR /etc/angie

# Use entrypoint for auto-setup, then run angie
ENTRYPOINT ["/entrypoint.sh"]
CMD ["angie", "-g", "daemon off;"]
