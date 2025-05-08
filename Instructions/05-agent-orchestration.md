---
lab:
  title: Développer une solution multi-agent
  description: Découvrez comment configurer plusieurs agents pour collaborer à l’aide du kit de développement logiciel (SDK) de noyau sémantique.
---

# Développer une solution multi-agent

Dans cet exercice, vous allez créer un projet qui orchestre deux agents IA à l’aide du kit de développement logiciel (SDK) de noyau sémantique. Un agent *gestionnaire d’incidents* analyse les fichiers journaux de service pour détecter les potentiels problèmes. S’il détecte un problème, le gestionnaire d’incidents recommande une action de résolution, un agent *assistant DevOps* reçoit la recommandation, appelle la fonction corrective et procède à la résolution. L’agent gestionnaire d’incidents passe ensuite en revue les journaux mis à jour pour vérifier que la résolution a fonctionné.

Pour cet exercice, vous disposez de quatre exemples de fichiers journaux. Le code de l’agent assistant DevOps met uniquement à jour les exemples de fichiers journaux avec des exemples de messages journaux.

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

Vous avez effectué toutes les préparations nécessaires au déploiement d’un modèle de langage d’IA générative pour vos agents.

1. Dans le volet de gauche de votre projet, dans la section **Mes ressources**, sélectionnez la page **Modèles + points de terminaison**.
1. Sur la page **Modèles + points de terminaison**, dans l’onglet **Déploiements de modèles**, dans le menu **+ Déployer un modèle**, sélectionnez **Déployer le modèle de base**.
1. Recherchez le modèle **gpt-4o** dans la liste, puis sélectionnez-le et confirmez.
1. Déployez le modèle avec les paramètres suivants en sélectionnant **Personnaliser** dans les détails du déploiement :
    - **Nom du déploiement** : *nom valide pour votre modèle de déploiement*
    - **Type de déploiement** : standard global
    - **Mise à jour automatique de la version** : activée
    - **Version du modèle** : *sélectionnez la version la plus récente disponible.*
    - **Ressource IA connectée** : *sélectionnez votre connexion de ressources Azure OpenAI*
    - **Limite de jetons par minute (en milliers)**  : 60 *(ou le maximum disponible dans votre abonnement si inférieur à 60 000)*
    - **Filtre de contenu** : DefaultV2

    > **Remarque** : La réduction du nombre de jetons par minute permet d’éviter une surutilisation du quota disponible dans l’abonnement que vous utilisez. 60 000 jetons par minute (TPM) sont suffisants pour les données utilisées dans cet exercice. Si votre quota disponible est inférieur à ce montant, vous serez en mesure d’effectuer l’exercice, mais vous devrez peut-être patienter et soumettre à nouveau les invites si la limite de jetons est dépassée.

