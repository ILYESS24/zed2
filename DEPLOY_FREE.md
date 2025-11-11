# 🆓 Déploiement Gratuit - Guide Complet

Ce guide présente toutes les options **gratuites** pour déployer le serveur Zed collab.

## ⚠️ Limitations des Plans Gratuits

Les plans gratuits ont généralement :
- **RAM limitée** : 256 MB - 512 MB (insuffisant pour le build)
- **CPU limité** : 0.25 - 0.5 vCPU
- **Timeout** : Services qui s'endorment après inactivité
- **Limites de bande passante** : Quotas mensuels
- **Pas de support prioritaire**

## 🏆 Top 3 Options Gratuites

### 1. Fly.io ⭐⭐⭐⭐⭐

**Le meilleur gratuit pour Rust**

#### ✅ Offre Gratuite
- **3 VMs gratuites** (256 MB RAM chacune)
- **160 GB sortie réseau/mois** gratuit
- **Build illimité** (pas de limite de RAM)
- **Pas de timeout** (services toujours actifs)

#### ⚠️ Limitations
- 256 MB RAM par VM (limité pour runtime)
- Partage de CPU (peut être lent)

#### 💡 Solution : Build Externe
1. **Builder l'image localement** ou sur GitHub Actions
2. **Pousser vers Docker Hub** (gratuit)
3. **Utiliser l'image pré-buildée** sur Fly.io
4. **256 MB suffit pour runtime** si image optimisée

#### 🚀 Déploiement Rapide
```bash
# Installer Fly.io CLI
curl -L https://fly.io/install.sh | sh

# Se connecter
fly auth login

# Initialiser (utilise Dockerfile.render)
fly launch

# Configurer les secrets
fly secrets set DATABASE_URL="postgresql://..."

# Déployer
fly deploy
```

**Guide complet** : Voir `DEPLOY_FLYIO.md`

---

### 2. Google Cloud Run ⭐⭐⭐⭐

**Gratuit jusqu'à 2M requêtes/mois**

