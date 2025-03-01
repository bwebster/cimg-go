<div align="center">
	<p>
		<img alt="CircleCI Logo" src="https://raw.github.com/CircleCI-Public/cimg-go/main/img/circle-circleci.svg?sanitize=true" width="75" />
		<img alt="Docker Logo" src="https://raw.github.com/CircleCI-Public/cimg-go/main/img/circle-docker.svg?sanitize=true" width="75" />
		<img alt="Go Logo" src="https://raw.github.com/CircleCI-Public/cimg-go/main/img/circle-go.svg?sanitize=true" width="75" />
	</p>
	<h1>CircleCI Convenience Images => Go</h1>
	<h3>A Continuous Integration focused Go Docker image built to run on CircleCI</h3>
</div>

[![CircleCI Build Status](https://circleci.com/gh/CircleCI-Public/cimg-go.svg?style=shield)](https://circleci.com/gh/CircleCI-Public/cimg-go) [![Software License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/CircleCI-Public/cimg-go/main/LICENSE) [![Docker Pulls](https://img.shields.io/docker/pulls/cimg/go)](https://hub.docker.com/r/cimg/go) [![CircleCI Community](https://img.shields.io/badge/community-CircleCI%20Discuss-343434.svg)](https://discuss.circleci.com/c/ecosystem/circleci-images) [![Repository](https://img.shields.io/badge/github-README-brightgreen)](https://github.com/CircleCI-Public/cimg-go)

**_This image is designed to supercede the legacy CircleCI Go image, `circleci/golang`._**

`cimg/go` is a Docker image created by CircleCI with continuous integration builds in mind.
Each tag contains a complete Go version and toolchain, the testing wrapper `gotestsum`, and any binaries and tools that are required for builds to complete successfully in a CircleCI environment.

## Support Policy

The CircleCI Docker Convenience Image support policy can be found on the [CircleCI docs](https://circleci.com/docs/convenience-images-support-policy) site. This policy outlines the release, update, and deprecation policy for CircleCI Docker Convenience Images.

## Table of Contents

- [Getting Started](#getting-started)
- [How This Image Works](#how-this-image-works)
  - [Changes From Legacy Image](#changes-from-legacy-image)
- [Development](#development)
- [Contributing](#contributing)
- [Additional Resources](#additional-resources)
- [License](#license)

## Getting Started

This image can be used with the CircleCI `docker` executor.
For example:

```yaml
jobs:
  build:
    docker:
      - image: cimg/go:1.17
    steps:
      - checkout
      - run: go version
```

In the above example, the CircleCI Go Docker image is used for the primary container.
More specifically, the tag `1.17` is used meaning the version of Go will be Go v1.17.
You can now use Go within the steps for this job.

## How This Image Works

This image contains the Go programming language and its complete toolchain.
This includes support for Go modules, the official Go Proxy Server, etc.

### Changes From Legacy Image

If you're coming from the legacy version of this image, `circleci/golang`, here's any changes that you might want to be aware of:

- `GOPATH` - this envar has changed from `/go` to `$HOME/go`.
The latter will expand to the full path of `/home/circleci/go`.
On first run, this change may affect caching and some other commands if you don’t correct the page in your config.

### Variants

Variant images typically contain the same base software, but with a few additional modifications.

#### Node.js

The Node.js variant is the same Go image but with Node.js also installed.
The Node.js variant will be used by appending `-node` to the end of an existing `cimg/go` tag.

```yaml
jobs:
  build:
    docker:
      - image: cimg/go:1.17-node
    steps:
      - checkout
      - run: go version
      - run: node --version
```

#### Browsers

The browsers variant is the same Go image but with Node.js, Java, Selenium, and browser dependencies pre-installed via apt.
The browsers variant can be used by appending `-browser` to the end of an existing `cimg/go` tag.
The browsers variant is designed to work in conjunction with the [CircleCI Browser Tools orb](https://circleci.com/developer/orbs/orb/circleci/browser-tools).
You can use the orb to install a version of Google Chrome and/or Firefox into your build. The image contains all of the supporting tools needed to use both the browser and its driver.

```yaml
orbs:
  browser-tools: circleci/browser-tools@1.1
jobs:
  build:
    docker:
      - image: cimg/go:1.17-browsers
    steps:
      - browser-tools/install-browser-tools
      - checkout
      - run: |
          go version
          node --version
          java --version
          google-chrome --version
```

### Tagging Scheme

This image has the following tagging scheme:

```
cimg/go:<go-version>[-variant]
```

`<go-version>` - The version of Go to use.
This can be a full SemVer point release (such as `1.12.7`) or just the minor release (such as `1.12.0`).
**Note: For some reason the Go Team releases the first minor release without the `.0` at the end.
So instead of releasing Go v1.18.0, they release v1.18.
For this image, we add the `.0` anyway so that the tagging scheme works with our tooling.**
If you use the minor release tag, it will automatically point to future patch updates as they are released by the Go Team.
For example, the tag `1.17` points to Go v1.17 now, but when the next release comes out, it will point to Go v1.17.1.

`[-variant]` - Variant tags, if available, can optionally be used.
Once the Node.js variant is available, it could be used like this: `cimg/go:1.17-node`.

## Development

Images can be built and run locally with this repository.
This has the following requirements:

- local machine of Linux (Ubuntu tested) or macOS
- modern version of Bash (v4+)
- modern version of Docker Engine (v19.03+)

### Cloning For Community Users (no write access to this repository)

Fork this repository on GitHub.
When you get your clone URL, you'll want to add `--recurse-submodules` to the clone command in order to populate the Git submodule contained in this repo.
It would look something like this:

```bash
git clone --recurse-submodules <my-clone-url>
```

If you missed this step and already cloned, you can just run `git submodule update --init` to populate the submodule.
Then you can optionally add this repo as an upstream to your own:

```bash
git remote add upstream https://github.com/CircleCI-Public/cimg-go.git
```

### Cloning For Maintainers ( you have write access to this repository)

Clone the project with the following command so that you populate the submodule:

```bash
git clone --recurse-submodules git@github.com:CircleCI-Public/cimg-go.git
```

### Generating Dockerfiles

Dockerfiles can be generated for a specific Go version using the `gen-dockerfiles.sh` script.
For example, to generate the Dockerfile for Go v1.18, you would run the following from the root of the repo:

```bash
./shared/gen-dockerfiles.sh 1.18.0
```

The generated Dockerfile will be located at `./1.18/Dockefile`.
To build this image locally and try it out, you can run the following:

```bash
cd 1.18
docker build -t test/go:1.18.0 .
docker run -it test/go:1.18.0 bash
```

**Note: For some reason the Go Team releases the first minor release without the `.0` at the end.
So instead of releasing Go v1.18.0, they release v1.18.
For this image, we add the `.0` anyway so that the tagging scheme works with our tooling.**

### Building the Dockerfiles

To build the Docker images locally as this repository does, you'll want to run the `build-images.sh` script:

```bash
./build-images.sh
```

This would need to be run after generating the Dockerfiles first.
When releasing proper images for CircleCI, this script is run from a CircleCI pipeline and not locally.

### Publishing Official Images (for Maintainers only)

The individual scripts (above) can be used to create the correct files for an image, and then added to a new git branch, committed, etc.
A release script is included to make this process easier.
To make a proper release for this image, let's use the fake Go version of Go v9.99, you would run the following from the repo root:

```bash
./shared/release.sh 9.99
```

This will automatically create a new Git branch, generate the Dockerfile(s), stage the changes, commit them, and push them to GitHub.
The commit message will end with the string `[release]`.
This string is used by CircleCI to know when to push images to Docker Hub.
All that would need to be done after that is:

- wait for build to pass on CircleCI
- review the PR
- merge the PR

The main branch build will then publish a release.

**Note: For some reason the Go Team releases the first minor release without the `.0` at the end.
So instead of releasing Go v1.18.0, they release v1.18.
For this image, we add the `.0` anyway so that the tagging scheme works with our tooling.**

### Incorporating Changes

How changes are incorporated into this image depends on where they come from.

**build scripts** - Changes within the `./shared` submodule happen in its [own repository](https://github.com/CircleCI-Public/cimg-shared).
For those changes to affect this image, the submodule needs to be updated.
Typically like this:

```bash
cd shared
git pull
cd ..
git add shared
git commit -m "Updating submodule for foo."
```

**parent image** - By design, when changes happen to a parent image, they don't appear in existing Go images.
This is to aid in "determinism" and prevent breaking customer builds.
New Go images will automatically pick up the changes.

If you _really_ want to publish changes from a parent image into the Go image, you have to build a specific image version as if it was a new image.
This will create a new Dockerfile and once published, a new image.

**Go specific changes** - Editing the `Dockerfile.template` file in this repo will modify the Go image specifically.
Don't forget that to see any of these changes locally, the `gen-dockerfiles.sh` script will need to be run again (see above).

## Contributing

We encourage [issues](https://github.com/CircleCI-Public/cimg-go/issues) and [pull requests](https://github.com/CircleCI-Public/cimg-go/pulls) against this repository.

Please check out our [contributing guide](.github/CONTRIBUTING.md) which outlines best practices for contributions and what you can expect from the images team at CircleCI.

## Additional Resources

[CircleCI Docs](https://circleci.com/docs/) - The official CircleCI Documentation website.
[CircleCI Configuration Reference](https://circleci.com/docs/2.0/configuration-reference/#section=configuration) - From CircleCI Docs, the configuration reference page is one of the most useful pages we have.
It will list all of the keys and values supported in `.circleci/config.yml`.
[Docker Docs](https://docs.docker.com/) - For simple projects this won't be needed but if you want to dive deeper into learning Docker, this is a great resource.

## License

This repository is licensed under the MIT license.
The license can be found [here](./LICENSE).
