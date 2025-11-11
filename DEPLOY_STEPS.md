# 🚀 Étapes de Déploiement sur Render.com

## ✅ Étape 1 : Pousser vers GitHub

Les fichiers de déploiement sont déjà commités. Maintenant, poussez vers GitHub :

```bash
git push origin main
```

**OU** si vous voulez aussi commiter les traductions chinoises :

```bash
# Ajouter les modifications de traduction
git add crates/
git commit -m "Translate Chinese text to English"
git push origin main
```

## 📋 Étape 2 : Créer un compte Render (si pas déjà fait)

1. Allez sur **[render.com](https://render.com)**
2. Cliquez sur **"Get Started for Free"**
3. Créez un compte avec :
   - Email
   - OU connectez-vous avec GitHub (recommandé)

## 🔗 Étape 3 : Connecter GitHub à Render

1. Dans le dashboard Render, allez dans **"Account Settings"**
2. Section **"Connected Accounts"**
3. Cliquez sur **"Connect GitHub"**
4. Autorisez Render à accéder à vos repositories

## 🎯 Étape 4 : Déployer via Blueprint

1. Dans le dashboard Render, cliquez sur **"New +"** (en haut à droite)
2. Sélectionnez **"Blueprint"**
3. Vous verrez la liste de vos repositories GitHub
4. **Sélectionnez votre repository** (celui contenant `render.yaml`)
5. Render détectera automatiquement le fichier `render.yaml`
6. Cliquez sur **"Apply"**

## ⚙️ Étape 5 : Configurer les services

Render va créer automatiquement :
- ✅ **Service Web** : `zed-collab`
- ✅ **PostgreSQL Database** : `zed-database`
- ✅ **PostgreSQL Database (LLM)** : `zed-llm-database` (optionnelle)

### Configuration des variables d'environnement

1. Une fois les services créés, cliquez sur le service **`zed-collab`**
2. Allez dans l'onglet **"Environment"**
3. Ajoutez les variables suivantes :

#### Variables OBLIGATOIRES :

```bash
# Générer un token API sécurisé (exécutez dans votre terminal)
# Windows PowerShell:
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Minimum 0 -Maximum 256 }))

# Ou Linux/Mac:
openssl rand -hex 32

# Ajoutez le résultat comme:
API_TOKEN=<le-token-généré>

# Les autres variables sont déjà configurées dans render.yaml
# Mais vérifiez que DATABASE_URL est bien injectée automatiquement
```

#### Variables OPTIONNELLES (pour fonctionnalités avancées) :

```bash
# Si vous avez un service S3/MinIO pour le stockage
BLOB_STORE_URL=https://votre-endpoint-s3.com
BLOB_STORE_REGION=us-east-1
BLOB_STORE_ACCESS_KEY=votre-access-key
BLOB_STORE_SECRET_KEY=votre-secret-key
BLOB_STORE_BUCKET=nom-du-bucket

# Pour LiveKit (audio/vidéo)
LIVEKIT_SERVER=wss://votre-livekit-server.com
LIVEKIT_KEY=votre-key
LIVEKIT_SECRET=votre-secret

# Pour les fonctionnalités LLM
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_AI_API_KEY=...
```

## ⏳ Étape 6 : Attendre le déploiement

1. Le premier build prendra **10-15 minutes**
2. Vous pouvez suivre la progression dans l'onglet **"Logs"**
3. Une fois terminé, vous verrez : **"Your service is live"**

## ✅ Étape 7 : Vérifier le déploiement

Votre service sera disponible à :
```
https://zed-collab.onrender.com
```

Testez le health check :
```bash
curl https://zed-collab.onrender.com/healthz
```

Vous devriez recevoir une réponse `200 OK`.

## 🔧 Configuration minimale pour démarrer

Si vous voulez juste tester rapidement, vous n'avez besoin que de :

1. **API_TOKEN** (générer avec la commande ci-dessus)
2. **DATABASE_URL** (injectée automatiquement par Render)

Les autres variables peuvent être ajoutées plus tard.

## 📝 Notes importantes

- ⚠️ **Plan gratuit** : Le service s'endort après 15 minutes d'inactivité
- 💰 **Plan payant** : Service toujours actif ($7/mois pour le service web)
- 🔄 Les migrations de base de données s'exécutent automatiquement au démarrage
- 🌐 Les WebSocket sont supportés nativement par Render

## 🆘 Besoin d'aide ?

- Vérifiez les logs dans Render Dashboard → Votre service → "Logs"
- Consultez `DEPLOY_RENDER.md` pour plus de détails
- [Documentation Render](https://render.com/docs)
- [Support Render](https://community.render.com/)

## 🎉 Félicitations !

Une fois déployé, votre serveur de collaboration Zed sera accessible publiquement !

