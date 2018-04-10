# Bash Commons

This repo contains a collection of reusable Bash functions for handling common tasks such as logging, assertions,
string manipulation, and more. It is our attempt to bring a little more sanity, predictability, and coding reuse to
our Bash scripts.




## Install

The first step is to download the code onto your computer.

One way to do this is to use Git to `clone` the code and then you can copy the [bash-commons src
folder](/modules/bash-commons/src) to its final destination (note, you'll need to replace `<VERSION>` below with a
version number from the [releases page](https://github.com/gruntwork-io/bash-commons/releases)):

```bash
git clone --branch <VERSION> https://github.com/gruntwork-io/bash-commons.git
cp -r bash-commons/modules/bash-commons/src /opt/gruntwork/bash-commons
```

Another option is to install the [Gruntwork Installer](https://github.com/gruntwork-io/gruntwork-installer) and to
let it do the work for your (default instlal location is `/opt/gruntwork/bash-commons`):

```bash
gruntwork-install --repo https://github.com/gruntwork-io/bash-commons --module-name bash-commons --tag <VERSION>
```

Now you can use `source` to "import" the modules you need and use them in your code:

```bash
source /opt/gruntwork/bash-commons/log.sh

log_info "Hello, World!"
```




## Available modules

Here's an overview of the modules available in `bash-commons`:

* `array.sh`: Helpers for working with Bash arrays, such as checking if an array contains an element, or joining an
  array into a string with a delimiter between elements.

* `assert.sh`: Assertions that check a condition and exit if the condition is not met, such as asserting a variable is
  not empty or that an expected app is installed. Useful for defensive programming.

* `aws.sh`: A collection of thin wrappers for direct calls to the [AWS CLI](https://aws.amazon.com/cli/) and [EC2
  Instance Metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html). These thin
  wrappers give you a shorthand way to fetch certain information (e.g., information about an EC2 Instance, such as its
  private IP, public IP, Instance ID, and region). Moreover, you can swap out `aws.sh` with a version that returns mock
  data to make it easy to run your code locally (e.g., in Docker) and to run unit tests.

* `aws-wrapper.sh`: A collection of "high level" wrappers for the [AWS CLI](https://aws.amazon.com/cli/) and [EC2
  Instance Metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html) to simplify common
  tasks such as looking up tags or IPs for EC2 Instances. Note that these wrappers handle all the data processing and
  logic, whereas all the direct calls to the AWS CLI and EC2 metadata endpoints are delegated to `aws.sh` to make unit
  testing easier.

* `file.sh`: A collection of helpers for working with files, such as checking if a file exists or contains certain text.

* `log.sh`: A collection of logging helpers that write logs to `stderr` with log levels (INFO, WARN, ERROR) and
  timestamps.

* `os.sh`: A collection of Operating System helpers, such as checking which flavor of Linux (e.g., Ubuntu, CentOS) is
  running and validating checksums.

* `string.sh`: A collection of string manipulation functions, such as checking if a string contains specific text,
  stripping prefixes, and stripping suffixes.




## Coding principles

The code in `bash-commons` follows the following principles:

1. [Code style](#code-style)
1. [Everything is a function](#everything-is-a-function)
1. [Namespacing](#namespacing)
1. [Testing](#testing)


### Code style

All the code should follow mainly follow the [Google Shell Style Guide](https://google.github.io/styleguide/shell.xml).
In particular:

* Use the Bash shell. We aim for compatibility with Bash 3.
* The first line of every script should be `#!/bin/bash`.
* All code should be defined in functions.
* Functions should exit or return 0 on success and non-zero on error.
* Functions should return output by writing it to `stdout`.
* Functions should log to `stderr`.
* All variables should be `local`. No global variables are allowed at all.
* Make as many variables `readonly` as possible.
* If calling to a subshell and storing the output in a variable (foo=`$( ... )`), do NOT use `local` and `readonly`
  in the same statement or the [exit code will be
  lost](https://blog.gruntwork.io/yak-shaving-series-1-all-i-need-is-a-little-bit-of-disk-space-6e5ef1644f67). Instead,
  declare the variable as `local` on one line and then call the subshell on the next line.
* Quote all strings.
* Use `[[ ... ]]` instead of `[ ... ]`.
* Use snake_case for function and variable names. Use UPPER_SNAKE_CASE for constants.


### Everything in a function

It's essential that ALL code is defined in a function. That allows you to use `source` to "import" that code without
anything actually being executed.


### Namespacing

Bash does not support namespacing, so we fake it using a convention on the function names: if you create a file
`<foo.sh>`, all functions in it should start with `foo_`. For example, all the functions in `log.sh` start with `log_`
(`log_info`, `log_error`) and all the functions in `string.sh` start with `string_` (`string_contains`,
`string_strip_prefix`). That makes it easier to tell which functions came from which modules.

For readability, that means you should typically give files a name that is a singular noun. For example, `log.sh`
instead of `logging.sh` and `string.sh` instead of `strings.sh`.


### Testing

Every function should be tested:

* Automated tests are in the [test](/test) folder.

* We use [Bats](https://github.com/sstephenson/bats) as our unit test framework for Bash code. Note: Bats has not been
  maintained the last couple years, so we may need to change to the [bats-core](https://github.com/bats-core/bats-core)
  fork at some point (see [#150](https://github.com/sstephenson/bats/issues/150)).

* [Install Bats](https://github.com/sstephenson/bats/wiki/Install-Bats-Using-a-Package).

* To run all the tests: `bats test`.

* To run one test file: `bats test/array.bats`.

* CircleCI will run all Bats tests automatically after every build. The build runs in the Docker image defined in
  [.circleci/Dockerfile](/.circleci/Dockerfile). To build a new version of the image:

    ```bash
    cd .circleci
    docker build -t gruntwork/bash-commons-circleci-tests .
    docker push gruntwork/bash-commons-circleci-tests
    ```




## TODO

1. Add automated tests for `aws.sh` and `aws-wrapper.sh`. We have not tested these as they require either running an
   EC2 Instance or run something like [LocalStack](https://github.com/localstack/localstack).

1. Add automated tests for `assert_uid_is_root_or_sudo` and `os_user_is_root_or_sudo`. These methods depend on the
   current user, which is different in CircleCI (which runs in a Docker image as sudo) and local dev. We'll probably
   need to run all tests in Docker eventually.