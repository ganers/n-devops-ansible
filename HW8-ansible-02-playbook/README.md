# Домашнее задание к занятию "08.02 Работа с Playbook"

## Подготовка к выполнению
1. Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.
2. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
3. Подготовьте хосты в соотвтествии с группами из предподготовленного playbook. 
4. Скачайте дистрибутив [java](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) и положите его в директорию `playbook/files/`. 

## Основная часть
1. Приготовьте свой собственный inventory файл `prod.yml`.
2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает kibana.
3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, сгенерировать конфигурацию с параметрами.
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
10. Готовый playbook выложите в свой репозиторий, в ответ предоставьте ссылку на него.

---

#### Ответ:
Playbook подключается к двум docker контейнерам с ubuntu и устанвливает на них Java JDK из файла заранее скаченного в дирректорию /files. Для контейнера elastic скачивается и устанавливается дистрибутив Elasticsearch. Для контейнера Kibana скачивается и устанвливается дистрибутив Kibana.
В Group Vars находится информация о версиях устанавливаемого ПО. Для настройки установленного ПО используются .j2 шаблоны. Используя теги java elastic и kibana можно запустить задачи для установки соответствуего ПО.

```
#ansible-lint не выдает ошибок и предупреждений
ganers@netologydevops:~/netologydevops/HW/n-devops-ansible/HW8-ansible-02-playbook$ ansible-lint site.yml

#запуск Playbook с флагом --check и два запуска с флагом --diff
ganers@netologydevops:~/netologydevops/HW/n-devops-ansible/HW8-ansible-02-playbook$ sudo ansible-playbook --check site.yml -i inventory/prod.yml
[WARNING]: Found both group and host with same name: kibana

PLAY [Install Java] ******************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************
ok: [kibana]
ok: [elastic]

TASK [Set facts for Java 11 vars] ****************************************************************************************************************************************************************************
ok: [elastic]
ok: [kibana]

TASK [Upload .tar.gz file containing binaries from local storage] ********************************************************************************************************************************************
changed: [elastic]
changed: [kibana]

TASK [Ensure installation dir exists] ************************************************************************************************************************************************************************
changed: [kibana]
changed: [elastic]

TASK [Extract java in the installation directory] ************************************************************************************************************************************************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: NoneType: None
fatal: [kibana]: FAILED! => {"changed": false, "msg": "dest '/opt/jdk/11.0.14' must be an existing dir"}
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: NoneType: None
fatal: [elastic]: FAILED! => {"changed": false, "msg": "dest '/opt/jdk/11.0.14' must be an existing dir"}

PLAY RECAP ***************************************************************************************************************************************************************************************************
elastic                    : ok=4    changed=2    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
kibana                     : ok=4    changed=2    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0

ganers@netologydevops:~/netologydevops/HW/n-devops-ansible/HW8-ansible-02-playbook$ sudo ansible-playbook --diff site.yml -i inventory/prod.yml
[WARNING]: Found both group and host with same name: kibana

PLAY [Install Java] ******************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************
ok: [kibana]
ok: [elastic]

TASK [Set facts for Java 11 vars] ****************************************************************************************************************************************************************************
ok: [kibana]
ok: [elastic]

TASK [Upload .tar.gz file containing binaries from local storage] ********************************************************************************************************************************************
diff skipped: source file size is greater than 104448
changed: [kibana]
diff skipped: source file size is greater than 104448
changed: [elastic]

TASK [Ensure installation dir exists] ************************************************************************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/jdk/11.0.14",
-    "state": "absent"
+    "state": "directory"
 }

changed: [kibana]
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/jdk/11.0.14",
-    "state": "absent"
+    "state": "directory"
 }

changed: [elastic]

TASK [Extract java in the installation directory] ************************************************************************************************************************************************************
changed: [elastic]
changed: [kibana]

TASK [Export environment variables] **************************************************************************************************************************************************************************
--- before
+++ after: /root/.ansible/tmp/ansible-local-1099053f4jjms5/tmpq5_iasi6/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/11.0.14
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [elastic]
--- before
+++ after: /root/.ansible/tmp/ansible-local-1099053f4jjms5/tmpgip1l3ff/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/11.0.14
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [kibana]

PLAY [Install Elasticsearch] *********************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************
ok: [elastic]

TASK [Upload tar.gz Elasticsearch from remote URL] ***********************************************************************************************************************************************************
changed: [elastic]

TASK [Create directrory for Elasticsearch] *******************************************************************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/elastic/7.10.1",
-    "state": "absent"
+    "state": "directory"
 }

changed: [elastic]

TASK [Extract Elasticsearch in the installation directory] ***************************************************************************************************************************************************
changed: [elastic]

TASK [Set environment Elastic] *******************************************************************************************************************************************************************************
--- before
+++ after: /root/.ansible/tmp/ansible-local-1099053f4jjms5/tmpddo_k09x/elk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export ES_HOME=/opt/elastic/7.10.1
+export PATH=$PATH:$ES_HOME/bin
\ No newline at end of file

changed: [elastic]

PLAY [Install Kibana] ****************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************
ok: [kibana]

TASK [Upload tar.gz Kibana from remote URL] ******************************************************************************************************************************************************************
changed: [kibana]

TASK [Create directrory for Kibana] **************************************************************************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/kibana/7.10.1",
-    "state": "absent"
+    "state": "directory"
 }

changed: [kibana]

TASK [Extract Kibana in the installation directory] **********************************************************************************************************************************************************
changed: [kibana]

TASK [Set environment Kibana] ********************************************************************************************************************************************************************************
--- before
+++ after: /root/.ansible/tmp/ansible-local-1099053f4jjms5/tmp16grs0mo/kibana.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export KBN_PATH_CONF=/opt/kibana/7.10.1
+export PATH=$PATH:$KBN_PATH_CONF/bin

changed: [kibana]

PLAY RECAP ***************************************************************************************************************************************************************************************************
elastic                    : ok=11   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
kibana                     : ok=11   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

ganers@netologydevops:~/netologydevops/HW/n-devops-ansible/HW8-ansible-02-playbook$ sudo ansible-playbook --diff site.yml -i inventory/prod.yml
[WARNING]: Found both group and host with same name: kibana

PLAY [Install Java] ******************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************
ok: [kibana]
ok: [elastic]

TASK [Set facts for Java 11 vars] ****************************************************************************************************************************************************************************
ok: [elastic]
ok: [kibana]

TASK [Upload .tar.gz file containing binaries from local storage] ********************************************************************************************************************************************
ok: [elastic]
ok: [kibana]

TASK [Ensure installation dir exists] ************************************************************************************************************************************************************************
ok: [kibana]
ok: [elastic]

TASK [Extract java in the installation directory] ************************************************************************************************************************************************************
skipping: [kibana]
skipping: [elastic]

TASK [Export environment variables] **************************************************************************************************************************************************************************
ok: [kibana]
ok: [elastic]

PLAY [Install Elasticsearch] *********************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************
ok: [elastic]

TASK [Upload tar.gz Elasticsearch from remote URL] ***********************************************************************************************************************************************************
ok: [elastic]

TASK [Create directrory for Elasticsearch] *******************************************************************************************************************************************************************
ok: [elastic]

TASK [Extract Elasticsearch in the installation directory] ***************************************************************************************************************************************************
skipping: [elastic]

TASK [Set environment Elastic] *******************************************************************************************************************************************************************************
ok: [elastic]

PLAY [Install Kibana] ****************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************
ok: [kibana]

TASK [Upload tar.gz Kibana from remote URL] ******************************************************************************************************************************************************************
ok: [kibana]

TASK [Create directrory for Kibana] **************************************************************************************************************************************************************************
ok: [kibana]

TASK [Extract Kibana in the installation directory] **********************************************************************************************************************************************************
skipping: [kibana]

TASK [Set environment Kibana] ********************************************************************************************************************************************************************************
ok: [kibana]

PLAY RECAP ***************************************************************************************************************************************************************************************************
elastic                    : ok=9    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
kibana                     : ok=9    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

```