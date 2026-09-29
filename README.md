# Jee's AI

Interface de chat façon ChatGPT, dans un seul fichier HTML autonome (`jees-ai.html`). Aucune installation, aucun serveur à faire tourner côté site — tu ouvres le fichier dans un navigateur et tu parles à ton propre modèle (Ollama, ou toute API compatible OpenAI).

## Démarrage rapide

1. Ouvre `jees-ai.html` avec un éditeur de texte.
2. Cherche le bloc `CONFIG` tout en haut du `<script>` :

   ```js
   const CONFIG = {
     ENDPOINT: "http://localhost:11434/v1/chat/completions",
     API_KEY:  "ollama",
     MODEL:    "richardyoung-qwen2.5-14b-instruct-abliterated-latest-64k",
     VISION:   false
   };
   ```

3. Remplace `ENDPOINT` par l'URL de ton serveur (Ollama local, tunnel Cloudflare vers Kaggle, ou toute API type OpenAI), `API_KEY` par ta clé si besoin, et `MODEL` par le nom exact du modèle tel qu'il apparaît dans `ollama list` (ou l'équivalent côté serveur).
4. Si ton modèle comprend les images (ex. `qwen2.5vl`), passe `VISION` à `true`.
5. Ouvre le fichier dans un navigateur. C'est prêt.

Ces valeurs sont les **valeurs par défaut** : elles peuvent être surchargées sans toucher au code, directement dans Paramètres (voir plus bas).

## Fonctionnalités

- **Historique des discussions** dans la barre latérale, avec renommage implicite (titre = premier message), suppression, nouvelle discussion.
- **Mode incognito** (bouton 🕶️ en haut) : la discussion en cours n'est pas enregistrée et disparaît si tu changes de discussion.
- **Thème clair / sombre / système**, bouton rapide 🌓 dans l'en-tête.
- **Réponses en streaming**, avec bouton d'arrêt, copier et régénérer.
- **Affichage du raisonnement (« thinking »)** pour les modèles qui le fournissent (DeepSeek-R1, QwQ, Qwen3, gpt-oss…) : un bloc repliable « Réflexion » apparaît au-dessus de la réponse.
- **Pièces jointes** via le bouton 📎, le glisser-déposer ou le collage :
  - Images (si `VISION: true` ou un modèle vision est sélectionné)
  - Fichiers texte et code (`.txt`, `.md`, `.py`, `.js`, `.json`, etc.)
  - **PDF** : texte extrait automatiquement (via pdf.js)
  - **Word `.docx`** : texte extrait automatiquement (via mammoth.js)
  - `.doc` (ancien format Word) : non pris en charge, un message le signale
- **Plusieurs profils serveur** (Paramètres → Serveurs) :
  - Bouton **+ Ajouter un serveur** pour créer un profil (nom, endpoint, clé, modèle)
  - Sélection du profil actif par **bouton radio**
  - Case **« Basculer automatiquement »** : si le serveur actif échoue (panne, quota, erreur réseau), le site essaie automatiquement le suivant dans la liste
- **Export de l'historique** en JSON, et suppression complète des données.

## Paramètres (⚙️)

| Réglage | Effet |
|---|---|
| Thème | Clair / sombre / système |
| Serveurs | Profils endpoint/clé/modèle, en plus du profil « Par défaut » (= `CONFIG`) |
| Basculement automatique | Essaie le profil suivant si celui actif échoue |
| Instructions système | Message système envoyé à chaque conversation |
| Température | Créativité des réponses (0 à 2) |
| Exporter l'historique | Télécharge toutes les discussions en `.json` |
| Tout supprimer | Efface l'historique et les réglages locaux |

Un champ de profil laissé vide hérite automatiquement de la valeur correspondante dans `CONFIG`.

## Prérequis côté serveur

- Une API compatible OpenAI (`/v1/chat/completions`), avec support du streaming SSE.
- **CORS activé** si le site et le serveur ne sont pas sur le même domaine. Avec Ollama :

  ```bash
  OLLAMA_ORIGINS=* ollama serve
  ```

- Si la page est servie en `https://`, l'endpoint doit aussi être en `https://` (un tunnel Cloudflare fonctionne bien pour exposer un serveur local ou Kaggle).

## Limites connues

- La clé API est visible dans le code source de la page et dans le stockage local du navigateur. Ne publie pas le fichier tel quel si ta clé doit rester secrète — passe par un petit proxy côté serveur dans ce cas.
- L'historique est stocké dans le `localStorage` du navigateur (quelques Mo). Beaucoup d'images jointes peuvent le remplir ; au-delà, les nouvelles discussions ne s'enregistrent plus.
- L'extraction de PDF et Word charge deux bibliothèques externes (pdf.js, mammoth.js) depuis un CDN au premier usage : une connexion internet est nécessaire à ce moment, même si le modèle tourne en local.
- Le basculement automatique entre serveurs n'intervient que sur un échec de connexion initial, pas si une réponse a déjà commencé à s'afficher.
