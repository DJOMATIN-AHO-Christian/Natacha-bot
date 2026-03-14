# NATACHA - Next-gen Automated TAsk & Calendar Helper with AI

NATACHA est votre assistant personnel intelligent propulsé par [n8n](https://n8n.io/) et **Google Gemini**.

## Prérequis
- [Docker & Docker Compose](https://docs.docker.com/compose/install/) installés sur votre machine.
- Une clé API [Google Gemini](https://aistudio.google.com/).
- Des comptes Slack et Telegram (ou Google Tasks) pour les notifications et la gestion des tâches.
- Des identifiants Google Cloud Console (OAuth 2.0) pour connecter Gmail, Calendar et Tasks.

## Démarrage rapide

### 1. Lancer n8n via Docker
Dans le dossier racine du projet, démarrez le conteneur n8n :
```bash
docker-compose up -d
```
Cela démarrera **n8n** sur le port local : [http://localhost:5678](http://localhost:5678)

### 2. Configurer n8n
- Ouvrez n8n dans votre navigateur : [http://localhost:5678](http://localhost:5678).
- Importez les fichiers de workflows situés dans le dossier `workflows/` (Menu du workflow > Import from File).
- Vous devrez configurer ou créer des *Credentials* (identifiants) pour chaque nœud externe (Google, Slack, Telegram).
- Activez les workflows avec le bouton en haut à droite.

### 3. Configuration des Identifiants (Credentials)

#### A. Google Gemini (Intelligence Artificielle)
C'est le cerveau de NATACHA.
1. Sur le nœud nommé "Google Gemini", double-cliquez pour l'ouvrir.
2. Dans "Credential for Google Gemini API", créez un nouvel identifiant.
3. Collez simplement votre clé API secrète (obtenue sur Google AI Studio) et sauvegardez.

#### B. Google (Gmail, Calendar, Tasks) - OAuth2
Pour lier vos comptes Google, créez une application OAuth sur la Google Cloud Console :
1. Allez sur [Google Cloud Console](https://console.cloud.google.com/).
2. Créez un projet et activez **Gmail API**, **Google Calendar API**, et **Google Tasks API**.
3. Allez dans **Écran de consentement OAuth** (configurez-le en "Externe" et ajoutez votre email en utilisateur test).
4. Allez dans **Identifiants > Créer des identifiants > ID client OAuth**. Choisissez "Application Web". 
5. En "URI de redirection autorisés", ajoutez l'URL de callback de votre n8n local : `http://localhost:5678/rest/oauth2-credential/callback`
6. Dans **n8n**, créez un nouveau credential "Google OAuth2 API".
7. Copiez-y le **Client ID** et le **Client Secret**.
8. **IMPORTANT** : N'oubliez pas de renseigner le champ **Scope** selon l'API ciblée :
   - Pour Gmail : `https://www.googleapis.com/auth/gmail.readonly`
   - Pour le Calendrier : `https://www.googleapis.com/auth/calendar.readonly`
   - Pour Tâches : `https://www.googleapis.com/auth/tasks`
9. Cliquez sur "Sign in with Google".

#### C. Slack / Telegram (Notifications)
- **Slack** : Créez une app sur `api.slack.com/apps`, ajoutez le scope OAuth `chat:write`, installez-la sur votre Workspace et copiez le "Bot User OAuth Token" (xoxb-...) dans un credential n8n "Slack API".
- **Telegram** : Parlez au bot `@BotFather` sur l'app, tapez `/newbot`, obtenez le Token et collez-le dans un credential n8n "Telegram API".

## Architecture
Le projet entier est orchestré nativement dans n8n.
1. **Déclencheurs (Triggers)** : n8n écoute l'arrivée d'un nouvel e-mail ou déclenche le flux à heure fixe.
2. **Analyse (Google Gemini Node)** : Le texte brut est envoyé au modèle Gemini 1.5 Flash pour être résumé et évalué sur une échelle d'importance de 1 à 10.
3. **Priorisation (Code JS Node)** : Un algorithme mathématique sur mesure en JavaScript croise l'importance sémantique de l'IA avec la métrique temporelle (le délai disponible) pour générer un score d'urgence final sur 100.
4. **Action (If condition)** : Si c'est urgent (> 75/100), une alerte est poussée sur Slack/Telegram. Sinon, c'est mis de côté dans Google Tasks pour plus tard.
