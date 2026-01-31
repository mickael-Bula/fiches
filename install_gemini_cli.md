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
from google import genai

client = genai.Client(api_key=os.environ["GEMINI_API_KEY"])

def ask():
    # On récupère tous les arguments après le nom du script
    args = sys.argv[1:]
    if not args:
        print("Usage: gemini 'ma question' OU gemini fichier.txt")
        return

    # Nettoyage : si l'utilisateur met "-f", on l'ignore pour trouver le fichier
    potential_file = args[1] if args[0] == "-f" and len(args) > 1 else args[0]

    if os.path.isfile(potential_file):
        with open(potential_file, 'r', encoding='utf-8') as f:
            prompt = f.read()
    else:
        # Si ce n'est pas un fichier, on traite tout comme du texte
        prompt = " ".join(args)

    try:
        response = client.models.generate_content(
            model='gemini-flash-latest', 
            contents=prompt
        )
        print(response.text)
    except Exception as e:
        print(f"Erreur : {e}")

if __name__ == "__main__":
    ask()
'@ | Out-File -FilePath "C:\Users\mon_user\.local\bin\ask.py" -Encoding utf8
```

## Étape 4 : Création de la commande gemini (Alias Cmder) 

Ouvrir le fichier `C:\cmder\config\user_aliases.cmd` (ou le dossier cmder) et ajouter :

```DOS
gemini=python "C:\Users\bulam\.local\bin\ask.py" $*
```
