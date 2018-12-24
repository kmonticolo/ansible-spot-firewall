# ansible-spot-firewall

Ansible managed firewall for SPOT (SPT-4740)

Requirements:

- Ansible and pywinrm (https://docs.ansible.com/ansible/latest/user_guide/windows_winrm.html) and ntlm-auth installed on Ansible host
- Powershell >=3.0 installed on SPOT site
- SPOT site configured using https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1
- SPOT site must be available for an Ansible host
- Firewall service must be configured as automatically enabled 

Check for hosts availability:
ansible -m win_ping -i inventory_qasite3 all

Usage:
ansible-playbook firewall_spot.yml -i inventory_qasite3 -l mdb
