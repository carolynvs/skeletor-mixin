# A Porter Mixin Skeleton

This repository contains the skeleton structure of a Porter Mixin. You can clone this repository and use it as a starting point to build new mixins. The structure of this project matches closely with existing Porter Mixins like [Porter Azure](https://github.com/deislabs/porter-azure), [Porter Helm](https://github.com/deislabs/porter-helm), and [Porter Terraform](https://github.com/deislabs/porter-terraform).

1. Create a new repository in GitHub [using this repository as a
   template](https://help.github.com/en/articles/creating-a-repository-from-a-template).
1. Clone your new repository following the usual Go conventions. If you are using
   Go 1.11, clone it into your GOPATH such as
   ~/go/src/github.com/YOURNAME/YOURMIXIN. If you are using Go 1.12+ with go
   modules, you may chose to clone it outside of the GOPATH.
1. Rename the `cmd/skeletor` and `pkg/skeletor` directories to `cmd/YOURMIXIN` and
   `pkg/YOURMIXIN`.
1. Find the text `github.com/deislabs/pkg/skeletor` in the repository and change it to 
    `github.com/YOURNAME/pkg/YOURMIXIN`.
1. Find any remaining `skeletor` text in the repository and replace it with `YOURMIXIN`.
1. Run `make build xbuild test` to try out all the make targets and
   verify that everything executes without failing.
1. Run `make install` to install your mixin into the Porter home directory. If
   you don't already have Porter installed, [install](https://porter.sh/install) it first.
1. Now your mixin is installed, you are ready start customizing and iterating on
   your mixin!

## Customize your mixin

Start by modifying the `InstallArguments` struct defined in
`pkg/skeletor/install.go`. This structure defines the yaml snippet for your
mixin. Take a look at the helm mixin to get a feel for what you can do 

https://github.com/deislabs/porter-helm/blob/7c5a656f0c38d23e7d4efc8a4ba26d03f06c06e8/pkg/helm/install.go#L20-L32

Next you can use the properties that you added in the `Install` function. Look
for the `TODO` in that function to know where to put your code. For
many mixins, this may end up being a call out to an executable in the
invocationImage but it could be an API call or something else too. Here is
another example from the helm mixin, which uses the yaml input to build a call
to the helm executable.

https://github.com/deislabs/porter-helm/blob/7c5a656f0c38d23e7d4efc8a4ba26d03f06c06e8/pkg/helm/install.go#L55

If your mixin requires anything installed in the invocation image, for example a
client tool like `helm`, edit the `Build` function in `pkg/skeletor/build.go`.
Here you can add any Dockerfile lines that you require to download and install
additional tools, configuration files, etc necessary for your mixin. The Build
function should write the Dockerfile lines to `m.Out` which is a pipe from the
mixin back to porter.

Here is an example from the helm mixin, where it downloads a particular version
of the helm binary and initialize the helm client.

https://github.com/deislabs/porter-helm/blob/7c5a656f0c38d23e7d4efc8a4ba26d03f06c06e8/pkg/helm/build.go#L7-L19

This is enough to have a working mixin. Run `make build install` and then test
it out with a bundle. Once you have a feel for it, you will want to implement
upgrade and uninstall as well, most likely reusing the same code from install
though that is up to you.

That will get you started but make sure to read the mixin developer
documentation for how to create a full featured mixin:

* [Mixin Architecture](https://porter.sh/mixin-architecture/)
* [Mixin Commands](https://porter.sh/mixin-commands/)
* [Distributing Mixins](https://porter.sh/mixin-distribution/)

## Project Structure

In the `cmd/skeletor` directory, you will find a cli built using [spf13/cobra](https://github.com/spf13/cobra). The CLI contains a go file for each basic capability a Mixin should implement:

* build
* schema
* verison
* install
* upgrade
* delete

Each of these command implementations have a corresponding Mixin implemention in the `pkg/skeletor` directory. Each of the commands above is wired into an empty implementation in `pkg/skeletor` that needs to be completed. In order to build a new Mixin, you need to complete these implementations with the relevant technology. For example, to build a [Cloud Formation](https://aws.amazon.com/cloudformation/) mixin, you might implement the methods in `pkg/skeletor` using the [AWS Go SDK](https://docs.aws.amazon.com/sdk-for-go/api/service/cloudformation/).

## Provided capabilities

This skeleton mixin project brings some free capabilities:

### File System Access and Context

Porter provides a [Context](https://github.com/deislabs/porter/tree/master/pkg/context) package that has helpful mechanisms for accessing the File System using [spf13/afero](https://github.com/spf13/afero). This makes it easy to provide mock File System implementations during testing. The Context package also provides a mechanism to encapsualte stdin, stdout and stderr so that they can easily be passed from `cmd/skeletor` code to implementing `pkg/skeletor` code.  

### Template and Static Asset Handling

The project already includes [Packr V2](https://github.com/gobuffalo/packr/tree/master/v2) for dealing with static files, such as templates or other content that is best modeled outside of a Go file. You can see an example of this in `pkg/skeletor/schema.go`.

### Basic Schema

The project provides an implementation of the `skeletor schema` command that is mostly functional. To fully implement this for your mixin, you simply need to provide a valid JSON schema. For reference, consult `pkg/skeletor/schema/skeletor.json`.

### Basic Tests

The project provides some very basic test skeletons that you can use as a starting point for building tests for your mixin.

### Makefile

The project also includes a Makefile that will can be used to both build and install the mixin. The Makefile also includes a TODO `publish` target that shows how you might publish the mixin and generate an mixin feed for easily sharing your mixin.