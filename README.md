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


--------------------------------------------------------------------

Voici les étapes détaillées pour installer Ansible sur Ubuntu dans VirtualBox et configurer l'interaction avec des équipements Cisco (points d'accès et contrôleur Wi-Fi) :

### Pré-requis

1. **VirtualBox installé** : Assurez-vous d'avoir VirtualBox installé sur votre machine.
2. **ISO Ubuntu** : Téléchargez la dernière version d'Ubuntu.
3. **Accès à Internet** : Pour télécharger et installer les paquets nécessaires.

### Étape 1 : Création de la machine virtuelle

1. **Ouvrir VirtualBox et créer une nouvelle VM :**
    - Cliquez sur "New".
    - Donnez un nom à la VM (par exemple, "Ubuntu-Ansible").
    - Sélectionnez "Linux" comme type et "Ubuntu (64-bit)" comme version.
    - Allouez au moins 2 Go de RAM.
    - Créez un disque dur virtuel (20 Go est suffisant).

2. **Configurer la machine virtuelle :**
    - Allez dans "Settings" > "Storage".
    - Ajoutez le fichier ISO d'Ubuntu comme lecteur optique.

### Étape 2 : Installation d'Ubuntu

1. **Démarrer la VM :**
    - Cliquez sur "Start" pour démarrer la VM avec le fichier ISO monté.
    - Suivez les instructions pour installer Ubuntu.
    - Configurez le réseau pour utiliser le NAT ou le pontage selon vos besoins.

### Étape 3 : Installation d'Ansible

1. **Mettre à jour le système :**
    ```bash
    sudo apt update
    sudo apt upgrade -y
    ```

2. **Installer Ansible :**
    ```bash
    sudo apt install ansible -y
    ```

### Étape 4 : Configuration de l'inventaire Ansible

1. **Créer un fichier d'inventaire :**
    ```bash
    sudo nano /etc/ansible/hosts
    ```

2. **Ajouter les équipements Cisco à l'inventaire :**
    ```ini
    [cisco]
    ap1 ansible_host=192.168.1.2 ansible_user=admin ansible_password=admin_password
    ap2 ansible_host=192.168.1.3 ansible_user=admin ansible_password=admin_password
    controller1 ansible_host=192.168.1.1 ansible_user=admin ansible_password=admin_password
    ```

### Étape 5 : Installer les collections Ansible nécessaires

1. **Installer la collection Cisco IOS :**
    ```bash
    ansible-galaxy collection install cisco.ios
    ```

### Étape 6 : Création d'un playbook Ansible

1. **Créer un fichier de playbook :**
    ```bash
    mkdir -p ~/ansible/playbooks
    nano ~/ansible/playbooks/configure_cisco_wifi.yml
    ```

2. **Ajouter les tâches de configuration :**
    ```yaml
    - name: Configurer le contrôleur Wi-Fi
      hosts: controller1
      gather_facts: false
      tasks:
        - name: Configurer le SSID sur le contrôleur
          cisco.ios.ios_config:
            lines:
              - wlan CorporateWiFi 10
              - security wpa psk set-key ascii StrongPassword
            provider:
              host: "{{ ansible_host }}"
              username: "{{ ansible_user }}"
              password: "{{ ansible_password }}"
              authorize: yes
              auth_pass: "{{ ansible_password }}"

    - name: Configurer les points d'accès
      hosts: ap1,ap2
      gather_facts: false
      tasks:
        - name: Configurer le SSID sur les points d'accès
          cisco.ios.ios_config:
            lines:
              - dot11 ssid CorporateWiFi
              - vlan 10
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

### Étape 7 : Exécution du playbook

1. **Exécuter le playbook :**
    ```bash
    ansible-playbook -i /etc/ansible/hosts ~/ansible/playbooks/configure_cisco_wifi.yml
    ```

### Vérification

1. **Vérifier la configuration sur les équipements Cisco :**
    - Connectez-vous aux équipements et utilisez les commandes Cisco appropriées pour vérifier les configurations appliquées.

### Notes supplémentaires

- **SSH Configuration** : Assurez-vous que les équipements Cisco sont configurés pour permettre les connexions SSH.
- **Modules spécifiques** : Selon le modèle et la version du logiciel de vos équipements Cisco, vous devrez peut-être ajuster les modules et les commandes dans les playbooks Ansible.
- **Documentation Ansible** : Consultez la documentation officielle d'Ansible pour les modules Cisco pour des configurations spécifiques et des exemples avancés.

Ce guide couvre les étapes essentielles pour installer et configurer Ansible sur Ubuntu dans VirtualBox et pour interagir avec des équipements Cisco. Assurez-vous d'adapter les configurations et les commandes selon les spécificités de votre environnement réseau.

