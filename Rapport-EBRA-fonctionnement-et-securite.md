# EBRA — Fonctionnement, circulation des données et sécurité

**Note destinée aux utilisateurs, responsables métier et équipes techniques**  
**Établie par : Mohamed Amine**  
**Date : 1er octobre 2026**  
**Version examinée : archive EBRA fournie pour cette vérification**

## Synthèse

EBRA comprend deux interfaces, **Press Service** et **Manager**, lancées sur le poste de l’utilisateur. Elles consultent et mettent à jour les données métier sur les emplacements configurés, proposent des fonctions de messagerie via Outlook et intègrent un studio de retouche d’image, Photoshop/Magic.

L’examen des scripts fournis ne montre **aucun mécanisme d’envoi automatique des images à un service d’IA externe**. Magic traite la photo dans le navigateur à l’aide de fichiers installés avec EBRA. Les deux serveurs de l’application écoutent sur le poste local : ils ne publient pas directement leur interface sur Internet ou sur le réseau de l’entreprise.

EBRA utilise cependant, par sa fonction métier, **les dossiers partagés et Outlook**. Il serait donc inexact de dire que *toutes* les données restent exclusivement sur le poste ou que l’application présente « zéro risque ». Les échanges de messagerie suivent le fonctionnement normal d’Outlook ; la destination effective d’un message dépend de la boîte et de l’action de l’utilisateur. Les principaux points de sécurité relevés concernent l’ouverture de pièces jointes et la protection des fichiers de l’application sur le partage.

## 1. Périmètre et méthode

Cette note couvre les scripts et interfaces des répertoires `EBRA-Press-Service` et `EBRA-Manager` de l’archive transmise : serveurs PowerShell, pages et scripts JavaScript, intégration Outlook et studio Photoshop/Magic.

Il s’agit d’une **revue du code fourni**. Nous n’avons pas capturé le trafic réseau sur les postes, vérifié les permissions Windows des partages, testé l’infrastructure de messagerie ni analysé le contenu binaire des composants tiers. Les conclusions ci-dessous distinguent donc ce que le code démontre de ce qui demanderait une vérification en environnement réel.

## 2. Comment l’application travaille

| Composant | Rôle | Données concernées |
|---|---|---|
| Interface Press Service | Prise en charge, suivi et modification des traitements | Fichiers JSON métier configurés |
| Interface Manager | Consolidation, validation, suivi d’équipe et configuration des sources | JSON métier et journaux |
| Serveurs locaux PowerShell | Fournissent les pages et les API aux navigateurs du même poste | Données lues ou modifiées par l’utilisateur Windows |
| Messagerie Outlook | Consulte les boîtes autorisées, ouvre une réponse ou un transfert, déplace des messages et ouvre des pièces jointes | E-mails et pièces jointes |
| Photoshop/Magic | Importe, retouche, détoure et exporte des images | Photos choisies par l’utilisateur et fichiers du modèle local |

Les interfaces utilisent respectivement les adresses locales `127.0.0.1:8765` et `127.0.0.1:8766`. Les serveurs PowerShell sont configurés pour écouter sur l’interface **Loopback**, accessible depuis le poste lui-même. Les fichiers métier situés sur un partage réseau sont naturellement lus ou modifiés par le réseau interne lorsque l’utilisateur utilise les fonctions concernées.

## 3. Où vont les images dans Photoshop/Magic ?

L’utilisateur sélectionne une image. `photo375.js` la lit avec les fonctions locales du navigateur, la place dans une zone de travail graphique et applique les retouches. Pour détecter une personne, il charge le modèle `selfie_segmenter.tflite` et les composants associés depuis le dossier `/ai/` **servi par le serveur local EBRA**. L’export crée un fichier à télécharger sur le poste.

Dans ce parcours examiné, `photo375.js` ne contient pas de requête d’envoi de l’image vers une API externe. Le terme commercial **Magic** désigne ici une vraie segmentation automatique exécutée localement. Cette conclusion concerne l’édition d’image : elle ne décrit pas les communications normales d’Outlook ou d’autres applications présentes sur le poste.

Un projet Photoshop exporté au format JSON peut contenir l’image intégrée dans le fichier. Comme tout document de travail, il doit être conservé dans un emplacement approprié si la photo est sensible.

## 4. Circulation des autres données

### Traitements et dossiers partagés

Les deux serveurs lisent les fichiers métier prévus dans leur configuration. Le Manager peut aussi enregistrer des changements, des validations et des événements de suivi. Ces accès utilisent les droits de la session Windows sur les emplacements configurés. La présence de chemins réseau ne signifie pas un envoi vers Internet : il s’agit des échanges avec les ressources internes que l’application est conçue pour utiliser.

### Outlook

