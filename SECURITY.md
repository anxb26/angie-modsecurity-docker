# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 1.0.x   | :white_check_mark: |

## Reporting a Vulnerability

**Please do not report security vulnerabilities through public GitHub issues.**

Instead, please report them via:
- **GitHub Security Advisories**: [Report a vulnerability](https://github.com/hvaclab/angie-modsecurity-docker/security/advisories/new)
- **Email**: security@hvaclab.ru (если есть)

Include:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

You will receive a response within 48 hours.

## Security Best Practices

When using this project:

1. **Never commit `.env` files** with real credentials
2. **Use strong passwords** for Keycloak secrets
3. **Keep dependencies updated** - check for new releases
4. **Review ModSecurity logs** regularly
5. **Enable auto-update for GeoIP** in production
6. **Use Let's Encrypt** for production SSL certificates

## Security Features

This project includes:
- ModSecurity WAF with OWASP CRS 4.x
- Fail2Ban for brute-force protection
- TLS 1.2/1.3 only (no legacy protocols)
- Security headers (HSTS, CSP, etc.)
- Rate limiting
