# 🚂 Déploiement sur Railway.app

Railway est une excellente alternative à Render.com, avec un meilleur pricing et plus de RAM pour le build.

## ✅ Avantages de Railway

- **8 GB RAM pour le build** (vs 2 GB sur Render Standard)
- **Pricing généreux** : $5/mois pour 8 GB RAM
- **Docker natif** : Support complet
- **PostgreSQL intégré** : Base de données managée
- **Interface simple** : Très facile à utiliser
- **Auto-deploy** : Déploiement automatique depuis GitHub

## 📋 Prérequis

- Compte GitHub
- Compte Railway (gratuit) : [railway.app](https://railway.app)

## 🚀 Déploiement Étape par Étape

### Étape 1 : Créer un compte Railway

1. Allez sur [railway.app](https://railway.app)
2. Cliquez sur **"Start a New Project"**
3. Connectez-vous avec **GitHub**
4. Autorisez Railway à accéder à vos repositories

### Étape 2 : Créer un nouveau projet

1. Cliquez sur **"New Project"**
2. Sélectionnez **"Deploy from GitHub repo"**
3. Choisissez votre repository **`zed2`**
4. Railway va détecter automatiquement le Dockerfile

### Étape 3 : Configurer le service

Railway devrait détecter automatiquement `Dockerfile.render`. Si non :

1. Dans les **Settings** du service
2. **Dockerfile Path** : `Dockerfile.render`
3. **Root Directory** : `.`

### Étape 4 : Créer la base de données PostgreSQL

1. Dans votre projet Railway, cliquez sur **"+ New"**
2. Sélectionnez **"Database"** → **"Add PostgreSQL"**
3. Railway créera automatiquement la base de données
4. Les variables d'environnement seront automatiquement injectées :
   - `DATABASE_URL` (automatique)
   - `PGHOST`, `PGPORT`, `PGUSER`, `PGPASSWORD`, `PGDATABASE`

### Étape 5 : Configurer les variables d'environnement

Dans les **Variables** du service, ajoutez :

#### Configuration de base
```bash
HTTP_PORT=8080
ZED_ENVIRONMENT=production
RUST_LOG=info
LOG_JSON=true
RUST_BACKTRACE=1
```

#### Base de données
```bash
# DATABASE_URL est automatiquement injecté par Railway PostgreSQL
# Mais vous pouvez l'ajouter manuellement si besoin
DATABASE_MAX_CONNECTIONS=10
```

#### Base de données LLM (optionnelle)
```bash
# Créez une deuxième base PostgreSQL si nécessaire
LLM_DATABASE_URL=<URL de la base LLM>
LLM_DATABASE_MAX_CONNECTIONS=5
```

#### API Token
```bash
# Générez un token sécurisé
API_TOKEN=<votre-token-securise>
```

#### LiveKit (optionnel)
```bash
LIVEKIT_SERVER=<votre-serveur-livekit>
LIVEKIT_KEY=<votre-cle>
LIVEKIT_SECRET=<votre-secret>
```

#### Blob Store (optionnel)
```bash
BLOB_STORE_URL=<votre-url-s3>
BLOB_STORE_REGION=<votre-region>
BLOB_STORE_ACCESS_KEY=<votre-access-key>
BLOB_STORE_SECRET_KEY=<votre-secret-key>
BLOB_STORE_BUCKET=<votre-bucket>
```

#### Clés API LLM (optionnelles)
```bash
OPENAI_API_KEY=<votre-cle>
ANTHROPIC_API_KEY=<votre-cle>
GOOGLE_AI_API_KEY=<votre-cle>
```

#### Configuration des invitations
```bash
INVITE_LINK_PREFIX=https://votre-domaine.com/invites/
```

### Étape 6 : Configurer le domaine (optionnel)

1. Dans les **Settings** du service
2. Cliquez sur **"Generate Domain"** pour un domaine Railway
3. Ou ajoutez un **Custom Domain** :
   - Ajoutez votre domaine
   - Configurez les DNS selon les instructions Railway

### Étape 7 : Déployer

Railway déploie automatiquement à chaque push sur la branche connectée.

Pour déployer manuellement :
1. Cliquez sur **"Deploy"** dans le dashboard
2. Ou faites un push sur GitHub

## 🔧 Configuration Avancée

### Railway.json (optionnel)

Créez un fichier `railway.json` à la racine pour personnaliser le déploiement :

```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "DOCKERFILE",
    "dockerfilePath": "Dockerfile.render"
  },
  "deploy": {
    "startCommand": "/app/collab serve all",
    "healthcheckPath": "/healthz",
    "healthcheckTimeout": 100,
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

### Limites de ressources

Dans les **Settings** → **Resources** :
- **RAM** : 512 MB - 8 GB (recommandé : 2 GB minimum)
- **CPU** : 0.5 - 4 vCPU

## 📊 Monitoring

Railway fournit :
- **Logs en temps réel** : Dans le dashboard
- **Métriques** : CPU, RAM, Network
- **Health checks** : Automatique sur `/healthz`

## 💰 Pricing

### Hobby Plan ($5/mois)
- 500 heures gratuites/mois
- 8 GB RAM pour build
- 512 MB - 8 GB RAM pour runtime
- Support communautaire

### Pro Plan ($20/mois)
- Illimité
- 8 GB RAM pour build
- 512 MB - 8 GB RAM pour runtime
- Support prioritaire
- Team collaboration

### PostgreSQL
- **Starter** : Gratuit (256 MB, 90 jours puis $5/mois)
- **Pro** : $5/mois (1 GB, toujours actif)

## 🐛 Dépannage

### Build échoue avec OOM

1. Vérifiez que vous avez assez de RAM allouée
2. Railway donne 8 GB pour le build par défaut
3. Si problème persiste, contactez le support

### Service ne démarre pas

1. Vérifiez les logs dans Railway dashboard
2. Vérifiez que toutes les variables d'environnement sont configurées
3. Vérifiez que `DATABASE_URL` est correct

### Erreurs de connexion à la base de données

1. Vérifiez que la base PostgreSQL est créée
2. Vérifiez que `DATABASE_URL` utilise l'URL interne Railway
3. Railway injecte automatiquement `DATABASE_URL` si la base est dans le même projet

## 🔄 Migration depuis Render

1. **Exporter les variables d'environnement** depuis Render
2. **Créer le projet Railway** et connecter GitHub
3. **Créer la base PostgreSQL** dans Railway
4. **Importer les variables d'environnement** (sauf DATABASE_URL)
5. **Mettre à jour DATABASE_URL** avec la nouvelle URL Railway
6. **Déployer** et tester
7. **Mettre à jour le DNS** si domaine personnalisé

## ✅ Avantages vs Render

| Feature | Railway | Render |
|---------|---------|--------|
| RAM Build | 8 GB | 2-4 GB |
| RAM Runtime | 512 MB - 8 GB | 512 MB - 4 GB |
| Pricing | $5/mois | $7-25/mois |
| PostgreSQL | Intégré | Intégré |
| Interface | Plus moderne | Standard |
| Auto-deploy | ✅ | ✅ |

## 🎯 Conclusion

Railway est une excellente alternative à Render avec :
- ✅ Meilleur pricing
- ✅ Plus de RAM pour le build
- ✅ Interface plus moderne
- ✅ Support Docker complet

Le déploiement est très similaire à Render, mais avec de meilleures ressources.

