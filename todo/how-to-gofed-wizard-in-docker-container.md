# How to gofed wizard in docker container

**Motivation**

As a golang maintainer, I would like:

* to run one command,
* disconnet,
* go on a lunch,
* return and,

get my precious Go package updated in a distribution.

## Acceptance criteria

* I want to ssh to a remote machine and run the wizard with one command
* I want to ssh to a remote machine and collect logs with one command
* I want to run one command on a local machine what will run wizard and collect logs on a remote machine (optional)

## How to wizard a package

```
$ gofed wizard --scratch [--(e)branches "master,epel7"]
```

## How to collect logs

One can query running (or run) container:

```
$ docker logs CONTAINER_ID
```

## Additional info

* one can run commands on a remote host via ssh: ``ssh HOST@IP COMMAND``
* worth integration into gofed the same way is ``koji scratch-build`` and its ``This build may be interrupted``
* "something" could trigger a Jenkins Job (with up to 10 concurrent builds sharing one slave?)
