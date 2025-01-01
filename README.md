# Configuration-de-serveur-Fedora-avec-Nginx



Voici les étapes pour installer et configurer **Nginx** sur une distribution Fedora :

---

### **Étape 1 : Installer Nginx**
1. **Mettre à jour le système :**
   ```bash
   sudo dnf update -y
   ```
2. **Installer Nginx :**
   ```bash
   sudo dnf install nginx -y
   ```

3. **Vérifier la version installée :**
   ```bash
   nginx -v
   ```

---

### **Étape 2 : Activer et démarrer le service Nginx**
1. **Activer le service pour qu'il démarre automatiquement au démarrage :**
   ```bash
   sudo systemctl enable nginx
   ```

2. **Démarrer le service :**
   ```bash
   sudo systemctl start nginx
   ```

3. **Vérifier le statut du service :**
   ```bash
   sudo systemctl status nginx
   ```

4. **Autoriser Nginx dans le pare-feu :**
   ```bash
   sudo firewall-cmd --permanent --add-service=http
   sudo firewall-cmd --permanent --add-service=https
   sudo firewall-cmd --reload
   ```

---

### **Étape 3 : Tester l'installation**
- Ouvrez un navigateur et accédez à l'adresse :
  ```
  http://localhost
  ```
  Si Nginx est installé et en cours d'exécution, vous verrez la page d'accueil par défaut de Nginx.

---

### **Étape 4 : Configuration de base**
1. **Fichiers de configuration principaux :**
   - Le fichier de configuration principal se trouve ici :
     ```bash
     /etc/nginx/nginx.conf
     ```

   - Les fichiers de configuration des sites se trouvent dans :
     ```bash
     /etc/nginx/conf.d/
     ```

2. **Créer un nouveau fichier de configuration pour un site :**
   Exemple de fichier `/etc/nginx/conf.d/mon_site.conf` :
   ```nginx
   server {
       listen 80;
       server_name monsite.com www.monsite.com;

       root /var/www/monsite;
       index index.html index.htm;

       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```

3. **Créer le répertoire pour le site web :**
   ```bash
   sudo mkdir -p /var/www/monsite
   ```

4. **Donner les permissions appropriées :**
   ```bash
   sudo chown -R nginx:nginx /var/www/monsite
   sudo chmod -R 755 /var/www/monsite
   ```

5. **Ajouter un fichier `index.html` de test :**
   ```bash
   echo "Bienvenue sur mon site !" | sudo tee /var/www/monsite/index.html
   ```

6. **Redémarrer Nginx pour appliquer les changements :**
   ```bash
   sudo systemctl restart nginx
   ```

---

### **Étape 5 : Vérifier la configuration**
1. **Tester la configuration de Nginx :**
   ```bash
   sudo nginx -t
   ```

2. **Afficher les journaux en cas de problème :**
   - Journaux d'erreurs :
     ```bash
     sudo journalctl -xeu nginx
     ```
   - Journaux d'accès :
     ```bash
     sudo tail -f /var/log/nginx/access.log
     ```

---

### **Étape 6 : Configurer HTTPS (Optionnel)**
1. **Installer Certbot :**
   ```bash
   sudo dnf install certbot python3-certbot-nginx -y
   ```

2. **Obtenir un certificat SSL :**
   Remplacez `monsite.com` par votre domaine réel.
   ```bash
   sudo certbot --nginx -d monsite.com -d www.monsite.com
   ```

3. **Renouveler automatiquement les certificats :**
   Ajoutez cette commande dans le cron :
   ```bash
   sudo crontab -e
   ```
   Ligne à ajouter :
   ```bash
   0 0 * * * certbot renew --quiet
   ```

---

Avec ces étapes, Nginx sera installé et configuré correctement sur votre Fedora. 


------------------------------------------

Voici un script shell pour installer et configurer **Nginx** sur Fedora. Ce script automatise les étapes de base, y compris la configuration d'un site web.

---

### **Script : `install_nginx.sh`**
```bash
#!/bin/bash

# Vérification des privilèges root
if [ "$EUID" -ne 0 ]; then
  echo "Veuillez exécuter ce script en tant qu'administrateur (root)."
  exit 1
fi

# Mettre à jour le système
echo "Mise à jour du système..."
dnf update -y

# Installer Nginx
echo "Installation de Nginx..."
dnf install nginx -y

# Activer et démarrer le service Nginx
echo "Activation et démarrage de Nginx..."
systemctl enable nginx
systemctl start nginx

# Vérifier le statut du service
echo "Statut de Nginx :"
systemctl status nginx | grep Active

# Configuration du pare-feu
echo "Configuration du pare-feu..."
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload

# Création d'un site web par défaut
SITE_NAME="monsite"
SITE_ROOT="/var/www/$SITE_NAME"
CONF_FILE="/etc/nginx/conf.d/$SITE_NAME.conf"

echo "Création du répertoire pour le site web..."
mkdir -p $SITE_ROOT

echo "Ajout d'une page d'accueil par défaut..."
echo "<!DOCTYPE html>
<html>
<head>
    <title>Bienvenue sur $SITE_NAME</title>
</head>
<body>
    <h1>Site $SITE_NAME est configuré avec succès !</h1>
</body>
</html>" > $SITE_ROOT/index.html

# Attribution des permissions
echo "Attribution des permissions..."
chown -R nginx:nginx $SITE_ROOT
chmod -R 755 $SITE_ROOT

# Configuration Nginx pour le site
echo "Création du fichier de configuration Nginx..."
echo "server {
    listen 80;
    server_name $SITE_NAME.local www.$SITE_NAME.local;

    root $SITE_ROOT;
    index index.html;

    location / {
        try_files \$uri \$uri/ =404;
    }
}" > $CONF_FILE

# Vérification et redémarrage de Nginx
echo "Test de configuration de Nginx..."
nginx -t

echo "Redémarrage de Nginx..."
systemctl restart nginx

# Message de fin
echo "Installation et configuration de Nginx terminées."
echo "Visitez http://localhost pour voir le site par défaut."
```

---

### **Instructions :**
1. **Créer le script :**
   - Enregistrez le script sous le nom `install_nginx.sh`.

2. **Donner les permissions d'exécution :**
   ```bash
   chmod +x install_nginx.sh
   ```

3. **Exécuter le script :**
   ```bash
   sudo ./install_nginx.sh
   ```

---

### **Notes :**
- Par défaut, le site sera accessible à `http://localhost`.
- Pour un usage en production, configurez un domaine réel et ajoutez un certificat SSL comme mentionné dans l'étape précédente. Vous pouvez intégrer cette partie dans le script si nécessaire.

