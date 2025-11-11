# ☁️ Déploiement sur Google Cloud Run (Gratuit)

Google Cloud Run est gratuit jusqu'à 2 millions de requêtes/mois, parfait pour le déploiement serverless.

## ✅ Avantages

- **Gratuit** : 2M requêtes/mois, 400K GB-secondes CPU
- **Serverless** : Scaling automatique à zéro
- **Docker natif** : Support complet
- **Global** : Déploiement multi-régions
- **Pas de timeout** : Jusqu'à 60 min par requête

## ⚠️ Limitations

- **Build** : Nécessite Google Cloud Build (payant) ou build externe
- **Cold start** : Délai au premier démarrage (~5-10 secondes)
- **Timeout** : 60 minutes max par requête

## 📋 Prérequis

- Compte Google Cloud (gratuit avec $300 crédit)
- `gcloud` CLI installé
- Docker installé (pour build local)

## 🚀 Installation

### Installer gcloud CLI

**Windows** :
```powershell
# Télécharger depuis https://cloud.google.com/sdk/docs/install
# Ou utiliser Chocolatey
choco install gcloudsdk
```

**macOS** :
```bash
brew install google-cloud-sdk
```

**Linux** :
```bash
curl https://sdk.cloud.google.com | bash
exec -l $SHELL
```

### Initialiser

```bash
gcloud init
```

## 🚀 Déploiement Étape par Étape

### Étape 1 : Créer un projet Google Cloud

```bash
# Créer un projet
gcloud projects create zed-collab --name="Zed Collab Server"

# Sélectionner le projet
gcloud config set project zed-collab

# Activer les APIs nécessaires
gcloud services enable run.googleapis.com
gcloud services enable containerregistry.googleapis.com
```

### Étape 2 : Builder l'image Docker

#### Option A : Build Local (Gratuit)

```bash
# Configurer Docker pour Google Container Registry
gcloud auth configure-docker

# Builder l'image
docker build -f Dockerfile.render -t gcr.io/zed-collab/zed-collab:latest .

# Pousser vers GCR
docker push gcr.io/zed-collab/zed-collab:latest
```

#### Option B : Build sur GitHub Actions (Gratuit)

Créez `.github/workflows/build-gcr.yml` :

```yaml
name: Build and Push to GCR

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}
      
      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v2
      
      - name: Configure Docker
        run: gcloud auth configure-docker
      
      - name: Build and push
        run: |
          docker build -f Dockerfile.render -t gcr.io/zed-collab/zed-collab:latest .
          docker push gcr.io/zed-collab/zed-collab:latest
```

### Étape 3 : Créer la base de données PostgreSQL

#### Option A : Cloud SQL (Payant, mais fiable)

```bash
# Créer une instance PostgreSQL
gcloud sql instances create zed-database \
  --database-version=POSTGRES_15 \
  --tier=db-f1-micro \
  --region=us-central1

# Créer la base de données
gcloud sql databases create zed --instance=zed-database

# Créer l'utilisateur
gcloud sql users create zed \
  --instance=zed-database \
  --password=votre-mot-de-passe
```

#### Option B : Supabase (Gratuit, Recommandé)

