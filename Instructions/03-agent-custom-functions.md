---
lab:
  title: Utiliser une fonction personnalisée dans un agent IA
  description: Découvrez comment utiliser des fonctions pour ajouter des fonctionnalités personnalisées à vos agents.
---

# Utiliser une fonction personnalisée dans un agent IA

Dans cet exercice, vous allez explorer la création d’un agent qui peut utiliser des fonctions personnalisées en tant qu’outil pour effectuer des tâches.

Vous allez créer un agent de support technique simple qui peut collecter les détails d’un problème technique et générer un ticket de support.

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
    - **Emplacement** : choisissez une région dans la liste suivante :\*
        - eastus
        - eastus2
        - centre de la suède
        - westus
        - westus3
    - **Connecter Azure AI Services ou Azure OpenAI** : *créer une nouvelle ressource AI Services*
    - **Connecter la Recherche Azure AI** : ignorer la connexion

    > \* Au moment de l’écriture, ces régions prennent en charge le modèle gpt-4o à utiliser dans les agents. La disponibilité des modèles est limitée par les quotas régionaux. Si une limite de quota est atteinte plus tard dans l’exercice, vous devrez peut-être créer un autre projet dans une autre région.

1. Sélectionnez **Suivant** et passez en revue votre configuration. Sélectionnez **Créer** et patientez jusqu’à ce que l’opération se termine.
1. Une fois votre projet créé, fermez les conseils affichés et passez en revue la page du projet dans le portail Azure AI Foundry, qui doit ressembler à l’image suivante :

    ![Capture d’écran des détails d’un projet Azure AI dans le portail Azure AI Foundry.](./Media/ai-foundry-project.png)

## Déployer un modèle d’IA générative

Vous êtes maintenant prêt à déployer un modèle de langage d’IA générative pour prendre en charge votre agent.

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

## Développer un agent qui utilise des outils de fonction

Maintenant que vous avez créé votre projet dans AI Foundry, nous allons développer une application qui implémente un agent à l’aide d’outils de fonction personnalisés.

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

    > **Conseil** : lorsque vous entrez des commandes dans le Cloud Shell, la sortie peut occuper une grande partie de la mémoire tampon d’écran et le curseur de la ligne en cours peut être masqué. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Entrez la commande suivante pour modifier le répertoire de travail vers le dossier contenant les fichiers de code, puis les répertorier tous.

    ```
   cd ai-agents/Labfiles/03-ai-agent-functions/Python
   ls -a -l
    ```

    Les fichiers fournis incluent le code d’application et un fichier pour les paramètres de configuration.

### Configurer les paramètres de l’application

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous allez utiliser :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects
    ```

    >**Note :** vous pouvez ignorer les messages d’avertissement ou d’erreur affichés pendant l’installation de la bibliothèque.

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez l’espace réservé **your_project_connection_string** par la chaîne de connexion de votre projet (copiée depuis la page **Vue d’ensemble** du projet dans le portail Azure AI Foundry), et remplacez **your_model_deployment** par le nom que vous avez attribué à votre déploiement du modèle gpt-4o.
1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Définir une fonction personnalisée

1. Saisissez la commande suivante pour modifier le fichier de code fourni pour votre code de fonction :

    ```
   code user_functions.py
    ```

1. Recherchez le commentaire **Créer une fonction pour envoyer un ticket de support** et ajoutez le code suivant, qui génère un numéro de ticket et enregistre un ticket de support sous forme de fichier texte.

    ```python
   # Create a function to submit a support ticket
   def submit_support_ticket(email_address: str, description: str) -> str:
        script_dir = Path(__file__).parent  # Get the directory of the script
        ticket_number = str(uuid.uuid4()).replace('-', '')[:6]
        file_name = f"ticket-{ticket_number}.txt"
        file_path = script_dir / file_name
        text = f"Support ticket: {ticket_number}\nSubmitted by: {email_address}\nDescription:\n{description}"
        file_path.write_text(text)
    
        message_json = json.dumps({"message": f"Support ticket {ticket_number} submitted. The ticket file is saved as {file_name}"})
        return message_json
    ```

1. Recherchez le commentaire **Définir un ensemble de fonctions pouvant être appelées** et ajoutez le code suivant, qui définit statiquement un ensemble de fonctions pouvant être appelées dans ce fichier de code (dans ce cas, il n’en existe qu’un seul, mais dans une solution réelle, vous pouvez avoir plusieurs fonctions que votre agent peut appeler) :

    ```python
   # Define a set of callable functions
   user_functions: Set[Callable[..., Any]] = {
        submit_support_ticket
    }
    ```
1. Enregistrez le fichier (*CTRL+S*).

### Écrire du code pour implémenter un agent qui peut utiliser votre fonction

1. Exécutez la commande suivante pour commencer à modifier le code d’agent.

    ```
    code agent.py
    ```

    > **Conseil** : lorsque vous ajoutez du code au fichier de code, veillez à conserver la mise en retrait correcte.

1. Passez en revue le code existant, qui récupère les paramètres de configuration de l’application et configure une boucle dans laquelle l’utilisateur peut entrer des invites pour l’agent. Le reste du fichier inclut des commentaires dans lesquels vous allez ajouter le code nécessaire pour implémenter votre agent de support technique.
1. Recherchez le commentaire **Ajouter des références** et ajoutez le code suivant pour importer les classes dont vous aurez besoin pour créer un agent Azure AI qui utilise votre code de fonction en tant qu’outil :

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import FunctionTool, ToolSet
   from user_functions import user_functions
    ```

