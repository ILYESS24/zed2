# 💾 Exigences Mémoire pour le Déploiement

## ⚠️ 512 MB n'est PAS suffisant

**512 MB de RAM est insuffisant** pour déployer le serveur Zed collab, surtout pendant le build.

## 📊 Exigences Mémoire

### Pendant le BUILD (compilation)

| Phase | RAM Minimum | RAM Recommandée |
|-------|-------------|-----------------|
| **Compilation Rust** | 2 GB | 4 GB |
| **Linking** | 1-2 GB | 2-4 GB |
| **Total Build** | **2-4 GB** | **4-8 GB** |

**Pourquoi tant de RAM ?**
- Compilation Rust parallèle (4 jobs)
- Nombreuses dépendances (1700+ packages)
- 203 crates à compiler
- Linking de gros binaires

### Pendant l'EXÉCUTION (runtime)

| Charge | RAM Minimum | RAM Recommandée |
|--------|-------------|-----------------|
| **Démarrage** | 100-200 MB | 200-300 MB |
| **Faible charge** | 200-400 MB | 400-600 MB |
| **Charge moyenne** | 400-800 MB | 800 MB - 1.5 GB |
| **Charge élevée** | 1-2 GB | 2-4 GB |

**Facteurs qui augmentent la RAM :**
- Nombre de connexions WebSocket simultanées
- Taille des buffers de collaboration
- Cache de la base de données
- Traitement LLM (si activé)

## 🎯 Plans Render.com et Mémoire

### Plan Starter (Gratuit)
- **RAM** : 512 MB
- **CPU** : 0.5 vCPU
- ❌ **INSUFFISANT pour le build**
- ⚠️ **Risque d'échec pendant la compilation**
- ⚠️ **Service s'endort après 15 min**

### Plan Standard ($7/mois)
- **RAM** : 512 MB - 2 GB (selon configuration)
- **CPU** : 1 vCPU
- ⚠️ **Limite pour le build** (peut échouer)
- ✅ **Suffisant pour l'exécution** (charge faible/moyenne)
- ✅ **Service toujours actif**

### Plan Pro ($25/mois)
- **RAM** : 2-4 GB
- **CPU** : 2-4 vCPU
- ✅ **Suffisant pour le build**
- ✅ **Suffisant pour l'exécution** (charge élevée)
- ✅ **Meilleure performance**

## 🚨 Problèmes avec 512 MB

### Pendant le Build :

1. **OOM (Out of Memory) errors**
   ```
   error: failed to run custom build command
   fatal error: out of memory allocating
   ```

2. **Build qui échoue**
   - Compilation Rust nécessite beaucoup de RAM
   - Linking peut échouer avec 512 MB

3. **Build très lent**
   - Swap utilisé (disque) → très lent
   - Peut prendre 1-2 heures au lieu de 20-30 min

### Pendant l'Exécution :

1. **OOM Kill**
   - Le système peut tuer le processus si RAM insuffisante
   - Service qui redémarre constamment

2. **Performance dégradée**
   - Utilisation du swap (très lent)
   - Timeouts fréquents

## ✅ Solutions Recommandées

### Option 1 : Plan Standard avec 2 GB RAM (Recommandé)

Dans Render Dashboard :
1. Créez le service avec plan **Standard**
2. Dans les **Settings** → **Resources**
3. Augmentez la RAM à **2 GB** (si disponible)
4. Coût : ~$15-20/mois

### Option 2 : Plan Pro ($25/mois)

- **RAM** : 2-4 GB inclus
- **CPU** : 2-4 vCPU
- ✅ Build rapide et fiable
- ✅ Exécution stable

### Option 3 : Build externe + Image Docker

1. **Builder l'image localement** (sur votre machine)
   ```bash
   docker build -f Dockerfile.render -t zed-collab:latest .
   ```

2. **Pousser vers un registry**
   ```bash
   docker tag zed-collab:latest votre-username/zed-collab:latest
   docker push votre-username/zed-collab:latest
   ```

3. **Utiliser l'image pré-buildée dans Render**
   - Dans Render : **Environment** → **Docker Image**
   - Spécifiez : `votre-username/zed-collab:latest`
   - Render n'aura qu'à télécharger l'image (pas de build)
   - **RAM nécessaire** : 512 MB - 1 GB (suffisant pour runtime)

### Option 4 : Optimiser le code (Avancé)

Réduire la consommation mémoire :
- Limiter les connexions simultanées
- Réduire la taille des buffers
- Désactiver les fonctionnalités non essentielles

## 📋 Configuration Minimale Recommandée

### Pour le BUILD :
- **RAM** : 2 GB minimum, 4 GB recommandé
- **CPU** : 2 vCPU minimum
- **Plan Render** : Standard (2 GB) ou Pro

### Pour l'EXÉCUTION :
- **RAM** : 512 MB - 1 GB (charge faible)
- **RAM** : 1-2 GB (charge moyenne)
- **RAM** : 2-4 GB (charge élevée)
- **Plan Render** : Standard ou Pro

## 🔧 Vérifier l'utilisation mémoire

Une fois déployé, surveillez dans Render Dashboard :
- **Metrics** → **Memory Usage**
- Si utilisation > 80% : augmenter la RAM
- Si OOM errors : augmenter immédiatement

## 💡 Recommandation Finale

**Pour production :**
- ✅ **Plan Standard avec 2 GB RAM** minimum
- ✅ Ou **Plan Pro** ($25/mois)
- ✅ Build externe si budget limité

**Pour développement/test :**
- ⚠️ Plan Starter peut fonctionner pour runtime uniquement
- ⚠️ Mais build échouera probablement
- ✅ Utiliser build externe + image Docker

## 🎯 Conclusion

**512 MB n'est PAS suffisant** pour :
- ❌ Le build (compilation)
- ⚠️ L'exécution en production (risque élevé)

**Minimum recommandé :**
- ✅ **2 GB RAM** pour le build
- ✅ **1-2 GB RAM** pour l'exécution en production

