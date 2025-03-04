# build

<!---MARKER_GEN_START-->
Build an image from a Dockerfile

### Aliases

`docker image build`, `docker build`, `docker buildx build`, `docker builder build`

### Options

| Name                                | Type          | Default   | Description                                                       |
|:------------------------------------|:--------------|:----------|:------------------------------------------------------------------|
| [`--add-host`](#add-host)           | `list`        |           | Add a custom host-to-IP mapping (`host:ip`)                       |
| [`--build-arg`](#build-arg)         | `list`        |           | Set build-time variables                                          |
| [`--cache-from`](#cache-from)       | `stringSlice` |           | Images to consider as cache sources                               |
| [`--cgroup-parent`](#cgroup-parent) | `string`      |           | Set the parent cgroup for the `RUN` instructions during build     |
| `--compress`                        |               |           | Compress the build context using gzip                             |
| `--cpu-period`                      | `int64`       | `0`       | Limit the CPU CFS (Completely Fair Scheduler) period              |
| `--cpu-quota`                       | `int64`       | `0`       | Limit the CPU CFS (Completely Fair Scheduler) quota               |
| `-c`, `--cpu-shares`                | `int64`       | `0`       | CPU shares (relative weight)                                      |
| `--cpuset-cpus`                     | `string`      |           | CPUs in which to allow execution (0-3, 0,1)                       |
| `--cpuset-mems`                     | `string`      |           | MEMs in which to allow execution (0-3, 0,1)                       |
| `--disable-content-trust`           |               |           | Skip image verification                                           |
| [`-f`](#file), [`--file`](#file)    | `string`      |           | Name of the Dockerfile (Default is `PATH/Dockerfile`)             |
| `--force-rm`                        |               |           | Always remove intermediate containers                             |
| `--iidfile`                         | `string`      |           | Write the image ID to the file                                    |
| [`--isolation`](#isolation)         | `string`      |           | Container isolation technology                                    |
| `--label`                           | `list`        |           | Set metadata for an image                                         |
| `-m`, `--memory`                    | `bytes`       | `0`       | Memory limit                                                      |
| `--memory-swap`                     | `bytes`       | `0`       | Swap limit equal to memory plus swap: -1 to enable unlimited swap |
| `--network`                         | `string`      | `default` | Set the networking mode for the RUN instructions during build     |
| `--no-cache`                        |               |           | Do not use cache when building the image                          |
| `--platform`                        | `string`      |           | Set platform if server is multi-platform capable                  |
| `--pull`                            |               |           | Always attempt to pull a newer version of the image               |
| `-q`, `--quiet`                     |               |           | Suppress the build output and print image ID on success           |
| `--rm`                              |               |           | Remove intermediate containers after a successful build           |
| [`--security-opt`](#security-opt)   | `stringSlice` |           | Security options                                                  |
| `--shm-size`                        | `bytes`       | `0`       | Size of `/dev/shm`                                                |
| [`--squash`](#squash)               |               |           | Squash newly built layers into a single new layer                 |
| [`-t`](#tag), [`--tag`](#tag)       | `list`        |           | Name and optionally a tag in the `name:tag` format                |
| [`--target`](#target)               | `string`      |           | Set the target build stage to build.                              |
| [`--ulimit`](#ulimit)               | `ulimit`      |           | Ulimit options                                                    |


<!---MARKER_GEN_END-->

## Description

The `docker build` command builds Docker images from a Dockerfile and a
"context". A build's context is the set of files located in the specified
`PATH` or `URL`. The build process can refer to any of the files in the
context. For example, your build can use a [*COPY*](https://docs.docker.com/engine/reference/builder/#copy)
instruction to reference a file in the context.

The `URL` parameter can refer to three kinds of resources: Git repositories,
pre-packaged tarball contexts and plain text files.

### Git repositories

When the `URL` parameter points to the location of a Git repository, the
repository acts as the build context. The system recursively fetches the
repository and its submodules. The commit history is not preserved. A
repository is first pulled into a temporary directory on your local host. After
that succeeds, the directory is sent to the Docker daemon as the context.
Local copy gives you the ability to access private repositories using local
user credentials, VPN's, and so forth.

> **Note**
>
> If the `URL` parameter contains a fragment the system will recursively clone
> the repository and its submodules using a `git clone --recursive` command.

Git URLs accept context configuration in their fragment section, separated by a
colon (`:`).  The first part represents the reference that Git will check out,
and can be either a branch, a tag, or a remote reference. The second part
represents a subdirectory inside the repository that will be used as a build
context.

For example, run this command to use a directory called `docker` in the branch
`container`:

```console
$ docker build https://github.com/docker/rootfs.git#container:docker
```

The following table represents all the valid suffixes with their build
contexts:

| Build Syntax Suffix            | Commit Used           | Build Context Used |
|--------------------------------|-----------------------|--------------------|
| `myrepo.git`                   | `refs/heads/master`   | `/`                |
| `myrepo.git#mytag`             | `refs/tags/mytag`     | `/`                |
| `myrepo.git#mybranch`          | `refs/heads/mybranch` | `/`                |
| `myrepo.git#pull/42/head`      | `refs/pull/42/head`   | `/`                |
| `myrepo.git#:myfolder`         | `refs/heads/master`   | `/myfolder`        |
| `myrepo.git#master:myfolder`   | `refs/heads/master`   | `/myfolder`        |
| `myrepo.git#mytag:myfolder`    | `refs/tags/mytag`     | `/myfolder`        |
| `myrepo.git#mybranch:myfolder` | `refs/heads/mybranch` | `/myfolder`        |

### Tarball contexts

If you pass an URL to a remote tarball, the URL itself is sent to the daemon:

```console
$ docker build http://server/context.tar.gz
```

The download operation will be performed on the host the Docker daemon is
running on, which is not necessarily the same host from which the build command
is being issued. The Docker daemon will fetch `context.tar.gz` and use it as the
build context. Tarball contexts must be tar archives conforming to the standard
`tar` UNIX format and can be compressed with any one of the 'xz', 'bzip2',
'gzip' or 'identity' (no compression) formats.

### Text files

Instead of specifying a context, you can pass a single `Dockerfile` in the
`URL` or pipe the file in via `STDIN`. To pipe a `Dockerfile` from `STDIN`:

```console
$ docker build - < Dockerfile
```

With Powershell on Windows, you can run:

```powershell
Get-Content Dockerfile | docker build -
```

If you use `STDIN` or specify a `URL` pointing to a plain text file, the system
places the contents into a file called `Dockerfile`, and any `-f`, `--file`
option is ignored. In this scenario, there is no context.

By default the `docker build` command will look for a `Dockerfile` at the root
of the build context. The `-f`, `--file`, option lets you specify the path to
an alternative file to use instead. This is useful in cases where the same set
of files are used for multiple builds. The path must be to a file within the
build context. If a relative path is specified then it is interpreted as
relative to the root of the context.

In most cases, it's best to put each Dockerfile in an empty directory. Then,
add to that directory only the files needed for building the Dockerfile. To
increase the build's performance, you can exclude files and directories by
adding a `.dockerignore` file to that directory as well. For information on
creating one, see the [.dockerignore file](https://docs.docker.com/engine/reference/builder/#dockerignore-file).

If the Docker client loses connection to the daemon, the build is canceled.
This happens if you interrupt the Docker client with `CTRL-c` or if the Docker
client is killed for any reason. If the build initiated a pull which is still
running at the time the build is cancelled, the pull is cancelled as well.

## Return code

On a successful build, a return code of success `0` will be returned.  When the
build fails, a non-zero failure code will be returned.

There should be informational output of the reason for failure output to
`STDERR`:

```console
$ docker build -t fail .

Sending build context to Docker daemon 2.048 kB
Sending build context to Docker daemon
Step 1/3 : FROM busybox
 ---> 4986bf8c1536
Step 2/3 : RUN exit 13
 ---> Running in e26670ec7a0a
INFO[0000] The command [/bin/sh -c exit 13] returned a non-zero code: 13
$ echo $?
1
```

See also:

[*Dockerfile Reference*](https://docs.docker.com/engine/reference/builder/).

## Examples

### Build with PATH

```console
$ docker build .

Uploading context 10240 bytes
Step 1/3 : FROM busybox
Pulling repository busybox
 ---> e9aa60c60128MB/2.284 MB (100%) endpoint: https://cdn-registry-1.docker.io/v1/
Step 2/3 : RUN ls -lh /
 ---> Running in 9c9e81692ae9
total 24
drwxr-xr-x    2 root     root        4.0K Mar 12  2013 bin
drwxr-xr-x    5 root     root        4.0K Oct 19 00:19 dev
drwxr-xr-x    2 root     root        4.0K Oct 19 00:19 etc
drwxr-xr-x    2 root     root        4.0K Nov 15 23:34 lib
lrwxrwxrwx    1 root     root           3 Mar 12  2013 lib64 -> lib
dr-xr-xr-x  116 root     root           0 Nov 15 23:34 proc
lrwxrwxrwx    1 root     root           3 Mar 12  2013 sbin -> bin
dr-xr-xr-x   13 root     root           0 Nov 15 23:34 sys
drwxr-xr-x    2 root     root        4.0K Mar 12  2013 tmp
drwxr-xr-x    2 root     root        4.0K Nov 15 23:34 usr
 ---> b35f4035db3f
Step 3/3 : CMD echo Hello world
 ---> Running in 02071fceb21b
 ---> f52f38b7823e
Successfully built f52f38b7823e
Removing intermediate container 9c9e81692ae9
Removing intermediate container 02071fceb21b
```

This example specifies that the `PATH` is `.`, and so all the files in the
local directory get `tar`d and sent to the Docker daemon. The `PATH` specifies
where to find the files for the "context" of the build on the Docker daemon.
Remember that the daemon could be running on a remote machine and that no
parsing of the Dockerfile happens at the client side (where you're running
`docker build`). That means that *all* the files at `PATH` get sent, not just
the ones listed to [*ADD*](https://docs.docker.com/engine/reference/builder/#add)
in the Dockerfile.

The transfer of context from the local machine to the Docker daemon is what the
`docker` client means when you see the "Sending build context" message.

If you wish to keep the intermediate containers after the build is complete,
you must use `--rm=false`. This does not affect the build cache.

### Build with URL

```console
$ docker build github.com/creack/docker-firefox
```

This will clone the GitHub repository and use the cloned repository as context.
The Dockerfile at the root of the repository is used as Dockerfile. You can
specify an arbitrary Git repository by using the `git://` or `git@` scheme.

```console
$ docker build -f ctx/Dockerfile http://server/ctx.tar.gz

Downloading context: http://server/ctx.tar.gz [===================>]    240 B/240 B
Step 1/3 : FROM busybox
 ---> 8c2e06607696
Step 2/3 : ADD ctx/container.cfg /
 ---> e7829950cee3
Removing intermediate container b35224abf821
Step 3/3 : CMD /bin/ls
 ---> Running in fbc63d321d73
 ---> 3286931702ad
Removing intermediate container fbc63d321d73
Successfully built 377c409b35e4
```

This sends the URL `http://server/ctx.tar.gz` to the Docker daemon, which
downloads and extracts the referenced tarball. The `-f ctx/Dockerfile`
parameter specifies a path inside `ctx.tar.gz` to the `Dockerfile` that is used
to build the image. Any `ADD` commands in that `Dockerfile` that refers to local
paths must be relative to the root of the contents inside `ctx.tar.gz`. In the
example above, the tarball contains a directory `ctx/`, so the `ADD
ctx/container.cfg /` operation works as expected.

### Build with -

```console
$ docker build - < Dockerfile
```

This will read a Dockerfile from `STDIN` without context. Due to the lack of a
context, no contents of any local directory will be sent to the Docker daemon.
Since there is no context, a Dockerfile `ADD` only works if it refers to a
remote URL.

```console
$ docker build - < context.tar.gz
```

This will build an image for a compressed context read from `STDIN`.  Supported
formats are: bzip2, gzip and xz.

### Use a .dockerignore file

```console
$ docker build .

Uploading context 18.829 MB
Uploading context
Step 1/2 : FROM busybox
 ---> 769b9341d937
Step 2/2 : CMD echo Hello world
 ---> Using cache
 ---> 99cc1ad10469
Successfully built 99cc1ad10469
$ echo ".git" > .dockerignore
$ docker build .
Uploading context  6.76 MB
Uploading context
Step 1/2 : FROM busybox
 ---> 769b9341d937
Step 2/2 : CMD echo Hello world
 ---> Using cache
 ---> 99cc1ad10469
Successfully built 99cc1ad10469
```

This example shows the use of the `.dockerignore` file to exclude the `.git`
directory from the context. Its effect can be seen in the changed size of the
uploaded context. The builder reference contains detailed information on
[creating a .dockerignore file](https://docs.docker.com/engine/reference/builder/#dockerignore-file).

When using the [BuildKit backend](https://docs.docker.com/build/buildkit/),
`docker build` searches for a `.dockerignore` file relative to the Dockerfile
name. For example, running `docker build -f myapp.Dockerfile .` will first look
for an ignore file named `myapp.Dockerfile.dockerignore`. If such a file is not
found, the `.dockerignore` file is used if present. Using a Dockerfile based
`.dockerignore` is useful if a project contains multiple Dockerfiles that expect
to ignore different sets of files.


### <a name="tag"></a> Tag an image (-t, --tag)

```console
$ docker build -t vieux/apache:2.0 .
```

This will build like the previous example, but it will then tag the resulting
image. The repository name will be `vieux/apache` and the tag will be `2.0`.
[Read more about valid tags](tag.md).

You can apply multiple tags to an image. For example, you can apply the `latest`
tag to a newly built image and add another tag that references a specific
version.
For example, to tag an image both as `whenry/fedora-jboss:latest` and
`whenry/fedora-jboss:v2.1`, use the following:

```console
$ docker build -t whenry/fedora-jboss:latest -t whenry/fedora-jboss:v2.1 .
```

### <a name="file"></a> Specify a Dockerfile (-f, --file)

```console
$ docker build -f Dockerfile.debug .
```

This will use a file called `Dockerfile.debug` for the build instructions
instead of `Dockerfile`.

```console
$ curl example.com/remote/Dockerfile | docker build -f - .
```

The above command will use the current directory as the build context and read
a Dockerfile from stdin.

```console
$ docker build -f dockerfiles/Dockerfile.debug -t myapp_debug .
$ docker build -f dockerfiles/Dockerfile.prod  -t myapp_prod .
```

The above commands will build the current build context (as specified by the
`.`) twice, once using a debug version of a `Dockerfile` and once using a
production version.

```console
$ cd /home/me/myapp/some/dir/really/deep
$ docker build -f /home/me/myapp/dockerfiles/debug /home/me/myapp
$ docker build -f ../../../../dockerfiles/debug /home/me/myapp
```

These two `docker build` commands do the exact same thing. They both use the
contents of the `debug` file instead of looking for a `Dockerfile` and will use
`/home/me/myapp` as the root of the build context. Note that `debug` is in the
directory structure of the build context, regardless of how you refer to it on
the command line.

> **Note**
>
> `docker build` returns a `no such file or directory` error if the
> file or directory does not exist in the uploaded context. This may
> happen if there is no context, or if you specify a file that is
> elsewhere on the Host system. The context is limited to the current
> directory (and its children) for security reasons, and to ensure
> repeatable builds on remote Docker hosts. This is also the reason why
> `ADD ../file` does not work.

### <a name="cgroup-parent"></a> Use a custom parent cgroup (--cgroup-parent)

When `docker build` is run with the `--cgroup-parent` option the containers
used in the build will be run with the [corresponding `docker run` flag](../run.md#specify-custom-cgroups).

### <a name="ulimit"></a> Set ulimits in container (--ulimit)

Using the `--ulimit` option with `docker build` will cause each build step's
container to be started using those [`--ulimit` flag values](run.md#ulimit).

### <a name="build-arg"></a> Set build-time variables (--build-arg)

You can use `ENV` instructions in a Dockerfile to define variable
values. These values persist in the built image. However, often
persistence is not what you want. Users want to specify variables differently
depending on which host they build an image on.

A good example is `http_proxy` or source versions for pulling intermediate
files. The `ARG` instruction lets Dockerfile authors define values that users
can set at build-time using the  `--build-arg` flag:

```console
$ docker build --build-arg HTTP_PROXY=http://10.20.30.2:1234 --build-arg FTP_PROXY=http://40.50.60.5:4567 .
```

This flag allows you to pass the build-time variables that are
accessed like regular environment variables in the `RUN` instruction of the
Dockerfile. Also, these values don't persist in the intermediate or final images
like `ENV` values do.   You must add `--build-arg` for each build argument.

Using this flag will not alter the output you see when the `ARG` lines from the
Dockerfile are echoed during the build process.

For detailed information on using `ARG` and `ENV` instructions, see the
[Dockerfile reference](https://docs.docker.com/engine/reference/builder/).

You may also use the `--build-arg` flag without a value, in which case the value
from the local environment will be propagated into the Docker container being
built:

```console
$ export HTTP_PROXY=http://10.20.30.2:1234
$ docker build --build-arg HTTP_PROXY .
```

This is similar to how `docker run -e` works. Refer to the [`docker run` documentation](run.md#env)
for more information.

### <a name="security-opt"></a> Optional security options (--security-opt)

This flag is only supported on a daemon running on Windows, and only supports
the `credentialspec` option. The `credentialspec` must be in the format
`file://spec.txt` or `registry://keyname`.

### <a name="isolation"></a> Specify isolation technology for container (--isolation)

This option is useful in situations where you are running Docker containers on
Windows. The `--isolation=<value>` option sets a container's isolation
technology. On Linux, the only supported is the `default` option which uses
Linux namespaces. On Microsoft Windows, you can specify these values:


| Value     | Description                                                                                                                                                                    |
|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `default` | Use the value specified by the Docker daemon's `--exec-opt` . If the `daemon` does not specify an isolation technology, Microsoft Windows uses `process` as its default value. |
| `process` | Namespace isolation only.                                                                                                                                                      |
| `hyperv`  | Hyper-V hypervisor partition-based isolation.                                                                                                                                  |

Specifying the `--isolation` flag without a value is the same as setting `--isolation="default"`.

### <a name="add-host"></a> Add entries to container hosts file (--add-host)

You can add other hosts into a build container's `/etc/hosts` file by using one
or more `--add-host` flags. This example adds static addresses for hosts named
`my-hostname` and `my_hostname_v6`:

```console
$ docker build --add-host my_hostname=8.8.8.8 --add-host my_hostname_v6=2001:4860:4860::8888 .
```

If you need your build to connect to services running on the host, you can use
the special `host-gateway` value for `--add-host`. In the following example,
build containers resolve `host.docker.internal` to the host's gateway IP.

```console
$ docker build --add-host host.docker.internal=host-gateway .
```

You can wrap an IPv6 address in square brackets.
`=` and `:` are both valid separators.
Both formats in the following example are valid:

```console
$ docker build --add-host my-hostname:10.180.0.1 --add-host my-hostname_v6=[2001:4860:4860::8888] .
```

### <a name="target"></a> Specifying target build stage (--target)

When building a Dockerfile with multiple build stages, `--target` can be used to
specify an intermediate build stage by name as a final stage for the resulting
image. Commands after the target stage will be skipped.

```dockerfile
FROM debian AS build-env
# ...

FROM alpine AS production-env
# ...
```

```console
$ docker build -t mybuildimage --target build-env .
```

### <a name="output"></a> Custom build outputs (--output)

> **Note**
>
> This feature requires the BuildKit backend. You can either
> [enable BuildKit](https://docs.docker.com/build/buildkit/#getting-started) or
> use the [buildx](https://github.com/docker/buildx) plugin which provides more
> output type options.

By default, a local container image is created from the build result. The
`--output` (or `-o`) flag allows you to override this behavior, and a specify a
custom exporter. For example, custom exporters allow you to export the build
artifacts as files on the local filesystem instead of a Docker image, which can
be useful for generating local binaries, code generation etc.

The value for `--output` is a CSV-formatted string defining the exporter type
and options. Currently, `local` and `tar` exporters are supported. The `local`
exporter writes the resulting build files to a directory on the client side. The
`tar` exporter is similar but writes the files as a single tarball (`.tar`).

If no type is specified, the value defaults to the output directory of the local
exporter. Use a hyphen (`-`) to write the output tarball to standard output
(`STDOUT`).

The following example builds an image using the current directory (`.`) as build
context, and exports the files to a directory named `out` in the current directory.
If the directory does not exist, Docker creates the directory automatically:

```console
$ docker build -o out .
```

The example above uses the short-hand syntax, omitting the `type` options, and
thus uses the default (`local`) exporter. The example below shows the equivalent
using the long-hand CSV syntax, specifying both `type` and `dest` (destination
path):

```console
$ docker build --output type=local,dest=out .
```

Use the `tar` type to export the files as a `.tar` archive:

```console
$ docker build --output type=tar,dest=out.tar .
```

The example below shows the equivalent when using the short-hand syntax. In this
case, `-` is specified as destination, which automatically selects the `tar` type,
and writes the output tarball to standard output, which is then redirected to
the `out.tar` file:

```console
$ docker build -o - . > out.tar
```

The `--output` option exports all files from the target stage. A common pattern
for exporting only specific files is to do multi-stage builds and to copy the
desired files to a new scratch stage with [`COPY --from`](https://docs.docker.com/engine/reference/builder/#copy).

The example `Dockerfile` below uses a separate stage to collect the
build-artifacts for exporting:

```dockerfile
FROM golang AS build-stage
RUN go get -u github.com/LK4D4/vndr

FROM scratch AS export-stage
COPY --from=build-stage /go/bin/vndr /
```

When building the Dockerfile with the `-o` option, only the files from the final
stage are exported to the `out` directory, in this case, the `vndr` binary:

```console
$ docker build -o out .

[+] Building 2.3s (7/7) FINISHED
 => [internal] load build definition from Dockerfile                                                                          0.1s
 => => transferring dockerfile: 176B                                                                                          0.0s
 => [internal] load .dockerignore                                                                                             0.0s
 => => transferring context: 2B                                                                                               0.0s
 => [internal] load metadata for docker.io/library/golang:latest                                                              1.6s
 => [build-stage 1/2] FROM docker.io/library/golang@sha256:2df96417dca0561bf1027742dcc5b446a18957cd28eba6aa79269f23f1846d3f   0.0s
 => => resolve docker.io/library/golang@sha256:2df96417dca0561bf1027742dcc5b446a18957cd28eba6aa79269f23f1846d3f               0.0s
 => CACHED [build-stage 2/2] RUN go get -u github.com/LK4D4/vndr                                                              0.0s
 => [export-stage 1/1] COPY --from=build-stage /go/bin/vndr /                                                                 0.2s
 => exporting to client                                                                                                       0.4s
 => => copying files 10.30MB                                                                                                  0.3s

$ ls ./out
vndr
```

### <a name="cache-from"></a> Specifying external cache sources (--cache-from)

> **Note**
>
> This feature requires the BuildKit backend. You can either
> [enable BuildKit](https://docs.docker.com/build/buildkit/#getting-started) or
> use the [buildx](https://github.com/docker/buildx) plugin. The previous
> builder has limited support for reusing cache from pre-pulled images.

In addition to local build cache, the builder can reuse the cache generated from
previous builds with the `--cache-from` flag pointing to an image in the registry.

To use an image as a cache source, cache metadata needs to be written into the
image on creation. This can be done by setting `--build-arg BUILDKIT_INLINE_CACHE=1`
when building the image. After that, the built image can be used as a cache source
for subsequent builds.

Upon importing the cache, the builder will only pull the JSON metadata from the
registry and determine possible cache hits based on that information. If there
is a cache hit, the matched layers are pulled into the local environment.

In addition to images, the cache can also be pulled from special cache manifests
generated by [`buildx`](https://github.com/docker/buildx) or the BuildKit CLI
(`buildctl`). These manifests (when built with the `type=registry` and `mode=max`
options) allow pulling layer data for intermediate stages in multi-stage builds.

The following example builds an image with inline-cache metadata and pushes it
to a registry, then uses the image as a cache source on another machine:

```console
$ docker build -t myname/myapp --build-arg BUILDKIT_INLINE_CACHE=1 .
$ docker push myname/myapp
```

After pushing the image, the image is used as cache source on another machine.
BuildKit automatically pulls the image from the registry if needed.

On another machine:

```console
$ docker build --cache-from myname/myapp .
```

### <a name="squash"></a> Squash an image's layers (--squash) (experimental)

#### Overview

Once the image is built, squash the new layers into a new image with a single
new layer. Squashing does not destroy any existing image, rather it creates a new
image with the content of the squashed layers. This effectively makes it look
like all `Dockerfile` commands were created with a single layer. The build
cache is preserved with this method.

The `--squash` option is an experimental feature, and should not be considered
stable.


Squashing layers can be beneficial if your Dockerfile produces multiple layers
modifying the same files, for example, files that are created in one step, and
removed in another step. For other use-cases, squashing images may actually have
a negative impact on performance; when pulling an image consisting of multiple
layers, layers can be pulled in parallel, and allows sharing layers between
images (saving space).

For most use cases, multi-stage builds are a better alternative, as they give more
fine-grained control over your build, and can take advantage of future
optimizations in the builder. Refer to the [use multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/)
section in the userguide for more information.


#### Known limitations

The `--squash` option has a number of known limitations:

- When squashing layers, the resulting image cannot take advantage of layer
  sharing with other images, and may use significantly more space. Sharing the
  base image is still supported.
- When using this option you may see significantly more space used due to
  storing two copies of the image, one for the build cache with all the cache
  layers intact, and one for the squashed version.
- While squashing layers may produce smaller images, it may have a negative
  impact on performance, as a single layer takes longer to extract, and
  downloading a single layer cannot be parallelized.
- When attempting to squash an image that does not make changes to the
  filesystem (for example, the Dockerfile only contains `ENV` instructions),
  the squash step will fail (see [issue #33823](https://github.com/moby/moby/issues/33823)).

#### Prerequisites

The example on this page is using experimental mode in Docker 23.03.

Experimental mode can be enabled by using the `--experimental` flag when starting
the Docker daemon or setting `experimental: true` in the `daemon.json` configuration
file.

By default, experimental mode is disabled. To see the current configuration of
the docker daemon, use the `docker version` command and check the `Experimental`
line in the `Engine` section:

```console
Client: Docker Engine - Community
 Version:           23.0.3
 API version:       1.42
 Go version:        go1.19.7
 Git commit:        3e7cbfd
 Built:             Tue Apr  4 22:05:41 2023
 OS/Arch:           darwin/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          23.0.3
  API version:      1.42 (minimum version 1.12)
  Go version:       go1.19.7
  Git commit:       59118bf
  Built:            Tue Apr  4 22:05:41 2023
  OS/Arch:          linux/amd64
  Experimental:     true
 [...]
```

To enable experimental mode, users need to restart the docker daemon with the
experimental flag enabled.

#### Enable Docker experimental

To enable experimental features, you need to start the Docker daemon with
`--experimental` flag. You can also enable the daemon flag via
`/etc/docker/daemon.json`, for example:

```json
{
    "experimental": true
}
```

Then make sure the experimental flag is enabled:

```console
$ docker version -f '{{.Server.Experimental}}'
true
```

#### Build an image with `--squash` argument

The following is an example of docker build with `--squash` argument

```dockerfile
FROM busybox
RUN echo hello > /hello
RUN echo world >> /hello
RUN touch remove_me /remove_me
ENV HELLO=world
RUN rm /remove_me
```

An image named `test` is built with `--squash` argument.

```console
$ docker build --squash -t test .

<...>
```

If everything is right, the history looks like this:

```console
$ docker history test

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
4e10cb5b4cac        3 seconds ago                                                       12 B                merge sha256:88a7b0112a41826885df0e7072698006ee8f621c6ab99fca7fe9151d7b599702 to sha256:47bcc53f74dc94b1920f0b34f6036096526296767650f223433fe65c35f149eb
<missing>           5 minutes ago       /bin/sh -c rm /remove_me                        0 B
<missing>           5 minutes ago       /bin/sh -c #(nop) ENV HELLO=world               0 B
<missing>           5 minutes ago       /bin/sh -c touch remove_me /remove_me           0 B
<missing>           5 minutes ago       /bin/sh -c echo world >> /hello                 0 B
<missing>           6 minutes ago       /bin/sh -c echo hello > /hello                  0 B
<missing>           7 weeks ago         /bin/sh -c #(nop) CMD ["sh"]                    0 B
<missing>           7 weeks ago         /bin/sh -c #(nop) ADD file:47ca6e777c36a4cfff   1.113 MB
```

We could find that a layer's name is `<missing>`, and there is a new layer with
COMMENT `merge`.

Test the image, check for `/remove_me` being gone, make sure `hello\nworld` is
in `/hello`, make sure the `HELLO` environment variable's value is `world`.
