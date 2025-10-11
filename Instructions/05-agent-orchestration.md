---
lab:
  title: Développez une solution multi-agent avec Microsoft Agent Framework
  description: Découvrez comment configurer plusieurs agents pour collaborer à l’aide du kit de développement logiciel (SDK) Microsoft Agent Framework
---

# Développer une solution multi-agent

Dans cet exercice, vous allez vous entraîner à utiliser le modèle d’orchestration séquentielle dans le kit de développement logiciel (SDK) Microsoft Agent Framework. Vous allez créer un pipeline simple de trois agents qui fonctionnent ensemble pour traiter les commentaires des clients et suggérer les étapes suivantes. Vous allez créer les agents suivants :

- L’agent de synthèse condense les commentaires bruts en une phrase courte et neutre.
- L’agent de classification catégorise les commentaires comme positifs, négatifs ou demandes de fonctionnalités.
- Enfin, l’agent d’action recommandé recommande une étape de suivi appropriée.

Vous allez apprendre à utiliser le kit de développement logiciel (SDK) Microsoft Agent Framework pour décomposer un problème, le router via les bons agents et produire des résultats actionnables. C’est parti !

Cet exercice devrait prendre environ **30** minutes.

> **Note** : certaines des technologies utilisées dans cet exercice sont en version préliminaire ou en cours de développement. Un comportement inattendu, des avertissements ou des erreurs peuvent se produire.

## Déployer un modèle dans un projet Azure AI Foundry

Commençons par déployer un projet Azure AI Foundry.

1. Dans un navigateur web, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure. Fermez les conseils ou les volets de démarrage rapide ouverts la première fois que vous vous connectez et, si nécessaire, utilisez le logo **Azure AI Foundry** en haut à gauche pour accéder à la page d’accueil, qui ressemble à l’image suivante (fermez le volet **Aide** s’il est ouvert) :

    ![Capture d’écran du portail Azure AI Foundry.](./Media/ai-foundry-home.png)

1. Dans la page d’accueil, dans la section **Explorer les modèles et les fonctionnalités**, recherchez le modèle `gpt-4o` ; que nous utiliserons dans notre projet.
1. Dans les résultats de la recherche, sélectionnez le modèle **gpt-4o** pour afficher ses détails, puis en haut de la page du modèle, sélectionnez **Utiliser ce modèle**.
1. Lorsque vous êtes invité à créer un projet, entrez un nom valide pour votre projet et développez **les options avancées**.
1. Confirmez les paramètres suivants pour votre projet :
    - **Ressource Azure AI Foundry** : *un nom valide pour votre ressource Azure AI Foundry.*
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Région** : *Sélectionnez n’importe quelle **recommandation d’AI Foundry***\*

    > \* Certaines ressources Azure AI sont limitées par des quotas de modèles régionaux. Si une limite de quota est atteinte plus tard dans l’exercice, vous devrez peut-être créer une autre ressource dans une autre région.

1. Sélectionnez **Créer** et attendez que votre projet, y compris le déploiement du modèle gpt-4 que vous avez sélectionné, soit créé.

1. Une fois le projet créé, le terrain de jeu de conversation instantanée s’ouvrira automatiquement.

1. Dans le volet de navigation de gauche, sélectionnez **Modèles et points de terminaison**, puis sélectionnez votre déploiement **gpt-4o**.

1. Dans le volet **Configuration**, notez le nom de votre modèle de déploiement ; il devrait s’agir de **gpt-4o**. Vous pouvez le confirmer en affichant le déploiement dans la page **Modèles et points de terminaison** (ouvrez simplement cette page dans le volet de navigation à gauche).
1. Dans le volet de navigation à gauche, sélectionnez **Vue d’ensemble** pour accéder à la page principale de votre projet ; elle se présente comme suit :

    ![Capture d’écran des détails d’un projet Azure AI dans le portail Azure AI Foundry.](./Media/ai-foundry-project.png)

## Créer une application cliente de l’agent IA

Vous avez effectué toutes les préparations nécessaires à la création d’une application cliente qui définit un agent et une fonction personnalisée. Nous vous avons fourni du code dans un référentiel GitHub.

### Préparer l’environnement

