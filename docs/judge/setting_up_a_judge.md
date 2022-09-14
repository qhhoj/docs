# Setting up a Judge

This guide goes through the process of installing a judge and connecting it to
the site. It is intended for Linux-based machines (WSL included); Windows is
not supported.

It is assumed that the site installation instructions have been followed, and
that a bridge instance is running.

## Site-side setup

First, add a judge on the admin page, located under
[https://example.com/admin/judge](https://example.com/admin/judge). Provide the
name of the judge and the authentication key for the judge. You may use the
`Regenerate` button to generate an authentication key.

In `local_settings.py`, find the `BRIDGED_JUDGE_ADDRESS`. This is the address
you will be connecting a judge to. By default, this should be `localhost:9999`.
If you are connecting from a different machine, you will need to change
`localhost` to an actual IP. Also, ensure that this port is **open**, or you
will receive cryptic error messages when attempting to connect a judge.

Finally, ensure the bridge is running. You should see something similar to the
following lines.

```shell-session
$ supervisorctl status
bridged RUNNING pid <pid>, uptime <uptime>
```

## Judge-side setup

The DMOJ supports installing judges through Docker and a PyPI package. We
recommend the Docker installation if you are able to use Docker, since we have
dealt with the hard problem of getting many runtimes co-existing on the same
machine and keeping them up-to-date. The PyPI package is also supported, and
may give you more control at the expense of more administrative complexity.

### With Docker

We maintain Docker images with all runtimes we support in the
[runtimes-docker](https://github.com/DMOJ/runtimes-docker) project.

Runtimes are split into three tiers of decreasing support. Tier 1 includes
Python 2/3, C/C++ (GCC only), Java 8, and Pascal. Tier 3 contains all the
runtimes we run on [dmoj.ca](https://dmoj.ca). Tier 2 contains some in-between
mix; read the `Dockerfile` for each tier for details. These images are rebuilt
and tested every week to contain the latest runtime versions.

The session below build a `judge-tier1`:

```shell-session
$ git clone --recursive https://github.com/VNOI-Admin/judge-server.git
$ cd judge/.docker
$ make judge-tier1
```

The session below spawns a tier 1 judge image in the same server as the site server.
**It expects problems to be placed on the host under `/mnt/problems`, and judge-specific
configuration to be in `/mnt/problems/judge.yml`.**

Here is an example `judge.yml`:
```yaml
id: Judge1
key: a_random_key
problem_storage_root:
  - /problems
```

```shell-session
$ docker run \
    --name judge \
    --network="host" \
    -v /mnt/problems:/problems \
    --cap-add=SYS_PTRACE \
    -d \
    --restart=always \
    vnoj/judge-tier1:latest \
    run -p 9999 -c /problems/judge.yml localhost  -A 0.0.0.0  -a 12345
```

If you changed the port that was specified in `BRIDGED_JUDGE_ADDRESS` of the
site's `local_settings.py`, you need to change the `-p 9999` to match the config as well

If you want to run multiple judges, you need to changes:
- Container name (`--name judge`): each judge need different name
- judge.yml file (`/problems/judge.yml`): each judge need different config file
- `-a 12345`: change to others ports

### Through PyPI

!> Not available for VNOJ

We are not maintaining our judge on PyPI, you should use the docker setup above.
