# Note de sécurité — EBRA Photoshop / Magic

**Périmètre :** import d’image, retouche et détourage Magic. La messagerie et les autres fonctions EBRA ne sont pas couvertes.

## Ce que fait la fonction

L’utilisateur importe une image dans le navigateur. Le détourage Magic utilise un modèle de segmentation et des composants WebAssembly fournis avec l’application. L’analyse se déroule localement dans le navigateur ; les retouches et l’export sont réalisés sur le poste de l’utilisateur.

## Ce que montre l’examen du code

- La fonction Photoshop/Magic ne comporte pas d’appel observé à une API d’IA externe pour transmettre l’image.
- Les composants du modèle sont servis depuis l’application sur `localhost`.
- Le serveur examiné écoute sur l’interface locale du poste, sans exposition directe aux autres postes ou à Internet.
- L’image importée pour cette fonction n’est pas envoyée à un fournisseur d’IA par le code Photoshop examiné.

**Conclusion :** dans la version examinée, rien n’indique que Photoshop/Magic transmette les images à l’extérieur ou nécessite une connexion à un service d’IA en ligne pour traiter les photos. Son usage normal n’expose pas directement l’infrastructure à Internet.

## Condition de sécurité à respecter

Les fichiers de l’application étant placés sur un partage réseau, les utilisateurs ordinaires doivent y avoir un **accès en lecture seule**. Seules les personnes chargées des mises à jour doivent pouvoir modifier les scripts et les fichiers du modèle. Les images exportées doivent être enregistrées dans les emplacements de travail autorisés.

Cette note repose sur un examen statique du code fourni. Elle ne vaut pas certification « risque zéro » pour le poste ou l’infrastructure : cela demanderait aussi un contrôle des droits du partage, de la configuration réelle du poste et du trafic réseau.

*Périmètre vérifié : fichiers Photoshop/Magic et configuration du serveur contenus dans l’archive EBRA fournie le 1er octobre 2026.*
