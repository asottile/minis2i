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
- `minis2i` does not include any messaging.

### dependencies

- `docker`
- `tar` (used for speed over `tarfile` module)
- `python3.6`

### example

This repository includes a sample incremental builder, this toy builder
copies an input file `input.txt` to `output.txt`.

```bash
docker build -t builder ./builder --quiet
docker rmi testapp-img >& /dev/null  # ensure non-incremental
./minis2i testapp builder testapp-img
./minis2i testapp builder testapp-img  # incremental
```

```console
$ docker build -t builder ./builder --quiet
sha256:ff83357c129cbc63cd89d9cf48357c55fa0f739f53aa6874c385ac30a401b2ac
$ docker rmi testapp-img >& /dev/null  # ensure non-incremental
$ ./minis2i testapp builder testapp-img
+ '[' -e /tmp/artifacts/output.txt ']'
+ cp /tmp/src/input.txt output.txt
sha256:73d4119c6c27216c37dc6000cecf483fd2f9cfb69396e2517fbeb3afb241f345
e2b6345b6b04b899b5fa4d10b59d1511fc624b90935636bf0489994b277ea2ae
$ ./minis2i testapp builder testapp-img  # incremental
+ exec tar cf - output.txt
+ '[' -e /tmp/artifacts/output.txt ']'
+ cp /tmp/artifacts/output.txt .
sha256:ab59665865a6203cea93ba88ef08168d529d467d4059f9470c42441c6fffe74d
1d00d6b26043f468a3f82b7fdcb91285359c4464a82327f3bace8095147cbccd
```

[s2i]: https://github.com/openshift/source-to-image
