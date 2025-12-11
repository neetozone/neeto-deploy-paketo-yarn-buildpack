# Paketo Buildpack for Yarn

The Yarn CNB provides the [Yarn Package manager](https://yarnpkg.com). The
buildpack installs `yarn` onto the `$PATH` which makes it available for
subsequent buildpacks and/or in the final running container. An example of
buildpack that might use yarn is the [Yarn Install
CNB](https://github.com/paketo-buildpacks/yarn-install)

## Integration

The Yarn CNB provides `yarn` as dependency. Downstream buildpacks, like [Yarn
Install CNB](https://github.com/paketo-buildpacks/yarn-install) can require the
`yarn` dependency by generating a [Build Plan
TOML](https://github.com/buildpacks/spec/blob/master/buildpack.md#build-plan-toml)
file that looks like the following:

```toml
[[requires]]

  # The name of the Yarn dependency is "yarn". This value is considered
  # part of the public API for the buildpack and will not change without a plan
  # for deprecation.
  name = "yarn"

  # The Yarn buildpack supports some non-required metadata options.
  [requires.metadata]

    # The version of the Yarn dependency is not required. In the case it
    # is not specified, the buildpack will provide the default version, which can
    # be seen in the buildpack.toml file.
    # If you wish to request a specific version, the buildpack supports
    # specifying a semver constraint in the form of "1.*", "1.22.*", or even
    # "1.22.4".
    version = "1.22.4"

    # Setting the build flag to true will ensure that the yarn
    # depdendency is available on the $PATH for subsequent buildpacks during
    # their build phase. If you are writing a buildpack that needs to run yarn
    # during its build process, this flag should be set to true.
    build = true

    # Setting the launch flag to true will ensure that the yarn
    # dependency is available on the $PATH for the running application. If you are
    # writing an application that needs to run yarn at runtime, this flag should
    # be set to true.
    launch = true
```

## Packaging

To package this buildpack for consumption:

```bash
./scripts/package.sh --version 2.2.6
```

This will build the buildpack for all target architectures specified in `buildpack.toml` (amd64 and arm64 by default) and create a single archive containing binaries for all architectures in the `build/` directory.

This will create a `buildpackage.cnb` file under the `build` directory which you
can use to build your app as follows:
```shell
pack build <app-name> \
  --path <path-to-app> \
  --buildpack <path/to/node-engine.cnb> \
  --buildpack build/buildpackage.cnb \
  --buildpack <path/to/cnb-that-requires-node-and-yarn>
```

Though the API of this buildpack does not require `node`, yarn is unusable without node.

## Publishing

To publish this buildpack to ECR:

```bash
# First, authenticate with ECR (if not already authenticated)
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin 348674388966.dkr.ecr.us-east-1.amazonaws.com

# Then publish the buildpack
./scripts/publish.sh \
  --image-ref 348674388966.dkr.ecr.us-east-1.amazonaws.com/neeto-deploy/paketo/buildpack/yarn:<version> \
  --buildpack-type buildpack
```

The script will automatically:
- Read target architectures from `buildpack.toml`
- Extract the buildpack archive
- Publish each architecture separately with arch-suffixed tags (e.g., `yarn:<version>-amd64`, `yarn:<version>-arm64`)
- Create and push a multi-arch manifest list

## Usage

## Run Tests

To run all unit tests, run:
```shell
./scripts/unit.sh
```

To run all integration tests, run:
```shell
/scripts/integration.sh
```
