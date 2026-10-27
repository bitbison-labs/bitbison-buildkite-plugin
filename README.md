# bitbison-labs/bitbison-buildkite-plugin

Capture build provenance in Buildkite with [Bitbison](https://bitbison.io/).

```yaml
steps:
  - label: build
    command: make
    plugins:
      - bitbison-labs/bitbison#v0.1.2: ~
```

The plugin reads the release channel URL, the submission token and the
instance URL from the cluster secrets `BITBISON_CHANNEL`, `BITBISON_TOKEN`
and `BITBISON_INSTANCE`, or from those variables when the pipeline sets
them. Other secret names:

```yaml
      - bitbison-labs/bitbison#v0.1.2:
          channel-secret: MY_CHANNEL
          token-secret: MY_TOKEN
```

It runs on Buildkite hosted Linux agents and on self-hosted agents
alike. buildguard is fetched from the release channel and verified
against its checksums on every job; it is kept under the agent's
plugins directory (or `BITBISON_PLUGIN_STORE`), so a self-hosted agent
downloads a release once. For installation, open your Bitbison user portal: the Builds page
carries the installation guide.

## Every job on a queue, with no plugin line

Instead of a plugin line on each step, the two hooks in `agent-hooks/` can
be installed once into an agent image, and every job on the queues using
it is captured, across all pipelines. Which repositories are captured
then follows the organization's build setting in Bitbison (all of them
by default), and a step that still carries the plugin line steps aside
when the hook has captured the job.

On Buildkite hosted agents:

1. Create the cluster secrets `BITBISON_CHANNEL` and `BITBISON_TOKEN`
   (the values are on the Bitbison Builds page), if they do not exist.
2. Under Agents, your cluster, Agent Images, create a new image (or add to
   the Dockerfile you already use) below the `FROM` line Buildkite fills in:

   ```dockerfile
   ENV BUILDKITE_ADDITIONAL_HOOKS_PATHS=/etc/bitbison/buildkite-hooks
   RUN set -e; \
       d=/etc/bitbison/buildkite-hooks; \
       u=https://raw.githubusercontent.com/bitbison-labs/bitbison-buildkite-plugin/v0.2.0/agent-hooks; \
       mkdir -p "$d"; \
       curl -fsSL "$u/environment" -o "$d/environment"; \
       curl -fsSL "$u/pre-exit" -o "$d/pre-exit"; \
       printf '%s  %s\n%s  %s\n' \
         61af5d3b2dc8033b96071bf50a86af4486313ae54c26c168cae2746f3dfce0c1 "$d/environment" \
         5bba62145f5a9c69d0e4f0db4a0c17f358be8269f16f3502b1b4cfbac4db1337 "$d/pre-exit" | sha256sum -c -; \
       chmod 0644 "$d/environment" "$d/pre-exit"
   ```

   If the image already sets `BUILDKITE_ADDITIONAL_HOOKS_PATHS`, put
   `/etc/bitbison/buildkite-hooks` first in its comma-separated list
   instead of replacing it: the agent runs each directory's hooks in list
   order, and Bitbison's `environment` hook should run before yours. An
   image managed in Buildkite cannot be edited; create a new one with your
   Dockerfile plus these lines and switch the queue to it.
3. When the image is Ready, select it under the queue's Base image tab.
   If you build your images in a pipeline instead (pushed to the cluster's
   registry, `$BUILDKITE_HOSTED_REGISTRY_URL`, and named by a step's or
   pipeline's `image:`), add the same lines to that Dockerfile.
4. Run any pipeline on the queue: the job log shows `buildguard: bbd
   started` and `buildguard: build ... opened` under the environment hook.

A step or pipeline that sets its own `image:` replaces the queue's image,
hooks included; give such steps the plugin line. On a self-hosted agent,
install the Bitbison sensor and run `buildguard init --buildkite`
instead, which writes the same hooks into the agent's hooks directory.

Repository code is licensed under [Apache-2.0](LICENSE). Downloaded
Bitbison components are covered by their own terms.
