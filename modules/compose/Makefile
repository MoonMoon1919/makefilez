# Docker compose stuff

COMPOSE_CMD := docker-compose

check/compose:
	@command -v $(COMPOSE_CMD) > /dev/null 2>&1 || { echo >&2 echo "${COMPOSE_CMD} not found, please install"; exit 1 }

compose/up: check/compose
	@$(COMPOSE_CMD) up -d

compose/down: check/compose
	@$(COMPOSE_CMD) down
