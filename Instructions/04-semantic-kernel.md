---
lab:
  title: Développer un agent Azure AI avec le kit de développement logiciel (SDK) de noyau sémantique
  description: Découvrez comment utiliser le kit de développement logiciel (SDK) de noyau sémantique pour créer et utiliser un agent Azure AI Agent Service.
---

# Développer un agent Azure AI avec le kit de développement logiciel (SDK) de noyau sémantique

Dans cet exercice, vous allez utiliser Azure AI Agent Service et le noyau sémantique pour créer un agent IA qui crée un e-mail de note de frais.

Cet exercice devrait prendre environ **30** minutes.

> **Note** : certaines des technologies utilisées dans cet exercice sont en version préliminaire ou en cours de développement. Un comportement inattendu, des avertissements ou des erreurs peuvent se produire.

## Créer un projet Azure AI Foundry

Commençons par créer un projet Azure AI Foundry.

1. Dans un navigateur web, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure. Fermez les conseils ou les volets de démarrage rapide ouverts la première fois que vous vous connectez et, si nécessaire, utilisez le logo **Azure AI Foundry** en haut à gauche pour accéder à la page d’accueil, qui ressemble à l’image suivante (fermez le volet **Aide** s’il est ouvert) :

    ![Capture d’écran du portail Azure AI Foundry.](./Media/ai-foundry-home.png)

1. Sur la page d’accueil, sélectionnez **+Créer un projet**.
1. Dans l’assistant **Créer un projet**, saisissez un nom valide et, si un hub existant est suggéré, choisissez l’option permettant d’en créer un. Passez ensuite en revue les ressources Azure qui seront créées automatiquement pour prendre en charge votre hub et votre projet.
1. Sélectionnez **Personnaliser** et spécifiez les paramètres suivants pour votre hub :
    - **Nom du hub** : *un nom valide pour votre hub*
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Emplacement** : sélectionnez l’une des régions suivantes :\*
        - eastus
        - eastus2
        - centre de la suède
        - westus
        - westus3
    - **Connecter Azure AI Services ou Azure OpenAI** : *créer une nouvelle ressource AI Services*
    - **Connecter la Recherche Azure AI** : ignorer la connexion

    > \* Au moment de l’écriture, ces régions prennent en charge le modèle gpt-4o à utiliser pour les agents. La disponibilité des modèles est limitée par les quotas régionaux. Si une limite de quota est atteinte plus tard dans l’exercice, vous devrez peut-être créer un autre projet dans une autre région.

1. Sélectionnez **Suivant** et passez en revue votre configuration. Sélectionnez **Créer** et patientez jusqu’à ce que l’opération se termine.
1. Une fois votre projet créé, fermez les conseils affichés et passez en revue la page du projet dans le portail Azure AI Foundry, qui doit ressembler à l’image suivante :

    ![Capture d’écran des détails d’un projet Azure AI dans le portail Azure AI Foundry.](./Media/ai-foundry-project.png)

## Déployer un modèle d’IA générative

Vous avez effectué toutes les préparations nécessaires au déploiement d’un modèle de langage d’IA générative pour votre agent.

1. Dans le volet de gauche de votre projet, dans la section **Mes ressources**, sélectionnez la page **Modèles + points de terminaison**.
1. Sur la page **Modèles + points de terminaison**, dans l’onglet **Déploiements de modèles**, dans le menu **+ Déployer un modèle**, sélectionnez **Déployer le modèle de base**.
1. Recherchez le modèle **gpt-4o** dans la liste, puis sélectionnez-le et confirmez.
1. Déployez le modèle avec les paramètres suivants en sélectionnant **Personnaliser** dans les détails du déploiement :
    - **Nom du déploiement** : *nom valide pour votre modèle de déploiement*
    - **Type de déploiement** : standard global
    - **Mise à jour automatique de la version** : activée
    - **Version du modèle** : *sélectionnez la version la plus récente disponible.*
    - **Ressource IA connectée** : *sélectionnez votre connexion de ressources Azure OpenAI*
    - **Limite de jetons par minute (en milliers)**  : 50 *(ou le maximum disponible dans votre abonnement si inférieur à 50 000)*
    - **Filtre de contenu** : DefaultV2

    > **Remarque** : La réduction du nombre de jetons par minute permet d’éviter une surutilisation du quota disponible dans l’abonnement que vous utilisez. 50 000 jetons par minute sont suffisants pour les données utilisées dans cet exercice. Si votre quota disponible est inférieur à ce montant, vous serez en mesure d’effectuer l’exercice, mais vous devrez peut-être patienter et soumettre à nouveau les invites si la limite de jetons est dépassée.

