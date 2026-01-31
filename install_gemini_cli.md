## Étape 1 : Installation du moteur (SDK)

```PowerShell
pip install -U google-genai
```

## Étape 2 : Configuration de la sécurité (Variable d'environnement)

```PowerShell
# Définit la clé de façon permanente pour l'utilisateur
[Environment]::SetEnvironmentVariable("GEMINI_API_KEY", "LA_CLE_API_ICI", "User")
# Note : Redémarrez le terminal après cette commande
```

## Étape 3 : Création du script 

Il s'agit du "Cerveau" (C:\Users\mon_user\.local\bin\ask.py) qui gère le prompt fourni comme texte OU comme fichier.

Voici la commande PowerShell pour créer le fichier Python avec son contenu :

```PowerShell
@'
import os
import sys

# Force le chemin vers les bibliothèques Laragon
site_packages = r"c:\laragon\bin\python\python-3.10\lib\site-packages"
if site_packages not in sys.path:
    sys.path.append(site_packages)

from google import genai

client = genai.Client(api_key=os.environ.get("GEMINI_API_KEY"))

def ask():
    pipe_content = ""
    file_content = ""
    user_query = ""

    # 1. Lecture du flux entrant (Pipe ou Redirection < )
    if not sys.stdin.isatty():
        pipe_content = sys.stdin.read().strip()

    # 2. Analyse des arguments
    args = sys.argv[1:]
    
    if "-f" in args:
        # Cas : gemini -f prompt.txt
        try:
            idx = args.index("-f")
            file_path = args[idx + 1]
            with open(file_path, 'r', encoding='utf-8') as f:
                file_content = f.read().strip()
            # On retire -f et le nom du fichier des arguments pour le texte restant
            args.pop(idx + 1)
            args.pop(idx)
        except (IndexError, FileNotFoundError):
            print("Erreur : Fichier spécifié après -f introuvable.")
            return

    # Le reste des arguments est considéré comme la question texte
    user_query = " ".join(args).strip()

    # 3. Assemblage intelligent du Prompt
    # On combine tout ce qu'on a trouvé (Pipe + Fichier + Texte)
    parts = []
    if pipe_content: parts.append(pipe_content)
    if file_content: parts.append(file_content)
    if user_query:   parts.append(user_query)

    prompt = "\n\n".join(parts)

    if not prompt:
        print("Usage:")
        print("  gemini 'Ma question'")
        print("  gemini -f instructions.txt")
        print("  cat code.php | gemini 'Analyse ce code'")
        print("  gemini < audit.txt")
        return

    try:
        response = client.models.generate_content(
            model='gemini-flash-latest',
            contents=prompt
        )
        print(response.text)
    except Exception as e:
        print(f"Erreur API : {e}")

if __name__ == "__main__":
    ask()
'@ | Out-File -FilePath "C:\Users\mon_user\.local\bin\ask.py" -Encoding utf8
```

## Étape 4 : Création de la commande gemini (Alias Cmder) 

Ouvrir le fichier `C:\cmder\config\user_aliases.cmd` (ou le dossier cmder) et ajouter :

```DOS
gemini="C:\laragon\bin\python\python-3.10\python.exe" C:\Users\bulam\.local\bin\ask.py $*
```
## Utilisation

Pour interroger **Gemini** depuis le terminal :

```bash
$ gemini "Génère un jeu du pendu en javascript"
```

Avec un fichier prompt.txt contenant la question précédente :

```bash
$ gemini -f prompt.txt
```

### Fournir du contexte

Poser une question en fournissant un fichier de contexte :

```bash
$ cat app.js | gemini "Ajoute un formulaire de connexion au fichier app.js" 
```

Pour lire le prompt et le code source ensemble :

```bash
$ cat prompt.txt app.js | gemini
```

### Redirection de fichier

Si tout le contenu (instructions + code) est mis dans un seul fichier **audit.txt** :

```Bash
gemini < audit.txt
```

## Considération

**⚠️ Attention à la taille**

Bien que Gemini Flash accepte énormément de texte, envoyer tout un projet (ex: le dossier vendor/), 
aura pour conséquence d'épuiser le quota inutilement et de "noyer" l'IA dans des informations inutiles.

L'idée est de Cibler toujours : Instruction + 1 ou 2 fichiers **maximum** pour une précision optimale.

## Coopération Gemini-Aider

Je cherche à optimiser un flux de travail utilisant Gemini comme cerveau et Aider comme acteur.