1. Attendez la fin du déploiement.

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
   pip install python-dotenv azure-identity semantic-kernel[azure] 
    ```

    > **Remarque** : l’installation du *semantic-kernel[azure]* entraîne l’installation automatique d’une version compatible avec le noyau sémantique des *azure-ai-projects*.

1. Entrez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez l’espace réservé **your_project_connection_string** par la chaîne de connexion de votre projet (copiée depuis la page **Vue d’ensemble** du projet dans le portail Azure AI Foundry), et remplacez l’espace réservé **your_model_deployment** par le nom que vous avez attribué au déploiement du modèle gpt-4o.

1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Créer des agents IA

Vous avez effectué toutes les préparation nécessaires à la création d’agents pour votre solution multi-agent ! C’est parti !

1. Entrez la commande suivante pour modifier le fichier **agent_chat.py** :

    ```
   code agent_chat.py
    ```

1. Examinez le code dans le fichier et notez qu’il contient les éléments suivants :
    - Des constantes qui définissent les noms et les instructions de vos deux agents.
    - Une fonction **principale** dans laquelle sera ajoutée la majeur partie du code pour implémenter votre solution multi-agent.
    - Une classe **SelectionStrategy**, que vous allez utiliser pour implémenter la logique requise pour déterminer l’agent qui doit être sélectionné à chaque tour de la conversation.
    - Une classe **ApprovalTerminationStrategy**, que vous allez utiliser pour implémenter la logique requise pour déterminer quand la conversation se termine.
    - Une classe **DevopsPlugin** qui contient des fonctions pour effectuer des opérations DevOps.
    - Une classe **LogFilePlugin** qui contient des fonctions permettant de lire et d’écrire des fichiers journaux.

    Tout d’abord, vous allez créer l’agent *gestionnaire d’incidents* qui analyse les fichiers journaux du service, identifie les problèmes potentiels et recommande des actions de résolution ou fait remonter les problèmes si nécessaire.

1. Examinez la chaîne **INCIDENT_MANAGER_INSTRUCTIONS**. Elle contient les instructions pour votre agent.

1. Dans la fonction **principale**, recherchez le commentaire **Créer l’agent gestionnaire d’incidents sur Azure AI Agent Service** et ajoutez le code suivant pour créer un agent Azure AI.

    ```python
   # Create the incident manager agent on the Azure AI agent service
   incident_agent_definition = await client.agents.create_agent(
        model=ai_agent_settings.model_deployment_name,
        name=INCIDENT_MANAGER,
        instructions=INCIDENT_MANAGER_INSTRUCTIONS
   )
    ```

    Ce code crée la définition de l’agent sur votre client de projet Azure AI.

1. Recherchez le commentaire **Créer un agent de noyau sémantique pour l’agent gestionnaire d’incidents Azure AI** et ajoutez le code suivant pour créer un agent de noyau sémantique basé sur la définition de l’agent Azure AI.

    ```python
   # Create a Semantic Kernel agent for the Azure AI incident manager agent
   agent_incident = AzureAIAgent(
        client=client,
        definition=incident_agent_definition,
        plugins=[LogFilePlugin()]
   )
    ```

    Ce code crée l’agent de noyau sémantique avec un accès à **LogFilePlugin**. Ce plug-in permet à l’agent de lire le contenu du fichier journal.

    Nous allons maintenant créer le deuxième agent, qui répondra aux problèmes et effectuera des opérations DevOps pour les résoudre.

1. En haut du fichier de code, prenez un moment pour examiner la chaîne **DEVOPS_ASSISTANT_INSTRUCTIONS**. Elle contient les instructions que vous allez fournir au nouvel agent assistant DevOps.

1. Recherchez le commentaire **Créer l’agent DevOps sur Azure AI Agent Service** et ajoutez le code suivant pour créer une définition de l’agent Azure AI :
    
    ```python
   # Create the devops agent on the Azure AI agent service
   devops_agent_definition = await client.agents.create_agent(
        model=ai_agent_settings.model_deployment_name,
        name=DEVOPS_ASSISTANT,
        instructions=DEVOPS_ASSISTANT_INSTRUCTIONS,
   )
    ```

1. Recherchez le commentaire **Créer un agent de noyau sémantique pour l’agent Azure AI DevOps**, puis ajoutez le code suivant pour créer un agent de noyau sémantique basé sur la définition de l’agent Azure AI.
    
    ```python
   # Create a Semantic Kernel agent for the devops Azure AI agent
   agent_devops = AzureAIAgent(
        client=client,
        definition=devops_agent_definition,
        plugins=[DevopsPlugin()]
   )
    ```

    Le plug-in **DevopsPlugin** permet à l’agent de simuler des tâches DevOps, telles que le redémarrage du service ou la restauration d’une transaction.

### Définir des stratégies de conversation de groupe

Vous devez maintenant fournir la logique servant à déterminer l’agent qui doit être sélectionné pour prendre le tour suivant d’une conversation et le moment où la conversation doit se terminer.

Commençons par **SelectionStrategy**, qui identifie l’agent qui doit prendre le tour suivant.

1. Dans la classe **SelectionStrategy** (sous la fonction **principale**), recherchez le commentaire **Sélectionner l’agent qui doit prendre le tour suivant de la conversation**, puis ajoutez le code suivant pour définir une fonction de sélection :

    ```python
   # Select the next agent that should take the next turn in the chat
   async def select_agent(self, agents, history):
        """"Check which agent should take the next turn in the chat."""

        # The Incident Manager should go after the User or the Devops Assistant
        if (history[-1].name == DEVOPS_ASSISTANT or history[-1].role == AuthorRole.USER):
            agent_name = INCIDENT_MANAGER
            return next((agent for agent in agents if agent.name == agent_name), None)
        
        # Otherwise it is the Devops Assistant's turn
        return next((agent for agent in agents if agent.name == DEVOPS_ASSISTANT), None)
    ```

    Ce code s’exécute à chaque tour pour déterminer l’agent qui doit répondre, en vérifiant l’historique des conversations pour voir qui a répondu en dernier.

    Nous allons maintenant implémenter la classe **ApprovalTerminationStrategy** pour vous aider à signaler lorsque l’objectif est rempli et que la conversation peut conclure.

1. Dans la classe **ApprovalTerminationStrategy**, recherchez le commentaire **Terminer la conversation si l’agent a indiqué qu’aucune action n’est nécessaire** et ajoutez le code suivant pour définir la fonction de conclusion :

    ```python
   # End the chat if the agent has indicated there is no action needed
   async def should_agent_terminate(self, agent, history):
        """Check if the agent should terminate."""
        return "no action needed" in history[-1].content.lower()
    ```

    Le noyau appelle cette fonction après la réponse de l’agent pour déterminer si les critères pour terminer la conversation sont remplis. Dans ce cas, l’objectif est atteint lorsque le gestionnaire d’incidents répond « Aucune action nécessaire ». Cette expression est définie dans les instructions de l’agent gestionnaire d’incidents.

### Implémenter la conversation de groupe

Maintenant que vous avez deux agents et des stratégies pour les aider à prendre des tours et à terminer une conversation, vous pouvez implémenter la conversation de groupe.

1. Plus haut dans la fonction principale, recherchez le commentaire **Ajouter les agents à une conversation de groupe avec une stratégie de conclusion et de sélection personnalisée**, puis ajoutez le code suivant pour créer la conversation de groupe :

    ```python
   # Add the agents to a group chat with a custom termination and selection strategy
   chat = AgentGroupChat(
        agents=[agent_incident, agent_devops],
        termination_strategy=ApprovalTerminationStrategy(
            agents=[agent_incident], 
            maximum_iterations=10, 
            automatic_reset=True
        ),
        selection_strategy=SelectionStrategy(agents=[agent_incident,agent_devops]),      
   )
    ```

    Dans ce code, vous créez un objet de conversation de groupe avec l’agent gestionnaire d’incidents et l’agent DevOps. Vous définissez également les stratégies de conclusion et de sélection pour la conversation. Notez que la classe **ApprovalTerminationStrategy** est liée uniquement à l’agent gestionnaire d’incidents, et non à l’agent DevOps. Cela signifie que l’agent gestionnaire d’incidents est chargé de signaler la fin de la conversation. La classe **SelectionStrategy** inclut tous les agents qui doivent prendre un tour dans la conversation.

    Notez que l’indicateur de réinitialisation automatique efface automatiquement la conversation lorsqu’elle se termine. De cette façon, l’agent peut continuer à analyser les fichiers sans que l’objet d’historique des conversations n’utilise trop de jetons inutiles. 

1. Recherchez le commentaire **Ajouter le fichier journal actuel à la conversation** et incorporez le code suivant pour ajouter le texte du fichier journal lu en dernier à la conversation :

    ```python
   # Append the current log file to the chat
   await chat.add_chat_message(logfile_msg)
   print()
    ```

1. Recherchez le commentaire **Appeler une réponse des agents** et ajoutez le code suivant pour appeler la conversation de groupe :

    ```python
   # Invoke a response from the agents
   async for response in chat.invoke():
        if response is None or not response.name:
            continue
        print(f"{response.content}")
    ```

    Il s’agit du code qui déclenche la conversation. Étant donné que le texte du fichier journal a été ajouté en tant que message, la stratégie de sélection détermine l’agent qui doit le lire et y répondre. Ensuite, la conversation continue entre les agents jusqu’à ce que les conditions de la stratégie de conclusion soient remplies ou que le nombre maximal d’itérations soit atteint.

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
   python agent_chat.py
    ```

    Vous devez obtenir une sortie similaire à la suivante :

    ```output
    
    INCIDENT_MANAGER > /home/.../logs/log1.log | Restart service ServiceX
    DEVOPS_ASSISTANT > Service ServiceX restarted successfully.
    INCIDENT_MANAGER > No action needed.

    INCIDENT_MANAGER > /home/.../logs/log2.log | Rollback transaction for transaction ID 987654.
    DEVOPS_ASSISTANT > Transaction rolled back successfully.
    INCIDENT_MANAGER > No action needed.

    INCIDENT_MANAGER > /home/.../logs/log3.log | Increase quota.
    DEVOPS_ASSISTANT > Successfully increased quota.
    (continued)
    ```

    > **Note** : l’application inclut du code pour attendre entre le traitement de chaque fichier journal afin de réduire le risque de dépassement de la limite du taux de TPM ainsi qu’une gestion des exceptions en cas de problème. Si le quota disponible dans votre abonnement est insuffisant, le modèle peut ne pas être en mesure de répondre.

1. Vérifiez que les fichiers journaux dans le dossier des **journaux** incluent les messages d’opération de résolution de l’assistant DevOps.

    Par exemple, log1.log devrait inclure les messages de journal suivants :

    ```log
    [2025-02-27 12:43:38] ALERT  DevopsAssistant: Multiple failures detected in ServiceX. Restarting service.
    [2025-02-27 12:43:38] INFO  ServiceX: Restart initiated.
    [2025-02-27 12:43:38] INFO  ServiceX: Service restarted successfully.
    ```

## Résumé

Dans cet exercice, vous avez utilisé Azure AI Agent Service et le kit de développement logiciel (SDK) de noyau sémantique pour créer des agents IA d’incidents et DevOps capables de détecter et de résoudre automatiquement les problèmes. Beau travail !

## Nettoyage

Si vous avez terminé d’explorer Azure AI Agent Service, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter de générer des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources dans lequel vous avez déployé les ressources utilisées dans cet exercice.

1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.

1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.