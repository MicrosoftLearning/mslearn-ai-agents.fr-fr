---
lab:
  title: Développer un agent IA
  description: Utilisez Azure AI Agent Service pour développer un agent qui utilise des outils intégrés.
---

# Développer un agent IA

Dans cet exercice, vous allez utiliser Azure AI Agent Service pour créer un agent simple qui analyse les données et crée des graphiques. L’agent peut utiliser l’outil intégré *Interpréteur de code* pour générer dynamiquement tout code nécessaire à l’analyse de données.

> **Conseil** : Le code utilisé dans cet exercice est basé sur le Kit de développement logiciel (SDK) Azure AI Foundry pour Python. Vous pouvez développer des solutions similaires à l’aide des Kits de développement logiciel (SDK) pour Microsoft .NET, JavaScript et Java. Pour plus d’informations, reportez-vous aux [bibliothèques clientes du Kit de développement logiciel (SDK) Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview).

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
    - **Région** : *sélectionnez n’importe quel emplacement pris en charge par les services d’IA***\*

    > \* Certaines ressources Azure AI sont limitées par des quotas de modèles régionaux. Si une limite de quota est atteinte plus tard dans l’exercice, vous devrez peut-être créer une autre ressource dans une autre région.

1. Sélectionnez **Créer** et attendez que votre projet soit créé.
1. Si vous y êtes invité, déployez un modèle **gpt-4o** en utilisant l’option de déploiement *Standard global* ou *Standard* (selon la disponibilité de votre quota).

    >**Remarque** : Si un quota est disponible, un modèle de base GPT-4o peut être déployé automatiquement lors de la création de votre Agent et de votre projet.

1. Une fois votre projet créé, le terrain de jeu Agents est ouvert.

1. Dans le volet de navigation à gauche, sélectionnez **Vue d’ensemble** pour accéder à la page principale de votre projet ; elle se présente comme suit :

    ![Capture d’écran d’une page de présentation d’un projet Azure AI Foundry.](./Media/ai-foundry-project.png)

1. Copiez les valeurs du **point de terminaison du projet Azure AI Foundry** dans un bloc-notes, car vous les utiliserez pour vous connecter à votre projet dans une application cliente.

## Créer une application cliente agent

Vous êtes maintenant prêt à créer une application cliente qui utilise un agent. Tout le code dont vous avez besoin est fourni dans un référentiel GitHub.

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
   cd ai-agents/Labfiles/02-build-ai-agent/Python
   ls -a -l
    ```

    Les fichiers fournis incluent le code d’application, les paramètres de configuration et les données.

### Configurer les paramètres de l’application

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous allez utiliser :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects
    ```

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez l’espace réservé **your_project_endpoint** par le point de terminaison de votre projet (copié à partir de la page **Vue d’ensemble** du projet dans le portail Azure AI Foundry) et vérifiez que la variable MODEL_DEPLOYMENT_NAME est définie sur votre nom de modèle de déploiement (qui doit être *gpt-4o*).
1. Une fois que vous avez remplacé l’espace réservé, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Écrire du code pour une application agent

> **Conseil** : lorsque vous ajoutez du code, veillez à conserver la mise en retrait correcte. Utilisez les niveaux de mise en retrait des commentaires comme guide.

1. Saisissez la commande suivante pour modifier le fichier de code fourni :

    ```
   code agent.py
    ```

