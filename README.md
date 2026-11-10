# Indexeur de pièces de dossier

Un outil gratuit pour avocats, qui tourne entièrement sur votre ordinateur, dans le navigateur. Il a trois fonctions :

- **Indexer un dossier de pièces.** Il lit chaque document (PDF, scans, Word, Excel, emails, ZIP…) et en tire une fiche : titre, type de pièce, date, parties, montants.
- **Retrouver les pièces d'un bordereau.** À partir de votre bordereau de communication de pièces, il retrouve le fichier qui correspond à chaque pièce, vous laisse valider, puis vous remet un ZIP où les fichiers sont renommés selon leur numéro de pièce.
- **Préparer l'index pour un assistant d'IA.** Il produit une copie allégée de l'index, en un ou plusieurs fichiers, que vous pouvez donner à Claude ou à un autre modèle pour discuter du dossier.

Ce dépôt contient le logiciel à télécharger déjà généré et **la recette** : un texte à coller dans un assistant d'IA, qui fabrique l'outil pour vous. Vous gardez ainsi la maîtrise de ce qui s'exécute sur votre ordinateur.

## Mode d'emploi

1. Copiez le prompt de la section [Le prompt à copier](#le-prompt-à-copier), plus bas : survolez le cadre gris et cliquez sur l'icône de copie qui apparaît en haut à droite.
2. Collez-le dans un assistant d'IA capable d'écrire un long programme, par exemple Claude ou ChatGPT.
3. Enregistrez le fichier qu'il vous rend sous le nom `pieces.html`, puis ouvrez-le d'un double-clic. Google Chrome ou Microsoft Edge sont recommandés : avec Firefox ou Safari, l'outil fonctionne, mais l'index doit être décompressé à la main dans le dossier des pièces.
4. Au premier lancement, le navigateur télécharge les composants (environ 200 Mo : Python, OCR français, petit modèle de recherche). Ils restent ensuite en cache.

Si quelque chose ne fonctionne pas, ouvrez la console du navigateur (touche F12), copiez les lignes rouges et collez-les dans l'assistant qui a fabriqué l'outil, en lui demandant de corriger. Les ajustements se font en français.

## Le prompt à copier

Copiez tout le contenu du cadre ci-dessous, sans rien modifier, et collez-le dans votre assistant d'IA.

````text
# Pièces du dossier — prompt à diffuser

Je suis avocat. Fabrique-moi une application, « Pièces du dossier », sous la forme d'un seul fichier HTML que j'ouvre d'un double-clic dans mon navigateur, sur Windows, Mac ou Linux. Je ne veux rien installer.

L'application a trois fonctions. D'abord, elle indexe un dossier de pièces clients : elle lit chaque document, OCR compris pour les scans, et en tire une fiche (titre, type, date, parties, extrait…). Ensuite, à partir d'un bordereau de communication de pièces, elle retrouve le fichier qui correspond à chaque pièce, me laisse valider, puis me remet un ZIP où les fichiers sont renommés selon leur numéro de pièce. Enfin, elle prépare une copie allégée de l'index, en un ou plusieurs fichiers, que je peux donner à un assistant d'IA comme Claude pour discuter du dossier.

Mes pièces sont confidentielles : aucun fichier, aucun extrait ne doit jamais quitter mon ordinateur. Tout le traitement se fait dans le navigateur. Seuls les outils (Python, OCR, un petit modèle de recherche) sont téléchargés au premier lancement, puis gardés en cache. Aucun grand modèle d'IA, aucun service en ligne. L'export pour un assistant d'IA se contente de produire des fichiers que je télécharge : c'est moi qui décide ensuite de les transmettre.

Les choix techniques ci-dessous ont déjà été testés : applique-les tels quels, sans les discuter ni proposer d'alternative.

## Ce que l'application doit faire

**Indexer un dossier**

- Je choisis un dossier de pièces. L'application lit tous ses sous-dossiers et prend en charge les PDF, les photos (jpg, png, webp, bmp), Word (docx et ancien doc), Excel (xlsx, xls), CSV, texte, emails (eml, avec leurs pièces jointes) et ZIP (avec leur contenu).
- Les pages scannées des PDF passent à l'OCR, en français : une page est considérée comme scannée quand elle contient moins de 50 caractères de texte.
- Pour chaque document, elle produit une fiche, **extraite** du texte et jamais rédigée : titre, type de pièce (facture, jugement, bulletin de paie, attestation, courrier…), date, parties, résumé (les phrases les plus représentatives), extrait recopié tel quel, mots-clés, ainsi que la liste des dates, montants et identifiants (IBAN, SIRET, numéro RG).
- L'index est rangé dans le dossier lui-même, dans un sous-dossier _index_pieces, en trois fichiers : index.json (les fiches), textes.json (le texte extrait, pour ne jamais refaire l'OCR) et vecteurs.bin (pour la recherche par le sens).
- Une nouvelle indexation ne traite que les fichiers nouveaux ou modifiés, et retire ceux qui ont disparu. L'index est sauvegardé tous les 10 fichiers : je peux arrêter, puis reprendre où j'en étais. Une case permet de tout réindexer.
- Un journal affiche chaque fichier traité, en vert ou en rouge avec la raison. Deux barres de progression : une pour le dossier, une pour l'OCR du document en cours. À la fin, un bilan : durée, fichiers indexés, inchangés et en erreur, pages passées à l'OCR.
- L'application ne modifie, ne renomme, ne déplace et ne supprime **jamais** une pièce. Elle n'écrit que dans _index_pieces.

