# stackup

[![Build Status](https://travis-ci.org/realestate-com-au/stackup.svg?branch=master)](https://travis-ci.org/realestate-com-au/stackup)

Stackup provides a CLI and a simplified Ruby API for dealing with
AWS CloudFormation stacks.

## Why?

Stackup provides some advantages over using `awscli` or `aws-sdk` directly:

  - It treats stack changes as synchronous, streaming stack events until the
    stack reaches a stable state.

  - A `Stack#up` facade for `create`/`update` frees you from having to know
    whether your stack already exists or not.

  - Changes are (mostly) idempotent: "no-op" operations - e.g. deleting a
    stack that doesn't exist, or updating without a template change - are
    handled gracefully (i.e. without error).

## Installation

    $ gem install stackup

## Command-line usage

The entry-point is the "stackup" command.

Most commands operate in the context of a named stack:

    $ stackup STACK-NAME ...

Called with `--list`, it will list stacks:

    $ stackup --list
    foo-bar-test
    zzz-production

The command-line support inputs (template and parameters) in either JSON or YAML format.

### Stack create/update

Use sub-command "up" to create or update a stack, as appropriate:

    $ stackup myapp-test up -t template.json

This will:

  * update (or create) the named CloudFormation stack, using the specified template
  * monitor events until the stack update is complete

### Stack deletion

Sub-command "delete" deletes the stack.

### Stack inspection

Inspect details of a stack with:

    $ stackup myapp-test status
    $ stackup myapp-test resources
    $ stackup myapp-test outputs

## Programmatic usage

Get a handle to a `Stack` object as follows:

    stack = Stackup.stack("my-stack")

You can pass an `Aws::CloudFormation::Client`, or client config,
to `Stackup`, e.g.

    stack = Stackup(credentials).stack("my-stack")

See {Stackup::Stack} for more details.

## Rake integration

Stackup integrates with Rake to generate handy tasks for managing a stack, e.g.

    require "stackup/rake_tasks"

    Stackup::RakeTasks.new("app") do |t|
      t.stack = "my-app"
      t.template = "app-template.json"
    end

providing tasks:

    rake app:diff       # Show pending changes to my-app stack
    rake app:down       # Delete my-app stack
    rake app:inspect    # Show my-app stack outputs and resources
    rake app:up         # Update my-app stack

Parameters and tags may be specified via files, or as a Hash, e.g.

    Stackup::RakeTasks.new("app") do |t|
      t.stack = "my-app"
      t.template = "app-template.json"
      t.parameters = "production-params.json"
      t.tags = { "environment" => "production" }
    end

## Docker image

Stackup is also published as a Docker image. Basic usage is:

    docker run --rm \
        -v `pwd`:/cwd \
        -e AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY -e AWS_SESSION_TOKEN \
        -e AWS_DEFAULT_REGION \
        realestate/stackup:latest ...

Replace "latest" with a specific version for added safety.

The default working-directory within the container is `/cwd`;
hence the volume mount to make files available from the host system.
