# beaststation-docker-compose

Docker compose files for my server (Beaststation).

## Required Environment Variables

- `DOCKER_PW`
  - password for docker repo
- `TELEGRAM_WATCHTOWER_TOKEN`
  - telegram token for watchtower bot
- `DB_PW`
  - password for databases
- `NEXTCLOUD_ADMIN_PASSWORD`
  - admin password for nextcloud
- `NEXTCLOUD_SMTP_PASSWORD`
  - password for <mail@nextcloud.dominik-schwaiger.ch>
- `JWT_SECRET`
  - secret for jwt's (onlyoffice)
- `WEBHOOK_SECRET`
  - github webhook secreto for compose-watcher
- `WATCHER_TELEGRAM_TOKEN`
  - telegram token for compose-watcher telegram bot
- `WATCHER_CHAT_ID`
  - chat id for compose watcher
- `BW_INSTALLATION_ID`
  - get from <https://bitwarden.com/host/>
- `BW_INSTALLATION_KEY`
  - get from <https://bitwarden.com/host/>
- `SCHWAIGER_ADMIN_PASSWORD`
  - password to enter admin panel of <https://dominik-schwaiger.ch>
- `GITLAB_SMTP_PASSWORD`
  - email password for gitlab

## Volumes

- `/mnt/raid5/openvpn`
- `/mnt/raid5/nextcloud/data`
- `/mnt/raid5/nextcloud/apps`
- `/mnt/raid5/nextcloud/config`
- `/mnt/raid5/nextcloud/themes`
- `/mnt/raid5/nextcloud/database`
- `/mnt/raid5/minecraft/server`
- `/mnt/raid5/minecraft/backups`
- `/mnt/raid5/portainer/data`
- `/mnt/raid5/bitwarden/data`
- `/mnt/raid5/bitwarden/database`
- `/mnt/raid5/bitwarden/logs`
- `/mnt/raid5/dominik-schwaiger.ch/images`
- `/mnt/raid5/gitlab/runner/config`
- `/mnt/raid5/gitlab/logs:/var/log/gitlab`
- `/mnt/raid5/gitlab/config:/etc/gitlab`
- `/mnt/raid5/gitlab/data:/var/opt/gitlab`
- `/var/run/docker.sock`
- `/home/domina/beaststation-docker-compose/proxy/nginx_config`
- `/home/domina/beaststation-docker-compose`
- `/home/domina/.ssh`

## Ports

- 80 (nginx)
- 443 (nginx)
- 25565 (Minecraft)
- 1194 (OpenVPN)
- 22 (Gitlab)
