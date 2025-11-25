---
lab:
  title: Développez une solution multi-assistants avec Microsoft Foundry
  description: Découvrez comment configurer plusieurs assistants pour collaborer à l’aide du service de l’agent Foundry
---

# Développer une solution multi-agent

Dans cet exercice, vous allez créer un projet qui orchestre plusieurs assistants IA à l’aide du service de l’agent Microsoft Foundry. Vous allez concevoir une solution IA qui facilitera le triage des tickets. Les agents connectés évaluent la priorité du ticket, suggèrent une affectation d’équipe et déterminent le niveau d’effort nécessaire pour terminer le ticket. C’est parti !

> **Conseil** : Le code utilisé dans cet exercice est basé sur le kit de développement logiciel (SDK) Foundry pour Python. Vous pouvez développer des solutions similaires à l’aide des kits de développement logiciel (SDK) pour Microsoft .NET, JavaScript et Java. Pour plus d’informations, consultez [Bibliothèques clientes du kit de développement logiciel (SDK) Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview).

Cet exercice devrait prendre environ **30** minutes.

> **Note** : certaines des technologies utilisées dans cet exercice sont en version préliminaire ou en cours de développement. Un comportement inattendu, des avertissements ou des erreurs peuvent se produire.

## Créer un projet Foundry

Commençons par créer un projet Foundry.

