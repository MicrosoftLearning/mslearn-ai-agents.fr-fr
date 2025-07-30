---
lab:
  title: Utiliser une fonction personnalisée dans un agent IA
  description: Découvrez comment utiliser des fonctions pour ajouter des fonctionnalités personnalisées à vos agents.
---

# Utiliser une fonction personnalisée dans un agent IA

Dans cet exercice, vous allez explorer la création d’un agent qui peut utiliser des fonctions personnalisées pour effectuer des tâches. Vous allez créer un agent de support technique simple capable de collecter des informations sur un problème technique et générer un ticket de support.

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

    > **Conseil** : lorsque vous saisissez des commandes dans le Cloud Shell, la sortie peut occuper une grande partie de la mémoire tampon d’écran et le curseur de la ligne actuelle peut être masqué. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

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
   pip install -r requirements.txt azure-ai-projects
    ```

    >**Remarque :** vous pouvez ignorer les messages d’avertissement ou d’erreur qui s’affichent pendant l’installation de la bibliothèque.

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez l’espace réservé **your_project_endpoint** par le point de terminaison de votre projet (copié à partir de la page **Vue d’ensemble** du projet dans le portail Azure AI Foundry) et vérifiez que la variable MODEL_DEPLOYMENT_NAME est définie sur votre nom de modèle de déploiement (qui doit être *gpt-4o*).
1. Une fois que vous avez remplacé l’espace réservé, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Définir une fonction personnalisée

1. Saisissez la commande suivante pour modifier le fichier de code fourni pour le code de la fonction :

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

1. Recherchez le commentaire **Définir un ensemble de fonctions pouvant être appelées** et ajoutez le code suivant, qui définit statiquement un ensemble de fonctions pouvant être appelées dans ce fichier de code (dans ce cas, il n’en existe qu’une seule, mais dans un scénario réel, vous pouvez avoir plusieurs fonctions que votre agent peut appeler) :

    ```python
   # Define a set of callable functions
   user_functions: Set[Callable[..., Any]] = {
        submit_support_ticket
    }
    ```
1. Enregistrez le fichier (*Ctrl+S*).

### Écrire du code pour implémenter un agent qui peut utiliser votre fonction

1. Exécutez la commande suivante pour commencer à modifier le code de l’agent.

    ```
    code agent.py
    ```

    > **Conseil** : lorsque vous ajoutez du code au fichier de code, veillez à conserver la bonne mise en retrait.

1. Passez en revue le code existant, qui récupère les paramètres de configuration d’application et définit une boucle dans laquelle l’utilisateur ou l’utilisatrice peut entrer des invites destinées à l’agent. Le reste du fichier contient des commentaires dans lesquels vous allez ajouter le code nécessaire pour implémenter votre agent de support technique.
1. Recherchez le commentaire **Ajouter des références** et ajoutez le code suivant pour importer les classes dont vous aurez besoin pour créer un agent Azure AI qui utilise votre code de fonction en tant qu’outil :

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import FunctionTool, ToolSet, ListSortOrder, MessageRole
   from user_functions import user_functions
    ```

1. Recherchez le commentaire **Se connecter au client de l’agent**, puis ajoutez le code suivant pour vous connecter au projet Azure AI à l’aide des informations d’identification Azure que vous utilisez actuellement.

    > **Conseil** : veillez à respecter le niveau de mise en retrait correct.

    ```python
   # Connect to the Agent client
   agent_client = AgentsClient(
       endpoint=project_endpoint,
       credential=DefaultAzureCredential
           (exclude_environment_credential=True,
            exclude_managed_identity_credential=True)
   )
    ```
    
