# database name (the database inside postgres)
DB_NAME ?= postgres
# database username
DB_USER ?= postgres
# database password
DB_PWD ?= mysecretpassword
# sqlc yaml file
SQLC_YAML ?= ./sqlc.yaml

# fixed container name
CONTAINER_NAME := test-postgres

.PHONY: postgresup postgresdown psql createdb teardown_recreate generate

postgresup:
	docker rm -f $(CONTAINER_NAME) || true
	docker run --name $(CONTAINER_NAME) \
		-v "$(PWD)":/usr/share/chapter1 \
		-e POSTGRES_PASSWORD="$(DB_PWD)" \
		-p 5432:5432 \
		-d postgres

postgresdown:
	docker stop $(CONTAINER_NAME) || true
	docker rm $(CONTAINER_NAME) || true

psql:
	docker exec -it $(CONTAINER_NAME) psql -U $(DB_USER) -d $(DB_NAME)

createdb:
	docker exec -it $(CONTAINER_NAME) psql -U $(DB_USER) -d $(DB_NAME) -f /usr/share/chapter1/db/schema.sql

teardown_recreate: postgresdown postgresup
	@echo "Waiting postgres to be ready..."
	until docker exec $(CONTAINER_NAME) pg_isready -U $(DB_USER); do sleep 1; done
	$(MAKE) createdb

generate:
	@echo "Generating Go models with sqlc"
	sqlc generate -f $(SQLC_YAML)
