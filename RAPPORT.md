# Rapport TP1 : Ansible + Semaphore

## Objectif
Déployer un portfolio Next.js avec un playbook Ansible structuré en rôles, puis industrialiser l'exécution avec Semaphore UI.

## Architecture
- vm-admin : poste de contrôle Ansible et Semaphore UI (Docker)
- vm-cible : Ubuntu 24.04 avec Node.js, PM2 (application sur le port 3000) et Nginx (reverse proxy sur le port 80)

## Partie A : Ansible
Quatre rôles : common, nodejs, app, nginx. Toutes les valeurs sont dans group_vars/all.yml (dépôt, version, dossier, port, nom du serveur).
Le redémarrage de l'application passe par un handler déclenché uniquement quand le dépôt change ; le rechargement de Nginx passe aussi par un handler.
Validation : --syntax-check OK, 1re exécution sans erreur, 2e exécution à changed=0 (captures/idempotence.png). Portfolio visible sur http://192.168.237.136 (captures/portfolio.png).

## Partie B : Semaphore
Objets créés : projet portfolio-deploy, clé SSH, dépôt GitHub, inventaire statique, groupe de variables (server_name, app_port), template « Deploy Portfolio » avec la variable de survey app_version (captures/semaphore-template.png).
Après modification d'une page du portfolio et push, la relance a cloné la nouvelle version, reconstruit l'application et déclenché le handler de redémarrage (captures/semaphore-run.png).

## Difficultés et solutions
- Port 80 intercepté par k3s sur la cible : arrêt de k3s pour la durée du TP.
- Pare-feu ufw bloquant le port 80 : ajout d'une tâche ufw dans le rôle nginx.
- Docker Desktop inutilisable (virtualisation non détectée) : Semaphore lancé dans Docker sur vm-admin.
- Erreur « no key found » dans Semaphore : création d'une clé SSH dédiée, sans passphrase.
- Résolution DNS intermittente dans le conteneur : DNS publics fixés avec l'option --dns.

## Conclusion
Le déploiement est reproductible et idempotent, et Semaphore permet de le lancer et de le rejouer depuis une interface, avec un historique des exécutions.
