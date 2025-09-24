# Connecter les agents IA aux outils à l’aide du protocole MCP (Model Context Protocol)

Dans cet exercice, vous allez créer un agent capable de se connecter à un serveur MCP et de découvrir les fonctions joignables de manière automatique.

Vous allez créer un agent d’évaluation d’inventaire simple pour un détaillant de produits cosmétiques. Grâce au serveur MCP, l’agent pourra récupérer des informations sur les inventaires et faire des suggestions de restockage ou de liquidation.

> **Conseil** : Le code utilisé dans cet exercice est basé sur Azure AI Foundry et les kits de développement logiciel (SDK) MCP pour Python. Vous pouvez développer des solutions similaires à l’aide des kits de développement logiciel (SDK) pour Microsoft .NET. Pour plus d’informations, reportez-vous à [Bibliothèques client SDK Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) et [SDK MCP C#](https://modelcontextprotocol.github.io/csharp-sdk/api/ModelContextProtocol.html).

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
    - **Région** : *Sélectionnez n’importe quelle **recommandation d’AI Foundry***\*

    > \* Certaines ressources Azure AI sont limitées par des quotas de modèles régionaux. Si une limite de quota est atteinte plus tard dans l’exercice, vous devrez peut-être créer une autre ressource dans une autre région.

1. Sélectionnez **Créer** et attendez que votre projet soit créé.
1. Si vous y êtes invité, déployez un modèle **gpt-4o** en utilisant l’option de déploiement *Standard global* ou *Standard* (selon la disponibilité de votre quota).

    >**Remarque** : Si un quota est disponible, un modèle de base GPT-4o peut être déployé automatiquement lors de la création de votre Agent et de votre projet.

1. Une fois votre projet créé, le terrain de jeu Agents est ouvert.

1. Dans le volet de navigation à gauche, sélectionnez **Vue d’ensemble** pour accéder à la page principale de votre projet ; elle se présente comme suit :

    ![Capture d’écran d’une page de présentation d’un projet Azure AI Foundry.](./Media/ai-foundry-project.png)

1. Copiez la valeur du **point de terminaison du projet Azure AI Foundry** dans un bloc-notes, car vous l’utiliserez pour vous connecter à votre projet dans une application cliente.

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
   cd ai-agents/Labfiles/03d-use-agent-tools-with-mcp/Python
   ls -a -l
    ```

    Les fichiers fournis incluent le code de l’application client et serveur. Le protocole MCP (Model Context Protocol) fournit un moyen standardisé de connecter des modèles d’IA à différentes sources de données et différents outils. Nous séparons `client.py` et `server.py` afin de conserver la modularité de la logique de l’agent et des définitions d’outils, et de simuler une architecture réelle. 
    
    `server.py` définit les outils que l’agent peut utiliser, en simulant les services d’arrière plan ou la logique professionnelle. 
    `client.py` gère la configuration de l’assistant IA, les invites utilisateur et l’appel des outils si nécessaire.

### Configurer les paramètres de l’application

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous allez utiliser :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects mcp
    ```

    >**Remarque :** vous pouvez ignorer les messages d’avertissement ou d’erreur qui s’affichent pendant l’installation de la bibliothèque.

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez l’espace réservé **your_project_endpoint** par le point de terminaison de votre projet (copié à partir de la page **Vue d’ensemble** du projet dans le portail Azure AI Foundry) et vérifiez que la variable MODEL_DEPLOYMENT_NAME est définie sur votre nom de modèle de déploiement (qui doit être *gpt-4o*).

1. Une fois que vous avez remplacé l’espace réservé, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Implémenter un serveur MCP

Un serveur MCP (Model Context Protocol) est un composant qui héberge des outils pouvant être appelés. Ces outils sont des fonctions Python qui peuvent être exposées à des agents IA. Lorsque les outils sont annotés avec `@mcp.tool()`, ils deviennent détectables par le client, ce qui permet à un agent IA de les appeler dynamiquement pendant une conversation ou une tâche. Dans cette tâche, vous allez ajouter des outils qui permettront à l’agent d’effectuer des vérifications d’inventaire.

1. Saisissez la commande suivante pour modifier le fichier de code fourni pour le code de la fonction :

    ```
   code server.py
    ```

    Dans ce fichier de code, vous allez définir les outils que l’agent peut utiliser pour simuler un service d’arrière plan pour le magasin de vente au détail. Notez le code de configuration du serveur en haut du fichier. Il utilise `FastMCP` pour démarrer rapidement une instance de serveur MCP nommée « Inventaire ». Ce serveur hébergera les outils que vous définissez et les rendra accessibles à l’agent pendant le laboratoire.

1. Recherchez le commentaire **Ajouter un outil de vérification de l’inventaire** et ajoutez le code suivant :

    ```python
   # Add an inventory check tool
   @mcp.tool()
   def get_inventory_levels() -> dict:
        """Returns current inventory for all products."""
        return {
            "Moisturizer": 6,
            "Shampoo": 8,
            "Body Spray": 28,
            "Hair Gel": 5, 
            "Lip Balm": 12,
            "Skin Serum": 9,
            "Cleanser": 30,
            "Conditioner": 3,
            "Setting Powder": 17,
            "Dry Shampoo": 45
        }
    ```

    Ce dictionnaire représente un exemple d’inventaire. L’annotation `@mcp.tool()` permet au LLM de découvrir votre fonction. 

1. Recherchez le commentaire **Ajouter un outil de vente hebdomadaire** et ajoutez le code suivant :

    ```python
   # Add a weekly sales tool
   @mcp.tool()
   def get_weekly_sales() -> dict:
        """Returns number of units sold last week."""
        return {
            "Moisturizer": 22,
            "Shampoo": 18,
            "Body Spray": 3,
            "Hair Gel": 2,
            "Lip Balm": 14,
            "Skin Serum": 19,
            "Cleanser": 4,
            "Conditioner": 1,
            "Setting Powder": 13,
            "Dry Shampoo": 17
        }
    ```

1. Enregistrez le fichier (*Ctrl+S*).

### Implémenter un client MCP

Un client MCP est le composant qui se connecte au serveur MCP pour détecter et appeler des outils. Vous pouvez le considérer comme le pont entre l’agent et les fonctions hébergées par le serveur, ce qui permet une utilisation dynamique de l’outil en réponse aux invites de l’utilisateur.

1. Exécutez la commande suivante pour commencer à modifier le code du client.

    ```
   code client.py
    ```

    > **Conseil** : lorsque vous ajoutez du code au fichier de code, veillez à conserver la bonne mise en retrait.

1. Recherchez le commentaire **Ajouter des références** et ajoutez le code suivant pour importer les classes :

    ```python
   # Add references
   from mcp import ClientSession, StdioServerParameters
   from mcp.client.stdio import stdio_client
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import FunctionTool, MessageRole, ListSortOrder
   from azure.identity import DefaultAzureCredential
    ```

1. Recherchez le commentaire **Démarrer le serveur MCP** et ajoutez le code suivant :

    ```python
   # Start the MCP server
   stdio_transport = await exit_stack.enter_async_context(stdio_client(server_params))
   stdio, write = stdio_transport
    ```

    Dans une configuration de production standard, le serveur s’exécute séparément du client. Mais dans le cadre de ce labo, le client est responsable du démarrage du serveur à l’aide du transport d’entrée/sortie standard. Cela crée un canal de communication léger entre les deux composants et simplifie la configuration du développement local.

1. Recherchez le commentaire **Créer une session cliente MCP** et ajoutez le code suivant :

    ```python
   # Create an MCP client session
   session = await exit_stack.enter_async_context(ClientSession(stdio, write))
   await session.initialize()
    ```

    Cela crée une session cliente à l’aide des flux d’entrée et de sortie de l’étape précédente. L’appel à `session.initialize` prépare la session pour détecter et appeler les outils inscrits sur le serveur MCP.

1. Sous le commentaire **Liste des outils disponibles**, ajoutez le code suivant pour vérifier que le client s’est connecté au serveur :

    ```python
   # List available tools
   response = await session.list_tools()
   tools = response.tools
   print("\nConnected to server with tools:", [tool.name for tool in tools]) 
    ```

    Votre session cliente est maintenant prête à être utilisée avec votre agent Azure AI.

### Connecter les outils MCP à votre agent

Dans cette tâche, vous allez préparer l’agent IA, accepter les invites utilisateur et appeler les outils de fonction.

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

1. Sous le commentaire **Liste des outils disponibles sur le serveur**, ajoutez le code suivant :

    ```python
   # List tools available on the server
   response = await session.list_tools()
   tools = response.tools
    ```

1. Sous le commentaire **Générer une fonction pour chaque outil** et ajouter le code suivant :

    ```python
   # Build a function for each tool
   def make_tool_func(tool_name):
        async def tool_func(**kwargs):
            result = await session.call_tool(tool_name, kwargs)
            return result
        
        tool_func.__name__ = tool_name
        return tool_func

   functions_dict = {tool.name: make_tool_func(tool.name) for tool in tools}
   mcp_function_tool = FunctionTool(functions=list(functions_dict.values()))
    ```

    Ce code encapsule dynamiquement les outils disponibles dans le serveur MCP afin qu’ils puissent être appelés par l’assistant IA. Chaque outil est convertit en une fonction asynchrone, puis regroupé dans un `FunctionTool` afin que l’agent puisse l’utiliser.

1. Recherchez le commentaire **Créer l’agent** et ajoutez le code suivant :

    ```python
   # Create the agent
   agent = agents_client.create_agent(
        model=model_deployment,
        name="inventory-agent",
        instructions="""
        You are an inventory assistant. Here are some general guidelines:
        - Recommend restock if item inventory < 10  and weekly sales > 15
        - Recommend clearance if item inventory > 20 and weekly sales < 5
        """,
        tools=mcp_function_tool.definitions
   )
    ```

1. Recherchez le commentaire **Activer l’appel automatique de fonction** et ajoutez le code suivant :

    ```python
   # Enable auto function calling
   agents_client.enable_auto_function_calls(tools=mcp_function_tool)
    ```

1. Sous le commentaire **Créer un fil de discussion pour la session de conversation**, ajoutez le code suivant :

    ```python
   # Create a thread for the chat session
   thread = agents_client.threads.create()
    ```

1. Recherchez le commentaire **Appeler l’invite** et ajoutez le code suivant :

    ```python
   # Invoke the prompt
   message = agents_client.messages.create(
        thread_id=thread.id,
        role=MessageRole.USER,
        content=user_input,
   )
   run = agents_client.runs.create(thread_id=thread.id, agent_id=agent.id)
    ```

1. Recherchez le commentaire **Récupérer l’outil de fonction correspondant** et ajoutez le code suivant :

    ```python
   # Retrieve the matching function tool
   function_name = tool_call.function.name
   args_json = tool_call.function.arguments
   kwargs = json.loads(args_json)
   required_function = functions_dict.get(function_name)

   # Invoke the function
   output = await required_function(**kwargs)
    ```

    Ce code utilise les informations de l’appel d’outil du fil de discussion de l’agent. Le nom de la fonction et les arguments sont récupérés et utilisés pour appeler la fonction correspondante.

1. Sous le commentaire **Ajouter le texte de sortie**, ajoutez le code suivant :

    ```python
   # Append the output text
   tool_outputs.append({
        "tool_call_id": tool_call.id,
        "output": output.content[0].text,
   })
    ```

1. Sous le commentaire **Soumettre la sortie de l’appel de l’outil**, ajoutez le code suivant :

    ```python
   # Submit the tool call output
   agents_client.runs.submit_tool_outputs(thread_id=thread.id, run_id=run.id, tool_outputs=tool_outputs)
    ```

    Ce code signale au fil de discussion de l’agent que l’action requise est terminée et met à jour les sorties de l’appel de l’outil.

1. Recherchez le commentaire **Afficher la réponse** et ajoutez le code suivant :

    ```python
   # Display the response
   messages = agents_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
        if message.text_messages:
            last_msg = message.text_messages[-1]
            print(f"{message.role}:\n{last_msg.text.value}\n")
    ```

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

1. Lorsque vous y êtes invité, entrez une requête telle que :

    ```
   What are the current inventory levels?
    ```

    > **Conseil** : si l’application échoue en raison du dépassement de la limite de débit, patientez quelques secondes, puis réessayez. Si le quota disponible dans votre abonnement est insuffisant, le modèle peut ne pas être en mesure de répondre.

    Vous devez obtenir une sortie similaire à la à celle-ci :

    ```
    MessageRole.AGENT:
    Here are the current inventory levels:

    - Moisturizer: 6
    - Shampoo: 8
    - Body Spray: 28
    - Hair Gel: 5
    - Lip Balm: 12
    - Skin Serum: 9
    - Cleanser: 30
    - Conditioner: 3
    - Setting Powder: 17
    - Dry Shampoo: 45
    ```

1. Vous pouvez continuer la conversation si vous le souhaitez. Le thread est *avec état* et conserve donc l’historique des conversations. Cela signifie que l’agent a le contexte complet de chaque réponse. 

    Essayez d’entrer des invites telles que :

    ```
   Are there any products that should be restocked?
    ```

    ```
   Which products would you recommend for clearance?
    ```

    ```
   What are the best sellers this week?
    ```

    Lorsque vous avez terminé, entrez `quit`.

## Nettoyage

Maintenant que vous avez terminé l’exercice, vous devez supprimer les ressources cloud que vous avez créées pour éviter une utilisation inutile des ressources.

1. Ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et affichez le contenu du groupe de ressources où vous avez déployé les ressources de hub utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
