# nucleihubquery

A small bash utility that extracts search dorks (`shodan-query`, `google-query`, `censys-query`, `fofa-query`, `hunter-query`, `zoomeye-query`) from the `nucleihub-templates` YAML collection and writes them into organized files.

This is useful when you want to build lists of reconnaissance/search queries (dorks) from vulnerability templates so you can run targeted scans or feed them into asset discovery tools.


## Features

* Scans all YAML templates under `nucleihub-templates`.
* Extracts the following metadata fields when present:

  * `shodan-query`
  * `google-query`
  * `censys-query`
  * `fofa-query`
  * `hunter-query`
  * `zoomeye-query`
* Produces one aggregated file per provider, plus per-severity files for each provider (`critical`, `high`, `medium`, `low`, `info`).
* Removes duplicate queries and filters out empty/null values.


## Requirements

* `bash` (POSIX shell)
* `yq` (YAML processor) — required to extract YAML fields
* `sed`, `egrep`, `find` — standard Unix utilities
* `unew` — used in the script to write unique lines quietly. If you don't have `unew`, you can substitute with `sort -u` or `awk` (see notes below).

> Note: If you use `sort -u` instead of `unew`, you may lose original ordering but will still deduplicate lines. Example replacement:
>
> ```bash
> ... | sort -u > shodan/shodan-query-all.txt
> ```


## Installation

There are two installation options shown in the repository README.

**Clone and install locally**

```bash
git clone https://github.com/rix4uni/nucleihubquery.git
cd nucleihubquery
chmod +x nucleihubquery
sudo mv nucleihubquery /usr/bin/
```

**Download single-file installer**

```bash
wget -q -O /usr/bin/nucleihubquery \
  https://raw.githubusercontent.com/rix4uni/nucleihubquery/refs/heads/main/nucleihubquery && \
  chmod +x /usr/bin/nucleihubquery
```

After installation, run with:

```bash
nucleihubquery
```

The script will create directories and write files in the current working directory where `nucleihub-templates` exists.


## What the script does (expanded)

1. Creates directories for each provider:

   * `shodan`, `google`, `censys`, `fofa`, `hunter`, `zoomeye`
2. Extracts **all** queries of each type and writes them to `*-query-all.txt` (one per provider).
3. For each severity level (`critical`, `high`, `medium`, `low`, `info`), it selects templates with that severity and writes provider-specific severity files, e.g. `shodan/shodan-query-critical.txt`.
4. Uses `yq` to read YAML paths like `.info.metadata."shodan-query"` from each YAML template.
5. Cleans output using `sed` to strip YAML list markers and remove `null` or ` items, and uses `unew` (or `sort -u`) to deduplicate.


## Example template (how fields are found)

A `nucleihub-templates` YAML snippet may look like this:

```yaml
id: unauthenticated-jenkins-rce

info:
  name: Unauthenticated Jenkins RCE Dashboard
  author: rix4uni
  severity: critical
  metadata:
    google-query:
      - intitle:"Dashboard [Jenkins]"
      - intitle:"Script Console [Jenkins]"
    shodan-query:
      - http.title:"Dashboard [Jenkins]"
      - http.title:"Script Console [Jenkins]"
    censys-query:
      - services.http.response.html_title="Dashboard [Jenkins]"
      - services.http.response.html_title="Script Console [Jenkins]"
    fofa-query:
      - title="Dashboard [Jenkins]"
      - title="Script Console [Jenkins]"
    hunter-query:
      - web.title="Dashboard [Jenkins]"
      - web.title="Script Console [Jenkins]"
```

The script reads these `*.yaml` files and extracts the list items under the provider keys.


## Output layout

After running `nucleihubquery` you will have a directory structure like this:

```yaml
shodan/
  shodan-query-all.txt
  shodan-query-critical.txt
  shodan-query-high.txt
  shodan-query-medium.txt
  shodan-query-low.txt
  shodan-query-info.txt
google/
  google-query-all.txt
  google-query-critical.txt
  ...
censys/
  censys-query-all.txt
  ...
fofa/
hunter/
zoomeye/
```

Each file contains one query per line.


## Troubleshooting

* If `yq` is missing or version incompatible, install it from: [https://github.com/mikefarah/yq](https://github.com/mikefarah/yq) (or via your package manager).
* If `unew` is missing, replace `unew -q` with `sort -u` or `awk '!seen[$0]++'` in the script.
* Ensure `nucleihub-templates` is present in the working directory and contains valid YAML files.

