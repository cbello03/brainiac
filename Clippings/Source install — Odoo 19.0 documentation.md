---
title: "Source install — Odoo 19.0 documentation"
source: "https://www.odoo.com/documentation/19.0/administration/on_premise/source.html"
author:
  - "[[Authorize.Net]]"
published:
created: 2025-11-13
description:
tags:
  - "clippings"
---
## Source install

The source ‘installation’ is not about installing Odoo but running it directly from the source instead.

Using the Odoo source can be more convenient for module developers as it is more easily accessible than using packaged installers.

It makes starting and stopping Odoo more flexible and explicit than the services set up by the packaged installers. Also, it allows overriding settings using [command-line parameters](https://www.odoo.com/documentation/19.0/developer/reference/cli.html#reference-cmdline) without needing to edit a configuration file.

Finally, it provides greater control over the system’s setup and allows to more easily keep (and run) multiple versions of Odoo side-by-side.

## Fetch the sources

There are two ways to obtain the source code of Odoo: as a ZIP **archive** or through **Git**.

### Archive

Community edition:

- [Odoo download page](https://www.odoo.com/page/download)
- [GitHub Community repository](https://github.com/odoo/odoo)
- [Nightly server](https://nightly.odoo.com/)

Enterprise edition:

- [Odoo download page](https://www.odoo.com/page/download)
- [GitHub Enterprise repository](https://github.com/odoo/enterprise)

### Git

To clone a Git repository, choose between cloning with HTTPS or SSH. In most cases, the best option is HTTPS. However, choose SSH to contribute to Odoo source code or when following the [Getting Started developer tutorial](https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101.html).

## Prepare

### Python

Odoo requires **Python 3.10** or later to run.

Changed in version 17: Minimum requirement updated from Python 3.7 to Python 3.10.

### PostgreSQL

Odoo uses PostgreSQL as its database management system.

Changed in version 19: Minimum requirement updated from PostgreSQL 12 to PostgreSQL 13.

By default, the only user is `postgres`. As Odoo forbids connecting as `postgres`, create a new PostgreSQL user.

### Dependencies

## Running Odoo

Once all dependencies are set up, Odoo can be launched by running `odoo-bin`, the command-line interface of the server. It is located at the root of the Odoo Community directory.

To configure the server, either specify [command-line arguments](https://www.odoo.com/documentation/19.0/developer/reference/cli.html#reference-cmdline-server) or a [configuration file](https://www.odoo.com/documentation/19.0/developer/reference/cli.html#reference-cmdline-config).

Common necessary configurations are:

- PostgreSQL user and password.
- Custom addon paths beyond the defaults to load custom modules.

A typical way to run the server would be:

After the server has started (the INFO log `odoo.modules.loading: Modules loaded.` is printed), open [http://localhost:8069](http://localhost:8069/) in a web browser and log into the Odoo database with the base administrator account: use `admin` as the email and, again, `admin` as the password.

[Edit on GitHub](https://github.com/odoo/documentation/edit/19.0/content/administration/on_premise/source.rst)