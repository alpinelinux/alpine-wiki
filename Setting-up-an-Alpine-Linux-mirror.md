[[_TOC_]]

## Introduction

This will guide you how you can setup your personal or official Alpine Linux mirror.

## Cron bassed syncing

This is the easiest method of syncing contents from one of our mirrors.

### Install rsync

`apk add rsync`

### Setup sync script

Add the following script to  /etc/periodic/hourly/sync-mirror

```sh
#!/bin/sh

# make sure we never run 2 rsync at the same time
lockfile="/tmp/alpine-mirror.lock"
if [ -z "$flock" ] ; then
  exec env flock=1 flock -n $lockfile "$0" "$@"
fi

src=rsync://rsync.alpinelinux.org/alpine/ 
dest=/var/www/localhost/htdocs/alpine/

# uncomment this to exclude old v2.x branches
#exclude="--exclude v2.*"

mkdir -p "$dest"
/usr/bin/rsync \
        --archive \
        --update \
        --hard-links \
        --delete \
        --delete-after \
        --delay-updates \
        --timeout=600 \
        $exclude \
        "$src" "$dest"
```

Make the script executable

`chmod +x /etc/periodic/hourly/sync-mirror`

## Setting up lighttpd

These are basic steps on how to setup lighttpd to serve your mirror.
Alternate httpd solutions are:

* darkhtttpd
* nginx

### Adding lighttpd

`apk add lighttpd`

### Modify lighttpd config file

Modify /etc/lighttpd/lighttpd.conf:

#### Enable dirlisting

```
dir-listing.activate      = "enable"
```

#### Cache control

Also set cache-control to force cache re-validate every 30 mins

```
"mod_setenv"
```

```
setenv.add-response-header += (           
        "Cache-Control" => "must-revalidate"
)
```

#### Start ligthtpd

`rc-service lighttpd start`

`rc-update add lighttpd`

## Setting up rsyncd

Add the following to /etc/rsyncd.conf

```
[alpine]
        path = /var/www/localhost/htdocs/alpine
        comment = My Alpine Linux Mirror
```

Optionally limit the rsyncd speed by adding following to /etc/conf.d/rsyncd

```
RSYNC_OPTS="--bwlimit=500"
```

## Mirror statistics

Simple mirror statistics can be generated by vnstat.

### Adding vnstat

`apk add vnstat`

### Modify configuration

Modify /etc/vnstat.conf to change default interface

```
Interface "eth0"
```

### Starting vnstatd

`rc-service vnstatd start`

`rc-update add vnstatd`

### Generate statistics

copy the following script to /etc/periodic/15min/stats

```sh
#!/bin/sh

output="/var/www/localhost/htdocs/.stats"
nic="eth0"

generate_index() {
    cat <<-EOF
    <!doctype html>
    <html lang="en">
    <head>
        <meta charset="utf-8">
        <meta http-equiv="cache-control" content=no-cache">
        <meta http-equiv="refresh" content="3000">
        <title>Alpine Linux mirror statistics</title>
    </head>
    <body>
        <table border="0">
            <tr><td><img src="summary.png" alt="summary"></td><td><img src="hours.png" alt="hours"></td></tr>
            <tr><td rowspan="2"><img src="days.png" alt="days"></td><td><img src="top10.png" alt="top10"></td></tr>
            <tr><td><img src="months.png" alt="months"></td></tr>
        </table>
    </body>
    </html>
    EOF
}

if  [ ! -f "$output"/index.html ]; then
    mkdir -p $output
    generate_index > "$output"/index.html
fi

for type in hours days months top10 summary hsummary vsummary; do
    vnstati --${type} -i $nic -o $output/${type}.png
done
```

# Updating via MQTT

### adding mqtt-exec

`apk add mqtt-exec`

### Copy/link the OpenRC scripst

mqtt-exec can run multiple times so we need to duplicate the initd/confd script to support this

`ln -s /etc/init.d/mqtt-exec /etc/init.d/mqtt-exec.sync-mirror`

`cp /etc/conf.d/mqtt-exec /etc/conf.d/mqtt-exec.sync-mirror`

### Edit confd

edit /etc/conf.d/mqtt-exec.sync-mirror and add

```
mqtt_topics="rsync/rsync.alpinelinux.org/#"
exec_user="buildozer"
exec_command="/usr/local/bin/sync-mirror"
```

### Create sync script

Copy the following file to /usr/local/bin/sync-mirror and make it executable (dont forget to update the variables).

```sh
#!/bin/sh

src="rsync://rsync.alpinelinux.org/alpine/"
dest="/var/www/localhost/htdocs/alpine/"
lock="/tmp/sync-mirror.lock"
topic="$1"
dir="$2"

[ -z "$flock" ] && exec env flock=1 flock $lock $0 "$@"

logger "Syncing directory: $dir"

if [ -n "$dir" ] && [ -d "$dest/${dir%/*}" ]; then
    src="${src}${dir%/}/"
    dest="${dest}${dir%/}/"
fi

/usr/bin/rsync -ua \
    --timeout=600 \
    --delay-updates \
    --delete-after \
    "$src" \
    "$dest"
```

### Start mqtt-exec

`rc-service mqtt-exec start`

`rc-update add mqtt-exec`

Now watch your syslog for sync messages