EBRA s’appuie sur l’instance locale d’Outlook pour afficher des messages, ouvrir les fenêtres de réponse ou de transfert et déplacer des e-mails. Le code examiné ouvre ces fenêtres dans Outlook ; il ne constitue pas une API d’IA. **Un message envoyé par l’utilisateur dans Outlook peut évidemment quitter l’organisation selon son destinataire et les règles de messagerie.** Il ne serait donc pas exact d’affirmer qu’EBRA, dans son ensemble, n’a aucun lien possible avec l’extérieur.

### Internet et dépendances

Les fichiers nécessaires à Magic sont présents dans l’archive examinée. Le parcours de traitement de la photo ne nécessite pas d’appel à un service cloud. Une installation ou une mise à jour de ces fichiers, si elle est effectuée séparément depuis des sources en ligne, constitue une opération distincte du traitement quotidien des photos.

## 5. Protections constatées dans le code

- Les serveurs écoutent sur l’adresse locale du poste, et les requêtes HTTP vérifient le nom d’hôte attendu.
- Les fichiers servis au navigateur sont explicitement énumérés ; le serveur ne propose pas de navigation générale dans les dossiers.
- Les actions POST vérifient l’origine de la page et un en-tête spécifique à EBRA.
- Les réponses indiquent au navigateur de ne pas conserver les données en cache et incluent des restrictions de chargement de contenu.
- La taille des requêtes est limitée, plusieurs champs métier sont validés, et le Manager limite les modifications aux sources configurées.

Ces dispositions réduisent certaines erreurs et expositions. Elles ne constituent pas à elles seules une authentification complète entre les programmes exécutés sur un même poste.

## 6. Points à maîtriser

| Point | Constat concret | Importance et suite recommandée |
|---|---|---|
| Ouverture des pièces jointes Outlook | Les scripts `outlook.ps1` enregistrent la pièce jointe, puis l’ouvrent avec `Start-Process`, sans liste de types autorisés. | **Prioritaire.** Un fichier exécutable reçu par e-mail pourrait être lancé avec les droits de l’utilisateur. Limiter l’ouverture automatique aux types de documents approuvés et prévoir une validation adaptée. |
| Intégrité des scripts sur le partage | Le comportement d’EBRA dépend des fichiers PowerShell et JavaScript déployés. | **Prioritaire à vérifier sur le terrain.** Les utilisateurs ordinaires ne doivent pas pouvoir modifier les fichiers du programme ou du modèle destinés aux autres. Les droits réels du partage et du système de fichiers n’étaient pas disponibles dans l’archive. |
| Accès aux API depuis le même poste | Les serveurs sont locaux, mais les contrôles `Origin` et l’en-tête demandé ne sont pas un mécanisme d’identité contre un autre programme local. | **À renforcer.** Un programme déjà exécuté sur le poste peut tenter de lire ou d’appeler des routes métier avec les droits de la session. Une authentification locale et une séparation des droits Manager/Agent amélioreraient la protection. |
| Sources configurables côté Manager | La configuration peut référencer des chemins Windows ou des partages réseau. | **À encadrer.** Les sources autorisées doivent correspondre au périmètre métier et être gérées par les personnes habilitées. |
| Images ou projets très volumineux | Le traitement de l’image se fait sur le poste et utilise sa mémoire. | **Disponibilité.** Une image exceptionnelle peut ralentir le navigateur ou consommer beaucoup de mémoire, sans constituer pour autant une fuite externe. |

Ces points sont des **observations techniques et mesures d’amélioration**, pas la preuve d’une intrusion ou d’une fuite constatée.

## 7. Conclusion communicable

**Dans le code EBRA examiné, Photoshop/Magic traite les photos localement : aucune transmission automatique de ces images vers un fournisseur d’IA externe n’a été constatée. Les interfaces sont servies sur le poste de l’utilisateur et ne sont pas directement exposées à Internet.**

**EBRA reste une application métier connectée aux ressources internes et à Outlook.** L’examen du code ne met pas en évidence une fuite externe automatique des photos, mais ne permet pas de certifier l’absence de tout risque pour le poste ou l’infrastructure. Pour sécuriser l’usage quotidien, les deux actions les plus utiles sont de maîtriser l’ouverture des pièces jointes et de vérifier les droits d’écriture sur les fichiers du programme partagé.

### Repères techniques pour les développeurs

- `EBRA-Press-Service/serverr.ps1` : écoute locale, routes HTTP et restrictions de réponse.
- `EBRA-Manager/serveur.ps1` : écoute locale, routes Manager, contrôle des sources et fichiers servis.
- `EBRA-Press-Service/web/photo375.js` et `EBRA-Manager/web/photo375.js` : import, segmentation, retouches et export local des images.
- `EBRA-Press-Service/outlook.ps1` et `EBRA-Manager/outlook.ps1` : actions Outlook et ouverture des pièces jointes.
- `EBRA-Press-Service/demarrer.ps1` et `EBRA-Manager/demarrer.ps1` : lancement sur les ports locaux.

*Cette note peut être diffusée avec la mention de son périmètre : revue statique de l’archive EBRA reçue le 1er octobre 2026, sans audit dynamique des postes et du réseau.*
