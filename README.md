minis2i
=======

[`s2i`][s2i] in ~80 lines of python.

I was curious how [source-to-image][s2i] (aka s2i) worked so I tried to
replicate it in python.

This repository provides a single executable `minis2i` which acts like:

```bash
exec s2i build \
    --copy \
    --incremental \
    --pull-policy=never \
    --incremental-pull-policy=never \
    "$@"
```

### cutting corners

- It assumes `io.openshift.s2i.scripts-url=image:///usr/local/s2i`.
- It does not support s2i `run` `CMD` instead inheriting the `CMD` from the
  builder image.
- `minis2i` opts for `/tmp/artifacts` to be a rw volume to save / retrieve
  artifacts from (instead of using `tar` over stdout).
- `/tmp/src` and `/tmp/artifacts` are mounted readonly (can't `rm` them)
- `minis2i` does not include any messaging.

### dependencies

- `docker`
- `python3.6`

### example

This repository includes a sample incremental builder, this toy builder
copies an input file `input.txt` to `output.txt`.

```bash
docker build -t builder ./builder --quiet
docker rmi testapp-img >& /dev/null  # ensure non-incremental
./minis2i --env=VAR=hello testapp builder testapp-img
./minis2i --env=VAR=hello testapp builder testapp-img  # incremental
```

```console
$ docker build -t builder ./builder --quiet
sha256:fd31c40b6e5c6543f771ab4466fa5168dc2bc1c98c5e97247ea9be5aa2dd77a8
$ docker rmi testapp-img >& /dev/null  # ensure non-incremental
$ ./minis2i --env=VAR=hello testapp builder testapp-img
+ echo hello
hello
+ '[' -e /tmp/artifacts/output.txt ']'
+ cp /tmp/src/input.txt output.txt
sha256:69d66828d4b5037e11585dd7e7d9b67ab70ab1f08d669819f0583c74e2eeb0b5
ad29512642eb425767247c23399325ba3d4d866e60b72b600abd5707a5f0fa9b
$ ./minis2i --env=VAR=hello testapp builder testapp-img  # incremental
+ cp output.txt /tmp/artifacts
+ echo hello
hello
+ '[' -e /tmp/artifacts/output.txt ']'
+ cp /tmp/artifacts/output.txt .
sha256:a57473c66941ec43dc24c1502cebc3ced976578600a5d3d722f8e4db390b5bb6
4168962cbcebbdd6015b3aa6a9612c9398b5d88ad0bb2a264b297495b0492f53
```

[s2i]: https://github.com/openshift/source-to-image
