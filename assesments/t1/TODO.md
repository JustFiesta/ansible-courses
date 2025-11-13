# TODO list

- Zainicjuj strukturę katalogów Ansible (inventories/, roles/, playbooks/, group_vars/, docker/, docs/).

- Utwórz rolę common instalującą podstawowe pakiety i konfigurującą system (np. firewall, curl).

- Utwórz rolę webserver instalującą Nginx, klonującą aplikację simple-go-invoice i budującą ją.

- Utwórz rolę loadbalancer instalującą HAProxy i konfigurującą backendy dynamicznie z groups['webservers'].

- Utwórz plik group_vars/all.yml z globalnymi zmiennymi (np. app_env, use_ssl).

- Utwórz group_vars/webservers.yml i group_vars/loadbalancers.yml z portami, ścieżkami i konfiguracją usług.

- Przygotuj pliki template Jinja2 (nginx.conf.j2, haproxy.cfg.j2, index.html.j2).

- Dodaj użycie Ansible Vault dla sekretów (np. hasła, klucze).

- Utwórz główny playbook playbooks/site.yml uruchamiający role w kolejności (common, webserver, loadbalancer).

- Uruchom provisioning (ansible-playbook playbooks/site.yml) i zweryfikuj działanie (curl do loadbalancera).

- Przeprowadź testy walidacyjne (sprawdź serwisy, porty, load balancing, zawartość HTML).

- Utwórz dokumentację w docs/ (setup, architektura, testy, troubleshoot).
