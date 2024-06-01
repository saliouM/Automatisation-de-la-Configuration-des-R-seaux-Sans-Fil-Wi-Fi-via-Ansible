# Automatisation-de-la-Configuration des Réseaux Sans Fil Wi-Fi via Ansible

L'automatisation de la configuration des réseaux sans fil (Wi-Fi) via Ansible peut grandement simplifier la gestion des points d'accès et des contrôleurs Wi-Fi, permettant des déploiements cohérents et reproductibles. Voici un guide détaillé pour automatiser cette tâche.

### Prérequis

1. **Ansible installé** : Assurez-vous qu'Ansible est installé sur votre machine de gestion.
2. **Accès SSH** : Points d'accès (AP) et contrôleurs Wi-Fi doivent être accessibles via SSH.
3. **Modules Ansible nécessaires** : En fonction du matériel utilisé (Cisco, Aruba, etc.), vous devrez installer les modules Ansible appropriés.
4. **Accès API** : Certains équipements Wi-Fi peuvent nécessiter l'accès à leur API pour certaines configurations.

### Étape 1 : Installer Ansible

Si Ansible n'est pas encore installé, vous pouvez l'installer en utilisant la commande suivante :

```bash
sudo apt update
sudo apt install ansible -y
```

### Étape 2 : Configurer l'Inventaire Ansible

Créez un fichier d'inventaire pour définir les hôtes et groupes des points d'accès et des contrôleurs.

```ini
# inventory/hosts.ini
[access_points]
ap1 ansible_host=192.168.1.2 ansible_user=admin ansible_password=admin_password
ap2 ansible_host=192.168.1.3 ansible_user=admin ansible_password=admin_password

[controllers]
controller1 ansible_host=192.168.1.1 ansible_user=admin ansible_password=admin_password
```

### Étape 3 : Créer des Playbooks Ansible

Les playbooks Ansible sont des fichiers YAML qui définissent les tâches à exécuter. Voici un exemple pour configurer un point d'accès et un contrôleur Wi-Fi.

#### Exemple pour Cisco AP et Contrôleur

1. **Installer les modules nécessaires**

```bash
ansible-galaxy collection install cisco.ios
```

2. **Créer un fichier de variables**

Définissez vos variables spécifiques pour Cisco :

```yaml
# vars/cisco_wifi_vars.yml
ssid_name: "CorporateWiFi"
ssid_password: "StrongPassword"
vlan_id: 10
```

3. **Créer le playbook principal**

```yaml
# playbooks/configure_wifi.yml
- name: Configurer le contrôleur Wi-Fi
  hosts: controllers
  gather_facts: false
  vars_files:
    - ../vars/cisco_wifi_vars.yml
  tasks:
    - name: Configurer le SSID sur le contrôleur
      cisco.ios.ios_config:
        lines:
          - wlan {{ ssid_name }} {{ vlan_id }}
          - security wpa psk set-key ascii {{ ssid_password }}
        provider:
          host: "{{ ansible_host }}"
          username: "{{ ansible_user }}"
          password: "{{ ansible_password }}"
          authorize: yes
          auth_pass: "{{ ansible_password }}"

- name: Configurer les points d'accès
  hosts: access_points
  gather_facts: false
  vars_files:
    - ../vars/cisco_wifi_vars.yml
  tasks:
    - name: Configurer le SSID sur le point d'accès
      cisco.ios.ios_config:
        lines:
          - dot11 ssid {{ ssid_name }}
          - vlan {{ vlan_id }}
          - authentication open
          - authentication key-management wpa
          - guest-mode
        provider:
          host: "{{ ansible_host }}"
          username: "{{ ansible_user }}"
          password: "{{ ansible_password }}"
          authorize: yes
          auth_pass: "{{ ansible_password }}"
```

4. **Exécuter le playbook**

```bash
ansible-playbook -i inventory/hosts.ini playbooks/configure_wifi.yml
```

### Étape 4 : Vérification de la Configuration

Après avoir exécuté le playbook, vérifiez que le SSID a été correctement créé sur le contrôleur et les points d'accès.

#### Exemple pour Aruba AP et Contrôleur

1. **Installer les modules nécessaires**

```bash
ansible-galaxy collection install arubanetworks.aos_switch
```

2. **Créer un fichier de variables**

Définissez vos variables spécifiques pour Aruba :

```yaml
# vars/aruba_wifi_vars.yml
ssid_name: "CorporateWiFi"
ssid_password: "StrongPassword"
vlan_id: 10
controller_ip: "192.168.1.1"
```

3. **Créer le playbook principal**

```yaml
# playbooks/configure_aruba_wifi.yml
- name: Configurer le contrôleur Wi-Fi Aruba
  hosts: controllers
  gather_facts: false
  vars_files:
    - ../vars/aruba_wifi_vars.yml
  tasks:
    - name: Configurer le SSID sur le contrôleur Aruba
      arubanetworks.aos_switch.aos_switch_config:
        config:
          - wlan ssid-profile "{{ ssid_name }}"
            ssid: "{{ ssid_name }}"
            opmode: wpa2-aes-psk
            wpa-passphrase: "{{ ssid_password }}"
          - vlan {{ vlan_id }}
            name: "WiFi_VLAN"
            tagged: "all"
        hostname: "{{ ansible_host }}"
        username: "{{ ansible_user }}"
        password: "{{ ansible_password }}"

- name: Configurer les points d'accès Aruba
  hosts: access_points
  gather_facts: false
  vars_files:
    - ../vars/aruba_wifi_vars.yml
  tasks:
    - name: Configurer le SSID sur le point d'accès Aruba
      arubanetworks.aos_switch.aos_switch_config:
        config:
          - wlan ssid-profile "{{ ssid_name }}"
            ssid: "{{ ssid_name }}"
            opmode: wpa2-aes-psk
            wpa-passphrase: "{{ ssid_password }}"
          - vlan {{ vlan_id }}
            name: "WiFi_VLAN"
            tagged: "all"
        hostname: "{{ ansible_host }}"
        username: "{{ ansible_user }}"
        password: "{{ ansible_password }}"
```

4. **Exécuter le playbook**

```bash
ansible-playbook -i inventory/hosts.ini playbooks/configure_aruba_wifi.yml
```

### Conclusion

En suivant ces étapes, vous pouvez automatiser la configuration des réseaux sans fil (Wi-Fi) via Ansible. Les playbooks fournis sont des exemples de base et peuvent être étendus pour inclure des configurations plus avancées selon les besoins spécifiques de votre infrastructure. L'utilisation d'Ansible permet de gérer efficacement les configurations et de garantir la cohérence à travers l'ensemble de votre réseau sans fil.

