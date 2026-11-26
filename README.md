# Domain Health Report: DNS, Email Auth, TLS, Expiry

Checks domains for registration expiry (RDAP/WHOIS), DNS, SPF/DMARC/DKIM/MTA-STS, TLS certificate and HTTPS redirect. Input: domains, URLs or emails. Returns one record per domain: 0-100 score, grade A-F, coded issues. Public data only, no login. DKIM found only on common selectors.

**▶ Run it on Apify Store: https://apify.com/far003/domain-health-report**

> This repository hosts documentation and examples; the Actor runs on the Apify platform.

## What it does

- Produces one record per domain with a 0-100 score, a letter grade and a list of coded issues.
- Registration: registrar, creation and expiry dates, days to expiry, EPP status locks, DNSSEC flag.
- DNS: apex A/AAAA, `www`, nameservers, MX, CAA.
- Email security: SPF, DMARC, DKIM on common selectors plus yours, MTA-STS, TLS-RPT, BIMI.
- HTTPS / TLS: handshake, certificate chain, expiry, name coverage, protocol version, HTTP to HTTPS redirect, HSTS.
- Accepts domains, URLs and email addresses, mixed. No login, no API key, no browser.

## Who it is for

- Anyone monitoring a portfolio of domains for expiring domains and certificates.
- Teams qualifying a lead list (broken SPF/DMARC, `p=none`).
- Due diligence before buying a domain.
- MSPs and agencies running health checks across clients.

## Example input

```json
{
  "domains": ["github.com", "https://www.cloudflare.com/", "repubblica.it", "expired.badssl.com"],
  "checkRegistration": true,
  "checkEmail": true,
  "checkTls": true,
  "dkimSelectors": ["mycompany2026"],
  "includeRawRecords": false,
  "timeoutSecs": 15,
  "concurrency": 10,
  "maxDomains": 0
}
```

## Example output

<details><summary>Show the record</summary>

```json
{
  "domain": "repubblica.it",
  "registrableDomain": "repubblica.it",
  "score": 84,
  "grade": "B",
  "criticalCount": 0,
  "issueCodes": [
    "NO_TRANSFER_LOCK",
    "NO_DNSSEC",
    "NO_CAA",
    "DMARC_POLICY_NONE",
    "MTA_STS_MISSING",
    "TLS_RPT_MISSING",
    "HSTS_MISSING"
  ],
  "registered": true,
  "registrar": "Gandi SAS",
  "expiresAt": "2027-01-16T00:00:00.000Z",
  "daysToExpiry": 101,
  "dnssec": false,
  "nameservers": [
    "ns-1477.awsdns-56.org",
    "ns-1541.awsdns-00.co.uk",
    "ns-353.awsdns-44.com",
    "ns-723.awsdns-26.net"
  ],
  "mailProvider": "Microsoft 365",
  "spfRecord": "v=spf1 ip4:178.23.254.18 include:spf-smtp.kataweb.it include:spf.protection.outlook.com include:musvc.com -all",
  "spfAll": "-",
  "dmarcPolicy": "none",
  "dkimSelectorsFound": [
    "selector1"
  ],
  "tlsTrusted": true,
  "certIssuer": "Amazon",
  "certDaysToExpiry": 153,
  "redirectsToHttps": true,
  "hsts": false,
  "coverageComplete": true,
  "checkErrors": [],
  "durationMs": 3699,
  "issues": [
    {
      "code": "DMARC_POLICY_NONE",
      "severity": "medium",
      "area": "email",
      "message": "DMARC p=none: monitoring only, spoofed mail is still delivered."
    }
  ]
}
```

</details>

## Use it from code

Set your token in the `APIFY_TOKEN` environment variable.

**Node.js** ([`apify-client`](https://docs.apify.com/api/client/js/))

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });
const run = await client.actor('far003/domain-health-report').call({
    "domains": [
        "github.com",
        "https://www.cloudflare.com/",
        "repubblica.it",
        "expired.badssl.com"
    ],
    "checkRegistration": true,
    "checkEmail": true,
    "checkTls": true,
    "dkimSelectors": [
        "mycompany2026"
    ],
    "includeRawRecords": false,
    "timeoutSecs": 15,
    "concurrency": 10,
    "maxDomains": 0
});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.log(items);
```

**Python** ([`apify-client`](https://docs.apify.com/api/client/python/))

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("far003/domain-health-report").call(run_input={
    "domains": ["github.com", "https://www.cloudflare.com/", "repubblica.it", "expired.badssl.com"],
    "checkRegistration": True,
    "checkEmail": True,
    "checkTls": True,
    "dkimSelectors": ["mycompany2026"],
    "includeRawRecords": False,
    "timeoutSecs": 15,
    "concurrency": 10,
    "maxDomains": 0
})
items = client.dataset(run["defaultDatasetId"]).list_items().items
print(items)
```

## Pricing

Pay per event, no subscription. The current price, and what is free, is on the [Apify Store page](https://apify.com/far003/domain-health-report).

## Data source & licence

The Actor reads only data that the domain owner publishes to the world: registry RDAP (RFC 9083), the Registro .it WHOIS for `.it`, public DNS, and a TLS handshake plus HTTP HEAD request on the domain. Registrant contact data is never extracted. Registries that publish nothing readable return `null` with a note. This is a technical screening, not a security audit or a legal compliance statement.

## Support

Open an issue in this repository or use the Issues tab on the Store page.

## Licence

The documentation and examples in this repository are released under the MIT licence (see `LICENSE`). The Actor itself is proprietary and runs on the Apify platform.
