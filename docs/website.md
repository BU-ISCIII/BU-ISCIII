# Website dev docs

## Info

- [repo](https://github.com/BU-ISCIII/buisciii.github.io)
- [template](https://kitian616.github.io/jekyll-TeXt-theme/docs/en/quick-start)

## Template installation

We are going to use docker to deploy locally our website:

```bash
docker run --rm -v "$PWD":/usr/src/app -w /usr/src/app ruby:3.0 bundle install
```

Deploy docker compose service:

```bash
docker compose -f ./docker/docker-compose.build-image.yml build
```

Run docker:

```bash
docker compose -f ./docker/docker-compose.default.yml up
```
