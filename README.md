# solarview
Docker support for [SolarView](https://www.solarview.info/solarview_fritzbox.aspx)

## User documentation
### Overview
This project allows to run [SolarView](https://www.solarview.info/solarview_fritzbox.aspx) in a Docker container.

Main features:
* Separation of SolarView from the main system
* Easy update of all SolarView applications with a single command (`docker pull`)
* Multi-architecture support (e.g. Raspberry Pi, x86 server)

Advantages:
* Up-to-date operating system and third-party packages for HTTP server, FTP upload and mail
* Separate containers for SolarView main application, SolarView proxy applications and HTTP server

### Requirements
* Docker
* Docker Compose

Installation on Debian-based systems:

```sudo apt install docker-ce docker-compose```

### Usage
#### Configuration
* Create a `docker-compose.yml`

See the examples on how to create a `docker-compose.yml` for your environment.

#### Start
* Run:

```docker-compose up -d```

When the option `restart: unless-stopped` is configured in `docker-compose.yml`, SolarView is started automatically at system boot.

#### Stop
* Run:

```docker-compose down```

### Migration from native SolarView

1. Stop native SolarView
1. Copy directory of native SolarView to `./data/solarview`
1. Copy temp directory of native SolarView to `./data/solarview-temp`

Example:

```
/home/pi/svrpi/stop.sh
cp -a /opt/svrpi ./data/solarview
cp -a /var/tmp ./data/solarview-temp
```

Subdirectories for proxy applications (e.g. `steca-fb`, `d0-fb`) are not used and may safely be deleted from `./data/solarview`.

## Developer documentation
In this section, the term _app_ is used for either the SolarView main application (`solarview-fb`) or a SolarView proxy application (e.g. `steca-fb`).

### How to build an image for a supported app?
To build the app `foo-fb`, run: `./image/build-image foo-fb`.

The build may be customized using the following environment variables:

* `IMAGE_REPOSITORY`: name of the docker image repository; default: `ckware`
* `IMAGE_NAME`: name of the docker image; default: `solarview-foo` (exception: `solarview` for SolarView main app)

To see a list of supported apps, run `./image/build-image`.

### How to add support for an unsupported app?
Steps to add support for the unsupported app `foo-fb`:

1. Create a directory `./image/app/foo-fb` containing
    * file `env`
    * directory `build-context`
    * directory `runtime-context`
1. Optional: add an app-dependent build hook `./image/app/foo-fb/build-context/hook` and/or runtime hook `./image/app/foo-fb/runtime-context/hook`

The `env` file and the directories may be empty. If you want to add an empty directory to a version control system like git, don't forget to create a dummy file (e.g. `.keep`).

* `env` may contain environment variables that are necessary to install `foo-fb`. Pre-defined variables:
    * `APP_NAME`: name of the main binary; default: `foo-fb`
    * `APP_ARCHIVE_URL`: URL to the app release archive; default: `http://www.solarview.info/downloads/${APP_NAME}.zip`
    * `APP_BINARIES`: list of binary files; default: `${APP_NAME}`
* `build-context` holds files that are necessary to install `foo-fb`.
* `runtime-context` holds files that are necessary to run `foo-fb`.
* The `hook` files have to be executables, they are run when an image is built.
    * `./image/arch/foo/build-context/hook` is run in the build container after the app release archive is unpacked
    * `./image/arch/foo/runtime-context/hook` is run in the runtime container

### How to add support for an unsupported architecture?
Steps to add support for the unsupported architecture `foo`:

* If `foo` is compatible to a supported architecture `bar`:
    * Create a symlink `./image/arch/foo` pointing to existing architecture directory `./image/arch/bar`
* Otherwise:
    1. Create a directory `./image/arch/foo` containing
        * file `env`
        * directory `build-context`
        * directory `runtime-context`
    1. Optional: add an arch-dependent build hook `./image/arch/foo/build-context/hook` and/or runtime hook `./image/arch/foo/runtime-context/hook`

The `env` file and the directories may be empty. If you want to add an empty directory to a version control system like git, don't forget to create a dummy file (e.g. `.keep`).

* `env` may contain environment variables that are necessary to add support the architecture `foo`.
    * `ARCH_ID`: id for the runtime architecure (currently one of `rpi`, `x86`, `7390`, `71xx`)
    * `RUNTIME_BASE_IMAGE`: docker base image for the runtime
* `build-context` holds files that are necessary to add support for the architecture `foo` at build time.
* `runtime-context` holds files that are necessary to add support for the architecture `foo` at runtime.
* The `hook` files have to be executables, they are run when an image is built.
    * `./image/arch/foo/build-context/hook` is run in the build container after the app release archive is unpacked
    * `./image/arch/foo/runtime-context/hook` is run in the runtime container

### What happens within a build of an app?
The following steps are performed when an app is built:

1. Create a docker image
    1. Download the ZIP archive of the app
    1. Select the binaries (variable `$APP_BINARIES`) that match the runtime architecture
    1. Run the arch build hook (e.g. copy additional arch-dependent files from the local filesystem)
    1. Run the app build hook (e.g. copy additional app-dependent files from the local filesystem)
    1. Detect compile date, release date and version of the app
    1. Run the arch runtime hook (e.g. install arch-dependent packages)
    1. Run the app runtime hook (e.g. install app-dependent packages)
1. Tag the docker image with version, compile date of the app binary, release date of the app archive and build date of the docker image
