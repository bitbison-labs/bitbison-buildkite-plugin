# bitbison-labs/bitbison-buildkite-plugin

Capture build provenance in Buildkite with [Bitbison](https://bitbison.io/).

```yaml
steps:
  - label: build
    command: make
    plugins:
      - bitbison-labs/bitbison#v0.1.0: ~
```

The plugin reads the release channel URL, the submission token and the
instance URL from the cluster secrets `BITBISON_CHANNEL`, `BITBISON_TOKEN`
and `BITBISON_INSTANCE`, or from those variables when the pipeline sets
them. Other secret names:

```yaml
      - bitbison-labs/bitbison#v0.1.0:
          channel-secret: MY_CHANNEL
          token-secret: MY_TOKEN
```

It runs on Buildkite hosted Linux agents and on self-hosted agents
alike. For installation, open your Bitbison user portal: the Builds page
carries the installation guide.

Repository code is licensed under [Apache-2.0](LICENSE). Downloaded
Bitbison components are covered by their own terms.
