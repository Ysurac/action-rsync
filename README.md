# action-rsync ![.github/workflows/main.yml](https://github.com/up9cloud/action-rsync/workflows/.github/workflows/main.yml/badge.svg) [![Docker Automated build](https://img.shields.io/docker/automated/sstc/action-rsync)](https://hub.docker.com/repository/docker/sstc/action-rsync)

- Small: Alpine based image with pre-installed rsync, see [sstc/rsync](https://hub.docker.com/r/sstc/rsync).
- Hooks: Basic pre and post scripts support.
- Pure 😊: No github specific inputs, outputs, it can be used on other platform!

## Quick start

> Github Action (`.github/workflows/*.yml`)

```yml
on: [push]
jobs:
  rsync:
    runs-on: ubuntu-latest
    steps:
      # Make sure to run checkout first, otherwise the folder will be empty. See: https://github.com/actions/checkout
      - uses: actions/checkout@v4
      # Replace `master` with a specific version; refer to https://github.com/marketplace/actions/action-rsync for available versions.
      - uses: up9cloud/action-rsync@master
        env:
          HOST: target.example.com
          KEY: ${{secrets.DEPLOY_SSH_KEY}}
          TARGET: /app/
```

> Drone CI (`.drone.yml`)

```yml
kind: pipeline
type: docker
name: default

steps:
  - name: deploy
    when:
      branch:
        - master
      event: [push]
    image: sstc/action-rsync
    settings:
      # lowercase attributes, see https://readme.drone.io/plugins/overview/#plugin-inputs
      key:
        from_secret: deploy_ssh_key
      host: target.example.com
      target: /app/
```

> Docker Container

```bash
docker run -it --rm \
  -v $(pwd):/app \
  -w /app \
  -e HOST="target.example.com" \
  -e KEY="$(cat ~/.ssh/id_rsa)"
  -e TARGET="/app/" \
  sstc/action-rsync
```

## ENVs

||Default Value|Description|
|---|---|---|
|**`HOST`**||SSH hostname or IP address of the remote server<br>**Required if** **`MODE`** is `push` or `pull`|
|**`REMOTE_HOSTS`**||Multiple remote hosts<br>You can provide a comma-separated list (e.g., 1.2.3.4,8.8.4.4)<br>or one host per line (e.g., 8.8.8.8\n111.111.111.111)<br>Pre & post scripts will be executed on each host<br>**Required if** **`MODE`** is `push` or `pull`|
|**`USER`**|`root`|SSH user for the remote server<br>Not used when **`MODE`** is `local`|
|**`PORT`**|`22`|SSH port for the remote server<br>Not used when **`MODE`** is `local`|
|**`KEY`**||SSH private key<br>**Required if** **`PASSWORD`** is not provided and **`MODE`** is `push` or `pull`|
|**`PASSWORD`**||SSH password<br>**Required if** **`KEY`** is not provided and **`MODE`** is `push` or `pull`|
|**`SOURCE`**|`./`|Source path for the folder or file|
|**`TARGET`**||Target path for the folder or file<br>**Required**|
|**`MODE`**|`push`|Allowed values:<br>`push`: local (SOURCE) to remote (TARGET)<br>`pull`: remote (SOURCE) to local (TARGET)<br>`local`: local (SOURCE) to local (TARGET)|
|**`VERBOSE`**|`false`|Set to `true` for more detailed output|
|**`ARGS`**|`-avz --delete --exclude=/.git/ --exclude=/.github/`|Arguments for rsync|
|**`ARGS_MORE`**||Additional rsync arguments. Appended to ARGS, so final rsync arguments are: `$ARGS $ARGS_MORE`.<br><br>For example, setting ARGS_MORE to `--no-o --no-g` (with default ARGS) results in:<br>`-avz --delete --exclude=/.git/ --exclude=/.github/ --no-o --no-g`|
|**`SSH_ARGS`**|`-p 22 -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -o LogLevel=quiet`|Arguments for SSH. The `-p` value is set dynamically from `PORT`, but if you set SSH_ARGS, the `PORT` value will be ignored.|
|**`RUN_SCRIPT_ON`**|`target`|Allowed values:<br>`target`: When **`MODE`** is `push`, runs pre and post scripts on the remote (target) server; for other modes, runs on local.<br>`source`: When **`MODE`** is `push`, runs scripts locally; for other modes, runs on remote.<br>`local`: Always runs scripts locally.<br>`remote`: Always runs scripts on remote.|
|**`PRE_SCRIPT`**||Script to run before rsync.<br>The target system defined by RUN_SCRIPT_ON must support the `mktemp` command|
|**`POST_SCRIPT`**||Script to run after rsync.<br>The target system defined by RUN_SCRIPT_ON must support the `mktemp` command|

### Example

```yml
on: [push]
jobs:
  rsync:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Deploy to my ❤️
      uses: up9cloud/action-rsync@master
      env:
        HOST: example.com
        KEY: ${{secrets.DEPLOY_SSH_KEY}}
        # PASSWORD: ${{secrets.DEPLOY_SSH_PASSWORD}} # less secure than using KEY; preferring KEY for security
        TARGET: /app/hello-service/

        VERBOSE: true
        USER: ubuntu
        # PORT: 2222 # no need to set this if you use $SSH_ARGS
        ARGS: -az --exclude=/.git/
        SSH_ARGS: '-p 2222 -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no'
        SOURCE: ./public/

        PRE_SCRIPT: |
          echo start at:
          date -u
        POST_SCRIPT: "echo done at: && date -u"
```

See also: [.github/workflows/main.yml](https://github.com/up9cloud/action-rsync/blob/master/.github/workflows/main.yml)

## TODO

- [ ] Verify SSH connection before execution
- [ ] Benchmark and compare performance with other JavaScript-based actions
- [ ] Pin the Docker image version
- [ ] Use more descriptive variable names, e.g., rename HOST to REMOTE_HOST
