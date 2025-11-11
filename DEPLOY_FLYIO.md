# 🪂 Déploiement sur Fly.io

Fly.io est excellent pour les applications Rust, avec un build illimité et un pricing flexible.

## ✅ Avantages de Fly.io

- **Build illimité** : Pas de limite de RAM pour la compilation
- **Gratuit pour commencer** : 3 VMs gratuites (256 MB RAM)
- **Rust-friendly** : Optimisé pour Rust
- **Global** : Déploiement multi-régions
- **Auto-scaling** : Scaling automatique intelligent
- **Pricing flexible** : Payez seulement ce que vous utilisez

## 📋 Prérequis

- Compte GitHub
- Compte Fly.io (gratuit) : [fly.io](https://fly.io)
- CLI Fly.io installé

## 🚀 Installation du CLI

### Windows (PowerShell)
```powershell
iwr https://fly.io/install.ps1 -useb | iex
```

### macOS/Linux
```bash
curl -L https://fly.io/install.sh | sh
```

Vérifiez l'installation :
```bash
fly version
```

## 🚀 Déploiement Étape par Étape

### Étape 1 : Créer un compte Fly.io

1. Allez sur [fly.io](https://fly.io)
2. Cliquez sur **"Sign Up"**
3. Connectez-vous avec **GitHub**

### Étape 2 : Se connecter via CLI

```bash
fly auth login
```

Cela ouvrira votre navigateur pour l'authentification.

### Étape 3 : Initialiser le projet

Dans le répertoire du projet :

```bash
fly launch
```

Le CLI va :
1. Détecter le Dockerfile
2. Demander un nom pour l'app (ex: `zed-collab`)
3. Demander une région (ex: `iad` pour Washington DC)
4. Créer un fichier `fly.toml`

### Étape 4 : Configurer fly.toml

Le fichier `fly.toml` sera créé. Modifiez-le :

```toml
app = "zed-collab"
primary_region = "iad"

[build]
  dockerfile = "Dockerfile.render"

[env]
  HTTP_PORT = "8080"
  ZED_ENVIRONMENT = "production"
  RUST_LOG = "info"
  LOG_JSON = "true"
  RUST_BACKTRACE = "1"

[http_service]
  internal_port = 8080
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[services]]
  protocol = "tcp"
  internal_port = 8080
  processes = ["app"]

  [[services.ports]]
    port = 80
    handlers = ["http"]
    force_https = true

  [[services.ports]]
    port = 443
    handlers = ["tls", "http"]

  [[services.http_checks]]
    interval = "10s"
    timeout = "2s"
    grace_period = "5s"
    method = "GET"
    path = "/healthz"
```

### Étape 5 : Configurer les variables d'environnement

#### Variables sensibles (secrets)

```bash
# Base de données PostgreSQL
fly secrets set DATABASE_URL="postgresql://user:pass@host:port/db"
fly secrets set DATABASE_MAX_CONNECTIONS="10"

# Base de données LLM (optionnelle)
fly secrets set LLM_DATABASE_URL="postgresql://user:pass@host:port/db"
fly secrets set LLM_DATABASE_MAX_CONNECTIONS="5"

# API Token
fly secrets set API_TOKEN="votre-token-securise"

# LiveKit (optionnel)
fly secrets set LIVEKIT_SERVER="votre-serveur"
fly secrets set LIVEKIT_KEY="votre-cle"
fly secrets set LIVEKIT_SECRET="votre-secret"

# Blob Store (optionnel)
fly secrets set BLOB_STORE_URL="votre-url"
fly secrets set BLOB_STORE_REGION="votre-region"
fly secrets set BLOB_STORE_ACCESS_KEY="votre-key"
fly secrets set BLOB_STORE_SECRET_KEY="votre-secret"
fly secrets set BLOB_STORE_BUCKET="votre-bucket"

# Clés API LLM (optionnelles)
fly secrets set OPENAI_API_KEY="votre-cle"
fly secrets set ANTHROPIC_API_KEY="votre-cle"
fly secrets set GOOGLE_AI_API_KEY="votre-cle"

# Autres
fly secrets set LLM_API_SECRET="votre-secret"
fly secrets set AUTO_JOIN_CHANNEL_ID="votre-channel-id"
fly secrets set ZED_CLIENT_CHECKSUM_SEED="votre-seed"
```

#### Variables publiques (dans fly.toml)

```toml
[env]
  HTTP_PORT = "8080"
  ZED_ENVIRONMENT = "production"
  RUST_LOG = "info"
  LOG_JSON = "true"
  RUST_BACKTRACE = "1"
  INVITE_LINK_PREFIX = "https://votre-domaine.com/invites/"
```

### Étape 6 : Créer la base de données PostgreSQL

Fly.io propose PostgreSQL managé :

```bash
fly postgres create --name zed-database --region iad --vm-size shared-cpu-1x --volume-size 10
```

Cela créera une base PostgreSQL et injectera automatiquement `DATABASE_URL`.

Pour une base LLM séparée :
```bash
fly postgres create --name zed-llm-database --region iad --vm-size shared-cpu-1x --volume-size 10
```

Attacher la base à votre app :
```bash
fly postgres attach zed-database --app zed-collab
```

### Étape 7 : Configurer les ressources

Dans `fly.toml`, configurez les ressources :

```toml
[compute]
  memory_mb = 2048  # 2 GB RAM (minimum recommandé)
  cpu_kind = "shared"
  cpus = 1
```

Pour plus de RAM :
```bash
fly scale memory 4096  # 4 GB RAM
```

### Étape 8 : Déployer

```bash
fly deploy
```

Cela va :
1. Builder l'image Docker
2. La pousser vers Fly.io
3. Déployer l'application

### Étape 9 : Vérifier le déploiement

```bash
# Voir les logs
fly logs

# Voir le statut
fly status

# Ouvrir l'app
fly open
```

## 🔧 Configuration Avancée

### Multi-régions

Déployer dans plusieurs régions :

```bash
fly regions add fra  # Frankfurt
fly regions add sin  # Singapore
```

### Auto-scaling

```toml
[http_service]
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  max_machines_running = 5
```

### Health checks

```toml
[[services.http_checks]]
  interval = "10s"
  timeout = "2s"
  grace_period = "5s"
  method = "GET"
  path = "/healthz"
```

## 📊 Monitoring

```bash
# Logs en temps réel
fly logs

# Métriques
fly metrics

# Status
fly status
```

Dans le dashboard Fly.io :
- Logs
- Métriques (CPU, RAM, Network)
- Machines
- Scaling

## 💰 Pricing

### Gratuit
- 3 VMs partagées (256 MB RAM chacune)
- 160 GB sortie/mois
- Parfait pour tester

### Payant (Pay-as-you-go)
- **RAM** : $0.0000001/GB-seconde
- **CPU** : $0.0000001/vCPU-seconde
- **Stockage** : $0.15/GB/mois
- **Réseau** : $0.02/GB après 160 GB gratuits

**Exemple** : 2 GB RAM, 1 vCPU, 24/7 = ~$5-10/mois

### PostgreSQL
- **Starter** : $1.94/mois (256 MB RAM, 1 GB stockage)
- **Standard** : $5.83/mois (1 GB RAM, 10 GB stockage)

## 🐛 Dépannage

### Build échoue

```bash
# Voir les logs de build
fly logs --build

# Builder localement pour debug
docker build -f Dockerfile.render -t zed-collab .
```

### Service ne démarre pas

```bash
# Voir les logs
fly logs

# Vérifier les secrets
fly secrets list

# Vérifier le status
fly status
```

### Erreurs de connexion à la base de données

```bash
# Vérifier la connexion PostgreSQL
fly postgres connect -a zed-database

# Vérifier les variables d'environnement
fly ssh console -C env | grep DATABASE
```

## 🔄 Migration depuis Render

1. **Exporter les variables d'environnement** depuis Render
2. **Installer Fly.io CLI** et se connecter
3. **Créer l'app** avec `fly launch`
4. **Créer PostgreSQL** avec `fly postgres create`
5. **Configurer les secrets** avec `fly secrets set`
6. **Déployer** avec `fly deploy`
7. **Mettre à jour le DNS** si domaine personnalisé

## ✅ Avantages vs Render

| Feature | Fly.io | Render |
|---------|--------|--------|
| RAM Build | Illimité | 2-4 GB |
| RAM Runtime | 256 MB - 8 GB | 512 MB - 4 GB |
| Pricing | Pay-as-you-go | $7-25/mois |
| Multi-régions | ✅ | ❌ |
| Auto-scaling | ✅ | ⚠️ |
| Rust-optimized | ✅ | ❌ |

## 🎯 Conclusion

Fly.io est excellent pour Rust avec :
- ✅ Build illimité (pas de limite de RAM)
- ✅ Gratuit pour commencer
- ✅ Multi-régions
- ✅ Auto-scaling
- ✅ Pricing flexible

Plus complexe que Railway/Render, mais plus puissant.

