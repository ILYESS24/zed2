# ⚡ Déploiement Rapide - Web Service Render

Guide ultra-rapide pour déployer en 5 minutes.

## 🚀 Étapes rapides

### 1. Créer la base de données (2 min)

1. Render Dashboard → **"New +"** → **"PostgreSQL"**
2. Configuration :
   - Name: `zed-database`
   - Database: `zed`
   - User: `zed`
   - Plan: `Starter` (gratuit)
   - Version: `15`
3. **Créer** et **copier l'Internal Database URL**

### 2. Créer le Web Service (2 min)

1. Render Dashboard → **"New +"** → **"Web Service"**
2. Connecter le repository `zed2`
3. Configuration :
   - Name: `zed-collab`
   - Environment: `Docker`
   - Dockerfile Path: `Dockerfile.render`
   - Region: Même que la DB
   - Plan: `Starter` (gratuit) ou `Standard` ($7/mois)

### 3. Variables d'environnement (1 min)

Dans **Environment**, ajoutez uniquement :

```bash
HTTP_PORT=8080
ZED_ENVIRONMENT=production
RUST_LOG=info
DATABASE_URL=<Internal Database URL copiée à l'étape 1>
API_TOKEN=<générer: openssl rand -hex 32>
INVITE_LINK_PREFIX=https://zed-collab.onrender.com/invites/
```

### 4. Déployer

Cliquez sur **"Create Web Service"** et attendez 10-15 minutes.

### 5. Vérifier

```bash
curl https://zed-collab.onrender.com/healthz
```

✅ **C'est tout !** Votre service est en ligne.

## 💡 Astuce

Pour générer `API_TOKEN` rapidement :
- **Windows PowerShell**: 
  ```powershell
  -join ((65..90) + (97..122) + (48..57) | Get-Random -Count 32 | ForEach-Object {[char]$_})
  ```
- **Linux/Mac**: 
  ```bash
  openssl rand -hex 32
  ```

## 📚 Documentation complète

Voir `DEPLOY_WEB_SERVICE.md` pour tous les détails.

