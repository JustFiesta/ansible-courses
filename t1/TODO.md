# TODO list

1. Utwórz nowy folder w repo labów, np. lab-docker-infra/.

2. Zainicjuj strukturę katalogów Ansible (inventories/, roles/, playbooks/, group_vars/, docker/, docs/).

3. Skonfiguruj ansible.cfg z lokalnym inventory (inventory = inventories/docker/hosts.ini) i ścieżką do ról.

4. Utwórz plik docker/docker-compose.yml, który uruchamia trzy kontenery Ubuntu (web1, web2, lb).

5. Zbuduj prosty Dockerfile.base dla kontenerów z Pythonem i SSH lub ansible_connection=docker.

6. Przetestuj start kontenerów (docker compose up -d) i sprawdź, czy są widoczne (docker ps).

7. Utwórz inventory inventories/docker/hosts.ini z grupami webservers i loadbalancers.

8. Przetestuj połączenie Ansible do kontenerów (ansible all -m ping).

9. Utwórz rolę common instalującą podstawowe pakiety i konfigurującą system (np. firewall, curl).

10. Utwórz rolę webserver instalującą Nginx, klonującą aplikację simple-go-invoice i budującą ją.

11. Utwórz rolę loadbalancer instalującą HAProxy i konfigurującą backendy dynamicznie z groups['webservers'].

12. Utwórz plik group_vars/all.yml z globalnymi zmiennymi (np. app_env, use_ssl).

13. Utwórz group_vars/webservers.yml i group_vars/loadbalancers.yml z portami, ścieżkami i konfiguracją usług.

14. Przygotuj pliki template Jinja2 (nginx.conf.j2, haproxy.cfg.j2, index.html.j2).

15. Dodaj użycie Ansible Vault dla sekretów (np. hasła, klucze).

16. Utwórz główny playbook playbooks/site.yml uruchamiający role w kolejności (common, webserver, loadbalancer).

17. Uruchom provisioning (ansible-playbook playbooks/site.yml) i zweryfikuj działanie (curl do loadbalancera).

18. Przeprowadź testy walidacyjne (sprawdź serwisy, porty, load balancing, zawartość HTML).

19. Utwórz dokumentację w docs/ (setup, architektura, testy, troubleshoot).

20. Dodaj sekcję w głównym README repo z krótkim opisem tego labu i linkiem do jego katalogu.

21. Oczyść kontenery (docker compose down) i upewnij się, że lab jest w pełni powtarzalny (up + ansible-playbook).
