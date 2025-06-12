# Plutus
# Contrat Intelligent "Guessing Game" sur Cardano

Ce dépôt contient un exemple simple de smart contract écrit en Plutus V2 pour la blockchain Cardano. Il s'agit d'un "jeu de devinette" conçu à des fins éducatives pour illustrer les concepts fondamentaux de Plutus : le **Datum**, le **Redeemer** et le **Validator**.

## 📖 Table des Matières

1.  [Fonctionnement du Contrat]
2.  [Composants du Smart Contract]
3.  [Prérequis]
4.  [Structure du Fichier]
5.  [Comment Compiler le Contrat]
6.  [Prochaines Étapes]

-----

## 🎲 Fonctionnement du Contrat

Le principe du jeu est simple et se déroule en deux étapes principales, correspondant à deux transactions sur la blockchain.

### 1\. Création du jeu (Verrouillage des fonds)

  - Le créateur du jeu choisit un mot secret .
  - Il calcule le hachage SHA-256 de ce mot.
  - Il soumet une transaction qui verrouille une certaine quantité d'ADA à l'adresse du script. Cette transaction inclut le Datum, qui contient le hachage du mot secret. Le mot en clair n'est jamais révélé sur la chaîne.

### 2\. Tenter de deviner (Déverrouillage des fonds)

  - Un joueur voit les fonds verrouillés sur le contrat.
  - Pour tenter de les récupérer, il construit une transaction qui essaie de dépenser ces fonds.
  - Cette transaction doit inclure un Redeemer, qui contient sa proposition de mot (par exemple, "plutus").
  - Le script de validation s'exécute alors :
      - Il hache la proposition du joueur ("plutus").
      - Il compare ce nouveau hachage avec celui stocké dans le Datum.
      - Si les hachages correspondent, la validation réussit, et le joueur reçoit les ADA.
      - Sinon, la validation échoue, et la transaction est rejetée.

-----

## 🧩 Composants du Smart Contract

Le code, situé dans `GuessingGame.hs`, est structuré autour des trois piliers de Plutus.

### Datum : `GameDatum`

C'est l'état du contrat, attaché aux fonds verrouillés (UTXO).

```haskell
newtype GameDatum = GameDatum { secretHash :: BuiltinByteString }
```

  - `secretHash`: Contient le hachage SHA-256 du mot secret.

### Redeemer : `Guess`

C'est l'action ou la "preuve" fournie par l'utilisateur pour déverrouiller les fonds.

```haskell
newtype Guess = Guess { clearWord :: BuiltinByteString }
```

  - `clearWord`: Contient la proposition de mot en clair.

### Validator : `mkGuessingGameValidator`

C'est la logique centrale du contrat. Sa fonction est de retourner `True` ou `False`.
La logique est la suivante :

```haskell
sha2_256 (clearWord redeemer) == secretHash datum
```

Elle vérifie si le hachage de la proposition du `Redeemer` est identique au hachage stocké dans le `Datum`.

-----

## 🛠️ Prérequis

Pour travailler sur ce projet, vous aurez besoin de :

  - [Nix](https://nixos.org/) installé sur votre système.
  - Un environnement de développement Plutus fonctionnel, comme celui fourni par le [Plutus Starter Template](https://github.com/IntersectMBO/plutus.git).
  - Des connaissances de base du langage de programmation [Haskell](https://www.haskell.mooc/).

-----

## 📂 Structure du Fichier

  - `GuessingGame.hs`: Contient l'intégralité du code "on-chain" : définitions du Datum et du Redeemer, et la logique du validateur.
  - `cabal.project` / `package.yaml`: Fichiers de configuration du projet Haskell.

-----

## Comment Compiler le Contrat

Le code Haskell doit être compilé en Plutus Script (`.plutus`) pour être déployable sur la blockchain. Pour ce faire, vous pouvez utiliser un REPL (Read-Eval-Print Loop) Haskell.

1.  Lancez le REPL depuis la racine de votre projet :

    ```bash
    cabal repl
    ```

2.  Chargez le module du jeu :

    ```haskell
    > :l GuessingGame
    ```

3.  Exécutez la fonction pour sauvegarder le script compilé. Vous aurez besoin d'une petite fonction d'aide (à ajouter dans un module `Utils` ou directement dans `Main` de votre projet) pour écrire le fichier. Voici un exemple :

    ```haskell
    -- Code à ajouter dans un fichier utilitaire pour l'écriture
    -- import qualified Data.ByteString.Short as SBS
    -- import qualified Plutus.V2.Ledger.Api      as V2
    --
    -- writeValidator :: FilePath -> V2.Validator -> IO (Either (AsContractError e) ())
    -- writeValidator file = writeFileTextEnvelope file Nothing . PlutusScriptSerialised . SBS.toShort . LBS.toStrict . serialise

    -- Dans le REPL, après avoir importé votre fonction d'aide :
    > writeValidator "game.plutus" validator
    ```

    Cela créera un fichier `game.plutus` dans votre répertoire. C'est ce fichier qui contient le smart contract prêt à être utilisé par des outils comme `cardano-cli` ou des bibliothèques off-chain comme Lucid.
