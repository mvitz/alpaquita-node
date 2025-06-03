# Alpaquita Node

The `pom.xml` uses `com.github.eirslett:frontend-maven-plugin` to download `node` and `npm` and executes `npm ci` afterwards.

The `run` script starts a given Docker container and executes a given script. By default this is `./execute` which calls `./mvnw clean package` in which the above should happen.

## ✅ ./run bellsoft/liberica-openjdk-debian:21.0.7-9

Using the debian based liberica image everything works as expected:

```
[INFO] Scanning for projects...
...
[INFO] --- frontend:1.15.1:install-node-and-npm (install node and npm) @ alpaquite-node ---
[INFO] Installing node version v22.16.0
[INFO] Unpacking /root/.m2/repository/com/github/eirslett/node/22.16.0/node-22.16.0-linux-arm64.tar.gz into /build/target/node/tmp
[INFO] Copying node binary from /build/target/node/tmp/node-v22.16.0-linux-arm64/bin/node to /build/target/node/node
[INFO] Extracting NPM
[INFO] Installed node locally.
[INFO]
[INFO] --- frontend:1.15.1:npm (npm install) @ alpaquite-node ---
[INFO] Running 'npm ci' in /build
[INFO]
[INFO] added 2 packages, and audited 3 packages in 616ms
[INFO]
[INFO] 2 packages are looking for funding
[INFO]   run `npm fund` for details
[INFO]
[INFO] found 0 vulnerabilities
[INFO] npm notice
[INFO] npm notice New major version of npm available! 10.9.2 -> 11.4.1
[INFO] npm notice Changelog: https://github.com/npm/cli/releases/tag/v11.4.1
[INFO] npm notice To update run: npm install -g npm@11.4.1
[INFO] npm notice
...
[INFO] BUILD SUCCESS
...
```

## ⛔️ ./run bellsoft/liberica-openjdk-alpine:21.0.7-9 ./apk

Using the alpine glibc based liberica image the frontend plugin is not able to download node because for some reason it tries to download a musl based node.

```
[INFO] Scanning for projects...
...
[INFO] --- frontend:1.15.1:install-node-and-npm (install node and npm) @ alpaquite-node ---
[INFO] Installing node version v22.16.0
[INFO] Downloading https://unofficial-builds.nodejs.org/download/release/v22.16.0/node-v22.16.0-linux-arm64-musl.tar.gz to /root/.m2/repository/com/github/eirslett/node/22.16.0/node-22.16.0-linux-arm64-musl.tar.gz
[INFO] No proxies configured
[INFO] No proxy was configured, downloading directly
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
...
[ERROR] Failed to execute goal com.github.eirslett:frontend-maven-plugin:1.15.1:install-node-and-npm (install node and npm) on project alpaquite-node: Could not download Node.js: Got error code 404 from the server. -> [Help 1]
...
```

## ⛔️ ./run bellsoft/liberica-openjdk-alpine-musl:21.0.7-9 ./apk

Using the alpine musl based liberica image the same as with the glibc one happens. This is expected.

## ✅ ./run bellsoft/liberica-runtime-container:jdk-21.0.7_9-stream-glibc ./apk

Using the Alpaquita glibc based liberica image `npm ci` fails because the `libstdc++.so.6` is not found.
This can be solved by adding libstdc++ using apk with `apk add libstdc++`.

## ⛔️ ./run bellsoft/liberica-runtime-container:jdk-21.0.7_9-stream-musl ./apk

Using the Alpaquita based liberica image `npm ci` fails with a `No such file or directory` exception.

```
[INFO] Scanning for projects...
...
[INFO] --- frontend:1.15.1:install-node-and-npm (install node and npm) @ alpaquite-node ---
[INFO] Installing node version v22.16.0
[INFO] Unpacking /root/.m2/repository/com/github/eirslett/node/22.16.0/node-22.16.0-linux-arm64.tar.gz into /build/target/node/tmp
[INFO] Copying node binary from /build/target/node/tmp/node-v22.16.0-linux-arm64/bin/node to /build/target/node/node
[INFO] Extracting NPM
[INFO] Installed node locally.
[INFO]
[INFO] --- frontend:1.15.1:npm (npm install) @ alpaquite-node ---
[INFO] Running 'npm ci' in /build
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
...
[ERROR] Failed to execute goal com.github.eirslett:frontend-maven-plugin:1.15.1:npm (npm install) on project alpaquite-node: Failed to run task: 'npm ci' failed. java.io.IOException: Cannot run program "/build/target/node/node" (in directory "/build"): error=2, No such file or directory -> [Help 1]
...
```
