## Mono Static

This image contains the full Mono development environment and a gcc environment, which can be used to create a statically compiled Mono binary.

As part of the build, it expects a `build.sh` to be present in the working directory to compile the app.

An example `buildi-app.sh` file:

```
#!/bin/bash

# This is used from the CI image to compile and build the app into a static binary.
nuget restore -NonInteractive

PACKAGES_DIR=$(find /usr/src/app/source/packages/ -name *.dll)
PACKAGES=""

for f in $PACKAGES_DIR; do
   PACKAGES="$PACKAGES $f"
done

echo "Found packages: ${PACKAGES}"
xbuild /p:Configuration=Release /property:OutDir=/usr/src/app/build/ ./console.sln
mkbundle --deps --static ./obj/x86/Release/console.exe $PACKAGES -o consoleapp
```

## Usage

This image uses the `ONBUILD` Docker feature which is great for use as an intermediate / CI container to produce your static artifacts
for use in smaller containers.

### Create your CI 
Assuming you have the `build-app.sh` from above in the root dir, you can now build an intermediate image containing your app:

Dockerfile

```
FROM mfellows/mono-static

MAINTAINER Matt Fellows <matt.fellows@onegeek.com.au>

ONBUILD WORKDIR /usr/src/app/build
CMD [ "echo", "build completed!" ]
```

This will produce a container with a compiled binary in the `/usr/src/app/build` directory with the name `consoleapp`.
Extract the binary from the container for embedding into a smaller runtime container, such as busybox:

```
docker build -t tmp-build .                                         # Run the CI build
docker run -d --name tmp-build tmp-build                            # Run the 'ci' container and get the artifact
docker cp tmp-build:/usr/src/app/source/packages .                  # This is a sort of package caching mechanism
docker cp tmp-build:/usr/src/app/source/consoleapp ./distribution/  # Put static binary with distribution container build
docker rm -f tmp-build
```

Now create `distribution\Dockerfile`:

```
FROM progrium/busybox
RUN opkg-install libc-dev
ADD ./consoleapp console
CMD [ "./console" ]
```

You can build this with `docker build -t mfellows/mono-api distribution`

## Tutorial

See [A Nancy .NET Microservice running on Docker in under 20mb](http://www.onegeek.com.au/articles/a-nancy-net-microservice-running-on-docker-in-under-20mb) for a walkthrough.
