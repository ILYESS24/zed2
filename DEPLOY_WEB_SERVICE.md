# 🚀 Déploiement sur Render.com via Web Service (Sans Blueprint)

Ce guide vous explique comment déployer le serveur Zed collab directement comme Web Service sur Render, sans utiliser Blueprint.

## ✅ Avantages du Web Service

- ✅ Plus simple et direct
- ✅ Plus de contrôle sur la configuration
- ✅ Pas besoin de render.yaml
- ✅ Configuration étape par étape via l'interface

## 📋 Étapes de déploiement

### Étape 1 : Créer un compte Render

1. Allez sur [render.com](https://render.com)
2. Créez un compte ou connectez-vous
3. Connectez votre compte GitHub dans **Account Settings** → **Connected Accounts**

### Étape 2 : Créer la base de données PostgreSQL

1. Dans le dashboard Render, cliquez sur **"New +"** (en haut à droite)
2. Sélectionnez **"PostgreSQL"**
3. Configurez :
   - **Name**: `zed-database`
   - **Database**: `zed`
   - **User**: `zed`
   - **Region**: `Oregon` (ou la région la plus proche)
   - **Plan**: 
     - `Starter` (gratuit, 90 jours puis $7/mois)
     - `Standard` (payant, $7/mois, toujours actif)
   - **PostgreSQL Version**: `15`
4. Cliquez sur **"Create Database"**
5. ⚠️ **IMPORTANT** : Notez l'**Internal Database URL** (vous en aurez besoin)
   - Format : `postgresql://user:password@host:port/database`

### Étape 3 : Créer le Web Service

1. Dans le dashboard Render, cliquez sur **"New +"**
2. Sélectionnez **"Web Service"**
3. Connectez votre repository GitHub :
   - Si pas encore connecté, cliquez sur **"Connect GitHub"**
   - Autorisez Render à accéder à vos repositories
   - Sélectionnez le repository **`zed2`**
4. Configurez le service :

#### Configuration de base :

- **Name**: `zed-collab`
- **Region**: `Oregon` (même région que la base de données)
- **Branch**: `main`
- **Root Directory**: `.` (laisser vide ou mettre `.`)

#### Configuration du build :

- **Environment**: `Docker`
- **Dockerfile Path**: `Dockerfile.render`
- **Docker Context**: `.`
- **Build Command**: (laisser vide - Docker gère le build)
- **Start Command**: (laisser vide - défini dans Dockerfile)

#### Plan :

- **Plan**: 
  - `Starter` (gratuit, s'endort après 15 min d'inactivité)
  - `Standard` ($7/mois, toujours actif) - **Recommandé pour production**

### Étape 4 : Configurer les variables d'environnement

Dans la section **"Environment"** du service, ajoutez les variables suivantes :

#### Variables OBLIGATOIRES :

```bash
# Port HTTP
HTTP_PORT=8080

# Environnement
ZED_ENVIRONMENT=production

# Logging
RUST_LOG=info
LOG_JSON=true
RUST_BACKTRACE=1

# Base de données (utilisez l'Internal Database URL de l'étape 2)
DATABASE_URL=<Internal Database URL de zed-database>
DATABASE_MAX_CONNECTIONS=10

# Token API (générer un token sécurisé)
# Windows PowerShell:
# [Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Minimum 0 -Maximum 256 }))
# Linux/Mac:
# openssl rand -hex 32
API_TOKEN=<générez-un-token-sécurisé>

# Configuration des invitations
INVITE_LINK_PREFIX=https://zed-collab.onrender.com/invites/
```

#### Variables OPTIONNELLES (pour fonctionnalités avancées) :

```bash
# Base de données LLM (si vous créez une deuxième DB)
LLM_DATABASE_URL=<Internal Database URL de zed-llm-database>
LLM_DATABASE_MAX_CONNECTIONS=5
LLM_API_SECRET=<secret-optionnel>

# LiveKit (pour audio/vidéo)
LIVEKIT_SERVER=wss://votre-livekit-server.com
LIVEKIT_KEY=votre-key
LIVEKIT_SECRET=votre-secret

# Blob Store (S3/MinIO)
BLOB_STORE_URL=https://votre-s3-endpoint.com
BLOB_STORE_REGION=us-east-1
BLOB_STORE_ACCESS_KEY=votre-access-key
BLOB_STORE_SECRET_KEY=votre-secret-key
BLOB_STORE_BUCKET=nom-du-bucket

# Clés API LLM
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_AI_API_KEY=...

# Autres
AUTO_JOIN_CHANNEL_ID=1
ZED_CLIENT_CHECKSUM_SEED=<générer-avec-openssl-rand-hex-32>
```

### Étape 5 : Configurer le Health Check

Dans la section **"Health Check Path"** :
- **Health Check Path**: `/healthz`
- Render vérifiera automatiquement que le service est en ligne

### Étape 6 : Déployer

1. Cliquez sur **"Create Web Service"**
2. Render va :
   - Cloner votre repository
   - Builder l'image Docker (10-15 minutes la première fois)
   - Démarrer le service
3. Suivez la progression dans l'onglet **"Logs"**

### Étape 7 : Vérifier le déploiement

Une fois le déploiement terminé :

1. Votre service sera disponible à : `https://zed-collab.onrender.com`
2. Testez le health check :
```bash
curl https://zed-collab.onrender.com/healthz
```

Vous devriez recevoir une réponse `200 OK`.

## 🔧 Configuration minimale pour démarrer

Pour tester rapidement, vous n'avez besoin que de :

1. ✅ **DATABASE_URL** (de l'étape 2)
2. ✅ **API_TOKEN** (générer avec la commande ci-dessus)
3. ✅ **HTTP_PORT=8080**
4. ✅ **ZED_ENVIRONMENT=production**

Les autres variables peuvent être ajoutées plus tard.

## 📝 Notes importantes

### Plan gratuit vs payant

- **Starter (gratuit)** :
  - ⚠️ Service s'endort après 15 minutes d'inactivité
  - ⚠️ Premier démarrage après sommeil : 30-60 secondes
  - ✅ Parfait pour tester

- **Standard ($7/mois)** :
  - ✅ Service toujours actif
  - ✅ Meilleure performance
  - ✅ Recommandé pour production

### Migrations de base de données

Les migrations s'exécutent automatiquement au premier démarrage du service. Vérifiez les logs pour confirmer.

### WebSocket

Render supporte nativement les WebSocket. Assurez-vous d'utiliser `wss://` (WebSocket Secure) pour les connexions.

## 🔄 Mises à jour

Les mises à jour se font automatiquement quand vous poussez sur la branche `main`.

Pour forcer un redéploy :
1. Dashboard → Votre service
2. **"Manual Deploy"** → **"Deploy latest commit"**

## 🆘 Dépannage

### Le service ne démarre pas

1. Vérifiez les logs dans Render Dashboard
2. Assurez-vous que `DATABASE_URL` est correcte (utilisez Internal Database URL)
3. Vérifiez que `API_TOKEN` est configuré
4. Vérifiez que le Dockerfile est correct

### Erreur de connexion à la base de données

1. Utilisez l'**Internal Database URL** (pas External)
2. Vérifiez que le service web et la DB sont dans la même région
3. Vérifiez les credentials dans l'URL

### Build échoue

1. Vérifiez que `Dockerfile.render` existe dans le repository
2. Vérifiez les logs de build pour voir l'erreur exacte
3. Le build peut prendre 15 minutes la première fois

### Service s'endort (plan gratuit)

C'est normal avec le plan Starter. Le service se réveillera automatiquement à la prochaine requête (30-60 secondes de délai).

## 🎉 Félicitations !

Votre serveur de collaboration Zed est maintenant déployé sur Render.com !

## 🔗 Liens utiles

- [Documentation Render](https://render.com/docs)
- [Support Render](https://community.render.com/)
- [Pricing Render](https://render.com/pricing)