1. Attendez la fin du déploiement.

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

1. Lorsque le référentiel a été cloné, entrez la commande suivante pour déplacer le répertoire de travail vers le dossier contenant les fichiers de code et les répertorier tous.

    ```
   cd ai-agents/Labfiles/04-semantic-kernel/python
   ls -a -l
    ```

    Les fichiers fournis incluent le code d’application et un fichier pour les paramètres de configuration.

### Configurer les paramètres de l’application

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous allez utiliser :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity semantic-kernel[azure] 
    ```

    > **Remarque** : l’installation d’un *noyau sémantique[azure]* entraîne l’installation automatique d’une version compatible avec le noyau sémantique des *azure-ai-projects*.

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez l’espace réservé **your_project_connection_string** par la chaîne de connexion de votre projet (copiée depuis la page **Vue d’ensemble** du projet dans le portail Azure AI Foundry), et remplacez l’espace réservé **your_model_deployment** par le nom que vous avez attribué au déploiement du modèle gpt-4o.
1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Écrire du code pour une application agent

> **Conseil** : lorsque vous ajoutez du code, veillez à conserver la mise en retrait correcte. Utilisez les commentaires existants pour vous guider, et entrez le nouveau code au même niveau de mise en retrait.

1. Entrez la commande suivante pour modifier le fichier de code de l’agent fourni :

    ```
   code semantic-kernel.py
    ```

1. Examinez le code inclus dans ce fichier. Il contient :
    - des instructions d’**import** pour ajouter des références à des espaces de noms couramment utilisés.
    - une fonction *principale* qui définit les données d’une note de frais (dans un scénario réel, elles seraient probablement envoyées sous forme de fichier) et appelle ensuite...
    - une fonction **create_expense_claim** à laquelle il faut ajouter le code qui permettra de créer et d’utiliser votre agent.
    - une classe **EmailPlugin** qui inclut une fonction noyau nommée **send_email** et qui sera utilisée par votre agent pour simuler les fonctionnalités utilisées pour envoyer un e-mail.

1. En haut du fichier, après l’instruction d’**import** existante, recherchez le commentaire **Ajouter des références**. Ensuite, ajoutez le code suivant pour référencer les espaces de noms dans les bibliothèques dont vous avez besoin pour implémenter votre agent :

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity.aio import DefaultAzureCredential
   from semantic_kernel.agents import AzureAIAgent, AzureAIAgentSettings, AzureAIAgentThread
   from semantic_kernel.functions import kernel_function
   from typing import Annotated
    ```

1. En bas du fichier, recherchez le commentaire **Créer un plug-in pour la fonctionnalité de messagerie** et ajoutez le code suivant afin de définir une classe pour un plug-in contenant une fonction que votre agent utilisera pour envoyer des e-mails (les plug-ins constituent un moyen d’ajouter des fonctionnalités personnalisées aux agents de noyau sémantique).

    ```python
   # Create a Plugin for the email functionality
   class EmailPlugin:
       """A Plugin to simulate email functionality."""
    
       @kernel_function(description="Sends an email.")
       def send_email(self,
                      to: Annotated[str, "Who to send the email to"],
                      subject: Annotated[str, "The subject of the email."],
                      body: Annotated[str, "The text body of the email."]):
           print("\nTo:", to)
           print("Subject:", subject)
           print(body, "\n")
    ```

    > **Remarque** : la fonction *simule* l’envoi d’un e-mail en l’affichant dans la console. Dans un scénario réel, vous utiliseriez un service SMTP ou similaire pour envoyer l’e-mail.

1. Remontez dans le nouveau code de la classe **EmailPlugin**, dans la fonction **create_expense_claim**, et recherchez le commentaire **Obtenir les paramètres de configuration**. Ensuite, ajoutez le code suivant pour charger le fichier de configuration et créer un objet **AzureAIAgentSettings** (il inclura automatiquement les paramètres de l’agent Azure AI à partir de la configuration).

    (Veillez à maintenir le niveau de mise en retrait)

    ```python
   # Get configuration settings
   load_dotenv()
   ai_agent_settings = AzureAIAgentSettings()
    ```

1. Recherchez le commentaire **Se connecter au projet Azure AI Foundry** et ajoutez le code suivant afin de vous connecter à votre projet Azure AI Foundry à l’aide des mêmes informations d’identification Azure que vous utilisez actuellement pour votre session.

    (Veillez à maintenir le niveau de mise en retrait)

    ```python
   # Connect to the Azure AI Foundry project
   async with (
        DefaultAzureCredential(
            exclude_environment_credential=True,
            exclude_managed_identity_credential=True) as creds,
        AzureAIAgent.create_client(
            credential=creds
        ) as project_client,
   ):
    ```