J'ai installé les deux outils globalement :

- Gemini est fourni par le SDK genai sous Python 3.10 (installation avec pip)
- Aider est installé avec pipx (donc dans un environnement virtuel)

L'idée est de faire coopérer ces deux outils en récupérant la sortie de l'un pour la passer à l'autre.

pour ne pas perdre l'historique des discussions, tout en fournissant un fichier qui contient uniquement les informations pertinentes,
j'enregistre la dernière sortie dans un fichier dernier_plan.md, dont j'ajoute ensuite le contenu au fichier de journalisation historique_global.md

La commande pourra être :

```bash
$ gemini "Ma question" > dernier_plan.md & type dernier_plan.md >> historique_global.md
```

Un alias pratique pourrait être :

```bash
$ glog=gemini $* > dernier_plan.md & type dernier_plan.md >> historique_global.md & type dernier_plan.md
```

Il s'utiliserait de cette façon :

```bash
$ glog "Analyse ce contrôleur pour PHP 8.2"
```

Et la sortie serait alors produite dans les fichiers `dernier_plan.md` et `historique_global.md` de manière automatique.

Pour demander à Aider d'exécuter les dernières instructions listées par Gemini :

```bash
$ aider src/Controller/OldController.php --message "$(cat dernier_plan.md)"
```

Pour que Gemini produise un résultat exploitable par Aider, je crée également un fichier de contexte global, `prompt_system.txt`.
Ce fichier pourrait contenir ceci :

```text
Tu es un expert en migration Symfony et PHP 8.2+.
Ta mission est d'analyser le code fourni et de proposer des améliorations de modernisation.

STRUCTURE TA RÉPONSE :
1. ANALYSE : Explique brièvement (en français) les changements nécessaires et pourquoi (ex: typage, attributs, performances).
2. INSTRUCTIONS POUR AIDER : Termine ta réponse par une section claire commençant par "### ACTIONS POUR AIDER". Dans cette section, donne des instructions directes et techniques que l'outil Aider pourra exécuter (ex: "Remplace les annotations @Route par des Attributs #[Route]", "Ajoute le typage string à la propriété $name").

Sois précis et technique. Évite les bavardages inutiles.
```

Il serait intégrer automatiquement lors de l'appel du script Python `ask.py` :

```python
import os
import sys

# ... (garder les imports et le sys.path.append précédents) ...

def ask():
    # --- NOUVEAU : Lecture du Prompt Système ---
    system_prompt_path = r"C:\Users\bulam\.local\bin\prompt_system.txt"
    system_content = ""
    if os.path.exists(system_prompt_path):
        with open(system_prompt_path, 'r', encoding='utf-8') as f:
            system_content = f.read().strip()
    
    pipe_content = ""
    file_content = ""
    user_query = ""

    # 1. Lecture stdin (Pipe)
    if not sys.stdin.isatty():
        pipe_content = sys.stdin.read().strip()

    # 2. Analyse des arguments (Gestion du -f)
    args = sys.argv[1:]
    if "-f" in args:
        try:
            idx = args.index("-f")
            file_path = args[idx + 1]
            with open(file_path, 'r', encoding='utf-8') as f:
                file_content = f.read().strip()
            args.pop(idx + 1); args.pop(idx)
        except: pass
    user_query = " ".join(args).strip()

    # 3. Assemblage du Prompt Final avec le Système
    all_parts = []
    if system_content: all_parts.append(f"INSTRUCTIONS SYSTÈME :\n{system_content}")
    if pipe_content:   all_parts.append(f"CODE SOURCE / CONTEXTE :\n{pipe_content}")
    if file_content:   all_parts.append(f"INSTRUCTIONS SUPPLÉMENTAIRES :\n{file_content}")
    if user_query:     all_parts.append(f"QUESTION UTILISATEUR :\n{user_query}")

    prompt = "\n\n---\n\n".join(all_parts)
    
    # ... (Reste de l'appel API avec client.models.generate_content) …
```

>NOTE : pour que l'encodage UTF-8 généré par Gemini en raison du markdown ne génère pas d'erreur, je force l'encodage UTF-8 pour Python.
>Pour ne pas avoir à saisir cette configuration à chaque ouverture d'un terminal, je la place dans le fichier `Cmder C:\laragon\etc\cmder\user_profile.cmd` :

```bash
$ set PYTHONIOENCODING=utf-8
```