**Retrouver les pièces d'un bordereau**

- J'ouvre le dossier indexé, puis je donne le bordereau (PDF, même scanné, Word ou texte).
- L'application lit la liste numérotée. Une pièce commence par un numéro, précédé ou non de « Pièce » ou « Pièce n° », suivi d'un point, d'une virgule, d'un deux-points, d'un tiret ou d'une parenthèse. Les pièces peuvent être une par ligne, ou à la suite sur une même ligne (« 1. Contrat ; 2. Facture ; 3. Attestation »). Les sous-numéros (5 puis 5-1) et les mentions bis ou ter sont reconnus. Si le bordereau se trouve à la fin de conclusions, il est trouvé grâce à son titre « Bordereau… ».
- Elle m'affiche la liste dans un tableau que je peux corriger, compléter ou réduire avant de lancer la recherche.
- Pour chaque pièce, elle cherche le bon fichier en combinant le sens de la description et les indices précis qu'elle contient : mots rares, dates, montants, type de pièce, nom du fichier. Elle classe chaque pièce en « Trouvée », « À vérifier » ou « Non trouvée », et n'attribue jamais le même fichier à deux pièces.
- Je valide dans un tableau : pour chaque pièce, une liste des cinq meilleurs candidats, plus « Aucun fichier » et « Autre fichier de l'index… » (saisie avec suggestions). Une pièce « À vérifier » n'a rien de présélectionné : c'est à moi de trancher.
- Je télécharge le ZIP des pièces retenues, renommées piece_001.pdf, piece_002.docx, piece_005-1.jpg… dans leur format d'origine, y compris celles qui viennent d'un ZIP ou d'un email. Un rapport JSON séparé (jamais dans le ZIP) liste les pièces retenues, à vérifier et introuvables.
- Si le dossier n'a pas encore d'index, je peux charger un index.json produit par un autre outil, au même format de fiches.

**Préparer l'index pour un assistant d'IA**

