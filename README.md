# Odoo service for Kubernetes on Wodby

Build and run Odoo Community Edition on Kubernetes with Wodby.

This repository defines the Wodby service manifests and operational
configuration for Odoo.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Start with a boilerplate

Use one of the boilerplates exposed by this service to start with compatible
build configuration and Wodby CI:

- [Odoo modules boilerplate](https://github.com/wodby/odoo-boilerplate)

## Wodby stacks using this service

- [Odoo application stack](https://github.com/wodby/stack-odoo)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `odoo` |
| Type | Application service |
| Image | [`wodby/odoo`](https://github.com/wodby/odoo) |
| Versions | `20` by default, `19`, `18` |
| Endpoints | `odoo`: `http` 8069 and `websocket` 8072, both private |
| Service links | PostgreSQL, required; Odoo data storage, required; SMTP server |
| Settings | Admin login, Admin password (required), Admin email, Language, Apps to install, Workers, Scheduled action workers |
| Volumes | `filestore`, shared |
| Application build | Git source connection enabled; Dockerfile: `Dockerfile`; boilerplates: Odoo modules boilerplate |
| Operations | 2 actions |

## Use this service

Use this service through [Odoo application stack](https://github.com/wodby/stack-odoo), or reference `odoo` from a
custom Wodby stack together with the [Odoo Nginx service](https://github.com/wodby/service-odoo-nginx), which routes
live updates to Odoo's second port.

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id>
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).
