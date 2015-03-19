## Mono Static

This image contains the full Mono development environment and a gcc environment, which can be used to create a statically compiled Mono binary.

As part of the build, it expects a `build.sh` to be present in the working directory to compile the app.

An example `build.sh` file:

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