- Cet onglet produit une copie allégée de l'index, à donner à un assistant comme Claude ou à un modèle hébergé chez OVH. L'index du dossier n'est **jamais** modifié, ni le fichier ni les fiches en mémoire : chaque calcul repart de l'index d'origine et fabrique de nouvelles fiches.
- Je choisis l'index : le dossier déjà ouvert dans un autre onglet, un autre dossier indexé, ou un fichier index.json. L'onglet lit le JSON directement, sans attendre Python ni le modèle de recherche.
- Je choisis le modèle visé : Claude (contexte de 200 000 jetons), OVH AI Endpoints (requête limitée à 2 Mo) ou un autre modèle dont j'indique le contexte. Je règle aussi la part de ce contexte réservée à l'index, 60 % par défaut. Ces réglages sont mémorisés par le navigateur.
- Je choisis la méthode d'allègement parmi quatre boutons. **Automatique**, le réglage par défaut, retient la méthode la plus complète qui tient dans la place réservée. **Tout garder** ne retire rien. **Allègement recommandé** retire l'extrait, les mots-clés et les champs techniques, ramène le résumé à 250 caractères et tout autre texte long à 400. **Allègement maximal** ne garde que l'identité de la pièce : chemin, titre, type, date, parties, et les liens avec un ZIP, un email ou d'autres versions.
- Les champs techniques sont l'empreinte, la date d'indexation, la version, la taille, le format, les listes de dates et de mois, les nombres de fichiers et de pièces jointes, et le message d'erreur. Une section repliable permet aussi de régler champ par champ (garder, raccourcir à tant de caractères, retirer), avec pour chaque champ sa présence, sa longueur moyenne et sa part du poids total.
- Quatre options, cochées par défaut. Regrouper les versions d'un même document : même nom hors extension dans le même dossier, comme un Word et son PDF ; la fiche la plus lisible est gardée, de préférence le Word, et les autres chemins vont dans un champ autres_versions. Dédoublonner les listes. Retirer les champs vides. JSON compact.
- Je choisis enfin entre « Un seul fichier », par défaut, et « Plusieurs fichiers si la copie dépasse la place réservée ». Le découpage ne se fait **que si je le demande**. Il garde ensemble les pièces d'un même sous-dossier quand c'est possible, et répartit fiche par fiche un sous-dossier trop gros.
- L'onglet affiche le poids avant et après, le pourcentage d'allègement, une estimation en jetons (3,3 caractères par jeton) et une jauge par rapport à la limite du modèle : verte, orange au-delà de 85 % de la place réservée, rouge au-delà de 100 %. Si un fichier unique dépasse, un message le dit et propose trois remèdes : un allègement plus fort, une part réservée plus grande, ou plusieurs fichiers.
- Le bouton indique ce qu'il va produire (« Générer le fichier », « Générer les 3 fichiers ») et, juste à côté, la méthode appliquée, par exemple « Méthode : Allègement maximal (choisi automatiquement) ». Au clic, il recalcule à partir de la méthode affichée, télécharge nomdudossier_index_llm.json ou nomdudossier_index_llm_1-sur-3.json et les suivants, puis confirme les noms des fichiers générés.

**En toutes circonstances**

- Si le modèle de recherche ne se charge pas, l'application continue avec une recherche par mots, dates et montants, et propose de l'installer à la main (voir les choix techniques).
- Sous Firefox ou Safari, qui ne savent pas écrire dans un dossier, l'index m'est remis en fin d'indexation sous forme d'un fichier _index_pieces.zip à décompresser dans le dossier des pièces. Un bandeau me l'explique.
- Les problèmes sont expliqués en une phrase simple : fichier trop volumineux (plus de 50 Mo), protégé, endommagé, composant non chargé.

## À quoi elle doit ressembler

