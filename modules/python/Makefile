# Make targets for python
PWD ?= $(shell pwd)
MAKEFILES_DIR := $(PWD)/makefiles/modules

include $(MAKEFILES_DIR)/project/Makefile

# All cli tools should be here because they are referenced in many places
TEST_CMD := pytest
TYPE_CMD := mypy
LINT_CMD := black
POETRY_CMD := poetry
SORT_CMD := isort
CLEAN_IMPORTS_CMD := autoflake
DEAD_CODE_CMD := vulture
DOCS_CMD := pydocstyle
DEPS_SEC_CMD := safety
CODE_SEC_CMD := bandit
COMPLEXITY_CMD := flake8

# Flags
CLEAN_IMPORTS_FLAGS := --check -i -r $(PROJECT_NAME) --quiet
DEP_SEC_FLAGS := --full-report
CODE_SEC_FLAGS := -r

# equality check variables
DOC_CONVENTION := google

# Generic validators
validate-tomlq:
	@command -v tomlq > /dev/null 2>&1 || { echo >&2 "tomlq not found, please install tomlq via the yq package with 'poetry add yq --dev"; exit 1; }

# Project stuff
check/poetry:
	@command -v $(POETRY_CMD) > /dev/null 2>&1 || { echo >&2 echo "${POETRY_CMD} not found, please install"; exit 1; }

init%: check/poetry
	@$(POETRY_CMD) init

check/venv-config: validate-tomlq
	@[ -f poetry.toml ] || { echo >&2 echo "${POETRY_CMD} config not found, please execute 'touch ${POETRY_CMD}.toml'"; exit 1; }
	@tomlq -t .virtualenvs ${POETRY_CMD}.toml > /dev/null 2>&1 || { echo >&2 "Config for virtualenvs not found in ${POETRY_CMD}.toml, please add!"; exit 1; }
# confirms the venv will be created in project
ifeq ($(shell tomlq -t ".virtualenvs" poetry.toml | grep 'in-project' | cut -d '=' -f 2 | cut -d ' ' -f 2), false)
	@{ echo >&2 "virtualenvs must be configured as 'in-project = true' in ${POETRY_CMD}.toml, please update!"; exit 1; }
endif

venv%: check/poetry check/venv-config
	@$(POETRY_CMD) shell

# Typing
check/types:
	@command $(TYPE_CMD) --version > /dev/null 2>&1 || { echo >&2 "${TYPE_CMD} not found, please install with 'poetry add ${TYPE_CMD} --dev'"; exit 1; }

config/types: validate-tomlq
	@tomlq -t .tool.${TYPE_CMD} pyproject.toml > /dev/null 2>&1 || { echo >&2 "Config for ${TYPE_CMD} not found in pyproject.toml, please add!"; exit 1; }

types%: check/types config/types
	@echo "Checking types with ${TYPE_CMD}..."
	@$(TYPE_CMD) $(PROJECT_NAME)

# Lint
check/lint:
	@command $(LINT_CMD) --help > /dev/null 2>&1 || { echo >&2 "${LINT_CMD} not found, please install with 'poetry add ${LINT_CMD} --dev'"; exit 1; }

config/lint: validate-tomlq
	@tomlq -t .tool.${LINT_CMD} pyproject.toml > /dev/null 2>&1 || { echo >&2 "Config for ${LINT_CMD} not found in pyproject.toml, please add!"; exit 1; }

config/lint/line-length:
	@if [ $(shell tomlq -t .tool.${LINT_CMD} pyproject.toml | grep "line-length" | cut -d "=" -f 2 | cut -d ' ' -f 2) -gt 100 ]; then \
		{ echo >&2 "Line length config for ${LINT_CMD} exceeds 100, please adjust to 100 or lower!"; exit 1; }; \
	fi

lint-super: check/lint config/lint config/lint/line-length
	@echo "Running ${LINT_CMD} to lint the codebase..."
	@$(LINT_CMD) $(PROJECT_NAME)

# Import sorting and cleanup
check/sort:
	@command $(SORT_CMD) > /dev/null 2>&1 || { echo >&2 "${SORT_CMD} not found, please install with 'poetry add ${SORT_CMD} --dev'"; exit 1; }

config/sort: validate-tomlq
	@tomlq -t .tool.${SORT_CMD} pyproject.toml > /dev/null 2>&1 || { echo >&2 "Config for ${SORT_CMD} not found in pyproject.toml, please add!"; exit 1; }

config/sort/linter: validate-tomlq
ifneq ($(shell tomlq -t .tool.${SORT_CMD} pyproject.toml | grep "profile" | cut -d "=" -f 2 | cut -d ' ' -f 2), "${LINT_CMD}")
	@{ echo >&2 "${SORT_CMD} is not set to use the ${LINT_CMD} profile, please update profile to ${LINT_CMD}!"; exit 1; };
endif

config/sort/line-length: validate-tomlq
	@if [ $(shell tomlq -t .tool.${SORT_CMD} pyproject.toml | grep "line_length" | cut -d "=" -f 2 | cut -d ' ' -f 2) -gt 100 ]; then \
		{ echo >&2 "Line length config for ${SORT_CMD} exceeds 100, please adjust to 100 or lower or match ${LINT_CMD} line-length setting"; exit 1; }; \
	fi

sort: check/sort config/sort config/sort/linter config/sort/line-length
	@echo "Running ${SORT_CMD} to sort all imports..."
	@$(SORT_CMD) $(PROJECT_NAME)

