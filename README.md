# Junk Net

> Old laptops don't die. They join the Junk Net.

**Junk Net** is a community project that turns donated old laptops into
a free, community-owned, S3-compatible object store. Every donated
machine is wiped, health-checked, and imaged into a storage node that
joins an encrypted [Nebula](https://github.com/slackhq/nebula) mesh and
pools its disk into a replicated [Garage](https://garagehq.deuxfleurs.fr/)
cluster. Donate a laptop, get free storage. Starting in Brisbane,
Australia.

This repository holds the project documentation, published at
**[junknet.au](https://junknet.au/)**.

## Working on the docs

The site is built with [Zensical](https://zensical.org/) and deployed
to GitHub Pages by [`.github/workflows/docs.yml`](.github/workflows/docs.yml)
on every push to `main`.

```bash
pip install zensical
zensical serve     # live-reloading preview at http://localhost:8000
zensical build     # static build into site/
```

Content lives in [`docs/`](docs/); site configuration in
[`zensical.toml`](zensical.toml).

## Get involved

Laptops, host households, and curious people are all welcome — see
[the Brisbane pilot](https://junknet.au/contribute/brisbane/) or email
[luke@aquainnis.com](mailto:luke@aquainnis.com).

A community project by [Aquainnis](https://aquainnis.com).