Une interface sobre, digne d'un cabinet d'avocats, en français, sur fond gris très clair (#f3f5f6) avec des surfaces blanches et des filets gris. Une seule couleur d'accent : un vert d'archive (#2d5b4f). Les titres sont dans une police à empattements installée sur l'ordinateur (Iowan Old Style, Palatino, Book Antiqua, Georgia), le reste dans la police du système, sans aucune police téléchargée. Un thème sombre s'applique automatiquement si mon ordinateur est réglé en mode sombre. Pas de dégradés, pas d'animations décoratives.

En haut, un en-tête souligné d'un trait épais : à gauche le titre « Pièces du dossier » et une phrase d'explication ; à droite, trois témoins, Python, Recherche et OCR, chacun avec une pastille (grise en attente, orange pendant le chargement, verte quand c'est prêt, rouge en cas d'erreur) et un mot d'état (« téléchargement 45 % », « prêt », « au besoin »).

En dessous, trois onglets, « Indexer un dossier », « Retrouver des pièces » et « Exporter pour un LLM ».

1. L'onglet Indexer : une phrase sur l'emplacement de l'index, le bouton « Choisir le dossier » avec le nom du dossier choisi, la case « Tout réindexer », les boutons « Lancer l'indexation » et « Arrêter », les barres de progression, le bilan et le journal.
2. L'onglet Retrouver : quatre étapes numérotées dans des cercles et reliées par un trait vertical. 1. Ouvrir le dossier indexé ; 2. Lire le bordereau ; 3. Rechercher les pièces ; 4. Télécharger. Une étape reste inactive tant que la précédente n'est pas faite.

L'onglet Exporter reprend les quatre étapes numérotées de l'onglet Retrouver : 1. Choisir l'index ; 2. Choisir le modèle visé ; 3. Alléger ; 4. Générer. Les étapes 2 à 4 n'apparaissent qu'une fois l'index chargé. À l'étape 3, les quatre méthodes sont de grands boutons en grille, chacun avec son nom en gras et une ligne d'explication ; la méthode active est encadrée de vert. Viennent ensuite les options, un encadré « Fichiers produits » avec ses deux choix, et le tableau des champs, replié. À l'étape 4 : les chiffres clés en grand dans la police à empattements, la jauge, la comparaison d'une fiche avant et après (repliée), puis le bouton principal et la méthode appliquée.

Le tableau des pièces retrouvées se lit comme un bordereau : les numéros de pièce en grand, en gras, dans la police à empattements, en vert (en orange pour une pièce à vérifier, en gris pour une pièce introuvable). Les statuts sont de petites étiquettes colorées : vert pour « Trouvée » ou « Choisie », orange pour « À vérifier », rouge pour « Non trouvée » ou « Écartée ».

Tout en bas, une section repliable « Réglages » : nombre de processus OCR en parallèle, résolution des scans, seuils de la recherche, et un lien pour revenir aux valeurs par défaut. Puis le rappel : « Tout est traité sur cet ordinateur : aucun document ne quitte la machine. »

Les messages s'affichent dans un encadré à bordure gauche colorée (vert, orange ou rouge) et disent quoi faire. L'application reste utilisable au clavier, lisible par un lecteur d'écran, et s'adapte aux petits écrans.

## Choix techniques imposés

- **Un seul fichier HTML, tout compris.** Ouvert d'un double-clic, il tourne à une adresse en file://, où le navigateur refuse de charger des fichiers locaux séparés. Donc : la feuille de style dans la page, tout le JavaScript dans un unique script classique (pas de module) placé en fin de page, et le code Python dans des balises script de type text/x-python, une par module. Au démarrage, le JavaScript écrit ces modules dans le système de fichiers de Pyodide, puis les importe.
- **Les seuls éléments chargés depuis Internet** sont ces outils, dans ces versions exactes, depuis jsDelivr : Pyodide 0.27.7 ; pdf.js (paquet pdfjs-dist) 4.10.38, avec son worker de même version ; Tesseract.js 5.1.1 avec la langue française ; Transformers.js (paquet @huggingface/transformers) 3.7.6. Pyodide et Tesseract.js se chargent par une balise script, pdf.js et Transformers.js par un import dynamique à la première utilisation.
- **Le modèle de recherche** est Xenova/multilingual-e5-small en version 8 bits (environ 120 Mo), téléchargé une fois depuis Hugging Face. Il attend le préfixe « query: » devant une description de pièce et « passage: » devant un texte de document.
- **Paquets Python** : numpy et xlrd par le chargeur de Pyodide ; openpyxl (Excel) et mammoth (Word) par micropip. Pas de PyMuPDF, qui n'existe pas pour Pyodide.
- **Partage du travail.** Le JavaScript gère le dossier, les PDF (pdf.js), l'OCR (Tesseract.js), le modèle, les vecteurs et l'interface. Python gère l'extraction des autres formats, l'analyse des fiches, la lecture du bordereau, la recherche des pièces, le ZIP et le rapport. Entre les deux, les fonctions Python ne reçoivent que du texte et des nombres et renvoient du JSON ; les octets passent par des fichiers temporaires du système de fichiers de Pyodide.
- **Accès au dossier.** Avec Chrome et Edge, le sélecteur de dossier du navigateur, en lecture-écriture. Avec les autres navigateurs, un champ de fichier en mode dossier, en lecture seule. Les deux modes offrent les mêmes opérations au reste du code.
- **Recherche des pièces.** Score = similarité de sens (meilleur passage du document) + bonus : mots rares en commun (0,15), nom du fichier (0,05), date (0,08, ou 60 % si seul le mois correspond), montant (0,08), type de pièce (0,04). Statut fixé par l'écart du meilleur candidat à la moyenne des documents, mesuré en écarts-types : « Trouvée » à partir de 3 avec une avance d'au moins 0,02 sur le second, « À vérifier » à partir de 2. Sous 8 documents, des seuils fixes (0,86 et 0,80). Ces trois seuils sont réglables dans l'interface.
- **Vecteurs** : un par passage d'environ 700 caractères (16 passages au plus par document, plus une fiche résumée), stockés sur 8 bits dans vecteurs.bin, avec le nom du modèle, pour les recalculer si le modèle change.
- **Sécurité** : 50 Mo maximum par fichier. Pour les ZIP : 5 000 fichiers et 500 Mo décompressés au maximum, refus des chemins piégés (remontée de dossier, chemin absolu, lien symbolique) et des taux de compression supérieurs à 100. Tout est lu en mémoire, jamais décompressé sur le disque.
- **Export pour un assistant d'IA** : tout en JavaScript, sur le JSON de l'index, dans un bloc de code à part qui ne modifie aucune fonction d'indexation ou de recherche. Il se contente d'appeler en lecture le choix du dossier et la lecture de index.json. Sous Firefox ou Safari, juste après une indexation, si le dossier ne contient pas encore index.json, il lit l'index gardé en mémoire par Python.

## Pièges déjà rencontrés, à éviter

- Ouverte d'un double-clic, la page refuse les fichiers séparés : tout doit être intégré dans le fichier HTML.
- En file://, le cache du navigateur où Transformers.js range le modèle peut être refusé, ce qui provoque une « network error » au chargement. Range le modèle dans IndexedDB, au moyen du cache personnalisé que prévoit Transformers.js, et désactive son cache habituel.
- N'interdis jamais à Transformers.js de télécharger un modèle, même quand le modèle est déjà en cache : il refuserait de démarrer. Il consulte toujours son cache en premier, donc un modèle déjà rangé ne se retélécharge pas.
- Si le modèle ne se charge vraiment pas, affiche un panneau avec les liens vers ses quatre fichiers (config.json, tokenizer.json, tokenizer_config.json, onnx/model_quantized.onnx), à enregistrer par clic droit, puis un champ pour me les faire sélectionner. Reconnais-les même si mon navigateur les a renommés « config (1).json ». En attendant, la recherche fonctionne par mots, dates et montants.
- Certains PDF perdent leurs retours à la ligne à l'extraction : reconstitue les lignes d'après la position verticale du texte sur la page.
- La lecture du bordereau est le piège le plus fréquent : elle a sa propre section, juste en dessous. Applique-la à la lettre.
- Les noms des parties se lisent sur une seule ligne, sinon on obtient « SARL MARTIN 12 ». Pour le titre d'un jugement, ignore les en-têtes comme « République française » et préfère une ligne qui nomme le type de document.
- pdf.js s'approprie les octets qu'on lui confie : ne les réutilise pas ensuite.
- Un ancien index créé sous Windows contient des chemins avec des antislashs : convertis-les en barres obliques.
- Installe les paquets Python avec micropip en exécutant du code Python dans Pyodide, pas en l'appelant depuis JavaScript.
- Ne change pas le nombre de processus OCR pendant un traitement : le réglage s'applique au prochain chargement de la page.
- L'autorisation d'écriture du navigateur porte sur tout le dossier : rassure-moi dans l'interface, en indiquant que seul le sous-dossier _index_pieces est écrit.
- Aucun message technique brut à l'écran : une phrase claire pour moi, les détails dans la console du navigateur.
- Dans l'onglet d'export, ne reconstruis pas les boutons de méthode ni le tableau des champs à chaque recalcul : crée-les une fois, puis mets-les à jour sur place. Sinon, quand je saisis une taille de contexte puis clique sur un bouton, le recalcul déclenché par la sortie du champ détruit le bouton et mon clic est perdu.
- Ne découpe jamais d'office : si « Allègement maximal » produit plusieurs fichiers sans que je l'aie demandé, j'ai l'impression que mon choix est ignoré.
- Affiche toujours la méthode appliquée à côté du bouton de génération. Deux méthodes peuvent donner le même nombre de fichiers, et sans ce repère on croit que le changement n'a pas été pris en compte.
- Donne aux fonctions de l'export leur propre portée et des noms distincts : le script unique contient déjà des fonctions et des variables (téléchargement, messages, état de la recherche, dossier ouvert) qu'un doublon casserait.

## Le piège principal : la lecture du bordereau

**Le problème constaté.** Les versions précédentes ne reconnaissaient que les lignes qui commencent par « Pièce » ou « Pièce n° ». Quand le bordereau numérote directement ses pièces, elles ne trouvaient rien. Or c'est la forme la plus courante : « 1. Contrat de bail ; 2. Facture ; 3. Attestation ». Souvent, les pièces se suivent sur une même ligne, ou le PDF perd ses retours à la ligne à l'extraction, si bien que toute la liste arrive en un seul bloc de texte.

**La règle.** Le mot « Pièce » est facultatif : ne l'exige jamais. Ce qui définit une pièce, c'est un numéro. Ne te fie pas non plus aux débuts de ligne.

**La méthode, en quatre étapes.**

1. Reconstitue les lignes du PDF d'après la position verticale du texte (voir les pièges ci-dessus).
2. Repère tous les débuts de pièce possibles, **n'importe où dans le texte** : un numéro de 1 à 3 chiffres, éventuellement suivi de bis ou ter ou d'un sous-numéro (5-1), éventuellement précédé de « Pièce », « Pièce n° » ou « n° ». Ce numéro doit être au début du texte ou suivre une espace ou une ponctuation, et être suivi d'un point, d'une virgule, d'un deux-points, d'un tiret, d'une parenthèse fermante ou d'une barre verticale (ou seulement d'une espace s'il est en début de ligne), puis d'une lettre. Une date comme « 12.03.2023 » n'est pas un début de pièce.
3. Parmi ces candidats, garde la plus longue suite de numéros qui se suivent : elle commence à 1, 2 ou 3, puis chaque numéro vaut le précédent plus un. Un sous-numéro (5 puis 5-1) est admis ; un petit saut (de 4 à 6) est toléré. Les autres candidats sont de faux débuts, comme « article 3. » ou « facture n° 45. », et restent dans la description de la pièce qui les contient.
4. La description d'une pièce est le texte qui va de son numéro jusqu'au numéro suivant, débarrassé du point-virgule, de la virgule ou du point final. Arrête la lecture à « Fait à », « Sous toutes réserves » ou « Par ces motifs ».

Si le texte contient un titre commençant par « Bordereau », cherche la liste après lui ; s'il y en a plusieurs (bordereau en fin de conclusions), garde celui qui donne le plus de pièces. Le titre s'arrête au premier deux-points ou au premier numéro de pièce.

**Exemples à tester avant de me rendre le fichier, avec le résultat attendu.**

| Texte du bordereau | Pièces attendues |
| --- | --- |
| 1. Contrat de bail ; 2. Facture du 12/03/2023 ; 3. Attestation de Mme Leroy (sur une seule ligne) | 1 Contrat de bail · 2 Facture du 12/03/2023 · 3 Attestation de Mme Leroy |
| Les mêmes, une par ligne, terminées par un point-virgule ou une virgule | Les mêmes trois pièces, sans ponctuation finale |
| 1, Contrat de bail puis 2, Facture (numéros suivis d'une virgule) | 1 Contrat de bail · 2 Facture |
| Bordereau de communication : 1. Bail ; 2. Facture | Titre « Bordereau de communication » · 1 Bail · 2 Facture |
| Pièce n° 1 : Bail puis Pièce n° 2 : Facture (une par ligne) | 1 Bail · 2 Facture |
| 1. Contrat, article 3. Le preneur s'engage ; 2. Facture n° 45. Montant ; 3. Attestation | 3 pièces ; « article 3 » et « n° 45 » restent dans les descriptions |
| 5. Courrier ; 5-1. Accusé de réception ; 6. Jugement | 5 Courrier · 5-1 Accusé de réception · 6 Jugement |
| Tableau Word : une ligne « 1 \| Contrat de bail », une ligne « 2 \| Facture » | 1 Contrat de bail · 2 Facture |

Si un seul de ces exemples échoue, corrige avant de livrer.

## Ce que tu me rends

Le fichier pieces.html complet, prêt à être ouvert d'un double-clic. Si tu travailles en plusieurs parties, assemble-les en ce seul fichier.

Si tu peux exécuter du code, teste d'abord la partie Python : la lecture de bordereaux, avec tous les exemples du tableau de la section précédente, plus un bordereau placé à la fin de conclusions ; l'analyse d'une facture, d'un jugement et d'un bulletin de paie ; un ZIP contenant des chemins piégés ; la recherche des pièces à partir des seuls indices de mots, de dates et de montants. Teste aussi l'export sur un faux index d'environ 800 fiches, dont des paires Word et PDF : chaque méthode, en un seul fichier puis en plusieurs, en vérifiant que le bouton et la méthode affichée changent à chaque choix et que l'index d'origine reste identique.

Ajoute ensuite, en quelques lignes simples et sans jargon, ce que tu as vérifié, ce que tu n'as pas pu vérifier, et ce que je dois observer au premier lancement : les témoins Python et Recherche passent au vert, puis une indexation d'essai sur quelques fichiers crée le sous-dossier _index_pieces, et l'onglet d'export en tire une copie allégée. Rappelle-moi qu'en cas de problème, il suffit d'ouvrir la console du navigateur (touche F12) et de te recopier les lignes rouges.
````

## Confidentialité

Tout le traitement se fait sur votre ordinateur : aucune pièce, aucun extrait n'est envoyé sur Internet. L'outil ne modifie, ne renomme, ne déplace et ne supprime jamais vos pièces ; il écrit seulement son index dans un sous-dossier `_index_pieces`.

La fonction d'export pour un assistant d'IA se contente de produire des fichiers que vous téléchargez. C'est vous qui décidez ensuite de les transmettre, ou non, à un service en ligne, au regard du secret professionnel.

## Précautions

- Testez d'abord l'outil sur de fausses pièces, avant de l'utiliser sur un dossier réel.
- Vérifiez toujours le résultat : la recherche des pièces propose, c'est vous qui validez.
- Chaque assistant d'IA fabrique sa propre version de l'outil à partir de la recette. Le résultat peut donc varier d'un assistant à l'autre, et d'une fois à l'autre.
- L'outil est fourni tel quel, sans aucune garantie.

## Vos retours

Un bug, une amélioration, un piège rencontré ? Ouvrez une « Issue » dans l'onglet du même nom, en haut de cette page. Chaque piège signalé enrichit la recette pour tout le monde.

## Licence

Ce texte et la recette sont publiés sous [licence MIT](LICENSE) : vous pouvez les utiliser, les modifier et les redistribuer librement, à condition de conserver la mention de copyright et le texte de la licence.
