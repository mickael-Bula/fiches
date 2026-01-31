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
aura pour conséquences suivantes d'épuiser le quota inutilement et de "noyer" l'IA dans des informations inutiles.

L'idée est de Cibler toujours : Instruction + 1 ou 2 fichiers **maximum** pour une précision optimale.