1. Ouvrez un nouvel onglet de navigateur (en gardant le portail Azure AI Foundry ouvert dans l’onglet existant). Dans un nouvel onglet du navigateur, ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.

    Fermez les notifications de bienvenue pour afficher la page d’accueil du portail Azure.

1. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell*** avec aucun stockage dans votre abonnement.

    Cloud Shell fournit une interface de ligne de commande via un volet situé en bas du portail Azure. Vous pouvez redimensionner ou agrandir ce volet pour faciliter le travail.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Dans le volet Cloud Shell, saisissez les commandes suivantes pour cloner le référentiel GitHub contenant les fichiers de code pour cet exercice (saisissez la commande, ou copiez-la dans le presse-papiers puis effectuez un clic droit dans la ligne de commande pour la coller en texte brut) :

    ```
   rm -r ai-agents -f
   git clone https://github.com/MicrosoftLearning/mslearn-ai-agents ai-agents
    ```

    > **Conseil** : lorsque vous saisissez des commandes dans le Cloud Shell, la sortie peut occuper une grande partie de la mémoire tampon d’écran et le curseur de la ligne actuelle peut être masqué. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Lorsque le référentiel a été cloné, entrez la commande suivante pour déplacer le répertoire de travail vers le dossier contenant les fichiers de code et les répertorier tous.

    ```
   cd ai-agents/Labfiles/05-agent-orchestration/Python
   ls -a -l
    ```

    Les fichiers fournis incluent le code d’application et un fichier pour les paramètres de configuration.