1. Allez sur [supabase.com](https://supabase.com)
2. Créez un projet gratuit
3. Notez la **Connection String**

### Étape 4 : Déployer sur Cloud Run

```bash
gcloud run deploy zed-collab \
  --image gcr.io/zed-collab/zed-collab:latest \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --memory 512Mi \
  --cpu 1 \
  --min-instances 0 \
  --max-instances 10 \
  --port 8080 \
  --timeout 300 \
  --set-env-vars HTTP_PORT=8080,ZED_ENVIRONMENT=production,RUST_LOG=info,LOG_JSON=true
```

### Étape 5 : Configurer les secrets

```bash
# Créer les secrets
echo -n "postgresql://user:pass@host:port/db" | gcloud secrets create database-url --data-file=-

# Donner accès à Cloud Run
gcloud secrets add-iam-policy-binding database-url \
  --member="serviceAccount:PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"

# Déployer avec secrets
gcloud run deploy zed-collab \
  --image gcr.io/zed-collab/zed-collab:latest \
  --update-secrets DATABASE_URL=database-url:latest,API_TOKEN=api-token:latest
```

### Étape 6 : Configurer toutes les variables d'environnement

```bash
gcloud run services update zed-collab \
  --update-env-vars \
    HTTP_PORT=8080,\
    ZED_ENVIRONMENT=production,\
    RUST_LOG=info,\
    LOG_JSON=true,\
    RUST_BACKTRACE=1,\
    DATABASE_MAX_CONNECTIONS=10,\
    INVITE_LINK_PREFIX=https://votre-domaine.com/invites/
```

## 🔧 Configuration Avancée

### Health Checks

Cloud Run vérifie automatiquement le port configuré. Assurez-vous que `/healthz` répond correctement.

### Custom Domain

```bash
# Mapper un domaine personnalisé
gcloud run domain-mappings create \
  --service zed-collab \
  --domain votre-domaine.com \
  --region us-central1
```

### Auto-scaling

```bash
# Configurer le scaling
gcloud run services update zed-collab \
  --min-instances 0 \
  --max-instances 10 \
  --concurrency 80 \
  --cpu-throttling
```

### VPC Connector (pour Cloud SQL)

```bash
# Créer un VPC connector
gcloud compute networks vpc-access connectors create zed-connector \
  --region=us-central1 \
  --subnet=default \
  --subnet-project=zed-collab

# Déployer avec VPC connector
gcloud run deploy zed-collab \
  --vpc-connector zed-connector \
  --vpc-egress private-ranges-only
```

## 📊 Monitoring

### Logs

```bash
# Voir les logs
gcloud run services logs read zed-collab --limit 50

# Logs en temps réel
gcloud run services logs tail zed-collab
```

### Métriques

Dans Google Cloud Console :
- **Cloud Run** → **Services** → **zed-collab**
- Métriques : Requêtes, Latence, Erreurs, CPU, Mémoire

## 💰 Pricing

### Gratuit (Always Free)

- **2 millions de requêtes/mois**
- **400,000 GB-secondes** CPU/mois
- **200,000 GB-secondes** mémoire/mois
- **1 GB sortie réseau/mois**

### Payant (après limites gratuites)

- **Requêtes** : $0.40 par million
- **CPU** : $0.00002400 par GB-seconde
- **Mémoire** : $0.00000250 par GB-seconde
- **Réseau** : $0.12 par GB

**Exemple** : 100K requêtes/mois, 512 MB RAM, 1 vCPU = **GRATUIT**

## 🐛 Dépannage

### Service ne démarre pas

```bash
# Voir les logs
gcloud run services logs read zed-collab

# Vérifier la configuration
gcloud run services describe zed-collab
```

### Erreurs de connexion à la base de données

```bash
# Vérifier les secrets
gcloud secrets versions access latest --secret=database-url

# Tester la connexion
gcloud run services update zed-collab --update-env-vars DATABASE_URL=...
```

### Cold start trop long

```bash
# Garder au moins 1 instance active
gcloud run services update zed-collab --min-instances 1
```

## 🔄 Migration depuis Render

1. **Exporter les variables d'environnement** depuis Render
2. **Builder l'image** et pousser vers GCR
3. **Créer la base PostgreSQL** (Cloud SQL ou Supabase)
4. **Déployer sur Cloud Run** avec les variables
5. **Mettre à jour le DNS** si domaine personnalisé

## ✅ Avantages vs Render

| Feature | Cloud Run | Render |
|---------|-----------|--------|
| Pricing | Gratuit (2M req) | $7-25/mois |
| Scaling | Auto (0-10 instances) | Manuel |
| Timeout | 60 min | Illimité |
| Cold start | ~5-10s | Instantané |
| Build | Externe | Intégré |

## 🎯 Conclusion

Google Cloud Run est excellent pour :
- ✅ Déploiement serverless gratuit
- ✅ Scaling automatique
- ✅ Pay-as-you-go après limites gratuites

Idéal pour :
- Applications avec trafic variable
- Développement/test
- Production avec trafic modéré (< 2M req/mois)

