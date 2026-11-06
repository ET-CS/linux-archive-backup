# Linux Archive Backup

Create rotating `.tar.gz` archives of selected directories and optional MySQL dumps on Linux.

> For experienced users. Read the script and these instructions carefully, and use at your own risk. Root access may be required.

## Features

- Backup selected directories and files
- Dump MySQL databases
- Compress each session into `.tgz` archives
- Keep a fixed number of rotated copies (6 by default)
- Store each run in a numbered folder (`backup/0` … `backup/N`)
- Optional remote destination via SMB/CIFS

## Installation

1. Clone or copy the project to your preferred directory:

   ```bash
   mkdir -p /scripts
   cd /scripts
   git clone <your-repo-url> linux-archive-backup
   cd linux-archive-backup
   ```

2. Create settings from the example and edit them:

   ```bash
   cp settings.cfg.example settings.cfg
   ```

   Set MySQL credentials if you use DB backups, and set `workdir` to this script’s location (for example `/scripts/linux-archive-backup`).

## Configure backups

Create `lst/folders.lst` with the paths to archive (space-separated on one line):

```text
/var/log/ /var/www/ /usr/files/ /tftpd/
```

Optional MySQL list file: `lst/db.lst` (one database name per line). When `BACKUP_MYSQL=true`, the script can also refresh that list from the server.

## Remote backup (SMB/CIFS)

Disabled by default (`BACKUP_REMOTELY=false`).

To enable:

1. Set `BACKUP_REMOTELY=true` in `settings.cfg`
2. Configure `REMOTE_HOST`, `REMOTE_SHARE`, `REMOTE_USER`, and `REMOTE_PASS`
3. The share is mounted at `$backupdir` for the run
4. Leave `unmountremote=true` to unmount when finished (default)

The run fails if the mount settings are wrong — fix whatever the terminal reports before retrying.

## How backups are stored

By default, archives land under `$workdir/backup`, one subdirectory per generation:

- `BACKUP_COPIES` controls how many older copies to keep (default `6`)
- Newest backup is always `backup/0`
- On each run, folders shift: `0` → `1`, `1` → `2`, … and the oldest slot is removed
- Override the destination with the `backupdir` setting if needed

## Enable the script

The script starts in test mode (`WRITE_CHANGES=false`) and does not write backups until you turn that off.

1. Configure settings and run once in test mode
2. Confirm the console output looks right
3. Set `WRITE_CHANGES=true` in `settings.cfg`

You can also set `DISABLED=true` to skip runs without removing cron.

## Cron

Example daily run at midnight:

```bash
crontab -e
```

```cron
0 0 * * * /scripts/linux-archive-backup/backup.sh
```

Settings can also be loaded from `/etc/backup.cfg` if that file exists; otherwise `settings.cfg` next to the script is used.

Authors: [RaveMaker](http://ravemaker.net) & [ET](http://etcs.me).