1. Recherchez le commentaire **Définir un agent qui peut utiliser des fonctions personnalisées** et ajoutez le code suivant afin d’ajouter votre code de fonction à un ensemble d’outils. Ensuite, créez un agent capable d’utiliser l’ensemble d’outils et un thread sur lequel exécuter la session de conversation.

    ```python
   # Define an agent that can use the custom functions
   with agent_client:

        functions = FunctionTool(user_functions)
        toolset = ToolSet()
        toolset.add(functions)
        agent_client.enable_auto_function_calls(toolset)
            
        agent = agent_client.create_agent(
            model=model_deployment,
            name="support-agent",
            instructions="""You are a technical support agent.
                            When a user has a technical issue, you get their email address and a description of the issue.
                            Then you use those values to submit a support ticket using the function available to you.
                            If a file is saved, tell the user the file name.
                         """,
            toolset=toolset
        )

        thread = agent_client.threads.create()
        print(f"You're chatting with: {agent.name} ({agent.id})")

    ```

1. Recherchez le commentaire **Envoyer une invite à l’agent** et ajoutez le code suivant pour ajouter l’invite de l’utilisateur ou l’utilisatrice en tant que message et exécuter le thread.

    ```python
   # Send a prompt to the agent
   message = agent_client.messages.create(
        thread_id=thread.id,
        role="user",
        content=user_prompt
   )
   run = agent_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
    ```

    > **Remarque** : l’utilisation de la méthode **create_and_process** pour exécuter le thread permet à l’agent de rechercher automatiquement vos fonctions et de choisir de les utiliser en fonction de leurs noms et paramètres. Autrement, vous pouvez utiliser la méthode **create_run**, auquel cas vous devez écrire du code pour interroger l’état d’exécution afin de déterminer quand un appel de fonction est requis, appeler la fonction et renvoyer les résultats à l’agent.

1. Recherchez le commentaire **Vérifier l’état d’exécution des échecs** et ajoutez le code suivant pour afficher les erreurs qui se produisent.

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

1. Recherchez le commentaire **Obtenir l’historique des conversations** et ajoutez le code suivant pour afficher les messages du thread de conversation en les classant par ordre chronologique.

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
   print("Deleted agent")
    ```

1. Passez en revue le code en utilisant les commentaires pour comprendre comment il effectue les tâches suivantes :
    - Ajouter votre ensemble de fonctions personnalisées à un ensemble d’outils.
    - Créer un agent qui utilise l’ensemble d’outils.
    - Exécuter un thread avec un message d’invite de l’utilisateur ou l’utilisatrice.
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

1. Lorsque le système vous y invitera, entrez une invite telle que :

    ```
   I have a technical problem
    ```

    > **Conseil** : si l’application échoue en raison du dépassement de la limite de débit, patientez quelques secondes, puis réessayez. Si le quota disponible dans votre abonnement est insuffisant, le modèle peut ne pas être en mesure de répondre.

1. Affichez la réponse. L’agent peut demander votre adresse e-mail et une description du problème. Vous pouvez utiliser n’importe quelle adresse e-mail (par exemple, `alex@contoso.com`) et n’importe quelle description de problème (par exemple, `my computer won't start`).

    Lorsqu’il dispose d’informations suffisantes, l’agent doit choisir d’utiliser votre fonction selon les besoins.

1. Vous pouvez continuer la conversation si vous le souhaitez. Le thread est *avec état* et conserve donc l’historique des conversations. Cela signifie que l’agent a le contexte complet de chaque réponse. Lorsque vous avez terminé, entrez `quit`.
1. Examinez les messages de conversation récupérés à partir du thread et les tickets générés.
1. L’outil doit avoir enregistré des tickets de support dans le dossier de l’application. Vous pouvez utiliser la commande `ls` pour vérifier, puis la commande `cat` pour afficher le contenu du fichier, comme suit :

    ```
   cat ticket-<ticket_num>.txt
    ```

## Nettoyage

Maintenant que vous avez terminé l’exercice, vous devez supprimer les ressources cloud que vous avez créées pour éviter une utilisation inutile des ressources.

1. Ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et affichez le contenu du groupe de ressources où vous avez déployé les ressources de hub utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
