# makefilez

A collection of helpful makefiles for projects.

So far I've built what I found useful for some personal and professional products.

## Why?

Every time I created a new project I found myself recreating Makefile with largely the same targets.

## Why make?

- Widely available
- Easy to add new targets/features
- Didn't require building a wrapper CLI

## Available makefiles

- [docker](./modules/docker/Makefile)
- [docker compose](./modules/compose/Makefile)
- [git](./modules/git/Makefile)
- [project](./modules/project/Makefile)
- [python](./modules/python/Makefile)

## Usage

First, add this repo as a submodule to your repo.

```sh
git submodule add git@github.com:MoonMoon1919/makefilez.git
```

Then, create a Makefile.

```sh
touch Makefile
```

Then you can use an `include` statement in your Makefile pointing to the Makefile(s) you want to import into your Makefile. An example Makefile for a Python project can be found below.

```Makefile
MAKEFILES_DIR := $(PWD)/makefilez/modules

include $(MAKEFILES_DIR)/python/Makefile

venv:
	@make venv-super

types:
	@make types-super

lint:
	@make lint-super

organize-imports:
	@make imports-super

dead-code:
	@make dead-code-super

docs:
	@make docs-super

security:
	@make security-super

complexity:
	@make complexity-super

test/unit:
	@make tests/unit-super

test/integration:
	@make tests/integration-super

test/e2e:
	@make tests/e2e-super
```

Note that the Python makefile makes use of the wildcard `%`, which allows Make to match the initial target string plus any additional characters. For example, the unit test target is implemented as `test/unit%`, which can then be used in your makefile as `test/unit-super` or `test/unit-foo` or anything you dream up. `super`, while not required, is a convenient word to use to signal to your future self or teammates that the target you're calling came from another file.

## Contributing

If you would like to add more features and functionality, please create an issue requesting new functionality first to ensure that similar work is not already being done. After that, please feel free to submit PRs!
