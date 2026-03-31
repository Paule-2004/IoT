Paule SAMBA
MCS26.3

Rapport de Travaux Pratiques : Sécurisation MQTT via mTLS

Sujet : Mise en place d'une authentification mutuelle TLS (mTLS) avec Mosquitto et OpenSSL
Environnement : WSL

1. Introduction et Objectifs
L'objectif de ce TP est de sécuriser les communications entre un client IoT et un broker MQTT. Contrairement au TLS classique (où seul le serveur est identifié), le mTLS (mutual TLS) impose que le client présente également un certificat valide signé par une autorité de confiance (CA).
Objectifs clés :
•	Créer une Autorité de Certification (CA) racine.
•	Générer et signer des certificats X.509 pour le broker et le client.
•	Configurer Mosquitto pour exiger l'authentification par certificat.
•	Valider la chaîne de confiance par une publication de message.

2. Architecture de la Chaîne de Confiance
Pour ce TP, nous avons mis en place une hiérarchie de confiance :
1.	CA (ca.crt) : Le pivot de confiance. Elle sert à signer tous les autres certificats.
2.	Certificat Broker (broker.crt) : Permet au client de vérifier qu'il parle au bon serveur.
3.	Certificat Client (client.crt) : Permet au broker de vérifier l'identité du client IoT.

3. Déroulement Technique
Étape A : Génération de l'Autorité de Certification (CA)
Nous avons généré une clé privée RSA de 2048 bits, puis un certificat racine auto-signé valable 10 ans.
openssl genrsa -out ca.key 2048
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt -subj "/CN=MQTT-CA"
 
 
Étape B : Génération et Signature des certificats
Nous avons créé des demandes de signature (CSR) pour le broker et le client, puis nous les avons signées avec notre propre CA.
•	Broker : CN=localhost
•	Client : CN=paule-client
 
 

Étape C : Configuration du Broker Mosquitto
Le fichier mosquitto.conf a été configuré pour écouter sur le port standard sécurisé 8883 et exiger un certificat client.
 

4. Tests et Validation
Le test a été effectué en deux temps :
1.	Lancement du broker en mode verbeux : mosquitto -v -c mosquitto.conf
2.	Publication d'un message depuis un autre terminal avec les certificats.
 

Commande de test :
mosquitto_pub -h localhost -p 8883 --cafile ca.crt --cert client.crt --key client.key -t "test" -m "Victoire"
 

5. Résolution des Problèmes (Troubleshooting)
Durant le TP, plusieurs erreurs ont été rencontrées et résolues :
1.	Error: File not found : Résolue en s'assurant que la commande mosquitto_pub était lancée depuis le dossier contenant les fichiers .crt et .key.
 
2.	TLS Alert Internal Error : Cette erreur survient souvent lorsque les certificats ne correspondent pas à la CA actuelle. Une régénération complète de la chaîne a résolu le problème.
 
3.	Connection Refused (Not authorised) : Résolue en ajustant les paramètres use_identity_as_username et allow_anonymous dans la configuration.
 

6. Déploiement sur GitHub
Le code final a été poussé sur un dépôt GitHub public.
•	Sécurisation : Utilisation d'un fichier .gitignore pour exclure les clés privées (*.key).
•	Automatisation : Création d'un script setup.sh pour automatiser la génération des certificats pour les futurs utilisateurs.
•	Authentification : Utilisation d'un Personal Access Token (PAT) pour le push vers GitHub.
 
