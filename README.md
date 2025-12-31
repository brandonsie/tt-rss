Tiny Tiny RSS (tt-rss)
======================

Tiny Tiny RSS (tt-rss) is a free, flexible, open-source, web-based news feed (RSS/Atom/other) reader and aggregator.

## About this fork
* this is my personal fork of tt-rss/tt-rss. The functional change is that published articles are protected from purge (equivalent to how starred articles are not subject to purge). I want this behavior becuase I use star and publish to mark articles as interesting/uninteristing and use this data to train a model that scores interestingness of future articles.

steps taken to create this fork from original
* clone the source: 
```bash
git clone https://github.com/tt-rss/tt-rss.git
cd tt-rss
git checkout main
```
* make changes to the purge function. for me this was in. add a protection for published articles
classes/Feeds.php
```php
			$sth = $pdo->prepare("DELETE FROM ttrss_user_entries
				USING ttrss_entries
				WHERE ttrss_entries.id = ref_id AND
				marked = false AND
				published = false AND
				feed_id = ? AND
				$query_limit
				ttrss_entries.date_updated < NOW() - INTERVAL '$purge_interval days'");
			$sth->execute([$feed_id]);
```

* create a feature branch
```bash
git checkout -b protect-published
git commit -am "Protect published articles from purge"
```

* long term maintenance
```bash
git remote add upstream https://github.com/tt-rss/tt-rss.git
git fetch upstream
git checkout protect-published
git rebase upstream/main

```

create dockerfile at level of ttrss docker compose `Dockerfile.ttrss`
new
```Dockerfile
FROM ghcr.io/tt-rss/tt-rss:latest

RUN apk add --no-cache git

WORKDIR /var/www/html

RUN rm -rf tt-rss \
 && git clone https://github.com/brandonsie/tt-rss.git tt-rss \
 && cd tt-rss \
 && git checkout protect-published
```
move local plugins and htemes to /opt/
create config.d/10-local.php
```php
<?php
define('PLUGINS_LOCAL', '/opt/tt-rss/plugins.local');
define('THEMES_LOCAL',  '/opt/tt-rss/themes.local');
```

rebuild
```bash
docker compose down
docker volume rm ttrss_html # remove old volume
docker compose build --no-cache
docker compose up -d
```


sanity check
```bash
docker compose exec app   grep "marked = false" /var/www/html/tt-rss/classes/Feeds.php -A 5 -B 5


# should see ttrss_user_entries.published = false
```

later, to update
```
docker compose pull
docker compose build --no-cache ttrss
docker compose up -d
```


## Getting started

Please refer to [the installation guide](https://tt-rss.org/docs/Installation-Guide.html).

## Some notes about this project

* The original tt-rss project, hosted at https://tt-rss.org/ and its various subdomains, was retired on 2025-11-01.
  * Massive thanks to fox for creating tt-rss, and maintaining it (and absolutely everything else that went along with it) for so many years.
* This project (https://github.com/tt-rss/tt-rss) is a fork of tt-rss as of 2025-10-03, created by one of its long-time contributors (`wn_`/`wn_name` on `tt-rss.org`, `supahgreg` on `github.com`).
  * The goal is (as you might expect) to continue tt-rss development.
  * No major breaking changes are planned.
  * Like the original project:
    * The minimum PHP version supported by tt-rss will match [what's in Debian's current `stable` release](https://packages.debian.org/stable/php).
    * What's on the `main` branch (or `latest` and the most recent `sha-*` tag for the Docker images) is intended to be stable
      and safe for use.  Like all software, however, bugs sometimes slip through; the goal is to address those bugs promptly.
    * Using the latest code/image is strongly encouraged, and may be a prerequisite to getting support in certain situations.
  * Developer note: Due to use of `invalid@email.com` on `supahgreg`'s pre-2025-10-03 commits (which were done on `tt-rss.org`) GitHub incorrectly shows `ivanivanov884`
    (the GitHub user associated with that e-mail address) as the author instead of `wn_`/`supahgreg`.  Apologies for any confusion.  `¯\_(ツ)_/¯`
* Docker images (for `linux/amd64` and `linux/arm64`; drop-in replacements for the old images;
  see [the installation guide](https://tt-rss.org/docs/Installation-Guide.html)) are being built and published
  ([via GitHub Actions](https://github.com/tt-rss/tt-rss/actions/workflows/publish.yml)) to:
  * Docker Hub (as [supahgreg/tt-rss](https://hub.docker.com/r/supahgreg/tt-rss/) and [supahgreg/tt-rss-web-nginx](https://hub.docker.com/r/supahgreg/tt-rss-web-nginx/)).
  * GitHub Container Registry (as [ghcr.io/tt-rss/tt-rss](https://github.com/orgs/tt-rss/packages/container/package/tt-rss)
    and [ghcr.io/tt-rss/tt-rss-web-nginx](https://github.com/orgs/tt-rss/packages/container/package/tt-rss-web-nginx)).
* Documentation from https://tt-rss.org has been recreated in https://github.com/tt-rss/tt-rss.github.io,
  which is the new source for https://tt-rss.org content.
  * The original project's repository that held content for https://tt-rss.org was mirrored to https://github.com/tt-rss/tt-rss-web-static .
    Some content tweaks were made after mirroring (prior to the new repository being set up), and the repository is now archived.
* Plugins that were under https://gitlab.tt-rss.org/tt-rss/plugins have been mirrored to `https://github.com/tt-rss/tt-rss-plugin-*`.
  * Plugin repository names have changed to get a consistent `tt-rss-plugin-*` prefix.

## Development and contributing

Contributions (code, translations, reporting issues, etc.) are welcome. Please see [CONTRIBUTING.md](CONTRIBUTING.md) for more information.

## License

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.

Copyright (c) 2005 Andrew Dolgov (unless explicitly stated otherwise).
