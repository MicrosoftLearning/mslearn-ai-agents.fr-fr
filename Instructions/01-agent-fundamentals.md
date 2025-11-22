---
lab:
  title: Découvrir le développement d’agents IA
  description: Découvrez les étapes de développement d’assistants IA en explorant le service Azure AI Agent dans le portail Microsoft Foundry.
---

# Explorez le développement de l’agent d’IA.

Dans cet exercice, vous utilisez Azure AI Agent Service dans le portail Microsoft Foundry pour créer un assistant IA simple qui répond aux questions des employés relatives aux notes de frais.

Cet exercice prend environ **30** minutes.

> **Note** : certaines des technologies utilisées dans cet exercice sont en version préliminaire ou en cours de développement. Un comportement inattendu, des avertissements ou des erreurs peuvent se produire.

## Créer un projet et un assistant Foundry

Commençons par créer un projet Foundry.

1. Dans un navigateur web, ouvrez le [portail Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure. Fermez les conseils ou les volets de démarrage rapide ouverts la première fois que vous vous connectez et, si nécessaire, utilisez le logo **Foundry** en haut à gauche pour accéder à la page d’accueil, qui ressemble à l’image suivante (fermez le volet **Aide** s’il est ouvert) :

    ![Capture d’écran du portail Foundry.](./Media/ai-foundry-home.png)

    > **Important** : Assurez-vous que le bouton bascule **Nouveau Foundry** est *désactivé* pour ce labo.

1. Sur la page d’accueil, sélectionnez **Créer un agent**.
1. Lorsqu’il vous est demandé de créer un projet, entrez un nom valide pour votre projet.
1. Développez les **options avancées** et définissez les paramètres suivants :
    - **Ressource Foundry** : *Nom valide de votre ressource Foundry*
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *sélectionnez votre groupe de ressources ou créez-en un*.
    - **Région** : *Sélectionnez n’importe quelle **recommandation d’AI Foundry***\**

    > \* Certaines ressources Azure AI sont limitées par des quotas de modèles régionaux. Si une limite de quota est atteinte plus tard dans l’exercice, vous devrez peut-être créer une autre ressource dans une autre région.

1. Sélectionnez **Créer** et attendez que votre projet soit créé.
1. Si vous y êtes invité, déployez un modèle **gpt-4o** en utilisant le type de déploiement **norme Global** ou **Standard** (en fonction de la disponibilité des quotas) et personnalisez les détails du déploiement pour définir une **Limite de taux de jetons par minute** de 50 000 (ou le maximum disponible s’il est inférieur à 50 000).

    > **Remarque** : La réduction du nombre de jetons par minute permet d’éviter une surutilisation du quota disponible dans l’abonnement que vous utilisez. 50 000 jetons par minute sont suffisants pour les données utilisées dans cet exercice. Si votre quota disponible est inférieur à cette valeur, vous pourrez tout de même terminer l’exercice, mais vous pourriez rencontrer des erreurs en cas de dépassement de la limite.

1. Une fois le projet créé, le terrain de jeu Agents s’ouvrira automatiquement pour vous permettre de sélectionner ou de déployer un modèle :

    ![Capture d’écran du terrain de jeu des assistants d’un projet Foundry.](./Media/ai-foundry-agents-playground.png)

    >**Remarque** : un modèle de base GPT-4o est automatiquement déployé lors de la création de votre agent et de votre projet.

Vous verrez qu’un agent portant un nom par défaut a été créé pour vous, ainsi que votre modèle de déploiement de base.

## Créer un agent

Maintenant que vous avez déployé un modèle, tout est prêt pour créer un agent IA. Dans cet exercice, vous allez créer un agent simple qui répond aux questions en fonction de la stratégie de dépenses d’une entreprise. Vous allez télécharger le document de stratégie de dépenses et l’utiliser comme données *de base* pour l’agent.

1. Ouvrez un nouvel onglet de navigateur, téléchargez le fichier [Expenses_policy.docx](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-agent-fundamentals/Expenses_Policy.docx) à partir de `https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-agent-fundamentals/Expenses_Policy.docx`, et enregistrez-le sur votre ordinateur. Ce document contient les détails de la politique des dépenses pour la société Contoso fictive.
1. Revenez à l’onglet du navigateur contenant le terrain de jeu Foundry Agents et recherchez le volet **Configuration** (il peut être situé sur le côté ou sous la fenêtre de conversation).
1. Définissez le **nom de l’agent** sur `ExpensesAgent`, vérifiez que le modèle de déploiement gpt-4o que vous avez créé précédemment est sélectionné et définissez les **instructions** sur :

    ```prompt
   You are an AI assistant for corporate expenses.
   You answer questions about expenses based on the expenses policy data.
   If a user wants to submit an expense claim, you get their email address, a description of the claim, and the amount to be claimed and write the claim details to a text file that the user can download.
    ```

    ![Capture d’écran de la page de configuration de l’assistant IA dans le portail Foundry.](./Media/ai-agent-setup.png)

1. Plus loin dans le volet **Configuration**, en regard de l’en-tête **Connaissances**, sélectionnez **+ Ajouter**. Ensuite, dans la boîte de dialogue **Ajouter une connaissance**, sélectionnez **Fichiers**.
1. Dans la boîte de dialogue **Ajout de fichiers**, créez un magasin de vecteurs nommé `Expenses_Vector_Store`, chargez et enregistrez le fichier local **Expenses_policy.docx** que vous avez téléchargé précédemment.
1. Dans le volet **Configuration**, dans la section **Connaissances**, vérifiez que le fichier **Expenses_Vector_Store** apparaît et qu’il contient 1 fichier.
1. Sous la section **Connaissances**, en regard d’**Actions**, sélectionnez **+ Ajouter**. Ensuite, dans la boîte de dialogue **Ajouter une action**, sélectionnez **Interpréteur de code**, puis **Enregistrer** (vous n’avez pas besoin de charger de fichiers pour l’interpréteur de code).

    Votre agent utilisera le document que vous avez chargé comme source de connaissances pour *ancrer* ses réponses (en d’autres termes, il répond aux questions en fonction du contenu de ce document). Il utilise l’outil d’interpréteur de code comme nécessaire pour effectuer des actions en générant et en exécutant son propre code Python.

## Tester l’agent

Maintenant que vous avez créé un agent, vous pouvez le tester dans la conversation du terrain de jeu.

1. Dans la conversation du terrain de jeu, entrez l’invite `What's the maximum I can claim for meals?` et examinez la réponse de l’agent. Elle doit être basée sur les informations contenues dans le document de la politique de dépenses que vous avez ajouté en tant que connaissance lors de la configuration de l’agent.

    > **Remarque** : si l’agent ne répond pas, cela signifie que le taux limite a été dépassé. patientez quelques secondes, puis réessayez. Si le quota disponible dans votre abonnement est insuffisant, le modèle peut ne pas être en mesure de répondre. Si le problème persiste, essayez d’augmenter le quota de votre modèle sur la page **Modèles + points de terminaison**.

1. Essayez l’invite de suivi `I'd like to submit a claim for a meal.` et évaluez la réponse. L’agent doit vous demander les informations requises pour soumettre une demande.
1. Indiquez à l’agent une adresse e-mail, par exemple, `fred@contoso.com`. L’agent doit accuser réception de la réponse et demander les informations restantes requises pour la demande de frais (description et montant)
1. Envoyez une invite qui décrit la revendication et le montant, par exemple, `Breakfast cost me $20`.
1. L’agent doit utiliser l’interpréteur de code pour préparer le fichier texte de la demande de frais et fournir un lien pour pouvoir le télécharger.

    ![Capture d’écran du terrain de jeu des assistants dans le portail Foundry.](./Media/ai-agent-playground.png)

1. Téléchargez et ouvrez le document texte pour afficher les détails de la demande de frais.

## Nettoyage

Maintenant que vous avez terminé l’exercice, vous devez supprimer les ressources cloud que vous avez créées pour éviter une utilisation inutile des ressources.

1. Ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et affichez le contenu du groupe de ressources où vous avez déployé les ressources de hub utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
