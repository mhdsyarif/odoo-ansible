# Odoo Ansible Playbook 🚀

An **Automated Ansible-based solution** to install and configure Odoo (Community Edition) on Linux servers using Docker Compose, including database setup, persistent data, reverse proxy via Nginx, and SSL provisioning with Certbot.

## 📁 Repository Structure


```plaintext
odoo-ansible/
├── inventory.ini
├── plays/
│   ├── odoo/
├── roles/
│   ├── docker/
│   │   ├── tasks/
│   ├── odoo/
│   │   ├── tasks/
│   │   ├── templates/
│   │   └── vars/
│   ├── nginx/
│   │   ├── tasks/
│   │   ├── templates/
│   │   └── vars/
└── README.md
```

## 🛠️ Features

* Install specific Odoo version via Docker Compose
* Deploy PostgreSQL container with persistent storage
* Auto-generate `odoo.conf` using Jinja2 templates
* Configure Nginx reverse proxy (with SSL)
* Set up Certbot for Let's Encrypt certificate
* Enable proxy mode, memory/time limits, and remote access
* Optionally initialize database with base module

## ✅ Prerequisites

* Ansible 2.9+
* Docker Engine & Docker Compose installed on target server
* SSH access to target host
* Domain name properly pointed to server IP (A record)

## ⚙️ Usage

### 1. Clone the repo

```bash
git clone https://github.com/mhdsyarif/odoo-ansible.git
cd odoo-ansible
```

### 2. Configure inventory

In `inventory.ini`, add your Odoo server details:

```yaml
[odoo]
[ip-server] ansible_user=[ansible_user] ansible_ssh_private_key_file=~/.ssh/[ansible_key].pem
```

### 3. Set variables

Set the following variables in `odoo/vars/main.yml`:

```yaml
POSTGRES_VERSION : 15
POSTGRES_USER : odoo
POSTGRES_PASSWORD : [POSTGRES_PASSWORD]
POSTGRES_DB : odoo

ODOO_VERSION : 18
ODOO_ADMIN_PASSWORD : [ODOO_ADMIN_PASSWORD]
ODOO_PORT : 8069
ODOO_HOST : db
ODOO_DB : 
DB_FILTER : .*
ODOO_USER : odoo
ODOO_PASSWORD : [ODOO_PASSWORD]

docker_networks: docker_networks
```

Set the following variables in `nginx/vars/main.yml`:

```yaml
domain_name: ["example.com"]
email: ["example@email.com"]
docker_nginx_dir: /opt/nginx
docker_networks: docker_networks
```

### 4. Run the playbook

```bash
ansible-playbook -i inventory.ini plays/odoo/setup.yml
```

This will:

* Create Docker network
* Launch PostgreSQL & Odoo containers
* Deploy Nginx with HTTP→HTTPS redirect
* Issue Let’s Encrypt SSL certificate via Certbot

## 🧹 Configuration Templates Highlights

* `odoo.conf.j2`: includes options like `proxy_mode = True`, `xmlrpc_interface = 0.0.0.0`, memory and timeout limits.
* `nginx.conf.j2`: configures upstream proxy to `odoo:8069` and handles SSL termination.
* `docker-compose.yml.j2`: orchestrates Odoo, PostgreSQL, Nginx, and Certbot services if included.

## 🔒 Security & Optimization

* `proxy_mode` ensures Odoo trusts headers from Nginx (like X-Forwarded-Proto, Host).
* `xmlrpc_interface = 0.0.0.0` allows external access (via Docker network) to Odoo.
* Limits (`limit_memory_soft`, `limit_time_cpu`, etc.) protect your 1 GB RAM / 1 vCPU environment.

## ✅ Post‑Deployment Notes

* Default admin password: defined in `admin_passwd` variable
* Access Odoo via `https://<your-domain>`
* Use `docker-compose logs` or `docker exec odoo bash` for troubleshooting
* Run periodic Certbot renewals via cron or schedule

## 🚧 Troubleshooting

| Issue                  | Fix                                                             |
| ---------------------- | --------------------------------------------------------------- |
| Odoo redirect to /odoo | Reset `web.base.url` in database; ensure no `/odoo` path prefix |
| Nginx returns 502/404  | Confirm Odoo is running in Docker and reachable via `odoo:8069` |
| Certificate not issued | Check DNS, firewall, Certbot webroot path `/.well-known/`       |

## 💡 Why This Playbook?

* Idempotent: re-run safely
* Template-based: reuse across staging/production
* Modular roles: clear separation for Odoo, Nginx, Certbot
* Docker-friendly: minimal host footprint, isolated setup

## 🔭 Future Enhancements

* Multi-instance support
* Odoo database backup automation
* Cloud DNS / Cloudflare integration
* Slack notification on deployment success
* Support for Odoo 18 and future versions

## ⚠️ License

MIT License — free to use and customize

## 📞 Contact & Support

For issues or contributions, feel free to create GitHub issues or reach out via [GitHub Profile](https://github.com/mhdsyarif).
