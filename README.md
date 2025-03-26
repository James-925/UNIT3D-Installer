<h1 align="center">UNIT3D Community Edition Installer</h1>

<p align="center">
    🎉<b>A Big Thanks To All Our <a href="https://github.com/HDInnovations/UNIT3D-Community-Edition/graphs/contributors">Contributors</a> and <a href="https://github.com/sponsors/HDVinnie">Sponsors</a></b>🎉
</p>

<p align="center"><b>NOTE: This only works for a fresh server with nothing on it but a new OS install!</b></p>

## This Repository
Installer for the [UNIT3D-Community-Edition](https://github.com/HDInnovations/UNIT3D-Community-Edition).

**Officially Supported OS's**
- Ubuntu 22.04 LTS (Jammy Jellyfish)
- Ubuntu 20.04 LTS (Focal Fossa)

**Unstable WIP**
- Ubuntu 24.04 LTS (Noble Numbat)


**We offer install and tuning services for a small price if not comfortable installing and tuninng server yourself. Otherwise if want to install yurself run commannd below.**

**To install run the following:** (and follow the instructions. must be a fresh deicated server with nothing on it besides supported OS. Also must have a proper valid domain pointing to your server IP via A RECORD and CNAME for www)
```
sudo apt -y install git
git clone https://github.com/HDInnovations/UNIT3D-Installer.git installer
cd installer
sudo ./install.sh
```
**Install and Configure Meilisearch:**
1. Install Meilisearch:
```
curl -L https://install.meilisearch.com | sh
sudo mv ./meilisearch /usr/local/bin/
```

2. Set Up Directories:
```
sudo mkdir -p /var/lib/meilisearch/data /var/lib/meilisearch/dumps /var/lib/meilisearch/snapshots
sudo chown -R ubuntu:ubuntu /var/lib/meilisearch
sudo chmod -R 750 /var/lib/meilisearch
```

3. Generate and Record a Master Key:
```
openssl rand -hex 16
```
Record this key, as it will be used in the configuration file.

4. Configure Meilisearch:
```
sudo curl https://raw.githubusercontent.com/meilisearch/meilisearch/latest/config.toml -o /etc/meilisearch.toml
sudo nano /etc/meilisearch.toml
```
Update the following in /etc/meilisearch.toml:

```
env = "production"
master_key = "your_16_byte_master_key"
db_path = "/var/lib/meilisearch/data"
dump_dir = "/var/lib/meilisearch/dumps"
snapshot_dir = "/var/lib/meilisearch/snapshots"
```
5. Create and Enable Service:
```
sudo nano /etc/systemd/system/meilisearch.service
```
Add the following:
```
[Unit]
Description=Meilisearch
After=systemd-user-sessions.service

[Service]
Type=simple
WorkingDirectory=/var/lib/meilisearch
ExecStart=/usr/local/bin/meilisearch --config-file-path /etc/meilisearch.toml
User=ubuntu
Group=ubuntu
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Enable and start the service:
```
sudo systemctl enable meilisearch
sudo systemctl start meilisearch
sudo systemctl status meilisearch
```

6. Configure UNIT3D for Meilisearch
Update .env:
```
sudo nano /var/www/html/.env
```
Add the following:
```
SCOUT_DRIVER=meilisearch
MEILISEARCH_HOST=http://127.0.0.1:7700
MEILISEARCH_KEY=your_16_byte_master_key
Clear Configuration and Restart Services:
```
Clear Configuration and Restart Services (make sure you're inside /var/www/html):
```
sudo php artisan set:all_cache
sudo systemctl restart php8.3-fpm
sudo php artisan queue:restart
```

7. Finally run these comands while inside your UNIT3D folder (cd /var/www/html):
```
sudo php artisan scout:sync-index-settings && sudo php artisan auto:sync_torrents_to_meilisearch --wipe && sudo php artisan auto:sync_people_to_meilisearch
```


**NOTE: If you are running UNIT3D-Community-Edition on a non HTTPS instance you MUST change the following configs.**
```
.env  <-- SESSION_SECURE_COOKIE must be set to false
config/secure-headers.php   <-- HTTP Strict Transport Security must be set to false
config/secure-headers.php   <-- Content Security Policy must be disabled
```

### Suggestions and/or Bug Reporting
We encourage the use of [GitHub Issues](https://github.com/HDInnovations/UNIT3D-INSTALLER/issues/new)!