1. Passez en revue le code existant, qui récupère les paramètres de configuration de l’application et charge les données de *data.txt* à analyser. Le reste du fichier inclut des commentaires dans lesquels vous allez ajouter le code nécessaire pour implémenter votre agent d’analyse de données.
1. Recherchez le commentaire **Ajouter des références** et ajoutez le code suivant pour importer les classes dont vous aurez besoin pour créer un agent Azure AI qui utilise l’outil interpréteur de code intégré :

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import FilePurpose, CodeInterpreterTool, ListSortOrder, MessageRole
    ```

1. Recherchez le commentaire **Se connecter au client de l’agent**, puis ajoutez le code suivant pour vous connecter au projet Azure AI.

    > **Conseil** : veillez à respecter le niveau de mise en retrait correct.

    ```python
   # Connect to the Agent client
   agent_client = AgentsClient(
       endpoint=project_endpoint,
       credential=DefaultAzureCredential
           (exclude_environment_credential=True,
            exclude_managed_identity_credential=True)
   )
   with agent_client:
    ```

    Le code se connecte au projet Azure AI Foundry à l’aide des informations d’identification Azure actuelles. La dernière instruction *with agent_client* démarre un bloc de code qui définit l’étendue du client, en veillant à ce qu’il soit nettoyé lorsque le code dans le bloc est terminé.

1. Recherchez le commentaire **Charger le fichier de données et créer un CodeInterpreterTool**, dans le bloc *with agent_client*, puis ajoutez le code suivant pour charger le fichier de données dans le projet et créer un CodeInterpreterTool qui peut accéder aux données dans celui-ci :

    ```python
   # Upload the data file and create a CodeInterpreterTool
   file = agent_client.files.upload_and_poll(
        file_path=file_path, purpose=FilePurpose.AGENTS
   )
   print(f"Uploaded {file.filename}")

   code_interpreter = CodeInterpreterTool(file_ids=[file.id])
    ```
    
1. Recherchez le commentaire **Définir un agent qui utilise le CodeInterpreterTool** et ajoutez le code suivant pour définir un agent IA qui analyse les données et peut utiliser l’outil d’interpréteur de code que vous avez défini précédemment :

    ```python
   # Define an agent that uses the CodeInterpreterTool
   agent = agent_client.create_agent(
        model=model_deployment,
        name="data-agent",
        instructions="You are an AI agent that analyzes the data in the file that has been uploaded. Use Python to calculate statistical metrics as necessary.",
        tools=code_interpreter.definitions,
        tool_resources=code_interpreter.resources,
   )
   print(f"Using agent: {agent.name}")
    ```

1. Recherchez le commentaire **Créer un thread pour la conversation** et ajoutez le code suivant pour démarrer un thread sur lequel la session de conversation avec l’agent s’exécute :

    ```python
   # Create a thread for the conversation
   thread = agent_client.threads.create()
    ```
    
1. Notez que la section suivante du code configure une boucle pour qu’un utilisateur entre une invite. Elle se termine quand l’utilisateur entre « quit ».

1. Recherchez le commentaire **Envoyer une invite à l’agent** et ajoutez le code suivant pour ajouter un message utilisateur à l’invite (ainsi que les données du fichier chargé précédemment), puis exécutez le thread avec l’agent.

    ```python
   # Send a prompt to the agent
   message = agent_client.messages.create(
        thread_id=thread.id,
        role="user",
        content=user_prompt,
    )

   run = agent_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
    ```

1. Trouvez le commentaire **Vérifiez l’état d’exécution pour détecter les échecs** et ajoutez le code suivant pour rechercher d’éventuelles erreurs.

    ```python
   # Check the run status for failures
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")
    ```

1. Recherchez le commentaire **Afficher la dernière réponse de l’agent** et ajoutez le code suivant pour récupérer les messages du thread terminé et afficher le dernier envoyé par l’agent.

    ```python
   # Show the latest response from the agent
   last_msg = agent_client.messages.get_last_message_text_by_role(
       thread_id=thread.id,
       role=MessageRole.AGENT,
   )
   if last_msg:
       print(f"Last Message: {last_msg.text.value}")
    ```

1. Recherchez le commentaire **Obtenir l’historique des conversations**, qui se trouve après la fin de la boucle et ajoutez le code suivant pour imprimer les messages du thread de conversation. Inversez l’ordre pour les afficher dans une séquence chronologique.

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = agent_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
       if message.text_messages:
           last_msg = message.text_messages[-1]
           print(f"{message.role}: {last_msg.text.value}\n")
    ```

1. Recherchez le commentaire **Nettoyer** et ajoutez le code suivant pour supprimer l’agent et le thread quand cela n’est plus nécessaire.

    ```python
   # Clean up
   agent_client.delete_agent(agent.id)
    ```

1. Passez en revue le code en utilisant les commentaires pour comprendre comment il effectue les tâches suivantes :
    - Se connecte au projet AI Foundry.
    - Charge le fichier de données et crée un outil interpréteur de code qui peut y accéder.
    - Crée un agent qui utilise l’outil d’interpréteur de code et contient des instructions explicites pour utiliser Python si nécessaire pour l’analyse statistique.
    - Exécute un thread avec un message d’invite de l’utilisateur, ainsi que les données à analyser.
    - Vérifie le statut de l’exécution en cas d’échec.
    - Récupère les messages du thread terminé et affiche le dernier envoyé par l’agent.
    - Affiche l’historique des conversations.
    - Supprime l’agent et le thread lorsqu’ils ne sont plus nécessaires.

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
    python agent.py
    ```
    
    L’application s’exécute à l’aide des informations d’identification de votre session Azure authentifiée pour vous connecter à votre projet, et créer et exécuter l’agent.

1. Lorsque l’invite apparaît, affichez les données chargées par l’application à partir du fichier texte *data.txt*. Entrez ensuite une invite tel que :

    ```
   What's the category with the highest cost?
    ```

    > **Conseil** : si l’application échoue en raison du dépassement de la limite de débit, patientez quelques secondes, puis réessayez. Si le quota disponible dans votre abonnement est insuffisant, le modèle peut ne pas être en mesure de répondre.

1. Affichez la réponse. Entrez ensuite une autre invite, cette fois en demandant une visualisation :

    ```
   Create a text-based bar chart showing cost by category
    ```

1. Affichez la réponse. Entrez ensuite une autre invite, cette fois en demandant une métrique statistique :

    ```
   What's the standard deviation of cost?
    ```

    Affichez la réponse.

1. Vous pouvez continuer la conversation si vous le souhaitez. Le thread est *avec état* et conserve donc l’historique des conversations. Cela signifie que l’agent a le contexte complet de chaque réponse. Lorsque vous avez terminé, entrez `quit`.
1. Passez en revue les messages de conversation récupérés à partir de la conversation, qui peuvent inclure les messages générés par l’agent pour expliquer ses étapes lors de l’utilisation de l’outil d’interpréteur de code.

## Résumé

Dans cet exercice, vous avez utilisé le kit de développement logiciel (SDK) Azure AI Agent Service pour créer une application cliente qui utilise un agent IA. L’agent peut utiliser l’outil interpréteur de code intégré pour exécuter du code Python dynamique pour effectuer des analyses statistiques.

## Nettoyage

Si vous avez terminé d’explorer Azure AI Agent Service, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter de générer des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources dans lequel vous avez déployé les ressources utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
