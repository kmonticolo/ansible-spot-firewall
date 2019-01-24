# ansible-spot-firewall

Ansible managed firewall for SPOT (SPT-4740)

Requirements:

- Ansible and pywinrm (https://docs.ansible.com/ansible/latest/user_guide/windows_winrm.html) and ntlm-auth installed on Ansible host
- Powershell >=3.0 installed on SPOT site
- SPOT site configured using https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1
- SPOT site must be available for an Ansible host
- Firewall service must be configured as: "Status" is running and "Startup Type" is Automatic

Check for hosts availability:
ansible -m win_ping -i inventory_qasite3 all

Usage:
ansible-playbook firewall_spot.yml -i inventory_qasite3 -l mdb

###
Jeden wpis odpowiada jednej końcowej regule firewall.

następnie excela wyeksportowałem do formatu CSV z następującymi polami:

Name,rule_name,runs on,local port (number),action (allow block bypass),"direction (in,out)",protocol (tcp udp any)

 

przykładowa reguła:

scsetup,scsetup_mdb_139_in_tcp,mdb,139,allow,in,tcp

 

Opis formatu: 

Name - nazwa usługi w excelu pozwalająca przejść na powrót ścieżkę od reguły firewall do xls, gdyby zaszła potrzeba 

rule_name - konkatenacja pięciu pól w excelu stanowiąca samokomentującą się nazwę reguły w firewallu windows (bez zaglądania w regułę jest się w stanie stwierdzić co dana reguła robi)

runs on - nazwa hosta, na którym reguła ma być zaaplikowana 

local port (number) - numer portu

action - akcja do wykonania, możliwy wybór to allow, block, bypass - można pojedynczo blokować lub przepuszczać reguły - przydatne przy debugu i refiningu firewalla

direction - kierunek (przychodzący, wychodzący)

protocol - protokół, którego dotyczy reguła (tcp udp any)

 

za pomocą awk eksportuję wpisy z pliku csv do pliku stanowiącego konfigurację firewalla dla całego site, na przykład:

awk -F, '{print "- { name:",$2,", host:",$3,", localport:",$4,", action:",$5,", direction:",$6,", protocol:",$7,", state: present, enabled: yes }"}' <inserter.csv >insert

 

plik wynikowy firewall_port_rules.yml dla site ma przykładową konfigurację (na chwilę obecną 266 regułek):

---

firewall_port_rules:

  - { name: in3389rdp , host: Inserter, localport: 3389 , action: allow , direction: in , protocol: tcp , state: present, enabled: yes }

...

  - { name: gpi_Inserter_138_in_UDP , host: Inserter , localport: 138 , action: allow , direction: in , protocol: UDP , state: present, enabled: yes }

  - { name: tonescraper_Inserter_139_in_TCP , host: Inserter , localport: 139 , action: allow , direction: in , protocol: TCP , state: present, enabled: yes }

 

wartości w powyzszym pliku można swobodnie modyfikować wedle potrzeby, do debugu przydatne są zwłaszcza ostatnie dwa pola state i enabled.

 

wszystkie pliki potrzebne do zaaplikowania reguł firewalla są dostępne w repozytorium: https://gitlab.dcclabs.tv/AdvertisingTools/ansible-spot-firewall

 

plik inwentarza (inventory*) jest plikiem zawierającym spis wszystkich hostów, których mają dotyczyć reguły firewall. Jego podanie jest wymagane podczas uruchamiania playbooka ansbile. Dla wygody zawarłem w nim także wszystkie parametry uruchomieniowe oraz hasła wymagane do połączenia:

[mdb]QA750MDB3 ansible_host=128.168.53.10

[sdb]QA750SDB3 ansible_host=128.168.53.11

[mvl]QA750MVL3 ansible_host=128.168.53.12

[tti]QA750TONE3 ansible_host=128.168.53.14

[inserter]QA750TSI03 ansible_host=128.168.53.98 


plik playbooka firewall_spot.yml zawiera jedną stanzę per host i może być modyfikowany w zależności od potrzeb (TODO: dobrze byłoby przepisać go na pętlę)


Wymagania na hostach Windows:

1. Ansible and pywinrm (https://docs.ansible.com/ansible/latest/user_guide/windows_winrm.html) and ntlm-auth powinny być zainstalowane na hoście, z którego uruchamiamy playbook

2. Powershell w wersji min. 3.0 musi być zainstalowany na hostach SPOT site

3. należy uruchomić skrypt powershellowy https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1 na wszystkich hostach windows,

4. hosty ze SPOT site muszą być dostępne dla hosta z Ansible

5. usługa Firewall musi być uruchomiona automatycznie przy starcie oraz uruchomiona: "Status" is running and "Startup Type" is Automatic

 

uruchomienie:

dobrze jest sprawdzić dostępność hostów przed wykonaniem playbooka: ansible -m win_ping -i inventory_qasite3 all
 

wykonywanie: ansible-playbook firewall_spot.yml -i inventory_qasite3 -l mdb

 

parametr -l ogranicza wykonanie playbooka do zadanych hostów.
