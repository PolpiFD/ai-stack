# 🌟 AI-Stack — déployer Open WebUI + Ollama + Caddy (HTTPS) en 10 minutes

Ce guide explique **pas-à-pas**, depuis un VPS Debian 11/12 flambant neuf, comment :

1. installer Docker Engine + Compose v2  
2. cloner ce dépôt et remplir un simple fichier `.env`  
3. lancer la pile en **une seule commande**  
4. télécharger vos modèles et les utiliser depuis WebUI *ou* n8n externe  

---

## 1. Préparer le VPS

> Toutes les commandes ci-dessous sont prévues pour l’utilisateur `debian` livré par défaut sur beaucoup de VPS.  
> Préfixez par `sudo` si vous n’êtes pas root ; gardez-les **dans l’ordre**.


# Mise à jour et outils de base
```
sudo apt update && sudo apt upgrade -y
sudo apt install -y ca-certificates curl gnupg lsb-release apt-transport-https
```
2. Installer Docker Engine + Compose v2

# Ajouter la clé GPG Docker
```
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release && echo "$ID")/gpg |
  sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg
```

# Activer le dépôt stable
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] \
https://download.docker.com/linux/$(. /etc/os-release && echo "$ID") \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
```

# Installer le moteur + compose-plugin
```
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

# Démarrer et activer le service
```
sudo systemctl enable --now docker
```


Vérification : 
```
docker run --rm hello-world
```

3. Cloner la stack
```
git clone https://github.com/<votre-compte>/ai-stack.git

cd ai-stack
```
4. Configurer les variables

Dupliquez le modèle :
```
cp .env.sample .env
```
Éditez .env :
```
EMAIL=admin@example.com                 # pour Let's Encrypt
DOMAIN_WEBUI=webui.mondomaine.com
DOMAIN_OLLAMA=ollama.mondomaine.com
DOMAIN_N8N=n8n.mondomaine.com           # IP ou sous-domaine de l’instance n8n externe
```
5. Démarrer
```
docker compose up -d
```
Caddy obtient automatiquement les certificats TLS.

Open WebUI : https://webui.mondomaine.com

API Ollama : https://ollama.mondomaine.com/v1/models

(premier lancement ≈ 1 minute le temps de télécharger les images)

6. Ajouter un modèle
a) via la ligne de commande
```
docker compose exec ollama ollama pull qwen3:4b
```
Le modèle apparaît alors :

dans WebUI (sélecteur de modèles)

dans nœud Ollama de n8n (Base URL = https://ollama.mondomaine.com)

7. Mettre à jour
```
cd ~/ai-stack

docker compose pull        # télécharge les nouvelles images
docker compose up -d       # redémarre sans perte de données
```
Volumes persistants :

Volume	Contenu
ollama-data
webui-data	
caddy-data

10. Désinstaller
```
docker compose down -v   # stoppe et supprime les volumes
sudo apt purge -y docker-ce\* containerd.io docker-compose-plugin
sudo rm -rf /var/lib/docker /var/lib/containerd
```
