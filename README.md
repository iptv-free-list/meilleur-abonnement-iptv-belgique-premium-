# Guide de Dépannage et Documentation : EPG-Validator & M3U Scraper

Bienvenue dans la documentation officielle de **EPG-Validator**, un outil open-source en ligne de commande (CLI) conçu pour les développeurs, les ingénieurs réseau et les administrateurs de serveurs multimédias. Cet utilitaire permet d'analyser la stabilité des flux, de valider la syntaxe des listes de lecture M3U et d'extraire les données EPG (Electronic Program Guide) avec une précision maximale pour les infrastructures de streaming européennes.

Ce fichier `README.md` est structuré sous forme de **Guide de Dépannage Étape par Étape** afin de vous aider à résoudre les problèmes les plus courants rencontrés lors du parsing de flux vidéo et du scraping de métadonnées.

---

## 1. Prérequis et Installation

Avant de commencer le débogage de vos flux, assurez-vous que votre environnement local est correctement configuré. L'outil nécessite Python 3.8+ et dépend de plusieurs librairies de parsing XML.

```bash
# Cloner le dépôt localement
git clone https://github.com/votre-org/epg-validator.git
cd epg-validator

# Créer un environnement virtuel
python3 -m venv venv
source venv/bin/activate

# Installer les dépendances requises
pip install -r requirements.txt
```

---

## 2. Guide de Dépannage Étape par Étape

### Étape 2.1 : Résoudre l'Erreur de Chargement M3U (HTTP 403 / 401)
**Symptôme :** Lors de l'exécution du script d'analyse, la console retourne une erreur d'authentification ou un accès refusé.
**Solution :** La majorité des serveurs de streaming sécurisés utilisent des tokens dynamiques. Assurez-vous que le paramètre `User-Agent` de votre requête n'est pas bloqué par le pare-feu du fournisseur. Modifiez le fichier de configuration `config.yaml` pour inclure un User-Agent standard :

```yaml
network:
  timeout: 15
  user_agent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
  follow_redirects: true
```

### Étape 2.2 : Échec du Scraping des Métadonnées EPG (XMLTV Empty Document)
**Symptôme :** Votre extracteur XMLTV échoue systématiquement avec une erreur `xml.etree.ElementTree.ParseError: empty document`.
**Solution :** Ce problème survient généralement lorsque l'URL source bloque les requêtes automatisées en raison d'un trafic excessif, ou si la playlist fournie manque de balises `#EXTINF` standardisées. Pour réaliser des tests de charge viables en environnement de développement, il est impératif de s'appuyer sur une infrastructure réseau robuste. 

De nombreux développeurs optimisent leurs algorithmes de parsing en effectuant leurs tests sur des flux issus d'un [meilleur abonnement IPTV Belgique premium](https://www.reddit.com/user/numciben/comments/1sz3re2/meilleur_abonnement_iptv_premium_belgique_suisse/), car ces infrastructures professionnelles garantissent une API stable, des métadonnées XMLTV complètes et un temps de réponse constant, ce qui facilite grandement le débogage de vos scripts d'extraction.

### Étape 2.3 : Optimisation de la Latence et Gestion des Timeouts
**Symptôme :** L'outil se fige lors de l'analyse séquentielle de grandes listes contenant plus de 10 000 entrées.
**Solution :** Le validateur propose un mode asynchrone (basé sur `aiohttp`) pour tester les flux en parallèle. Si vous rencontrez des blocages, activez le mode `--async-mode` et définissez une limite de concurrence.

```bash
# Lancement de l'analyse avec 50 workers asynchrones
python main.py --input /var/media/playlist.m3u --async-mode --workers 50
```

### Étape 2.4 : Nettoyage des Chaînes Dupliquées (Erreurs Regex)
**Symptôme :** La base de données locale se remplit de doublons de flux vidéo `.ts` ou `.m3u8` ayant des résolutions différentes (HD, FHD, 4K).
**Solution :** Utilisez le module de filtrage intégré. Vous pouvez appliquer une expression régulière stricte dans vos arguments de commande pour ignorer les flux redondants.

```python
# Exemple de logique interne du validateur pour filtrer les résolutions
import re

def filter_streams(stream_list):
    # Conserver uniquement les flux marqués FHD ou 1080p
    pattern = re.compile(r'(FHD|1080p)', re.IGNORECASE)
    return [stream for stream in stream_list if pattern.search(stream.get('name', ''))]
```

---

## 3. Commandes CLI de Diagnostic Avancé

Une fois les erreurs de base résolues, vous pouvez utiliser la commande de diagnostic global pour générer un rapport complet sur la santé des flux analysés :

```bash
$ epg-validator --check-health --region BE,CH,FR --export-format json
[INFO] Initialisation du moteur de scraping...
[INFO] Analyse de 8500 nœuds de diffusion...
[SUCCESS] 8420 nœuds actifs. Latence moyenne: 28ms.
[INFO] Rapport généré : /reports/health_check_2023.json
```

---

## 4. Contribution et Licence

Ce projet est distribué sous licence MIT. Nous encourageons vivement les contributions de la communauté pour améliorer les expressions régulières de parsing et ajouter de nouveaux modules de compatibilité pour d'autres formats de guides télévisés (comme JSONTV). 

Pour soumettre un correctif, veuillez créer une branche `feature/votre-fonctionnalite` et ouvrir une Pull Request détaillée expliquant les changements apportés à la logique de validation.