1. Recherchez le commentaire **Définir un agent Azure AI qui envoie un e-mail de note de frais** et ajoutez le code suivant pour créer une définition d’agent Azure AI pour votre agent.

    (Veillez à maintenir le niveau de mise en retrait)

    ```python
   # Define an Azure AI agent that sends an expense claim email
   expenses_agent_def = await project_client.agents.create_agent(
        model= ai_agent_settings.model_deployment_name,
        name="expenses_agent",
        instructions="""You are an AI assistant for expense claim submission.
                        When a user submits expenses data and requests an expense claim, use the plug-in function to send an email to expenses@contoso.com with the subject 'Expense Claim`and a body that contains itemized expenses with a total.
                        Then confirm to the user that you've done so."""
   )
    ```

1. Recherchez le commentaire **Créer un agent de noyau sémantique** et ajoutez le code suivant pour créer un objet d’agent de noyau sémantique pour votre agent Azure AI et inclure une référence au plug-in **EmailPlugin**.

    (Veillez à maintenir le niveau de mise en retrait)

    ```python
   # Create a semantic kernel agent
   expenses_agent = AzureAIAgent(
        client=project_client,
        definition=expenses_agent_def,
        plugins=[EmailPlugin()]
   )
    ```

1. Recherchez le commentaire **Utiliser l’agent pour générer un e-mail de note de frais** et ajoutez le code suivant pour créer un thread sur lequel votre agent va s’exécuter, puis appelez-le en écrivant un message.

    (Veillez à conserver le niveau de mise en retrait) :

    ```python
   # Use the agent to generate an expense claim email
   thread: AzureAIAgentThread = AzureAIAgentThread(client=project_client)
   try:
        # Add the input prompt to a list of messages to be submitted
        prompt_messages = [f"Create an expense claim for the following expenses: {expenses_data}"]
        # Invoke the agent for the specified thread with the messages
        response = await expenses_agent.get_response(thread_id=thread.id, messages=prompt_messages)
        # Display the response
        print(f"\n# {response.name}:\n{response}")
   except Exception as e:
        # Something went wrong
        print (e)
   finally:
        # Cleanup: Delete the thread and agent
        await thread.delete() if thread else None
        await project_client.agents.delete_agent(expenses_agent.id)
    ```

1. Relisez le code finalisé de votre agent en vous aidant des commentaires pour mieux comprendre ce que fait chaque bloc de code, puis enregistrez les modifications du code (**Ctrl+S**).

### Se connecter à Azure et exécuter l’application

1. Dans le volet de ligne de commande Cloud Shell, sous l’éditeur de code, entrez la commande suivante pour vous connecter à Azure.

    ```
    az login
    ```

    **<font color="red">Vous devez vous connecter à Azure, même si la session Cloud Shell est déjà authentifiée.</font>**

    > **Remarque** :dans la plupart des scénarios, l’utilisation d’*az login* suffit. Toutefois, si vous avez des abonnements dans plusieurs locataires, vous devrez peut-être spécifier le locataire à l’aide du paramètre *--tenant*. Pour plus d’informations, consultez [Se connecter à Azure de manière interactive à l’aide d’Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Lorsque l’invite apparaît, suivez les instructions pour ouvrir la page de connexion dans un nouvel onglet et entrez le code d’authentification fourni ainsi que vos informations d’identification Azure. Effectuez ensuite le processus de connexion dans la ligne de commande, en sélectionnant l’abonnement contenant votre hub Azure AI Foundry si nécessaire.
1. Une fois la connexion effectuée, entrez la commande suivante pour exécuter l’application :

    ```
   python semantic-kernel.py
    ```
    
    L’application s’exécute à l’aide des informations d’identification de votre session Azure authentifiée pour vous connecter à votre projet, et créer et exécuter l’agent.

    > **Conseil** : si l’application échoue en raison du dépassement de la limite de débit, patientez quelques secondes, puis réessayez. Si le quota disponible dans votre abonnement est insuffisant, le modèle peut ne pas être en mesure de répondre.

1. Une fois que l’application a terminé, vérifiez la sortie. L’agent doit avoir composé un e-mail pour une note de frais en fonction des données qui lui ont été fournies.

## Résumé

Dans cet exercice, vous avez utilisé le kit de développement logiciel (SDK) et le noyau sémantique d’Azure AI Agent Service pour créer un agent.

## Nettoyage

Si vous avez terminé d’explorer Azure AI Agent Service, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter de générer des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources dans lequel vous avez déployé les ressources utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
