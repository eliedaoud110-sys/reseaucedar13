# Réseau Cedar — version reconstruite

Cette version utilise Firebase Authentication, Cloud Firestore et Firebase Storage. Elle remplace l'ancienne démo locale qui permettait de choisir soi-même un rôle.

## Fonctions incluses
- Connexion par e-mail et mot de passe.
- Rôle attribué dans Firestore; les nouveaux comptes restent `pending` jusqu'à autorisation.
- Enseignants autorisés et administrateurs peuvent publier du texte et joindre des PDF, présentations, documents, images et MP3.
- Les MP3 s'affichent avec un lecteur audio.
- Les images s'affichent dans la publication; les autres fichiers s'ouvrent via un lien de téléchargement.
- Seul l'administrateur peut envoyer un lien externe, enregistrer un message vocal et supprimer une publication pour tous.
- Les suppressions de publications sont synchronisées avec tous les clients grâce à Firestore.

## A. Créer/choisir le projet Firebase
1. Ouvrez https://console.firebase.google.com/ et sélectionnez le projet Réseau Cedar.
2. Dans **Authentication → Sign-in method**, activez **Email/Password**.
3. Dans **Project settings → General → Your apps**, ajoutez une application Web si nécessaire et copiez sa configuration.
4. Dans **Build → Firestore Database**, créez la base de données.
5. Dans **Build → Storage**, configurez Firebase Storage. Si Firebase vous demande d'activer la facturation, vérifiez les conditions et tarifs affichés avant de continuer.

## B. Définir le premier administrateur (important)
1. Dans **Authentication → Users**, créez votre propre compte avec votre e-mail.
2. Copiez son **UID** exact.
3. Ouvrez `firebase-config.js`. Remplacez `REMPLACER_PAR_UID_ADMIN` par cet UID et remplacez les valeurs de configuration Firebase par celles de votre application Web.
4. Ouvrez `firestore.rules` et `storage.rules`. Remplacez aussi leur valeur `REMPLACER_PAR_UID_ADMIN` par le même UID.
5. Publiez les règles Firestore dans **Firestore Database → Rules**, puis publiez les règles Storage dans **Storage → Rules**.

**Ne donnez jamais le rôle admin à un utilisateur non fiable.** Les règles Firebase, et non le simple affichage de l'interface, contrôlent les opérations sensibles.

## C. Héberger le site
Les imports JavaScript utilisent des modules ES; n'ouvrez pas `index.html` comme un fichier local `file://`.
1. Installez Firebase CLI sur un ordinateur si vous l'avez, ou utilisez Firebase Hosting via Cloud Shell.
2. Déployez `index.html`, `app.js` et `firebase-config.js` sur Firebase Hosting.
3. Ouvrez l'URL HTTPS de votre site et connectez-vous.

Pour Firebase Hosting avec CLI, depuis ce dossier:
```bash
npm install -g firebase-tools
firebase login
firebase init hosting
# choisissez le projet existant et le dossier courant comme dossier public
firebase deploy --only hosting
```
Répondez **No** à la question qui propose de remplacer `index.html`.

## D. Autoriser les comptes
1. Chaque personne crée un compte dans l'écran de connexion ou l'admin le crée dans Authentication.
2. Les nouveaux utilisateurs ont le rôle `pending` et ne peuvent pas publier.
3. Dans Authentication, copiez l'UID de l'enseignant.
4. Connectez-vous avec le compte admin, ouvrez **Administration**, saisissez l'UID, choisissez `enseignant`, puis enregistrez.
5. L'utilisateur doit actualiser la page ou se reconnecter.

Les rôles disponibles sont `enseignant`, `eleve`, `parent` et `admin`. L'administrateur initial est reconnu par son UID défini dans la configuration et les règles.

## Types de fichiers
Fichiers acceptés par l'interface: PDF, PPT/PPTX, DOC/DOCX, ODT/ODP, XLS/XLSX, JPG/JPEG, PNG, GIF, WEBP et MP3. Limite de l'interface: 20 Mo par fichier. Ajustez les règles Storage seulement si vous souhaitez modifier cette limite.

## Notes importantes / limites de cette première version
- Pour les comptes élèves et parents, les messages peuvent être lus; seuls enseignants/admins peuvent publier.
- Les classes et destinataires sont une première base de démonstration. Les comptes peuvent recevoir un champ `className` dans `users/{uid}` pour filtrer les publications par classe.
- Le lien externe et l'enregistrement vocal sont visibles uniquement dans l'interface admin; les règles Firestore empêchent les enseignants d'enregistrer ces champs. Pour une école en production, envisagez également de valider les métadonnées et le nettoyage des fichiers avec une Cloud Function.
- Les fichiers et les règles Firebase sont des éléments séparés: après toute modification des règles, cliquez sur **Publish** dans la console Firebase.
- Ne mettez jamais de clé privée Admin SDK ou de mot de passe dans les fichiers du site.
