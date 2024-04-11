# Let's Encrypt HP iLO 
## History
This script is pased on the work of Basil Hendroff at https://github.com/basilhendroff/truenas-iocage-letsencrypt/, but adapted for more general use.  Basil's work created a jail on FreeNAS/TrueNAS CORE, and installed various software in it, including this script and its dependencies.  As a result, his version of this script expected certain files to be in certain locations, and hard-coded those paths into the script.

I no longer use TrueNAS CORE, but I do have two HPE servers with iLO on which I wanted to use Let's Encrypt certs, and I had another Linux VM that was already deploying certs to some other devices on my network.  That required changing some hardcoded paths in the script, and led to a few other changes as well.
## Changes
* Remove root privileges check
* Add configurable path for cert
* Add configurable path for acme.sh
* Change mode of created files (except the deployment script) to 600 rather than 700--they don't need to be executable.
* Make hostname/domain checking case-insensitive

## Prerequisites
* This script depends on both [python-hpilo](https://seveas.github.io/python-hpilo/index.html) and [acme.sh](https://github.com/acmesh-official/acme.sh).  Make sure these are installed and available in whatever environment you're using to run this script.
* This script will obtain a certificate from a trusted certificate authority, which means you must use a public domain name you control.
* This script uses DNS validation to obtain the script, so you must be using a [supported DNS provider](https://github.com/acmesh-official/acme.sh/wiki/dnsapi).  Fortunately, acme.sh supports over 150 DNS providers.

## Preparation
Before undertaking a deployment:
1. Ensure the iLO is updated with the latest firmware (iLO UI > Administration > Firmware) and the iLO hostname and domain fields (iLO UI > Network > iLO Dedicated Network Port > General) are configured.
2. Configure your local DNS resolver to resolve the FQDN of the iLO to its IP address. For example, `ilo.mydomain.com` must resolve to the iLO IP on the internal network.

## Deployment
1. Edit the file called `hpilo.cfg` with your favorite text editor. In its minimal form, it would look something like this:
```
USERNAME="Administrator"
PASSWORD="alakazam"
HOSTNAME="ilo"
DOMAIN="mydomain.com"
```
The mandatory options are:
- USERNAME: Username of the iLO administrator.
- PASSWORD: The iLO administrator password.
- HOSTNAME: The iLO hostname.
- DOMAIN:   Your registered domain name.

Other options with defaults include:
- STAGING:  While finding your way around this resource, you're encouraged to set STAGING to 1 to avoid hitting Let's Encrypt rate limits. The default is 0.
- DNSAPI:   A supported DNS provider for automatic DNS API integration https://github.com/acmesh-official/acme.sh/wiki/dnsapi. The default is Cloudflare (`dns_cf`). To use a different provider, for instance, Amazon Route53, set `DNSAPI="dns_aws"` in `hpilo.cfg`.
- ACMESH_DIR:  This is the directory in which the `acme.sh` script lives.  Default is `/root/.acme.sh`.
- SCRIPT_BASEDIR:  This is where the script will place the host-specific CSR, config, and script files.  Defaults to your current working directory when you run the script.

3. If this is your first deployment, set up the API credentials for your DNS provider https://github.com/acmesh-official/acme.sh/wiki/dnsapi, but do not issue a certificate just yet! For example, for Cloudflare:
```
export CF_Token="sdfsdfsdfljlbjkljlkjsdfoiwje"
export CF_Account_ID="xxxxxxxxxxxxx"
```
When a certificate is first issued, `CF_Token` and `CF_Account_ID` are saved in `$ACMESH_DIR/account.conf` and used for subsequent deployments.

4. Run the helper script `bash hpilo.sh` to issue and deploy a Let's Encrypt certificate to the iLO. 
5. Repeat the above steps for other iLOs on your network.

To list all issued certificates `acme.sh --list`. Acme.sh will manage the renewal and deployment of the certificates.
