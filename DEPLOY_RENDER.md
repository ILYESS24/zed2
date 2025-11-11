# Guide de déploiement sur Render.com

Ce guide vous explique comment déployer le serveur de collaboration Zed sur Render.com.

## Prérequis

1. Un compte Render.com (gratuit ou payant)
2. Un compte GitHub avec le repository cloné
3. (Optionnel) Un service S3-compatible pour le stockage de fichiers (MinIO, AWS S3, etc.)

## Méthode 1 : Déploiement via render.yaml (Recommandé)

### Étape 1 : Préparer le repository

1. Assurez-vous que les fichiers suivants sont dans votre repository :
   - `render.yaml`
   - `Dockerfile.render` (ou utilisez `Dockerfile-collab`)
   - Tous les fichiers du projet

2. Poussez les changements sur GitHub :
```bash
git add render.yaml Dockerfile.render
git commit -m "Add Render.com deployment configuration"
git push origin main
```

### Étape 2 : Créer les services sur Render

1. Allez sur [Render Dashboard](https://dashboard.render.com/)
2. Cliquez sur "New +" → "Blueprint"
3. Connectez votre repository GitHub
4. Sélectionnez le repository contenant `render.yaml`
5. Render détectera automatiquement la configuration

### Étape 3 : Configurer les variables d'environnement

Dans le dashboard Render, pour chaque service, configurez les variables d'environnement suivantes :

#### Variables OBLIGATOIRES :

```bash
# Base de données principale (utilisez l'Internal Database URL de la DB créée à l'étape 3)
DATABASE_URL=<Internal Database URL de zed-database>

# Token API (générer un token sécurisé)
API_TOKEN=<Générez un token aléatoire sécurisé, ex: openssl rand -hex 32>

# Blob Store (si vous utilisez S3/MinIO)
BLOB_STORE_URL=https://votre-s3-endpoint.com
BLOB_STORE_REGION=us-east-1
BLOB_STORE_ACCESS_KEY=votre-access-key
BLOB_STORE_SECRET_KEY=votre-secret-key
BLOB_STORE_BUCKET=nom-du-bucket
```

#### Variables OPTIONNELLES :

```bash
# Base de données LLM (si vous utilisez les fonctionnalités LLM)
LLM_DATABASE_URL=<URL de la base de données LLM>

# LiveKit (pour audio/vidéo)
LIVEKIT_SERVER=wss://votre-livekit-server.com
LIVEKIT_KEY=votre-key
LIVEKIT_SECRET=votre-secret

# Clés API LLM
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_AI_API_KEY=...

# Configuration
INVITE_LINK_PREFIX=https://votre-domaine.com/invites/
AUTO_JOIN_CHANNEL_ID=1
ZED_CLIENT_CHECKSUM_SEED=<générer avec: openssl rand -hex 32>
```

### Étape 4 : Déployer

1. Render va automatiquement :
   - Créer les services PostgreSQL
   - Builder l'image Docker
   - Déployer le service web
   - Configurer les variables d'environnement

2. Attendez que le déploiement soit terminé (5-10 minutes pour le premier build)

3. Votre service sera disponible à : `https://zed-collab.onrender.com`

## Méthode 2 : Déploiement manuel

### Étape 1 : Créer la base de données PostgreSQL

1. Dans Render Dashboard : "New +" → "PostgreSQL"
2. Configurez :
   - **Name**: `zed-database`
   - **Database**: `zed`
   - **User**: `zed`
   - **Region**: Choisissez la région la plus proche
   - **Plan**: Starter (gratuit) ou Standard (payant)
3. Notez l'**Internal Database URL** et **External Database URL**

### Étape 2 : Créer le service Web

1. "New +" → "Web Service"
2. Connectez votre repository GitHub
3. Configurez :
   - **Name**: `zed-collab`
   - **Environment**: `Docker`
   - **Dockerfile Path**: `Dockerfile.render` (ou `Dockerfile-collab`)
   - **Docker Context**: `.`
   - **Region**: Même région que la base de données
   - **Branch**: `main`
   - **Root Directory**: `.`
   - **Build Command**: (laissé vide, Render utilise Docker)
   - **Start Command**: (laissé vide, défini dans Dockerfile)

### Étape 3 : Configurer les variables d'environnement

Dans les settings du service web, ajoutez toutes les variables listées ci-dessus.

### Étape 4 : Configurer le Health Check

- **Health Check Path**: `/healthz`
- **Health Check Interval**: `10s`

## Configuration du Blob Store

### Option 1 : Utiliser AWS S3

1. Créez un bucket S3
2. Créez un utilisateur IAM avec accès au bucket
3. Utilisez les credentials dans les variables d'environnement

### Option 2 : Utiliser MinIO (auto-hébergé)

1. Déployez MinIO sur un serveur ou utilisez [MinIO Cloud](https://min.io/)
2. Créez un bucket
3. Configurez les variables d'environnement avec l'URL MinIO

### Option 3 : Utiliser Render Disk (limité)

⚠️ **Note**: Render Disk n'est pas compatible S3. Vous devrez adapter le code ou utiliser un service externe.

## Vérification du déploiement

1. **Vérifier les logs** :
   - Allez dans le dashboard Render
   - Cliquez sur votre service
   - Onglet "Logs"
   - Vérifiez qu'il n'y a pas d'erreurs

2. **Tester le health check** :
```bash
curl https://zed-collab.onrender.com/healthz
```

3. **Vérifier les migrations** :
   - Les migrations s'exécutent automatiquement au démarrage
   - Vérifiez dans les logs que les migrations sont appliquées

## Configuration du domaine personnalisé

1. Dans les settings du service web
2. Section "Custom Domains"
3. Ajoutez votre domaine
4. Configurez les DNS selon les instructions Render

## Mise à jour

Les mises à jour se font automatiquement quand vous poussez sur la branche `main` (ou la branche configurée).

Pour forcer un redéploy :
1. Dashboard Render → Votre service
2. Bouton "Manual Deploy" → "Deploy latest commit"

## Coûts estimés

- **Starter Plan** (gratuit) :
  - Web Service : Gratuit (avec limitations)
  - PostgreSQL : Gratuit (90 jours, puis $7/mois)
  - ⚠️ Service s'endort après 15 minutes d'inactivité

- **Standard Plan** :
  - Web Service : $7/mois
  - PostgreSQL : $7/mois
  - Service toujours actif

## Dépannage

### Le service ne démarre pas

1. Vérifiez les logs dans Render Dashboard
2. Vérifiez que toutes les variables d'environnement obligatoires sont configurées
3. Vérifiez que `DATABASE_URL` est correcte

### Erreur de connexion à la base de données

1. Utilisez l'**Internal Database URL** pour les connexions depuis Render
2. Vérifiez que le service web et la base de données sont dans la même région
3. Vérifiez les credentials de la base de données

### Build échoue

1. Vérifiez que `Dockerfile.render` est présent
2. Vérifiez les logs de build pour voir l'erreur exacte
3. Le build peut prendre 10-15 minutes la première fois

### WebSocket ne fonctionne pas

1. Render supporte les WebSocket nativement
2. Assurez-vous d'utiliser `wss://` (WebSocket Secure) pour les connexions
3. Vérifiez que le port 8080 est bien exposé

## Support

- [Documentation Render](https://render.com/docs)
- [Render Community](https://community.render.com/)
- [Issues GitHub Zed](https://github.com/zed-industries/zed/issues)

