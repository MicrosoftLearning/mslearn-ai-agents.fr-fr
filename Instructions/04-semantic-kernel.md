---
lab:
  title: Développez un agent Azure AI avec le kit de développement logiciel (SDK) Microsoft Agent Framework
  description: Découvrez comment utiliser le kit de développement logiciel (SDK) Microsoft Agent Framework pour créer et utiliser un agent de conversation Azure AI.
---

# Développez un agent de conversation Azure AI avec le kit de développement logiciel (SDK) Microsoft Agent Framework

Dans cet exercice, vous allez utiliser Azure AI Agent Service et Microsoft Agent Framework pour créer un agent IA qui traite les notes de frais.

Cet exercice devrait prendre environ **30** minutes.

> **Note** : certaines des technologies utilisées dans cet exercice sont en version préliminaire ou en cours de développement. Un comportement inattendu, des avertissements ou des erreurs peuvent se produire.

## Déployer un modèle dans un projet Azure AI Foundry

Commençons par déployer un projet Azure AI Foundry.

1. Dans un navigateur web, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure. Fermez les conseils ou les volets de démarrage rapide ouverts la première fois que vous vous connectez et, si nécessaire, utilisez le logo **Azure AI Foundry** en haut à gauche pour accéder à la page d’accueil, qui ressemble à l’image suivante (fermez le volet **Aide** s’il est ouvert) :

    ![Capture d’écran du portail Azure AI Foundry.](./Media/ai-foundry-home.png)

1. Sur la page d’accueil, dans la section **Explorer les modèles et les fonctionnalités**, recherchez le modèle `gpt-4o` que nous utiliserons dans notre projet.
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
1. Dans le volet **Configuration**, notez le nom de votre modèle de déploiement ; il devrait s’agir de **gpt-4o**. Vous pouvez le confirmer en affichant le déploiement dans la page **Modèles et points de terminaison** (ouvrez simplement cette page dans le volet de navigation à gauche).
1. Dans le volet de navigation à gauche, sélectionnez **Vue d’ensemble** pour accéder à la page principale de votre projet ; elle se présente comme suit :

    ![Capture d’écran des détails d’un projet Azure AI dans le portail Azure AI Foundry.](./Media/ai-foundry-project.png)

## Créer une application cliente agent

Vous avez effectué toutes les préparations nécessaires à la création d’une application cliente qui définit un agent et une fonction personnalisée. Tout le code dont vous avez besoin est fourni dans un référentiel GitHub.

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

1. Lorsque le référentiel a été cloné, entrez la commande suivante pour modifier le répertoire de travail vers le dossier contenant les fichiers de code, puis les répertorier tous.

    ```
   cd ai-agents/Labfiles/04-agent-framework/python
   ls -a -l
    ```

    Les fichiers fournis incluent le code d’application, un fichier pour les paramètres de configuration et un fichier contenant des données de dépenses.