1. Recherchez le commentaire **Se connecter au projet Azure AI Foundry**, puis ajoutez le code suivant pour vous connecter à votre projet Azure AI Foundry à l’aide des identifiants Azure que vous utilisez actuellement :

    > **Conseil** : veillez à respecter le niveau de mise en retrait correct.

    ```python
   # Connect to the Azure AI Foundry project
   project_client = AIProjectClient.from_connection_string(
        credential=DefaultAzureCredential
            (exclude_environment_credential=True,
             exclude_managed_identity_credential=True),
        conn_str=PROJECT_CONNECTION_STRING
   )
    ```
    
1. Recherchez la section de commentaire **Définir un agent pouvant utiliser les fonctions personnalisées**, puis ajoutez le code suivant pour ajouter votre code de fonction à un ensemble d’outils. Ensuite, créez un agent qui peut utiliser l’ensemble d’outils, ainsi qu’un thread sur lequel exécuter la session de conversation.

    ```python
   # Define an agent that can use the custom functions
   with project_client:

        functions = FunctionTool(user_functions)
        toolset = ToolSet()
        toolset.add(functions)
        project_client.agents.enable_auto_function_calls(toolset=toolset)
            
        agent = project_client.agents.create_agent(
            model=MODEL_DEPLOYMENT,
            name="support-agent",
            instructions="""You are a technical support agent.
                            When a user has a technical issue, you get their email address and a description of the issue.
                            Then you use those values to submit a support ticket using the function available to you.
                            If a file is saved, tell the user the file name.
                         """,
            toolset=toolset
        )

        thread = project_client.agents.create_thread()
        print(f"You're chatting with: {agent.name} ({agent.id})")

    ```

1. Recherchez le commentaire **Envoyer une invite à l’agent** et ajoutez le code suivant pour ajouter l’invite de l’utilisateur en tant que message et exécuter le thread.

    ```python
   # Send a prompt to the agent
   message = project_client.agents.create_message(
        thread_id=thread.id,
        role="user",
        content=user_prompt
   )
   run = project_client.agents.create_and_process_run(thread_id=thread.id, agent_id=agent.id)
    ```

    > **Note** : l’utilisation de la méthode **create_and_process_run** pour exécuter le thread permet à l’agent de rechercher automatiquement vos fonctions et de choisir de les utiliser en fonction de leurs noms et paramètres. En guise d’alternative, vous pouvez utiliser la méthode **create_run**, auquel cas vous devez écrire du code pour interroger l’état d’exécution afin de déterminer quand un appel de fonction est requis, appeler la fonction et renvoyer les résultats à l’agent.

1. Recherchez le commentaire **Vérifier l’état d’exécution des échecs** et ajoutez le code suivant pour afficher les erreurs qui se produisent.

    ```python
   # Check the run status for failures
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")
    ```

1. Recherchez le commentaire **Afficher la dernière réponse de l’agent** et ajoutez le code suivant pour récupérer les messages du thread terminé et afficher le dernier envoyé par l’agent.

    ```python
   # Show the latest response from the agent
   messages = project_client.agents.list_messages(thread_id=thread.id)
   last_msg = messages.get_last_text_message_by_role("assistant")
   if last_msg:
        print(f"Last Message: {last_msg.text.value}")
    ```