check/clean-imports:
	@command $(CLEAN_IMPORTS_CMD) --help > /dev/null 2>&1 || { echo >&2 "${CLEAN_IMPORTS_CMD} not found, please install with 'poetry add $(CLEAN_IMPORTS_CMD) --dev'"; exit 1; }

config/clean-imports: validate-tomlq
	@tomlq -t .tool.${CLEAN_IMPORTS_CMD} pyproject.toml > /dev/null 2>&1 || { echo >&2 "Config for ${CLEAN_IMPORTS_CMD} not found in pyproject.toml, please add!"; exit 1; }

clean-imports: check/clean-imports config/clean-imports
	@echo "Running ${CLEAN_IMPORTS_CMD} to remove unused imports..."
	@$(CLEAN_IMPORTS_CMD) $(CLEAN_IMPORTS_FLAGS)

imports-super: sort clean-imports

# Dead code
check/dead-code:
	@command $(DEAD_CODE_CMD) > /dev/null 2>&1 || { echo >&2 "${DEAD_CODE_CMD} not found, please install with 'poetry add $(DEAD_CODE_CMD) --dev'"; exit 1; }

config/dead-code: validate-tomlq
	@tomlq -t .tool.${DEAD_CODE_CMD} pyproject.toml > /dev/null 2>&1 || { echo >&2 "Config for ${DEAD_CODE_CMD} not found in pyproject.toml, please add!"; exit 1; }

dead-code%: check/dead-code config/dead-code
	@echo "Running ${DEAD_CODE_CMD} to find unused code..."
	@$(DEAD_CODE_CMD)

# Docs
check/docs:
	@command $(DOCS_CMD) --help > /dev/null 2>&1 || { echo >&2 "${DOCS_CMD} not found, please install with 'poetry add $(DOCS_CMD) --dev'"; exit 1; }

config/docs: validate-tomlq
	@tomlq -t .tool.${DOCS_CMD} pyproject.toml > /dev/null 2>&1 || { echo >&2 "Config for ${DOCS_CMD} not found in pyproject.toml, please add!"; exit 1; }

config/docs/convention: validate-tomlq
ifneq ($(shell tomlq -t .tool.${DOCS_CMD} pyproject.toml | grep "convention" | cut -d "=" -f 2 | cut -d ' ' -f 2), "${DOC_CONVENTION}")
	@{ echo >&2 "${DOCS_CMD} is not set to use the ${DOC_CONVENTION} convention, please update!"; exit 1; };
endif

docs%: check/docs config/docs config/docs/convention
	@$(DOCS_CMD)

# Security
check/deps-security:
	@command $(DEPS_SEC_CMD) > /dev/null 2>&1 || { echo >&2 "${DEPS_SEC_CMD} not found, please install with 'poetry add $(DEPS_SEC_CMD) --dev'"; exit 1; }

deps-security: check/deps-security
	@echo "Checking dependencies for open CVEs..."
	@$(DEPS_SEC_CMD) check $(DEP_SEC_FLAGS)

check/code-security:
	@command $(CODE_SEC_CMD) --version > /dev/null 2>&1 || { echo >&2 "${CODE_SEC_CMD} not found, please install with 'poetry add $(CODE_SEC_CMD) --dev'"; exit 1; }

config/code-security: validate-tomlq
	@tomlq -t .tool.${CODE_SEC_CMD} pyproject.toml > /dev/null 2>&1 || { echo >&2 "Config for ${CODE_SEC_CMD} not found in pyproject.toml, please add!"; exit 1; }

code-security: check/code-security config/code-security
	@echo "Checking code for known security issues..."
	@$(CODE_SEC_CMD) $(CODE_SEC_FLAGS) $(PROJECT_NAME)

security-super: code-security deps-security

# Complexity
check/complexity:
	@command $(COMPLEXITY_CMD) > /dev/null 2>&1 || { echo >&2 "${COMPLEXITY_CMD} not found, please install with 'poetry add $(COMPLEXITY_CMD) --dev'"; exit 1; }

config/complexity:
	@cat setup.cfg > /dev/null 2>&1 || { echo >&2 "${COMPLEXITY_CMD} config not found, please execute 'touch setup.cfg'!"; exit 1; }
	@cat setup.cfg | grep "max-complexity" > /dev/null 2>&1 || { echo >&2 "Complexity config not found in setup.cfg, please add!"; exit 1; }

complexity-super: check/complexity config/complexity
	@echo "Checking code complexity..."
	@$(COMPLEXITY_CMD)

# Tests
check/tests:
	@command $(TEST_CMD) --help > /dev/null 2>&1 || { echo >&2 "${TEST_CMD} not found, please install with 'poetry add ${TEST_CMD} --dev'"; exit 1; }

config/tests: validate-tomlq
	@tomlq -t .tool.${TEST_CMD} pyproject.toml > /dev/null 2>&1 || { echo >&2 "Config for ${TEST_CMD} not found in pyproject.toml, please add!"; exit 1; }

tests/unit%: check/tests config/tests
	@echo "Running unit tests..."
	@$(TEST_CMD) $(UNIT_TEST_DIR)

tests/integration%: check/tests config/tests
	@echo "Running integration tests..."
	@$(TEST_CMD) $(INTEGRATION_TEST_DIR)

tests/e2e%: check/tests config/tests
	@echo "Running e2e tests..."
	@$(TEST_CMD) $(E2E_TEST_DIR)