#### ✅ Offre Gratuite
- **2 millions de requêtes/mois** gratuites
- **400,000 GB-secondes** de CPU/mois gratuites
- **200,000 GB-secondes** de mémoire/mois gratuites
- **1 GB sortie réseau/mois** gratuit
- **Pas de timeout** (jusqu'à 60 min par requête)

#### ⚠️ Limitations
- **Build** : Nécessite Google Cloud Build (payant) ou build externe
- **Cold start** : Délai au premier démarrage
- **Timeout** : 60 minutes max par requête

#### 💡 Solution
1. **Builder l'image** sur GitHub Actions (gratuit)
2. **Pousser vers Google Container Registry** (gratuit)
3. **Déployer sur Cloud Run** (gratuit jusqu'à 2M requêtes)

#### 🚀 Déploiement Rapide
```bash
# Installer gcloud CLI
# https://cloud.google.com/sdk/docs/install

# Se connecter
gcloud auth login

# Créer un projet
gcloud projects create zed-collab --name="Zed Collab"

# Configurer Docker pour GCR
gcloud auth configure-docker

# Builder et pousser (ou utiliser GitHub Actions)
docker build -f Dockerfile.render -t gcr.io/zed-collab/zed-collab .
docker push gcr.io/zed-collab/zed-collab

# Déployer sur Cloud Run
gcloud run deploy zed-collab \
  --image gcr.io/zed-collab/zed-collab \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --memory 512Mi \
  --cpu 1 \
  --port 8080 \
  --set-env-vars HTTP_PORT=8080
```

**Guide complet** : Voir `DEPLOY_GOOGLE_CLOUD_RUN.md`

---

### 3. Railway.app ⭐⭐⭐⭐

**500 heures gratuites/mois**

#### ✅ Offre Gratuite
- **500 heures gratuites/mois** (environ 20 jours 24/7)
- **8 GB RAM pour build** (génial !)
- **512 MB - 8 GB RAM** pour runtime
- **PostgreSQL gratuit** (256 MB, 90 jours puis $5/mois)

#### ⚠️ Limitations
- **500 heures/mois** : Service s'arrête après
- **PostgreSQL** : Gratuit 90 jours puis payant
- **Pas de support** : Communauté uniquement

#### 💡 Solution
1. **Utiliser pour développement/test**
2. **Ou builder localement** et utiliser Railway seulement pour runtime
3. **PostgreSQL externe** : Utiliser Supabase (gratuit) ou Neon (gratuit)

#### 🚀 Déploiement Rapide
1. Allez sur [railway.app](https://railway.app)
2. Connectez GitHub
3. Créez un nouveau projet
4. Sélectionnez votre repository
5. Railway détecte automatiquement `Dockerfile.render`
6. Configurez les variables d'environnement
7. Déployez !

**Guide complet** : Voir `DEPLOY_RAILWAY.md`

---

## 🌐 Autres Options Gratuites

### 4. Render.com (Plan Gratuit) ⭐⭐⭐

#### ✅ Offre Gratuite
- **512 MB RAM** (insuffisant pour build)
- **Service s'endort** après 15 min d'inactivité
- **PostgreSQL gratuit** (90 jours puis $7/mois)

#### ⚠️ Limitations
- **Build échouera** avec 512 MB RAM
- **Service inactif** : Redémarre lentement après inactivité
- **Pas de domaine personnalisé** sur plan gratuit

#### 💡 Solution
- **Build externe** : Builder sur GitHub Actions
- **Utiliser l'image pré-buildée** : Render télécharge l'image
- **512 MB suffit pour runtime** si image optimisée

---

### 5. Oracle Cloud Free Tier ⭐⭐⭐

#### ✅ Offre Gratuite
- **2 VMs Always Free** (AMD) : 1/8 OCPU, 1 GB RAM
- **4 VMs Always Free** (ARM) : 4 OCPU, 24 GB RAM
- **200 GB stockage** gratuit
- **10 TB sortie réseau/mois** gratuit

#### ⚠️ Limitations
- **1 GB RAM** (insuffisant pour build)
- **Configuration manuelle** : Plus complexe
- **Nécessite carte de crédit** (mais gratuit)

#### 💡 Solution
- **Build externe** : GitHub Actions
- **Déployer l'image** sur VM Oracle
- **1 GB RAM suffit pour runtime**

---

### 6. AWS Free Tier ⭐⭐

#### ✅ Offre Gratuite
- **12 mois gratuits** pour nouveaux comptes
- **EC2 t2.micro** : 1 vCPU, 1 GB RAM (750 heures/mois)
- **RDS PostgreSQL** : db.t2.micro (750 heures/mois)

#### ⚠️ Limitations
- **12 mois seulement** : Puis payant
- **1 GB RAM** : Insuffisant pour build
- **Complexe** : Configuration AWS requise

---

### 7. Azure Free Tier ⭐⭐

#### ✅ Offre Gratuite
- **12 mois gratuits** pour nouveaux comptes
- **App Service** : 1 GB RAM (limité)
- **PostgreSQL** : 32 GB (12 mois)

#### ⚠️ Limitations
- **12 mois seulement**
- **RAM limitée** : Insuffisant pour build
- **Complexe** : Configuration Azure requise

---

## 🎯 Stratégies pour Déploiement Gratuit

### Stratégie 1 : Build Externe + Runtime Gratuit

**Le plus économique et fiable**

1. **Builder l'image** sur GitHub Actions (gratuit, illimité)
2. **Pousser vers Docker Hub** (gratuit)
3. **Déployer sur Fly.io/Cloud Run** (gratuit)
4. **Utiliser PostgreSQL externe** : Supabase ou Neon (gratuit)

**Avantages** :
- ✅ Pas de limite de RAM pour build
- ✅ Build rapide (GitHub Actions)
- ✅ Runtime gratuit sur plusieurs plateformes

---

### Stratégie 2 : Railway 500h/mois

**Simple mais limité**

1. **Utiliser Railway** pour build et runtime
2. **500 heures/mois** = ~20 jours 24/7
3. **PostgreSQL externe** : Supabase (gratuit)

**Avantages** :
- ✅ Très simple
- ✅ 8 GB RAM pour build
- ⚠️ Limité à 500h/mois

---

### Stratégie 3 : Fly.io + Build Externe

**Le plus flexible**

1. **Builder sur GitHub Actions**
2. **Pousser vers Docker Hub**
3. **Déployer sur Fly.io** (3 VMs gratuites)
4. **PostgreSQL externe** : Supabase

**Avantages** :
- ✅ 3 VMs gratuites
- ✅ Pas de timeout
- ✅ Build illimité (via GitHub Actions)

---

## 🗄️ Bases de Données PostgreSQL Gratuites

### Supabase ⭐⭐⭐⭐⭐

- **Gratuit** : 500 MB base, 2 GB bande passante
- **PostgreSQL 15** : Full PostgreSQL
- **API REST** : Inclus
- **Dashboard** : Interface web
- **URL** : [supabase.com](https://supabase.com)

### Neon ⭐⭐⭐⭐

- **Gratuit** : 0.5 GB base, auto-pause
- **PostgreSQL** : Serverless
- **Auto-scaling** : Automatique
- **URL** : [neon.tech](https://neon.tech)

### Railway PostgreSQL ⭐⭐⭐

- **Gratuit** : 90 jours (256 MB)
- **Puis** : $5/mois
- **Intégré** : Si vous utilisez Railway

---

## 📋 Comparaison des Options Gratuites

| Plateforme | RAM Build | RAM Runtime | Timeout | PostgreSQL | Meilleur pour |
|------------|-----------|-------------|---------|------------|---------------|
| **Fly.io** | Illimité* | 256 MB | ❌ | ❌ | Rust, toujours actif |
| **Cloud Run** | Externe* | 128 MB - 8 GB | 60 min | ❌ | Serverless, scalable |
| **Railway** | 8 GB | 512 MB - 8 GB | ❌ | 90 jours | Simple, 500h/mois |
| **Render** | 512 MB | 512 MB | 15 min | 90 jours | Simple, limité |
| **Oracle** | Externe* | 1-24 GB | ❌ | ❌ | VPS, contrôle total |

*Build externe recommandé (GitHub Actions gratuit)

---

## 🚀 Guide Complet : Déploiement Gratuit avec Build Externe

### Étape 1 : Builder sur GitHub Actions

Créez `.github/workflows/build-docker.yml` :

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [main]
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile.render
          push: true
          tags: |
            ${{ secrets.DOCKER_USERNAME }}/zed-collab:latest
            ${{ secrets.DOCKER_USERNAME }}/zed-collab:${{ github.sha }}
```

### Étape 2 : Créer la base de données Supabase

1. Allez sur [supabase.com](https://supabase.com)
2. Créez un projet gratuit
3. Notez la **Connection String** (DATABASE_URL)

### Étape 3 : Déployer sur Fly.io

```bash
# Créer fly.toml
fly launch --image votre-username/zed-collab:latest

# Configurer les secrets
fly secrets set DATABASE_URL="postgresql://..."
fly secrets set API_TOKEN="votre-token"

# Déployer
fly deploy
```

---

## 💡 Recommandation Finale

**Pour un déploiement 100% gratuit et fiable** :

1. ✅ **Build** : GitHub Actions (gratuit, illimité)
2. ✅ **Registry** : Docker Hub (gratuit)
3. ✅ **Runtime** : Fly.io (3 VMs gratuites, 256 MB RAM)
4. ✅ **PostgreSQL** : Supabase (gratuit, 500 MB)

**Coût total** : $0/mois

**Limitations** :
- 256 MB RAM par VM (suffisant pour runtime léger)
- 500 MB base de données (suffisant pour développement/test)

---

## 🎯 Conclusion

**Meilleure option gratuite** : **Fly.io + Build Externe**
- ✅ Build illimité (GitHub Actions)
- ✅ 3 VMs gratuites (toujours actives)
- ✅ Pas de timeout
- ✅ PostgreSQL gratuit (Supabase)

**Alternative simple** : **Railway 500h/mois**
- ✅ Très simple
- ✅ 8 GB RAM pour build
- ⚠️ Limité à 500h/mois

Tous les guides détaillés sont disponibles dans le repository.

