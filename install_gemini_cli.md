# Workflow Gemini + Aider

## Prérequis

Python installé sur le poste. J'utilise ici la version 3.10.

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

Ouvrir le fichier `C:\laragon\bin\cmder\config\user_aliases.cmd` (ou le dossier cmder) et ajouter :

```DOS
gemini="C:\laragon\bin\python\python-3.10\python.exe" C:\Users\mon_user\.local\bin\ask.py $*
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

# Force le chemin vers les bibliothèques Laragon
site_packages = r"c:\laragon\bin\python\python-3.10\lib\site-packages"
if site_packages not in sys.path:
    sys.path.append(site_packages)

from google import genai

client = genai.Client(api_key=os.environ.get("GEMINI_API_KEY"))


def ask():
    # --- Lecture du Prompt Système ---
    system_prompt_path = r"C:\Users\mon_user\.local\bin\prompt_system.txt"
    system_content = ""
    if os.path.exists(system_prompt_path):
        with open(system_prompt_path, 'r', encoding='utf-8') as f:
            system_content = f.read().strip()

    pipe_content = ""
    file_content = ""

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
    parts: list[str] = []
    if system_content:
        parts.append(f"INSTRUCTIONS SYSTÈME :\n{system_content}")
    if pipe_content:
        parts.append(pipe_content)
    if file_content:
        parts.append(file_content)
    if user_query:
        parts.append(user_query)

    prompt = "\n\n---\n\n".join(parts)

    if not prompt:
        print("Usage :")
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
```

>NOTE : pour que l'encodage UTF-8 généré par Gemini en raison du markdown ne génère pas d'erreur, je force l'encodage UTF-8 pour Python.
>Pour ne pas avoir à saisir cette configuration à chaque ouverture d'un terminal, je la place dans le fichier `Cmder C:\laragon\etc\cmder\user_profile.cmd` :

```bash
$ set PYTHONIOENCODING=utf-8
```

## Déclaration de variables d'environnement

Pour simplifier et harmoniser les chemins appelés depuis les scripts Python et les alias Cmder, je déclare les variables suivantes dans `C:\laragon\bin\cmder\config\user_profile.cmd` :

```cmd
:: Ajout du dossier des scripts Gemini au PATH
set "PATH=C:\Users\mon_user\.local\bin;%PATH%"

:: Déclare l'encodage UTF-8 pour les scripts Python
set PYTHONIOENCODING=utf-8

:: Chemins vers les exécutables et scripts
set LOCAL_BIN=C:\Users\mon_user\.local\bin
set PYTHON_BIN=C:\laragon\bin\python\python-3.10\python.exe
set ASK_SCRIPT=C:\Users\mon_user\.local\bin\ask.py
```

## Alias Gemini

Pour simplifier au maximum les appels à **Gemini**, j'ai créé l'alias suivant dans le fichier `C:\laragon\bin\cmder\config\user_aliases.cmd` :

```cmd
glog=%PYTHON_BIN% %LOCAL_BIN%\glog.py $*
```

Cet alias appel le script Python suivant :

```python
import sys
import subprocess
import datetime
import os

def run():
    # 1. Vérification et création du dossier de scripts si nécessaire
    # On récupère le chemin depuis l'environnement ou on utilise celui par défaut
    local_bin = os.environ.get('LOCAL_BIN', r'C:\Users\mon_user\.local\bin')
    if not os.path.exists(local_bin):
        try:
            os.makedirs(local_bin, exist_ok=True)
        except Exception as e:
            print(f"Erreur lors de la création du dossier {local_bin} : {e}")

    # 2. Récupérer le prompt passé en argument
    prompt = " ".join(sys.argv[1:])
    if not prompt:
        print("Erreur : Aucun prompt fourni.")
        return

    # Configuration des chemins
    ask_script = os.environ.get('ASK_SCRIPT', os.path.join(local_bin, 'ask.py'))
    python_bin = os.environ.get('PYTHON_BIN', 'python')
    hist_file = 'historique_global.md'
    plan_file = 'dernier_plan.md'

    # 3. Préparer l'en-tête de l'historique
    timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    divider = "=" * 50
    header = f"\n{divider}\nDATE   : {timestamp}\nPROMPT : {prompt}\n{'-' * 50}\n"

    try:
        with open(hist_file, 'a', encoding='utf-8') as h:
            h.write(header)
    except Exception as e:
        print(f"Erreur lors de l'écriture dans l'historique : {e}")

    # 4. Exécuter ask.py et capturer la sortie
    # stdin=sys.stdin permet de transmettre le flux (ex: cat fichier | glog)
    try:
        result = subprocess.run(
            [python_bin, ask_script, prompt],
            capture_output=True,
            text=True,
            encoding='utf-8',
            stdin=sys.stdin  # Transmet le contenu du pipe (ex: cat file | ...)
        )

        if result.returncode != 0:
            # On affiche l'erreur sur le flux d'erreur standard
            print(f"Erreur lors de l'exécution de Gemini :\n{result.stderr}", file=sys.stderr)
            return

        # 5. Écrire dans dernier_plan.md et historique_global.md
        content = result.stdout
        
        with open(plan_file, 'w', encoding='utf-8') as p:
            p.write(content)

        with open(hist_file, 'a', encoding='utf-8') as h:
            h.write(content)

        # 6. Afficher le résultat dans le terminal
        print(content)

    except Exception as e:
        print(f"Une erreur système est survenue : {e}")

if __name__ == "__main__":
    run()
```

Ce script effectue les action ssuivantes :
- récupère le prompt passé en argument de la commande
- appelle le script `ask.py` avec le prompt précédent en argument
- affiche la réponse de Gemini en corrigeant l'encodage
- enregistre la réponse de Gemini dans le fichier `dernier_plan.md`
- ajoute la réponse de Gemini dans le fichier `historique_global.md`, précédée du prompt et datée

De cette manière, je conserve un historique complet du flux de questions et réponses de la discussion.

## Alias Aider

De même, pour simplifer l'appel à **Aider**, j'ai créé cet alias dans `C:\laragon\bin\cmder\config\user_aliases.cmd` :

```cmd
ago=aider --no-gitignore --no-auto-commits --message-file dernier_plan.md $*
```

Cet alias appel **Aider** en lui passant en argument le fichier `dernier_plan.md` généré par Gemini.
Il précise également de ne pas faire de commit (je m'en chargerai après validation) et de ne pas demander l'ajout du fichier `.env` à chaque appel.

