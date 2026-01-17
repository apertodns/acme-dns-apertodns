# ApertoDNS DNS Plugin for acme.sh

DNS-01 challenge plugin for [acme.sh](https://github.com/acmesh-official/acme.sh) to obtain SSL/TLS certificates from Let's Encrypt using [ApertoDNS](https://www.apertodns.com).

## Prerequisites

1. An ApertoDNS account with at least one domain
2. An API key from your [ApertoDNS dashboard](https://www.apertodns.com/dashboard)
3. acme.sh installed on your system

## Installation

### Option 1: Copy to acme.sh dnsapi folder

```bash
# Download the plugin
curl -o dns_apertodns.sh https://raw.githubusercontent.com/apertodns/acme.sh-plugin/main/dns_apertodns.sh

# Copy to acme.sh dnsapi folder
cp dns_apertodns.sh ~/.acme.sh/dnsapi/
```

### Option 2: Use from local path

```bash
# Clone the repository
git clone https://github.com/apertodns/acme.sh-plugin.git
cd acme.sh-plugin

# Set the path
export ACME_SH_DNSAPI_PATH=$(pwd)
```

## Configuration

Export your API key before running acme.sh:

```bash
export APERTODNS_API_KEY="apertodns_live_xxxxxxxxxxxxxxxx"
```

The API key will be saved to your acme.sh account configuration after the first successful use.

### Optional: Custom API URL

If you need to use a different API endpoint (e.g., for testing):

```bash
export APERTODNS_API_URL="https://api.apertodns.com"
```

## Usage

### Issue a new certificate

```bash
# Single domain
acme.sh --issue --dns dns_apertodns -d myhost.apertodns.com

# With www subdomain
acme.sh --issue --dns dns_apertodns -d myhost.apertodns.com -d www.myhost.apertodns.com

# Wildcard certificate
acme.sh --issue --dns dns_apertodns -d myhost.apertodns.com -d '*.myhost.apertodns.com'
```

### Renew a certificate

```bash
acme.sh --renew -d myhost.apertodns.com
```

### Force renewal

```bash
acme.sh --renew -d myhost.apertodns.com --force
```

## How It Works

1. When you request a certificate, Let's Encrypt asks you to prove domain ownership
2. acme.sh calls `dns_apertodns_add()` to create a TXT record at `_acme-challenge.yourdomain.apertodns.com`
3. Let's Encrypt verifies the TXT record
4. After verification, acme.sh calls `dns_apertodns_rm()` to remove the TXT record
5. Your certificate is issued

## Examples

### Nginx with automatic renewal

```bash
# Issue certificate
acme.sh --issue --dns dns_apertodns -d myhost.apertodns.com

# Install certificate to Nginx
acme.sh --install-cert -d myhost.apertodns.com \
  --key-file /etc/nginx/ssl/myhost.key \
  --fullchain-file /etc/nginx/ssl/myhost.crt \
  --reloadcmd "systemctl reload nginx"
```

### Apache

```bash
acme.sh --issue --dns dns_apertodns -d myhost.apertodns.com

acme.sh --install-cert -d myhost.apertodns.com \
  --cert-file /etc/apache2/ssl/myhost.crt \
  --key-file /etc/apache2/ssl/myhost.key \
  --fullchain-file /etc/apache2/ssl/myhost-fullchain.crt \
  --reloadcmd "systemctl reload apache2"
```

### Docker / Traefik

```bash
# Issue certificate with standalone mode
docker run --rm -it \
  -v ~/.acme.sh:/acme.sh \
  -e APERTODNS_API_KEY="your-api-key" \
  neilpang/acme.sh --issue --dns dns_apertodns -d myhost.apertodns.com
```

## Troubleshooting

### "You did not specify APERTODNS_API_KEY yet"

Make sure you've exported your API key:

```bash
export APERTODNS_API_KEY="apertodns_live_xxxxxxxx"
```

### "Domain must be under apertodns.com"

This plugin only works with domains ending in `.apertodns.com`. For custom domains, use the appropriate DNS provider plugin.

### "Failed to add TXT record"

Check that:
1. Your API key is valid and not expired
2. The domain exists in your ApertoDNS account
3. You have network connectivity to `api.apertodns.com`

### Debug mode

Run with debug output:

```bash
acme.sh --issue --dns dns_apertodns -d myhost.apertodns.com --debug 2
```

### Manual testing

Test the DNS API directly:

```bash
# Set TXT record
curl -X POST "https://api.apertodns.com/.well-known/apertodns/v1/update" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"hostname":"myhost.apertodns.com","txt":{"name":"_acme-challenge","value":"test-token","action":"set"}}'

# Verify with dig
dig TXT _acme-challenge.myhost.apertodns.com +short

# Delete TXT record
curl -X POST "https://api.apertodns.com/.well-known/apertodns/v1/update" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"hostname":"myhost.apertodns.com","txt":{"name":"_acme-challenge","action":"delete"}}'
```

## API Reference

The plugin uses the ApertoDNS IETF-compatible API:

- **Endpoint**: `POST https://api.apertodns.com/.well-known/apertodns/v1/update`
- **Authentication**: Bearer token
- **Content-Type**: application/json

### Set TXT Record

```json
{
  "hostname": "myhost.apertodns.com",
  "txt": {
    "name": "_acme-challenge",
    "value": "token-value",
    "action": "set"
  }
}
```

### Delete TXT Record

```json
{
  "hostname": "myhost.apertodns.com",
  "txt": {
    "name": "_acme-challenge",
    "action": "delete"
  }
}
```

## Getting Your API Key

1. Log in to [ApertoDNS Dashboard](https://www.apertodns.com/dashboard)
2. Go to **Settings** > **API Keys**
3. Click **Create API Key**
4. Copy the key (it starts with `apertodns_live_` or `apertodns_test_`)

## Support

- **Documentation**: [apertodns.com/docs](https://www.apertodns.com/docs)
- **Issues**: [GitHub Issues](https://github.com/apertodns/acme.sh-plugin/issues)
- **Email**: support@apertodns.com

## License

MIT License

## Author

Andrea Ferro <support@apertodns.com>