### Configurer les paramètres de l’application

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous allez utiliser :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install azure-identity agent-framework
    ```

1. Entrez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez l’espace réservé **your_openai_endpoint** par le point de terminaison de votre projet (copié à partir de la page **Vue d’ensemble** du projet dans le portail Azure AI Foundry). Remplacez l’espace réservé **your_model_deployment** par le nom que vous avez attribué à votre modèle de déploiement gpt-4o.

1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Créer des agents IA

Vous avez effectué toutes les préparation nécessaires à la création d’agents pour votre solution multi-agent ! C’est parti !

1. Entrez la commande suivante pour modifier le fichier **agents.py** :

    ```
   code agents.py
    ```

1. En haut du fichier, sous le commentaire **Ajouter des références**, ajoutez le code suivant pour référencer les espaces de noms dans les bibliothèques dont vous aurez besoin pour implémenter votre agent :

    ```python
   # Add references
   import asyncio
   from typing import cast
   from agent_framework import ChatMessage, Role, SequentialBuilder, WorkflowOutputEvent
   from agent_framework.azure import AzureAIAgentClient
   from azure.identity import AzureCliCredential
    ```

1. Dans la fonction **main**, prenez un moment pour passer en revue les instructions de l’agent. Ces instructions définissent le comportement de chaque agent dans l’orchestration.

1. Ajoutez le code suivant sous le commentaire **Créer le client de conversation** :

    ```python
   # Create the chat client
   credential = AzureCliCredential()
   async with (
       AzureAIAgentClient(async_credential=credential) as chat_client,
   ):
    ```

1. Ajoutez le code suivant sous le commentaire **Créer des agents** :

    (Veillez à maintenir le niveau de mise en retrait)

    ```python
   # Create agents
   summarizer = chat_client.create_agent(
       instructions=summarizer_instructions,
       name="summarizer",
   )

   classifier = chat_client.create_agent(
       instructions=classifier_instructions,
       name="classifier",
   )

   action = chat_client.create_agent(
       instructions=action_instructions,
       name="action",
   )
    ```

## Créer une orchestration séquentielle

1. Dans la fonction **main**, recherchez le commentaire **Initialiser le retour actuel** et ajoutez le code suivant :
    
    (Veillez à maintenir le niveau de mise en retrait)

    ```python
   # Initialize the current feedback
   feedback="""
   I use the dashboard every day to monitor metrics, and it works well overall. 
   But when I'm working late at night, the bright screen is really harsh on my eyes. 
   If you added a dark mode option, it would make the experience much more comfortable.
   """
    ```

1. Sous le commentaire **Générer une orchestration séquentielle**, ajoutez le code suivant pour définir une orchestration séquentielle avec les agents que vous avez définis :

    ```python
   # Build sequential orchestration
   workflow = SequentialBuilder().participants([summarizer, classifier, action]).build()
    ```

    Les agents traiteront le retour dans l’ordre dans lequel ils sont ajoutés à l’orchestration.

1. Ajoutez le code suivant sous le commentaire **Exécuter et collecter les sorties** :

    ```python
   # Run and collect outputs
   outputs: list[list[ChatMessage]] = []
   async for event in workflow.run_stream(f"Customer feedback: {feedback}"):
       if isinstance(event, WorkflowOutputEvent):
           outputs.append(cast(list[ChatMessage], event.data))
    ```

    Ce code exécute l’orchestration et collecte la sortie de chacun des agents participants.

1. Ajoutez le code suivant sous le commentaire **Affichage des sorties** :

    ```python
   # Display outputs
   if outputs:
       for i, msg in enumerate(outputs[-1], start=1):
           name = msg.author_name or ("assistant" if msg.role == Role.ASSISTANT else "user")
           print(f"{'-' * 60}\n{i:02d} [{name}]\n{msg.text}")
    ```

    Ce code met en forme et affiche les messages provenant des sorties du flux de travail que vous avez collectées depuis l’orchestration.

1. Utilisez la commande **Ctrl+S** pour enregistrer les modifications que vous avez apportées au fichier de code. Vous pouvez laisser le code ouvert (si vous devez le modifier pour corriger les erreurs) ou utiliser la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Se connecter à Azure et exécuter l’application

Vous avez effectué toutes les préparations nécessaires à l’exécution de votre code pour observer vos agents IA collaborer.

1. Dans le volet de la ligne de commande Cloud Shell, entrez la commande suivante pour vous connecter à Azure.

    ```
   az login
    ```

    **<font color="red">Vous devez vous connecter à Azure, même si la session Cloud Shell est déjà authentifiée.</font>**

    > **Remarque** :dans la plupart des scénarios, l’utilisation d’*az login* suffit. Toutefois, si vous avez des abonnements dans plusieurs locataires, vous devrez peut-être spécifier le locataire à l’aide du paramètre *--tenant*. Pour plus d’informations, consultez [Se connecter à Azure de manière interactive à l’aide d’Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).

1. Lorsque l’invite apparaît, suivez les instructions pour ouvrir la page de connexion dans un nouvel onglet et entrez le code d’authentification fourni ainsi que vos informations d’identification Azure. Effectuez ensuite le processus de connexion dans la ligne de commande, en sélectionnant l’abonnement contenant votre hub Azure AI Foundry si nécessaire.

1. Une fois la connexion effectuée, entrez la commande suivante pour exécuter l’application :

    ```
   python agents.py
    ```

    Vous devez obtenir une sortie similaire à la suivante :

    ```output
    ------------------------------------------------------------
    01 [user]
    Customer feedback:
        I use the dashboard every day to monitor metrics, and it works well overall.
        But when I'm working late at night, the bright screen is really harsh on my eyes.
        If you added a dark mode option, it would make the experience much more comfortable.

    ------------------------------------------------------------
    02 [summarizer]
    User requests a dark mode for better nighttime usability.
    ------------------------------------------------------------
    03 [classifier]
    Feature request
    ------------------------------------------------------------
    04 [action]
    Log as enhancement request for product backlog.
    ```

1. Vous pouvez également essayer d’exécuter le code en utilisant différentes entrées de retour, telles que :

    ```output
    I use the dashboard every day to monitor metrics, and it works well overall. But when I'm working late at night, the bright screen is really harsh on my eyes. If you added a dark mode option, it would make the experience much more comfortable.
    ```
    ```output
    I reached out to your customer support yesterday because I couldn't access my account. The representative responded almost immediately, was polite and professional, and fixed the issue within minutes. Honestly, it was one of the best support experiences I've ever had.
    ```

## Résumé

Dans cet exercice, vous avez pratiqué l’orchestration séquentielle avec le kit de développement logiciel (SDK) Microsoft Agent Framework, en combinant plusieurs agents en un flux de travail simplifié unique. Beau travail !

## Nettoyage

Si vous avez terminé d’explorer Azure AI Agent Service, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter de générer des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources dans lequel vous avez déployé les ressources utilisées dans cet exercice.

1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.

1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
