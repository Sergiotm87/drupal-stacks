SHELL:=/bin/bash
.ONESHELL:
.SHELLFLAGS := -eu -o pipefail -c
.DELETE_ON_ERROR:
MAKEFLAGS += --warn-undefined-variables
MAKEFLAGS += --no-builtin-rules

include .env
$(eval export $(grep '=' .env | cut -d= -f1))

build:
	docker-compose build

hosts:
	@echo -e "\n add 'PROJECT_BASE_URL' to /etc/hosts \n"
	nginx_ip=$$(docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}' $${PROJECT_NAME}_nginx)
	portainer_ip=$$(docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}' $${PROJECT_NAME}_portainer)
	echo "$$nginx_ip ${PROJECT_BASE_URL}" | sudo tee -a /etc/hosts
	echo "$$portainer_ip portainer.${PROJECT_BASE_URL}" | sudo tee -a /etc/hosts

compose: build
	@echo -e "\n launching docker-compose drupal stack \n"
	docker-compose up -d

up: compose hosts

drupal-bootstrap:
	@echo -e "\n create drupal project \n"
	docker exec ${PROJECT_NAME}_php composer create-project drupal-composer/drupal-project:8.x-dev /var/www/html --no-interaction

down:
	@echo -e "\n delete docker-compose drupal stack \n"
	docker-compose down
	sudo sed -i "/${PROJECT_BASE_DOMAIN}/d" /etc/hosts