1. Dans un navigateur web, ouvrez le [portail Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure. Fermez les conseils ou les volets de démarrage rapide ouverts la première fois que vous vous connectez et, si nécessaire, utilisez le logo **Foundry** en haut à gauche pour accéder à la page d’accueil, qui ressemble à l’image suivante (fermez le volet **Aide** s’il est ouvert) :

    ![Capture d’écran du portail Foundry.](./Media/ai-foundry-home.png)

    > **Important** : Assurez-vous que le bouton bascule **Nouveau Foundry** est *désactivé* pour ce labo.

1. Sur la page d’accueil, sélectionnez **Créer un agent**.
1. Lorsque vous êtes invité à créer un projet, entrez un nom valide pour votre projet et développez les **options avancées**.
1. Confirmez les paramètres suivants pour votre projet :
    - **Ressource Foundry** : *Nom valide de votre ressource Foundry*
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Région** : *Sélectionnez n’importe quelle **recommandation d’AI Foundry***\*

    > \* Certaines ressources Azure AI sont limitées par des quotas de modèles régionaux. Si une limite de quota est atteinte plus tard dans l’exercice, vous devrez peut-être créer une autre ressource dans une autre région.

1. Sélectionnez **Créer** et attendez que votre projet soit créé.
1. Si vous y êtes invité, déployez un modèle **gpt-4o** en utilisant l’option de déploiement *Standard global* ou *Standard* (selon la disponibilité de votre quota).

    >**Remarque** : Si un quota est disponible, un modèle de base GPT-4o peut être déployé automatiquement lors de la création de votre Agent et de votre projet.

1. Une fois votre projet créé, le terrain de jeu Agents est ouvert.

1. Dans le volet de navigation à gauche, sélectionnez **Vue d’ensemble** pour accéder à la page principale de votre projet ; elle se présente comme suit :

    ![Capture d’écran d’une page de vue d’ensemble d’un projet Foundry.](./Media/ai-foundry-project.png)

1. Copiez les valeurs **Point de terminaison du projet Foundry** dans un bloc-notes, car vous les utiliserez pour vous connecter à votre projet dans une application cliente.

## Créer une application cliente de l’agent IA

Vous êtes maintenant prêt à créer une application cliente qui définit les agents et les instructions. Nous vous avons fourni du code dans un référentiel GitHub.

### Préparer l’environnement

1. Ouvrez un nouvel onglet de navigateur (en gardant le portail Foundry ouvert dans l’onglet existant). Dans un nouvel onglet du navigateur, ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.

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
   cd ai-agents/Labfiles/03b-build-multi-agent-solution/Python
   ls -a -l
    ```

    Les fichiers fournis incluent le code d’application et un fichier pour les paramètres de configuration.

### Configurer les paramètres de l’application

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous allez utiliser :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects azure-ai-agents
    ```

1. Entrez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez l’espace réservé **your_project_endpoint** par le point de terminaison de votre projet (copié depuis la page **Vue d’ensemble** du projet dans le portail Foundry), et l’espace réservé **your_model_deployment** par le nom que vous avez attribué à votre modèle de déploiement gpt-4o (qui est `gpt-4o` par défaut).

1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Créer des agents IA

Vous avez effectué toutes les préparation nécessaires à la création d’agents pour votre solution multi-agent ! C’est parti !

1. Entrez la commande suivante pour modifier le fichier **agent_triage.py** :

    ```
   code agent_triage.py
    ```

1. Passez en revue le code dans le fichier, en notant qu’il contient des chaînes pour chaque nom et instructions de l’agent.

1. Recherchez le commentaire **Ajouter des références** et ajoutez le code suivant pour importer les classes dont vous aurez besoin :

    ```python
   # Add references
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import ConnectedAgentTool, MessageRole, ListSortOrder, ToolSet, FunctionTool
   from azure.identity import DefaultAzureCredential
    ```

1. Notez que le code permettant de charger le point de terminaison du projet et le nom du modèle à partir de vos variables d’environnement a été fourni.

1. Recherchez le commentaire **Se connecter au client des agents** et ajoutez le code suivant pour créer un AgentClient connecté à votre projet :

    ```python
   # Connect to the agents client
   agents_client = AgentsClient(
        endpoint=project_endpoint,
        credential=DefaultAzureCredential(
            exclude_environment_credential=True, 
            exclude_managed_identity_credential=True
        ),
   )
    ```

    Vous allez maintenant ajouter du code qui utilise AgentsClient pour créer plusieurs agents, chacun avec un rôle spécifique à jouer dans le traitement d’un ticket de support.

    > **Conseil** : Lors de l’ajout du code suivant, veillez à conserver le niveau de retrait approprié sous l’instruction `using agents_client:`.

1. Recherchez le commentaire **Créer un agent pour hiérarchiser les tickets de support** et entrez le code suivant (veillez à conserver le niveau de retrait approprié) :

    ```python
   # Create an agent to prioritize support tickets
   priority_agent_name = "priority_agent"
   priority_agent_instructions = """
   Assess how urgent a ticket is based on its description.

   Respond with one of the following levels:
   - High: User-facing or blocking issues
   - Medium: Time-sensitive but not breaking anything
   - Low: Cosmetic or non-urgent tasks

   Only output the urgency level and a very brief explanation.
   """

   priority_agent = agents_client.create_agent(
        model=model_deployment,
        name=priority_agent_name,
        instructions=priority_agent_instructions
   )
    ```

1. Recherchez le commentaire **Créer un agent pour affecter des tickets à l’équipe appropriée**, puis entrez le code suivant :

    ```python
   # Create an agent to assign tickets to the appropriate team
   team_agent_name = "team_agent"
   team_agent_instructions = """
   Decide which team should own each ticket.

   Choose from the following teams:
   - Frontend
   - Backend
   - Infrastructure
   - Marketing

   Base your answer on the content of the ticket. Respond with the team name and a very brief explanation.
   """

   team_agent = agents_client.create_agent(
        model=model_deployment,
        name=team_agent_name,
        instructions=team_agent_instructions
   )
    ```

1. Recherchez le commentaire **Créer un agent pour estimer l’effort d’un ticket de support**, puis entrez le code suivant :

    ```python
   # Create an agent to estimate effort for a support ticket
   effort_agent_name = "effort_agent"
   effort_agent_instructions = """
   Estimate how much work each ticket will require.

   Use the following scale:
   - Small: Can be completed in a day
   - Medium: 2-3 days of work
   - Large: Multi-day or cross-team effort

   Base your estimate on the complexity implied by the ticket. Respond with the effort level and a brief justification.
   """

   effort_agent = agents_client.create_agent(
        model=model_deployment,
        name=effort_agent_name,
        instructions=effort_agent_instructions
   )
    ```

    Jusqu’à présent, vous avez créé trois agents ; chacun d’entre eux a un rôle spécifique dans le triage d’un ticket de support. Nous allons maintenant créer des objets ConnectedAgentTool pour chacun de ces agents afin qu’ils puissent être utilisés par d’autres agents.

1. Recherchez le commentaire **Créer des outils d’agent connecté pour les agents de support**, puis entrez le code suivant :

    ```python
   # Create connected agent tools for the support agents
   priority_agent_tool = ConnectedAgentTool(
        id=priority_agent.id, 
        name=priority_agent_name, 
        description="Assess the priority of a ticket"
   )
    
   team_agent_tool = ConnectedAgentTool(
        id=team_agent.id, 
        name=team_agent_name, 
        description="Determines which team should take the ticket"
   )
    
   effort_agent_tool = ConnectedAgentTool(
        id=effort_agent.id, 
        name=effort_agent_name, 
        description="Determines the effort required to complete the ticket"
   )
    ```

    Vous êtes maintenant prêt à créer un agent principal qui coordonnera le processus de triage des tickets à l’aide des agents connectés en fonction des besoins.

1. Recherchez le commentaire **Créer un agent pour trier le traitement des tickets de support à l’aide d’agents connectés**, puis entrez le code suivant :

    ```python
   # Create an agent to triage support ticket processing by using connected agents
   triage_agent_name = "triage-agent"
   triage_agent_instructions = """
   Triage the given ticket. Use the connected tools to determine the ticket's priority, 
   which team it should be assigned to, and how much effort it may take.
   """

   triage_agent = agents_client.create_agent(
        model=model_deployment,
        name=triage_agent_name,
        instructions=triage_agent_instructions,
        tools=[
            priority_agent_tool.definitions[0],
            team_agent_tool.definitions[0],
            effort_agent_tool.definitions[0]
        ]
   )
    ```

    Maintenant que vous avez défini un agent principal, vous pouvez lui envoyer une invite et lui demander d’utiliser les autres agents pour trier un problème de support.

1. Recherchez le commentaire **Utiliser les agents pour trier un problème de support** et entrez le code suivant :

    ```python
   # Use the agents to triage a support issue
   print("Creating agent thread.")
   thread = agents_client.threads.create()  

   # Create the ticket prompt
   prompt = input("\nWhat's the support problem you need to resolve?: ")
    
   # Send a prompt to the agent
   message = agents_client.messages.create(
        thread_id=thread.id,
        role=MessageRole.USER,
        content=prompt,
   )   
    
   # Run the thread usng the primary agent
   print("\nProcessing agent thread. Please wait.")
   run = agents_client.runs.create_and_process(thread_id=thread.id, agent_id=triage_agent.id)
        
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")

   # Fetch and display messages
   messages = agents_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
        if message.text_messages:
            last_msg = message.text_messages[-1]
            print(f"{message.role}:\n{last_msg.text.value}\n")
   
    ```

1. Recherchez le commentaire **Nettoyer**et entrez le code suivant pour supprimer les agents lorsqu’ils ne sont plus nécessaires :

    ```python
   # Clean up
   print("Cleaning up agents:")
   agents_client.delete_agent(triage_agent.id)
   print("Deleted triage agent.")
   agents_client.delete_agent(priority_agent.id)
   print("Deleted priority agent.")
   agents_client.delete_agent(team_agent.id)
   print("Deleted team agent.")
   agents_client.delete_agent(effort_agent.id)
   print("Deleted effort agent.")
    ```
    

1. Utilisez la commande **Ctrl+S** pour enregistrer les modifications que vous avez apportées au fichier de code. Vous pouvez laisser le code ouvert (si vous devez le modifier pour corriger les erreurs) ou utiliser la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Se connecter à Azure et exécuter l’application

Vous avez effectué toutes les préparations nécessaires à l’exécution de votre code pour observer vos agents IA collaborer.

1. Dans le volet de la ligne de commande Cloud Shell, entrez la commande suivante pour vous connecter à Azure.

    ```
   az login
    ```

    **<font color="red">Vous devez vous connecter à Azure, même si la session Cloud Shell est déjà authentifiée.</font>**

    > **Remarque** :dans la plupart des scénarios, l’utilisation d’*az login* suffit. Toutefois, si vous avez des abonnements dans plusieurs locataires, vous devrez peut-être spécifier le locataire à l’aide du paramètre *--tenant*. Pour plus d’informations, consultez [Se connecter à Azure de manière interactive à l’aide d’Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).

1. Lorsque l’invite apparaît, suivez les instructions pour ouvrir la page de connexion dans un nouvel onglet et entrez le code d’authentification fourni ainsi que vos informations d’identification Azure. Effectuez ensuite le processus de connexion dans la ligne de commande, en sélectionnant l’abonnement contenant votre hub Foundry si nécessaire.

1. Une fois la connexion effectuée, entrez la commande suivante pour exécuter l’application :

    ```
   python agent_triage.py
    ```

1. Entrez une invite, par exemple `Users can't reset their password from the mobile app.`.

    Une fois que les agents traitent l’invite, vous devriez voir une sortie similaire à ce qui suit :

    ```output
    Creating agent thread.
    Processing agent thread. Please wait.

    MessageRole.USER:
    Users can't reset their password from the mobile app.

    MessageRole.AGENT:
    ### Ticket Assessment

    - **Priority:** High — This issue blocks users from resetting their passwords, limiting access to their accounts.
    - **Assigned Team:** Frontend Team — The problem lies in the mobile app's user interface or functionality.
    - **Effort Required:** Medium — Resolving this problem involves identifying the root cause, potentially updating the mobile app functionality, reviewing API/backend integration, and testing to ensure compatibility across Android/iOS platforms.

    Cleaning up agents:
    Deleted triage agent.
    Deleted priority agent.
    Deleted team agent.
    Deleted effort agent.
    ```

    Vous pouvez essayer de modifier l’invite à l’aide d’un autre scénario de ticket pour voir comment les agents collaborent. Par exemple, « Examiner les erreurs occasionnelles 502 à partir du point de terminaison de recherche. »

## Nettoyage

Si vous avez terminé d’explorer Azure AI Agent Service, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter de générer des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources dans lequel vous avez déployé les ressources utilisées dans cet exercice.

1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.

1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
