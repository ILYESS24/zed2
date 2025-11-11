# 🌊 Déploiement sur DigitalOcean App Platform

DigitalOcean App Platform est similaire à Render, avec une infrastructure éprouvée (déjà utilisée par Zed en production).

## ✅ Avantages de DigitalOcean

- **Infrastructure éprouvée** : Déjà utilisé par Zed en production (Kubernetes)
- **Stable et fiable** : Uptime excellent
- **Docker support** : Support complet
- **PostgreSQL intégré** : Base de données managée
- **Bon pricing** : $5-12/mois pour usage basique
- **Régions multiples** : Plusieurs datacenters
- **Monitoring intégré** : Métriques et logs

## 📋 Prérequis

- Compte GitHub
- Compte DigitalOcean : [digitalocean.com](https://digitalocean.com)

## 🚀 Déploiement Étape par Étape

### Étape 1 : Créer un compte DigitalOcean

1. Allez sur [digitalocean.com](https://digitalocean.com)
2. Créez un compte ou connectez-vous
3. Vérifiez votre email

### Étape 2 : Créer une App

1. Dans le dashboard DigitalOcean, cliquez sur **"Create"** → **"App"**
2. Connectez votre compte GitHub :
   - Cliquez sur **"GitHub"**
   - Autorisez DigitalOcean à accéder à vos repositories
   - Sélectionnez le repository **`zed2`**

### Étape 3 : Configurer le service

#### Configuration de base

1. **Source** :
   - **Repository** : `zed2`
   - **Branch** : `main`
   - **Root Directory** : `.`

2. **Environment** :
   - Sélectionnez **"Dockerfile"**
   - **Dockerfile Path** : `Dockerfile.render`

3. **Resources** :
   - **Plan** : Basic ($5/mois) ou Professional ($12/mois)
   - **RAM** : 512 MB - 2 GB (recommandé : 1-2 GB)
   - **CPU** : 0.5 - 1 vCPU

### Étape 4 : Créer la base de données PostgreSQL

1. Dans la configuration de l'App, cliquez sur **"+ Add Resource"**
2. Sélectionnez **"Database"** → **"PostgreSQL"**
3. Configurez :
   - **Database Name** : `zed-database`
   - **Plan** : 
     - **Basic** : $15/mois (1 GB RAM, 10 GB stockage)
     - **Professional** : $60/mois (2 GB RAM, 25 GB stockage)
   - **Region** : Même région que l'App
   - **Version** : PostgreSQL 15

4. DigitalOcean créera automatiquement la base et injectera `DATABASE_URL`

### Étape 5 : Configurer les variables d'environnement

Dans **"Environment Variables"**, ajoutez :

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
# DATABASE_URL est automatiquement injecté par DigitalOcean
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

1. Dans **"Settings"** → **"Domains"**
2. Cliquez sur **"Add Domain"**
3. Ajoutez votre domaine personnalisé
4. Configurez les DNS selon les instructions DigitalOcean

### Étape 7 : Déployer

1. Cliquez sur **"Next"** pour passer en revue la configuration
2. Cliquez sur **"Create Resources"**
3. DigitalOcean va :
   - Builder l'image Docker
   - Créer la base de données
   - Déployer l'application

Le premier déploiement peut prendre 15-30 minutes.

### Étape 8 : Vérifier le déploiement

1. Dans le dashboard, allez dans **"Runtime Logs"**
2. Vérifiez que l'application démarre correctement
3. Testez l'endpoint `/healthz`

## 🔧 Configuration Avancée

### Health Checks

Dans **"Settings"** → **"Health Checks"** :
- **Path** : `/healthz`
- **Initial Delay** : 30 secondes
- **Period** : 10 secondes
- **Timeout** : 5 secondes

### Auto-deploy

Par défaut, DigitalOcean déploie automatiquement à chaque push sur la branche connectée.

Pour désactiver :
- **Settings** → **"Deploy"** → Désactiver **"Auto Deploy"**

### Scaling

Dans **"Settings"** → **"Scaling"** :
- **Instance Count** : 1-10 instances
- **Auto-scaling** : Activer si nécessaire

## 📊 Monitoring

DigitalOcean fournit :
- **Runtime Logs** : Logs en temps réel
- **Metrics** : CPU, RAM, Network, Request Rate
- **Alerts** : Configurables pour CPU, RAM, etc.

## 💰 Pricing

### App Platform

- **Basic** : $5/mois (512 MB RAM, 0.5 vCPU)
- **Professional** : $12/mois (1 GB RAM, 1 vCPU)
- **Build** : 2-4 GB RAM inclus

### PostgreSQL

- **Basic** : $15/mois (1 GB RAM, 10 GB stockage)
- **Professional** : $60/mois (2 GB RAM, 25 GB stockage)

### Total estimé

- **App Basic + DB Basic** : ~$20/mois
- **App Professional + DB Professional** : ~$72/mois

## 🐛 Dépannage

### Build échoue

1. Vérifiez les **Build Logs** dans le dashboard
2. Vérifiez que `Dockerfile.render` existe
3. Vérifiez les limites de RAM (augmentez si nécessaire)

### Service ne démarre pas

1. Vérifiez les **Runtime Logs**
2. Vérifiez que toutes les variables d'environnement sont configurées
3. Vérifiez que `DATABASE_URL` est correct

### Erreurs de connexion à la base de données

1. Vérifiez que la base PostgreSQL est créée
2. Vérifiez que `DATABASE_URL` utilise l'URL interne DigitalOcean
3. DigitalOcean injecte automatiquement `DATABASE_URL` si la base est dans la même App

## 🔄 Migration depuis Render

1. **Exporter les variables d'environnement** depuis Render
2. **Créer l'App** dans DigitalOcean et connecter GitHub
3. **Créer la base PostgreSQL** dans DigitalOcean
4. **Importer les variables d'environnement** (sauf DATABASE_URL)
5. **Mettre à jour DATABASE_URL** avec la nouvelle URL DigitalOcean
6. **Déployer** et tester
7. **Mettre à jour le DNS** si domaine personnalisé

## ✅ Avantages vs Render

| Feature | DigitalOcean | Render |
|---------|--------------|--------|
| RAM Build | 2-4 GB | 2-4 GB |
| RAM Runtime | 512 MB - 2 GB | 512 MB - 4 GB |
| Pricing | $5-12/mois | $7-25/mois |
| Infrastructure | Éprouvée | Standard |
| PostgreSQL | Intégré | Intégré |
| Monitoring | ✅ | ✅ |

## 🎯 Conclusion

DigitalOcean App Platform est une excellente alternative à Render avec :
- ✅ Infrastructure éprouvée (déjà utilisée par Zed)
- ✅ Stable et fiable
- ✅ Bon pricing
- ✅ Support Docker et PostgreSQL complet

Similaire à Render, mais avec une infrastructure plus mature.

