# EBRA — Fonction Photoshop / Magic

Bonjour à tous,

Je souhaite apporter une précision sur la fonction Photoshop / Magic intégrée à EBRA, notamment sur le traitement des photos.

Lorsqu’une image est importée, les retouches et le détourage sont effectués directement sur le poste de l’utilisateur. La fonction utilise les fichiers déjà présents dans notre application et ne fait pas appel à un service d’IA en ligne pour traiter la photo.

Après vérification du code de cette fonction, **je n’ai constaté aucun envoi des images vers un service externe**. La fonction est accessible localement sur le poste ; elle ne rend pas notre infrastructure directement accessible depuis Internet.

Nous pouvons donc utiliser Photoshop / Magic pour ce traitement sans craindre que les photos soient automatiquement transmises à un fournisseur externe par cette fonction.

Cette précision concerne uniquement la fonction Photoshop / Magic dans la version d’EBRA examinée. Elle ne constitue pas un audit complet de l’ensemble des postes et de l’infrastructure.

Bien cordialement,
Mohamed Amine
