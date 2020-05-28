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
	echo "127.0.0.1 ${PROJECT_BASE_URL}" | sudo tee -a /etc/hosts

compose: build
	@echo -e "\n launching docker-compose drupal stack \n"
	docker-compose up -d

up: compose hosts

drupal-bootstrap:
	@echo -e "\n create drupal project \n"
	docker exec ${PROJECT_NAME}_php composer create-project drupal-composer/drupal-project:8.x-dev /var/www/html --no-interaction

multisite:
	declare -a DRUPAL_SITES=${DRUPAL_EXTRA_SITES}
	cp sites.php.tpl sites.php
	for DRUPAL_SITE in $${DRUPAL_SITES[@]}; do \
		docker exec -ti ${PROJECT_NAME}_mariadb /bin/bash -c "mysql -u root -p${DB_ROOT_PASSWORD} <<-EOSQL
		CREATE DATABASE IF NOT EXISTS $${DRUPAL_SITE};
		GRANT ALL ON $${DRUPAL_SITE}.* TO '${DB_USER}'@'%' ;
		EOSQL"
		docker exec -ti ${PROJECT_NAME}_php /bin/bash -c "cd /var/www/html/web/sites;                               \
	                                                  mkdir $${DRUPAL_SITE};                                        \
													  cp default/default.settings.php $${DRUPAL_SITE}/settings.php; \
													  chmod 644 $${DRUPAL_SITE}/settings.php"
		echo \$$sites[\'$${DRUPAL_SITE}.docker.localhost\'] = \'$${DRUPAL_SITE}\'\; >> sites.php
		echo "127.0.0.1 $${DRUPAL_SITE}.docker.localhost" | sudo tee -a /etc/hosts
	done
	docker cp sites.php ${PROJECT_NAME}_php:/var/www/html/web/sites/

down:
	@echo -e "\n delete docker-compose drupal stack \n"
	docker-compose down
	sudo sed -i "/${PROJECT_BASE_DOMAIN}/d" /etc/hosts