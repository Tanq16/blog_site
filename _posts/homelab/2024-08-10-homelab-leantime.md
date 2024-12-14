---
title: Leantime in Home Lab
date: 2024-08-10 13:37:19 -0400
categories: [Home Server]
tags: [time-tracking,project-management,leantime,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

[Leantime](https://leantime.io/) is an open-source project management solution that can be considered an alternative to Asana, Monday.com, or ClickUp. Leantime provides project-based task management using Kanban boards, tabular, and calendar views. Additionally, it supports tracking milestones and goals for a project through Gantt charts. The most amazing feature is support for time tracking, where users can easily track time against a given to-do until its completion. This allows amazing overall time management metrics for projects, making it easy for retrospectives.

It supports various authentication mechanisms and notification integration with Slack or Mattermost. It also supports multiple user roles, which make it easy to assign tasks to specific individuals. All metrics are easily exportable to CSV, making it easy to ingest into other analytics platforms.

Like always, doing this in a home lab setup is best when spun up using a container. I use [Dockge](https://dockge.kuma.pet/) to control my home lab setup with Docker. To begin the installation, first, perform the following commands for setting up data directories on the host for persistence and backups &rarr;

```bash
mkdir -p $HOME/leantime/{puserfiles,userfiles,plugins}
```

With the directories ready, the service can be now started with the following Dockge stack template &rarr;

```yaml
version: "3.8"
services:
  leantime_db:
    image: mysql:8.0
    container_name: mysql_leantime
    volumes:
      - /home/tanq/leantime/db:/var/lib/mysql
    restart: unless-stopped
    env_file: .env
    networks:
      - leantime-net
    command: --character-set-server=UTF8MB4 --collation-server=UTF8MB4_unicode_ci
  leantime:
    image: leantime/leantime:latest
    container_name: leantime
    restart: unless-stopped
    env_file: .env
    networks:
      - leantime-net
    volumes:
      - /home/tanq/leantime/puserfiles:/var/www/html/public/userfiles
      - /home/tanq/leantime/userfiles:/var/www/html/userfiles
      - /home/tanq/leantime/plugins:/var/www/html/app/Plugins
    ports:
      - 5005:80
    depends_on:
      - leantime_db
networks:
  leantime-net: null
```

The `.env` file can also be specified with Dockge because of its amazing features. An example config is as follows &rarr;

```
LEAN_PORT = '5005'                             # The port to expose and access Leantime
LEAN_APP_URL = ''                              # Base URL, needed for subfolder or proxy installs
LEAN_APP_DIR = ''                              # Base of application without trailing slash (used for cookies), e.g, /leantime

LEAN_DEBUG = 0                                 # Debug flag

# Database - MySQL container
MYSQL_ROOT_PASSWORD = 'lEaNtImE'               # Database password
MYSQL_DATABASE = 'leantime'                    # Database name
MYSQL_USER = 'lean'                            # Database username
MYSQL_PASSWORD = 'lEaNtImE'                    # Database password

# Database - leantime container
LEAN_DB_HOST = 'mysql_leantime'                # Database host 
LEAN_DB_USER = 'lean'                          # Database username (needs to be the same as MYSQL_USER)
LEAN_DB_PASSWORD = 'lEaNtImE'                  # Database password (needs to be the same as MYSQL_PASSWORD)
LEAN_DB_DATABASE = 'leantime'                  # Database name (needs to be the same as MYSQL_DATABASE)
LEAN_DB_PORT = '3306'                          # Database port


## Optional Configuration, you may omit these from your .env file

## Default Settings
LEAN_SITENAME = 'Leantime'                     # Name of your site, can be changed later
LEAN_LANGUAGE = 'en-US'                        # Default language
LEAN_DEFAULT_TIMEZONE = 'America/Toronto'      # Set default timezone
LEAN_ENABLE_MENU_TYPE = false                  # Enable to specifiy menu on a project by project basis
LEAN_SESSION_PASSWORD = 'lEaNtImE'             # Salting sessions. Replace with a strong password
LEAN_SESSION_EXPIRATION = 288000               # How many seconds after inactivity should we logout?  28800seconds = 8hours
LEAN_LOG_PATH = null                           # Default Log Path (including filename), if not set /logs/error.log will be used

## Look & Feel, these settings are available in the UI and can be overwritten there.
LEAN_LOGO_PATH = '/images/logo.svg'            # Default logo path, can be changed later
LEAN_PRINT_LOGO_URL = '/images/logo.jpg'       # Default logo URL use for printing (must be jpg or png format)
LEAN_DEFAULT_THEME = 'default'                 # Default theme
LEAN_PRIMARY_COLOR = '#1b75bb'                 # Primary Theme color
LEAN_SECONDARY_COLOR = '#81B1A8'               # Secondary Theme Color

## Fileuploads

# Local File Uploads
LEAN_USER_FILE_PATH = 'userfiles/'             # Local relative path to store uploaded files (if not using S3)
LEAN_DB_BACKUP_PATH = 'backupdb/'              # Local relative path to store backup files, need permission to write


## Email
LEAN_EMAIL_RETURN = ''                         # Return email address, needs to be valid email address format
LEAN_EMAIL_USE_SMTP = false                    # Use SMTP? If set to false, the default php mail() function will be used
LEAN_EMAIL_SMTP_HOSTS = ''                     # SMTP host
LEAN_EMAIL_SMTP_AUTH = true                    # SMTP authentication required
LEAN_EMAIL_SMTP_USERNAME = ''                  # SMTP username
LEAN_EMAIL_SMTP_PASSWORD = ''                  # SMTP password
LEAN_EMAIL_SMTP_AUTO_TLS = true                # SMTP Enable TLS encryption automatically if a server supports it
LEAN_EMAIL_SMTP_SECURE = ''                    # SMTP Security protocol (usually one of: TLS, SSL, STARTTLS)
LEAN_EMAIL_SMTP_SSLNOVERIFY = false            # SMTP Allow insecure SSL: Don't verify certificate, accept self-signed, etc.
LEAN_EMAIL_SMTP_PORT = ''                      # Port (usually one of 25, 465, 587, 2526)
```

Obviously, replace the password fields!

Now, you have an excellent project management tool ready with a minimal operational cost. The UI is initially slow due to all the tips and views loading for the first time, but after 10-15 mins of using it and adding a project, it should be very responsive. Enjoy project management with Leantime!
