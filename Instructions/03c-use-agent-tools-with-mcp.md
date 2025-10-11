---
lab:
  title: Connectez les agents IA à un serveur MCP distant
  description: Apprenez à intégrer les outils du protocole Model Context Protocol avec les agents IA
---

# Connecter les agents IA aux outils à l’aide du protocole MCP (Model Context Protocol)

Dans cet exercice, vous allez créer un agent qui se connecte à un serveur MCP hébergé dans le cloud. L'agent utilisera une recherche basée sur l'IA pour aider les développeurs à trouver des réponses précises et en temps réel dans la documentation officielle de Microsoft. Cela est utile pour créer des assistants qui aident les développeurs en leur fournissant des conseils actualisés sur des outils tels qu'Azure, .NET et Microsoft 365. L'agent utilisera l'outil fourni `microsoft_docs_search` pour interroger la documentation et renvoyer les résultats pertinents.

> **Conseil** : Le code utilisé dans cet exercice est basé sur le référentiel d'exemples de prise en charge MCP du service Azure AI Agent. Pour plus d'informations, consultez les [démonstrations Azure OpenAI](https://github.com/retkowsky/Azure-OpenAI-demos/blob/main/Azure%20Agent%20Service/9%20Azure%20AI%20Agent%20service%20-%20MCP%20support.ipynb) ou rendez-vous sur [Se connecter aux serveurs Model Context Protocol](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/tools/model-context-protocol).

Cet exercice devrait prendre environ **30** minutes.

> **Note** : certaines des technologies utilisées dans cet exercice sont en version préliminaire ou en cours de développement. Un comportement inattendu, des avertissements ou des erreurs peuvent se produire.

## Créer un projet Azure AI Foundry

Commençons par créer un projet Azure AI Foundry.

1. Dans un navigateur web, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure. Fermez les conseils ou les volets de démarrage rapide ouverts la première fois que vous vous connectez et, si nécessaire, utilisez le logo **Azure AI Foundry** en haut à gauche pour accéder à la page d’accueil, qui ressemble à l’image suivante (fermez le volet **Aide** s’il est ouvert) :

    ![Capture d’écran du portail Azure AI Foundry.](./Media/ai-foundry-home.png)

1. Sur la page d’accueil, sélectionnez **Créer un agent**.
1. Lorsque vous êtes invité à créer un projet, entrez un nom valide pour votre projet et développez les **options avancées**.
1. Confirmez les paramètres suivants pour votre projet :
    - **Ressource Azure AI Foundry** : *un nom valide pour votre ressource Azure AI Foundry.*
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Région** : *Sélectionnez l’un des emplacements pris en charge suivants :*\*
      * USA Ouest 2
      * USA Ouest
      * Norvège Est
      * Suisse Nord
      * Émirats arabes unis Nord
      * Inde Sud

    > \* Certaines ressources Azure AI sont limitées par des quotas de modèles régionaux. Si une limite de quota est atteinte plus tard dans l’exercice, vous devrez peut-être créer une autre ressource dans une autre région.

1. Sélectionnez **Créer** et attendez que votre projet soit créé.
1. Si vous y êtes invité, déployez un modèle **gpt-4o** en utilisant l’option de déploiement *Standard global* ou *Standard* (selon la disponibilité de votre quota).

    >**Remarque** : Si un quota est disponible, un modèle de base GPT-4o peut être déployé automatiquement lors de la création de votre Agent et de votre projet.

1. Une fois votre projet créé, le terrain de jeu Agents est ouvert.

1. Dans le volet de navigation à gauche, sélectionnez **Vue d’ensemble** pour accéder à la page principale de votre projet ; elle se présente comme suit :

    ![Capture d’écran d’une page de présentation d’un projet Azure AI Foundry.](./Media/ai-foundry-project.png)

1. Copiez la valeur du **point de terminaison du projet Azure AI Foundry**. Vous l’utiliserez pour vous connecter à votre projet dans une application cliente.

## Développer un agent qui utilise des outils de fonction MCP

Maintenant que vous avez créé votre projet dans AI Foundry, nous allons développer une application qui intègre un agent IA à un serveur MCP.

### Cloner le référentiel contenant le code de l’application

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

1. Entrez la commande suivante pour modifier le répertoire de travail vers le dossier contenant les fichiers de code, puis les répertorier tous.

    ```
   cd ai-agents/Labfiles/03c-use-agent-tools-with-mcp/Python
   ls -a -l
    ```

### Configurer les paramètres de l’application

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous allez utiliser :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt --pre azure-ai-projects mcp
    ```

    >**Remarque :** vous pouvez ignorer les messages d’avertissement ou d’erreur qui s’affichent pendant l’installation de la bibliothèque.

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez l’espace réservé **your_project_endpoint** par le point de terminaison de votre projet (copié à partir de la page **Vue d’ensemble** du projet dans le portail Azure AI Foundry) et vérifiez que la variable MODEL_DEPLOYMENT_NAME est définie sur votre nom de modèle de déploiement (qui doit être *gpt-4o*).

1. Une fois que vous avez remplacé l’espace réservé, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Connectez un agent Azure AI à un serveur MCP distant

Dans cette tâche, vous vous connecterez à un serveur MCP distant, préparerez l'agent IA et exécuterez une invite utilisateur.

1. Saisissez la commande suivante pour modifier le fichier de code fourni :

    ```
   code client.py
    ```

    Le fichier est ouvert dans l’éditeur de code.

1. Recherchez le commentaire **Ajouter des références** et ajoutez le code suivant pour importer les classes :

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import McpTool, ToolSet, ListSortOrder
    ```

1. Recherchez le commentaire **Se connecter au client des agents**, puis ajoutez le code suivant pour vous connecter au projet Azure AI à l’aide des informations d’identification Azure que vous utilisez actuellement.

    ```python
   # Connect to the agents client
   agents_client = AgentsClient(
        endpoint=project_endpoint,
        credential=DefaultAzureCredential(
            exclude_environment_credential=True,
            exclude_managed_identity_credential=True
        )
   )
    ```

1. Sous le commentaire **Initialiser l'outil MCP de l'agent**, Ajoutez le code suivant :

    ```python
   # Initialize agent MCP tool
   mcp_tool = McpTool(
        server_label=mcp_server_label,
        server_url=mcp_server_url,
   )
    
   mcp_tool.set_approval_mode("never")
    
   toolset = ToolSet()
   toolset.add(mcp_tool)
    ```

    Ce code se connectera au serveur MCP distant Microsft Learn Docs. Il s'agit d'un service hébergé dans le cloud qui permet aux clients d'accéder à des informations fiables et à jour directement à partir de la documentation officielle de Microsoft.

1. Sous le commentaire **Créer un agent**, ajoutez le code suivant :

    ```python
   # Create a new agent
   agent = agents_client.create_agent(
        model=model_deployment,
        name="my-mcp-agent",
        instructions="""
        You have access to an MCP server called `microsoft.docs.mcp` - this tool allows you to 
        search through Microsoft's latest official documentation. Use the available MCP tools 
        to answer questions and perform tasks."""
   )
    ```

    Dans ce code, vous fournissez des instructions à l'agent et lui fournissez les définitions de l'outil MCO.

1. Trouvez le commentaire **Créer un fil de discussion pour communiquer** et ajoutez le code suivant :

    ```python
   # Create thread for communication
   thread = agents_client.threads.create()
   print(f"Created thread, ID: {thread.id}")
    ```

1. Retrouvez le commentaire **Créer un message sur le fil de discussion** et ajoutez le code suivant :

    ```python
   # Create a message on the thread
   prompt = input("\nHow can I help?: ")
   message = agents_client.messages.create(
        thread_id=thread.id,
        role="user",
        content=prompt,
   )
   print(f"Created message, ID: {message.id}")
    ```

1. Recherchez le commentaire **Créer et traiter l’exécution de l’agent dans un thread à l’aide des outils MCP** et ajoutez le code suivant :

    ```python
   # Create and process agent run in thread with MCP tools
   run = agents_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id, toolset=toolset)
   print(f"Created run, ID: {run.id}")
    ```
    
    L'agent IA invoque automatiquement les outils MCP connectés pour traiter la requête immédiate. Pour illustrer ce processus, le code fourni sous le commentaire **Afficher les étapes d'exécution et les appels d'outils** affichera tous les outils invoqués à partir du serveur MCP.

1. Enregistrez le fichier de code (*Ctrl+S*) lorsque vous avez terminé. Vous pouvez également fermer l’éditeur de code (*Ctrl+Q*).Vous pouvez toutefois le laisser ouvert au cas où vous deviez apporter des modifications au code que vous avez ajouté. Dans les deux cas, laissez le volet de ligne de commande Cloud Shell ouvert.

### Se connecter à Azure et exécuter l’application

1. Dans le volet de la ligne de commande Cloud Shell, entrez la commande suivante pour vous connecter à Azure.

    ```
   az login
    ```

    **<font color="red">Vous devez vous connecter à Azure, même si la session Cloud Shell est déjà authentifiée.</font>**

    > **Remarque** :dans la plupart des scénarios, l’utilisation d’*az login* suffit. Toutefois, si vous avez des abonnements dans plusieurs locataires, vous devrez peut-être spécifier le locataire à l’aide du paramètre *--tenant*. Pour plus d’informations, consultez [Se connecter à Azure de manière interactive à l’aide d’Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Lorsque l’invite apparaît, suivez les instructions pour ouvrir la page de connexion dans un nouvel onglet et entrez le code d’authentification fourni ainsi que vos informations d’identification Azure. Effectuez ensuite le processus de connexion dans la ligne de commande, en sélectionnant l’abonnement contenant votre hub Azure AI Foundry si nécessaire.

1. Une fois la connexion effectuée, entrez la commande suivante pour exécuter l’application :

    ```
   python client.py
    ```

1. Lorsque vous y êtes invité, entrez une demande d’informations techniques telles que :

    ```
    Give me the Azure CLI commands to create an Azure Container App with a managed identity.
    ```

1. Attendez que l’agent traite votre invite à l’aide du serveur MCP pour trouver un outil approprié pour récupérer les informations demandées. Vous devez obtenir une sortie similaire à la suivante :

    ```
    Created agent, ID: <<agent-id>>
    MCP Server: mslearn at https://learn.microsoft.com/api/mcp
    Created thread, ID: <<thread-id>>
    Created message, ID: <<message-id>>
    Created run, ID: <<run-id>>
    Run completed with status: RunStatus.COMPLETED
    Step <<step1-id>> status: completed

    Step <<step2-id>> status: completed
    MCP Tool calls:
        Tool Call ID: <<tool-call-id>>
        Type: mcp
        Type: microsoft_docs_search


    Conversation:
    --------------------------------------------------
    ASSISTANT: You can use Azure CLI to create an Azure Container App with a managed identity (either system-assigned or user-assigned). Below are the relevant commands and workflow:

    ---

    ### **1. Create a Resource Group**
    '''azurecli
    az group create --name myResourceGroup --location eastus
    '''
    

    {{continued...}}

    By following these steps, you can deploy an Azure Container App with either system-assigned or user-assigned managed identities to integrate seamlessly with other Azure services.
    --------------------------------------------------
    USER: Give me the Azure CLI commands to create an Azure Container App with a managed identity.
    --------------------------------------------------
    Deleted agent
    ```

    Notez que l'agent a pu invoquer automatiquement l'outil MCP `microsoft_docs_search` pour répondre à la demande.

1. Vous pouvez réexécuter l’application (à l’aide de la commande `python client.py`) pour demander des informations différentes. Dans chaque cas, l’agent tente de trouver la documentation technique à l’aide de l’outil MCP.

## Nettoyage

Maintenant que vous avez terminé l’exercice, vous devez supprimer les ressources cloud que vous avez créées pour éviter une utilisation inutile des ressources.

1. Ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et affichez le contenu du groupe de ressources où vous avez déployé les ressources de hub utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
