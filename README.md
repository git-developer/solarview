# solarview
Docker support for SolarView

## User documentation
### Overview
This project allows to run [SolarView](https://www.solarview.info/solarview_linux.aspx) in a Docker container.

Main features:
* Separation of SolarView from the main system
* Easy update of all SolarView applications with a single command (`docker pull`)
* Multi-architecture support (Raspberry Pi, x86 server)

Advantages:
* Up-to-date operating system and third-party packages for HTTP server, FTP upload and mail
* Separate containers for SolarView main application, SolarView proxy applications and HTTP server

### Requirements
* Docker Compose

Installation on Debian-based systems:

```sudo apt install docker-compose```

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
There's no need to build an image because this is done automatically.
The build creates images for all supported architectures with appropriate tags and labels.
See one of the build repositories if you're interested in the details, e.g.
* [solarview-linux](../solarview-linux)
* [solarview-d0](../solarview-d0)
* [solarview-steca](../solarview-steca)
* [solarview-base](../solarview-base)

Alternatively, the build may be run manually for a single architecture and without docker labels,
by running a legacy shell script. See [shell-build](./tree/shell-build#how-to-build-an-image-for-a-supported-app) for details.

### How to update an image for a supported app to the current version?
Manually trigger the GitLab CI build or [create an issue](./issues) and ask the maintainer to do so.

### How to add support for an unsupported app?
Steps to add support for the unsupported app `foo-fb`:

1. Create a git repository `solarview-foo` containing the file `.gitlab-ci.yml` with the following content:
``` 
include:
  remote: 'https://github.com/git-developer/solarview-base/raw/develop/.gitlab-ci.yml'

variables:
  DOCKERFILE_URL: 'https://github.com/git-developer/solarview-base/raw/develop/Dockerfile'
  BUILD_ARGS: 'APP_NAME=foo-fb'
  IMAGE_TITLE: 'SolarView Foo-Proxy'
```
1. If the app needs special handling, customize the `.gitlab-ci.yml` and/or the Dockerfile
1. Build the repository via [GitLab CI](https://gitlab.com/).
   The images will be built and pushed to the registry that is configured in the GitLab project
   (e.g. Docker Hub or the GitLab internal registry).
