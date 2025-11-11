# Déploiement Rapide sur Render.com

## 🚀 Déploiement en 5 minutes

### 1. Préparer le repository

```bash
# Assurez-vous que les fichiers sont commités
git add render.yaml Dockerfile.render DEPLOY_RENDER.md .dockerignore
git commit -m "Add Render.com deployment"
git push origin main
```

### 2. Créer un compte Render

1. Allez sur [render.com](https://render.com)
2. Créez un compte (gratuit)
3. Connectez votre compte GitHub

### 3. Déployer via Blueprint

1. Dans Render Dashboard : **"New +"** → **"Blueprint"**
2. Sélectionnez votre repository GitHub
3. Render détectera automatiquement `render.yaml`
4. Cliquez sur **"Apply"**

### 4. Configurer les variables d'environnement

Render créera automatiquement :
- ✅ Service web `zed-collab`
- ✅ Base de données PostgreSQL `zed-database`
- ✅ Base de données LLM `zed-llm-database` (optionnelle)

**Variables OBLIGATOIRES à configurer manuellement :**

Dans le service web `zed-collab`, allez dans **"Environment"** et ajoutez :

```bash
# 1. Générer un token API sécurisé
API_TOKEN=$(openssl rand -hex 32)
# Copiez le résultat et ajoutez-le comme variable

# 2. La DATABASE_URL sera automatiquement injectée par Render
# Mais vous pouvez la vérifier dans les variables d'environnement

# 3. Si vous utilisez S3/MinIO pour le blob store
BLOB_STORE_URL=https://votre-endpoint-s3.com
BLOB_STORE_REGION=us-east-1
BLOB_STORE_ACCESS_KEY=votre-key
BLOB_STORE_SECRET_KEY=votre-secret
BLOB_STORE_BUCKET=nom-bucket
```

### 5. Attendre le déploiement

- Premier build : **10-15 minutes**
- Builds suivants : **5-10 minutes**

### 6. Vérifier

Une fois déployé, votre service sera disponible à :
```
https://zed-collab.onrender.com
```

Testez le health check :
```bash
curl https://zed-collab.onrender.com/healthz
```

## ⚙️ Configuration minimale pour démarrer

Si vous voulez juste tester sans toutes les fonctionnalités :

**Variables minimales nécessaires :**
```bash
HTTP_PORT=8080
DATABASE_URL=<auto-injecté par Render>
API_TOKEN=<générer avec: openssl rand -hex 32>
ZED_ENVIRONMENT=production
RUST_LOG=info
INVITE_LINK_PREFIX=https://zed-collab.onrender.com/invites/
```

Les autres variables (LiveKit, LLM, Blob Store) sont **optionnelles** et peuvent être ajoutées plus tard.

## 🔧 Dépannage rapide

**Le service ne démarre pas ?**
- Vérifiez les logs dans Render Dashboard
- Assurez-vous que `API_TOKEN` est configuré
- Vérifiez que `DATABASE_URL` est présent

**Erreur de build ?**
- Vérifiez que `Dockerfile.render` existe
- Le build peut prendre 15 minutes la première fois
- Vérifiez les logs de build pour plus de détails

**Base de données non accessible ?**
- Utilisez l'**Internal Database URL** (fournie automatiquement par Render)
- Vérifiez que le service web et la DB sont dans la même région

## 📝 Notes importantes

- **Plan gratuit** : Le service s'endort après 15 min d'inactivité
- **Plan payant** : Service toujours actif ($7/mois)
- Les migrations s'exécutent automatiquement au démarrage
- Les WebSocket sont supportés nativement par Render

## 🔗 Liens utiles

- [Documentation complète](./DEPLOY_RENDER.md)
- [Documentation Render](https://render.com/docs)
- [Support Render](https://community.render.com/)