### Configurer les paramètres de l’application

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous allez utiliser :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install azure-identity agent-framework
    ```

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez l’espace réservé **your_project_endpoint** par le point de terminaison de votre projet (copié depuis la page **Vue d’ensemble** du projet dans le portail Azure AI Foundry), et remplacez **your_model_deployment** par le nom que vous avez attribué à votre modèle de déploiement gpt-4o.
1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Écrire du code pour une application agent

> **Conseil** : lorsque vous ajoutez du code, veillez à conserver la mise en retrait correcte. Utilisez les commentaires existants pour vous guider, et entrez le nouveau code au même niveau de mise en retrait.

1. Entrez la commande suivante pour modifier le fichier de code de l’agent fourni :

    ```
   code agent-framework.py
    ```

1. Examinez le code inclus dans ce fichier. Il contient :
    - Certaines instructions d’**importation** pour ajouter des références à des espaces de noms couramment utilisés
    - Une fonction *principale* qui charge un fichier contenant des données de dépenses, demande à l’utilisateur des instructions, puis appelle...
    - Une fonction **process_expenses_data** dans laquelle le code permettant de créer et utiliser votre agent doit être ajouté.

1. En haut du fichier, après l’instruction d’**import** existante, recherchez le commentaire **Ajouter des références**. Ensuite, ajoutez le code suivant pour référencer les espaces de noms dans les bibliothèques dont vous avez besoin pour implémenter votre agent :

    ```python
   # Add references
   from agent_framework import AgentThread, ChatAgent
   from agent_framework.foundry import FoundryChatClient
   from azure.identity.aio import AzureCliCredential
   from pydantic import Field
   from typing import Annotated
    ```

1. Vers la fin du fichier, recherchez le commentaire **Créer une fonction outil pour la fonctionnalité e-mail**, puis ajoutez le code suivant pour définir une fonction que votre agent utilisera pour envoyer des e-mails (les outils permettent d’ajouter des fonctionnalités personnalisées aux agents)

    ```python
   # Create a tool function for the email functionality
   def send_email(
    to: Annotated[str, Field(description="Who to send the email to")],
    subject: Annotated[str, Field(description="The subject of the email.")],
    body: Annotated[str, Field(description="The text body of the email.")]):
        print("\nTo:", to)
        print("Subject:", subject)
        print(body, "\n")
    ```

    > **Remarque** : la fonction *simule* l’envoi d’un e-mail en l’affichant dans la console. Dans un scénario réel, vous utiliseriez un service SMTP ou similaire pour envoyer l’e-mail.

1. Au-dessus du code **send_email**, dans la fonction **process_expenses_data**, recherchez le commentaire **Créer un agent de conversation** et ajoutez le code suivant pour créer un objet **ChatAgent** avec les outils et les instructions.

    (Veillez à maintenir le niveau de mise en retrait)

    ```python
   # Create a chat agent
   async with (
       AzureCliCredential() as credential,
       ChatAgent(
           chat_client=FoundryChatClient(async_credential=credential),
           name="expenses_agent",
           instructions="""You are an AI assistant for expense claim submission.
                           When a user submits expenses data and requests an expense claim, use the plug-in function to send an email to expenses@contoso.com with the subject 'Expense Claim`and a body that contains itemized expenses with a total.
                           Then confirm to the user that you've done so.""",
           tools=send_email,
       ) as agent,
   ):
    ```

    Notez que l’objet **AzureCliCredential** inclura automatiquement les paramètres du projet Azure AI Foundry issus de la configuration.

1. Recherchez le commentaire **Utiliser l’agent pour traiter les données de dépenses** et ajoutez le code suivant pour créer un thread pour que votre agent s’exécute, puis appelez-le avec un message de conversation.

    Veillez à conserver le niveau de retrait :

    ```python
   # Use the agent to process the expenses data
   try:
       # Add the input prompt to a list of messages to be submitted
       prompt_messages = [f"{prompt}: {expenses_data}"]
       # Invoke the agent for the specified thread with the messages
       response = await agent.run(prompt_messages)
       # Display the response
       print(f"\n# {response.name}:\n{response}")
   except Exception as e:
       # Something went wrong
       print (e)
    ```

1. Vérifiez que le code de votre agent est achevé en utilisant les commentaires pour vous aider à comprendre ce que fait chaque bloc de code, puis enregistrer vos modifications de code (**Ctrl+S**).
1. Gardez l’éditeur de code ouvert au cas où vous devez corriger les fautes de frappe dans le code, mais redimensionnez les volets afin que vous puissiez voir davantage la console de ligne de commande.

### Connectez-vous à Azure et exécutez votre application.

1. Dans le volet de ligne de commande Cloud Shell, sous l’éditeur de code, entrez la commande suivante pour vous connecter à Azure.

    ```
    az login
    ```

    **<font color="red">Vous devez vous connecter à Azure, même si la session Cloud Shell est déjà authentifiée.</font>**

    > **Remarque** :dans la plupart des scénarios, l’utilisation d’*az login* suffit. Toutefois, si vous avez des abonnements dans plusieurs locataires, vous devrez peut-être spécifier le locataire à l’aide du paramètre *--tenant*. Pour plus d’informations, consultez [Se connecter à Azure de manière interactive à l’aide d’Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Lorsque l’invite apparaît, suivez les instructions pour ouvrir la page de connexion dans un nouvel onglet et entrez le code d’authentification fourni ainsi que vos informations d’identification Azure. Effectuez ensuite le processus de connexion dans la ligne de commande, en sélectionnant l’abonnement contenant votre hub Azure AI Foundry si nécessaire.
1. Une fois la connexion effectuée, entrez la commande suivante pour exécuter l’application :

    ```
   python agent-framework.py
    ```
    
    L’application s’exécute à l’aide des informations d’identification de votre session Azure authentifiée pour vous connecter à votre projet et créer et exécuter l’agent.

1. Lorsqu’on vous demande quoi faire avec les données de dépenses, entrez l’invite suivante :

    ```
   Submit an expense claim
    ```

1. Une fois l’application terminée, passez en revue la production. L’agent doit avoir composé un e-mail pour une demande de frais en fonction des données fournies.

    > **Conseil** : si l’application échoue en raison du dépassement de la limite de débit, patientez quelques secondes, puis réessayez. Si le quota disponible dans votre abonnement est insuffisant, le modèle peut ne pas être en mesure de répondre.

## Résumé

Dans cet exercice, vous avez utilisé le kit de développement logiciel (SDK) Microsoft Agent Framework pour créer un agent avec un outil personnalisé.

## Nettoyage

Si vous avez terminé d’explorer Azure AI Agent Service, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter de générer des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources dans lequel vous avez déployé les ressources utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
