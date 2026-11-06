# Avunu mail addons

Email and SMS transports for Odoo: email sent and received through Cloudflare with no SMTP or IMAP provider in the loop, and two-way SMS through Twilio, one Discuss thread per contact. They are Odoo 18.0 modules built and maintained by [Avunu LLC](https://avunu.net) for our own client work and published for anyone to use. The repository is named after [OCA/mail](https://github.com/OCA/mail), where OCA keeps the `mail_gateway` framework from 19.0; on 18.0 that framework, which the Twilio modules build on, is in [OCA/social](https://github.com/OCA/social).

## Modules

| Module | What it does | Licence |
| --- | --- | --- |
| [`mail_cloudflare`](mail_cloudflare/README.md) | Sends mail through the Cloudflare Email Sending API and receives it from the Cloudflare email relay over a signed webhook; no SMTP or IMAP provider in the loop | AGPL-3.0-or-later |
| [`mail_gateway_twilio`](mail_gateway_twilio/README.md) | Two-way SMS through Twilio, with inbound MMS media as attachments, one Discuss thread per contact, built on the OCA `mail_gateway` framework | AGPL-3.0-or-later |
| [`mail_gateway_twilio_sms_mirror`](mail_gateway_twilio_sms_mirror/README.md) | Mirrors Odoo's outbound core and marketing SMS into the matching Twilio gateway threads so replies sit next to what they answer | AGPL-3.0-or-later |

Each module's README explains why it exists and how it works.

## Branches

`develop` is the default branch and the one pull requests go to. Its code targets Odoo 18.0, the series named in `.odoo-series`. `18.0` follows `develop` automatically (a workflow fast-forwards it after every push) and is the branch to install or pin. When a newer Odoo series is maintained, it gets its own branch (`19.0`, …) and changes flow up to it from `develop`.

## Install

Requires Odoo 18.0 and a PostgreSQL database. Pick the module or modules you want; each pulls in the modules it depends on (see below), and those must be on your `addons_path` first.

With a plain Odoo checkout, clone this repository on its `18.0` branch and add it to the addons path:

```sh
git clone --branch 18.0 https://github.com/Avunu/mail-addons.git
odoo-bin --addons-path=odoo/addons,/path/to/mail-addons,<your other addons paths> \
  -d <database> -i <module> --stop-after-init
```

With [odoo-nix](https://github.com/Avunu/odoo-nix), add the repository from the dev shell. `odoo module add` accepts an `owner/repo` and a branch, adds it as a submodule, and the addons path is regenerated from the folders present (run `direnv reload` afterwards):

```sh
odoo module add Avunu/mail-addons 18.0
```

OCA dependencies can be added by module name, which also adds the OCA repository that holds them: `odoo module add mail_gateway` for the Twilio modules.

## Dependencies

Each module's `__manifest__.py` is the source of truth. Besides Odoo core modules (`mail`, `phone_validation`, `sms`), the modules depend on the following. The OCA repository listed is the 18.0 branch; clone it if your chosen modules need it, along with whatever its modules need in turn (their own manifests list it).

| Module | Needs from OCA or others | Python packages |
| --- | --- | --- |
| `mail_cloudflare` | none beyond core | none |
| `mail_gateway_twilio` | `mail_gateway` ([OCA/social](https://github.com/OCA/social)) | none |
| `mail_gateway_twilio_sms_mirror` | `mail_gateway_twilio` (this repository) | none |

`mass_mailing_sms` (SMS Marketing) is a soft dependency of `mail_gateway_twilio_sms_mirror`: its messages are mirrored when it is installed, and it is not required.

## Development

The repository carries its own development environment, built by [odoo-nix](https://github.com/Avunu/odoo-nix) in addons mode. OCB and the repositories these modules depend on are flake inputs pinned in `flake.lock`, and the Python environment is locked in `nix/`. You need Nix with flakes, and optionally direnv.

```sh
git clone https://github.com/Avunu/mail-addons.git && cd mail-addons
direnv allow                  # or: nix develop --no-pure-eval
devenv up                     # PostgreSQL, Odoo and Mailpit
odoo db provision             # a database with every module here installed
odoo test mail_cloudflare
```

After `nix flake update`, or when a module gains a `depends` or a Python requirement, run `odoo project relock` and commit `flake.lock` and `nix/`. A new dependency on another repository also needs its input and an `externalSources` entry in `flake.nix`. See odoo-nix's [A repository of addons](https://github.com/Avunu/odoo-nix/blob/main/docs/scaffolding/addons-repository.md).

## Tests

Odoo runs a module's tests when you install or update it with `--test-enable`. Select a module's tests with a `--test-tags` filter and use a throwaway database:

```sh
odoo-bin -c odoo.conf -d <testdb> -i <module> \
  --test-enable --test-tags /<module> --stop-after-init
```

All three modules have automated tests. Notes from the module READMEs:

- `mail_cloudflare` tests need the dev mail catcher off in an odoo-nix shell, because it is server-wide and would otherwise swallow every send the tests make: prefix the command with `ODOO_MAILCATCH_ENABLED=0`. The tests never contact Cloudflare: the REST responses are scripted and every request is captured, and the webhook tests sign real HTTP requests against the running test server.

## Part of the Cloudflare Email suite

Four open-source projects work together to give business systems email without SMTP credentials or IMAP polling:

| Project | Role |
| --- | --- |
| [cloudflare-email-relay](https://github.com/Avunu/cloudflare-email-relay) | Multi-tenant inbound Worker: stores each message in R2, then delivers it to the right system with signed, retried requests |
| [cloudflare_email_delivery](https://github.com/Avunu/cloudflare_email_delivery) | Frappe and ERPNext adapter |
| [mail_cloudflare](https://github.com/Avunu/mail-addons/tree/develop/mail_cloudflare) | Odoo adapter |
| [wordpress-cloudflare-email](https://github.com/Avunu/wordpress-cloudflare-email) | WordPress plugin: outbound mail and a delivery log |

Need it set up for your business? [Avunu](https://avunu.net) can help.

## Licence

Every module is licensed under the [GNU Affero General Public License v3.0 or later](LICENSE), as its `__manifest__.py` also states (`AGPL-3`, Odoo's identifier for version 3).

## Provenance

`mail_cloudflare`, `mail_gateway_twilio` and `mail_gateway_twilio_sms_mirror` moved here, with their history, from [Avunu/avunu-odoo-addons](https://github.com/Avunu/avunu-odoo-addons) in November 2026, when Avunu split that repository into one repository per OCA area.
