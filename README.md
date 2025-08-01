# Ansible PBS

Un playbook Ansible pour installer et configurer Proxmox Backup Server sur Debian/Ubuntu, avec envoi d’un récapitulatif sur Discord via webhook.



## Arborescence du projet

```
|-- ansible.cfg
|-- hosts.ini
|-- pbs-playbook.yml
|-- roles
|   `-- pbs
|       |-- defaults
|       |   `-- main.yml
|       |-- handlers
|       |   `-- main.yml
|       |-- tasks
|       |   `-- main.yml
|       `-- templates
|           `-- pbs-sources.list.j2
`-- vault.yml
```

---

## Prérequis

- Poste de contrôle avec Ansible 2.9+ installé  
- Machine cible Debian/Ubuntu accessible en SSH (partage de clés)  
- Fichier `vault.yml` contenant vos secrets (webhook Discord, mots de passe, etc.)  

---

## Clonez le repo sur votre machine srv ansible :  
   ```bash
   git clone https://github.com/anis-metref/ansible-pbs.git
   cd ansible-pbs
   ```

## Configuration de l’accès SSH par clés

- Sur la machine de contrôle :
  1. Générer une paire de clés SSH   
     ```bash
     ssh-keygen -t ed25519 -C "ansible@control"
     ```
  2. Copier la clé publique vers la cible   
     ```bash
     ssh-copy-id -i /home/{user}/.ssh/id_rsa.pub user@IP-CLient
     ```

- Sur la machine cible :
  1. Vérifier la connexion sans mot de passe   
     ```bash
     ssh user@IP-CLient
     ```
  2. (Optionnel) Renforcer la sécurité en désactivant l’authentification par mot de passe   
     - Éditer `/etc/ssh/sshd_config` et définir :
       ```ini
       PasswordAuthentication no
       ```
     - Redémarrer le service SSH   
       ```bash
       sudo systemctl restart ssh
       ```

## Installation des dépendances

```bash

# Installer Ansible et les outils de vérification
sudo apt install ansible ansible-lint
```

---

## Configuration

1. Éditez `hosts.ini` pour y ajouter vos machines cibles :

```ini
[pbs]
ansible-client ansible_host=IP-Client ansible_ssh_private_key_file=/home/{user}/.ssh/id_rsa    # indiquez la clé ssh privée

[pbs:vars]
ansible_python_interpreter=/usr/bin/python3
```

2. Placez vos variables publiques dans le fichier `vars.yml` 

3. Créez et chiffrez (ou modifiez) `vault.yml` :

   ```bash
   ansible-vault edit vault.yml
   ```

   ```yaml
   discord_webhook_url: "https://discord.com/api/webhooks/ID/TOKEN"    # vous mettez votre url webhook discord
   notify_username: "Ansibleserver"
   success_color: 3066993
   failure_color: 15158332
   `
3. Chiffrez le fichier `vault.yml` avec un mot de passe  :

   ```bash
   ansible-vault encrypt vault.yml
   `````

---

## Vérifications rapides

```bash
# Vérifier la syntaxe du playbook
ansible-playbook pbs-playbook.yml --syntax-check

# Afficher l’inventaire Ansible
ansible-inventory -i hosts.ini --list

# Linter vos rôles/tâches
ansible-lint roles/pbs/tasks/main.yml

# Voir le contenu chiffré de vault.yml
ansible-vault view vault.yml
```

---

## Exécution

```bash
ansible-playbook -i hosts.ini pbs-playbook.yml --ask-vault-pass -K
```

- `--ask-vault-pass` : saisie du mot de passe Vault  
- `-K` ou `--ask-become-pass` : saisie du mot de passe sudo si nécessaire  

---

## Mode “dry-run”

Pour simuler sans appliquer les changements :

```bash
ansible-playbook -i hosts.ini pbs-playbook.yml --check --diff --ask-vault-pass
```
---