1. Recherchez le commentaire **Obtenir l’historique des conversations**, qui se trouve après la fin de la boucle et ajoutez le code suivant pour imprimer les messages du thread de conversation. Inversez l’ordre pour les afficher dans une séquence chronologique.

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = project_client.agents.list_messages(thread_id=thread.id)
   for message_data in reversed(messages.data):
        last_message_content = message_data.content[-1]
        print(f"{message_data.role}: {last_message_content.text.value}\n")
    ```

1. Recherchez le commentaire **Nettoyer** et ajoutez le code suivant pour supprimer l’agent et le thread quand cela n’est plus nécessaire.

    ```python
   # Clean up
   project_client.agents.delete_agent(agent.id)
   project_client.agents.delete_thread(thread.id)
    ```

1. Passez en revue le code en utilisant les commentaires pour comprendre comment il effectue les tâches suivantes :
    - Ajoute votre ensemble de fonctions personnalisées à un ensemble d’outils.
    - Crée un agent qui utilise l’ensemble d’outils.
    - Exécute un thread avec un message d’invite de l’utilisateur.
    - Vérifie l’état de l’exécution en cas d’échec.
    - Récupère les messages du thread terminé et affiche le dernier envoyé par l’agent.
    - Affiche l’historique des conversations.
    - Supprime l’agent et le thread lorsqu’ils ne sont plus nécessaires.

1. Enregistrez le fichier de code (*CTRL+S*) lorsque vous avez terminé. Vous pouvez également fermer l’éditeur de code (*Ctrl+Q*). Vous pouvez toutefois le laisser ouvert, si vous devez apporter des modifications au code que vous avez ajouté. Dans les deux cas, maintenez le volet de ligne de commande cloud Shell ouvert.

### Connectez-vous à Azure et exécutez votre application.

1. Dans le volet en ligne de commande du Cloud Shell, saisissez la commande suivante pour vous connecter à Azure :

    ```
    az login
    ```

    **<font color="red">Vous devez vous connecter à Azure, même si la session Cloud Shell est déjà authentifiée.</font>**

    > **Note** : dans la plupart des scénarios, l’utilisation d’*az login* suffit. Toutefois, si vous avez des abonnements dans plusieurs locataires, vous devrez peut-être spécifier le locataire à l’aide du paramètre *--tenant*. Pour plus d’informations, consultez [Se connecter à Azure de manière interactive à l’aide d’Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Lorsque vous y êtes invité, suivez les instructions pour ouvrir la page de connexion dans un nouvel onglet et entrez le code d’authentification fourni et vos informations d’identification Azure. Terminez ensuite le processus de connexion dans la ligne de commande, en sélectionnant l’abonnement contenant votre hub Azure AI Foundry si vous y êtes invité.
1. Une fois connecté, entrez la commande suivante pour exécuter l’application :

    ```
   python agent.py
    ```
    
    L’application s’exécute à l’aide des informations d’identification de votre session Azure authentifiée pour vous connecter à votre projet et créer et exécuter l’agent.

1. Lorsque vous y êtes invité, entrez une invite telle que :

    ```
   I have a technical problem
    ```

    > **Conseil** : si l’application échoue en raison du dépassement de la limite de débit, patientez quelques secondes, puis réessayez. Si le quota disponible dans votre abonnement est insuffisant, le modèle peut ne pas être en mesure de répondre.

1. Affichez la réponse. L’agent peut demander votre adresse e-mail et une description du problème. Vous pouvez utiliser n’importe quelle adresse e-mail (par exemple, `alex@contoso.com`) et n’importe quelle description de problème (par exemple, `my computer won't start`).

    Lorsqu’il dispose d’informations suffisantes, l’agent doit choisir d’utiliser votre fonction selon les besoins.

1. Vous pouvez continuer la conversation si vous le souhaitez. Le thread est *avec état*. Il conserve donc l’historique des conversations, ce qui signifie que l’agent a le contexte complet de chaque réponse. Lorsque vous avez terminé, entrez `quit`.
1. Passez en revue les messages de conversation récupérés à partir du thread et les tickets générés.
1. L’outil doit avoir enregistré des tickets de support dans le dossier de l’application. Vous pouvez utiliser la commande `ls` pour vérifier, puis utiliser la commande `cat` pour afficher le contenu du fichier, comme suit :

    ```
   cat ticket-<ticket_num>.txt
    ```

## Nettoyage

Maintenant que vous avez terminé l’exercice, vous devez supprimer les ressources cloud que vous avez créées pour éviter une utilisation inutile des ressources.

1. Ouvrez le [portail Azure](https://portal.azure.com) sur `https://portal.azure.com` et affichez le contenu du groupe de ressources dans lequel vous avez déployé les ressources utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.