# Fiche de révision — Blockchain : Bitcoin, Ethereum et Solidity

> **Objectif :** réviser les notions essentielles des deux supports de cours de Fausto Spoto (*Blockchain Course — Bitcoin*, 90 diapositives ; *Blockchain Course — Ethereum*, 160 diapositives).
>
> La partie Solidity développe les notions du support Ethereum et les illustre avec des exemples de code en Solidity 0.8.x.

---

# Partie I — Bitcoin

## 1. Présentation générale

Bitcoin est un système de monnaie électronique **décentralisé**, fonctionnant sur un réseau pair-à-pair (*peer-to-peer*, P2P).

Les objectifs fondamentaux d'une monnaie sont :

- **Légalité :** empêcher la contrefaçon.
- **Unicité :** empêcher la double dépense.
- **Propriété :** empêcher quelqu'un de dépenser l'argent d'un autre.

Dans Bitcoin, il n'existe pas de serveur central qui conserve les soldes. Les participants du réseau partagent et vérifient l'historique des transactions.

### Vocabulaire essentiel

| Terme | Définition |
|---|---|
| Blockchain | Registre distribué organisé en blocs liés entre eux |
| Nœud (*node*) | Ordinateur participant au réseau |
| Wallet | Logiciel ou dispositif qui gère les clés et les transactions |
| Adresse | Identifiant auquel des bitcoins peuvent être envoyés |
| Transaction | Opération qui consomme des sorties et en crée de nouvelles |
| UTXO | *Unspent Transaction Output*, sortie de transaction non dépensée |
| Minage | Processus de création et de validation de blocs par preuve de travail |

## 2. Wallets, clés et adresses

Un wallet permet notamment de :

- générer et conserver des clés ;
- créer et signer des transactions ;
- diffuser les transactions au réseau ;
- consulter les transactions associées à ses adresses.

Types de wallets présentés dans le cours :

- desktop wallets ;
- mobile wallets ;
- web wallets ;
- hardware wallets ;
- paper wallets.

### Clé privée et clé publique

- La **clé privée** est secrète. Elle permet de signer des transactions.
- La **clé publique** est dérivée de la clé privée.
- L'**adresse** est obtenue à partir d'une représentation dérivée de la clé publique.

Pour une adresse Bitcoin historique de type P2PKH, le calcul utilise notamment :

```text
RIPEMD160(SHA256(public_key))
```

Cette formule concerne le format P2PKH standard présenté dans le cours ; elle ne décrit pas tous les formats d'adresses Bitcoin modernes.

**À retenir :** l'adresse peut être communiquée publiquement ; la clé privée ne doit jamais être divulguée.

### Génération des clés

La clé privée doit être créée avec un générateur aléatoire cryptographiquement sûr. Un générateur pseudo-aléatoire ordinaire n'est pas adapté à cet usage.

La création d'une clé et d'une adresse se fait localement : elle ne nécessite pas de transaction sur la blockchain.

## 3. Signatures numériques

Une transaction est signée avec la clé privée de son émetteur.

La signature permet aux nœuds de vérifier que l'auteur de la transaction contrôle la clé correspondant à la clé publique attendue.

Schéma simplifié :

1. Le wallet construit une transaction.
2. Il produit une signature avec la clé privée.
3. La transaction signée est diffusée au réseau.
4. Les nœuds vérifient la signature et les règles de validité.

La signature ne révèle pas la clé privée.

## 4. Transactions et modèle UTXO

Bitcoin ne stocke pas directement un solde par utilisateur dans la blockchain. Il utilise le modèle **UTXO**.

Une transaction contient principalement :

- des **inputs** : références à des sorties antérieures que la transaction dépense ;
- des **outputs** : nouvelles sorties créées par la transaction.

Une sortie non dépensée est un UTXO.

### Exemple de paiement avec monnaie rendue

Alice possède une sortie de 1 BTC et souhaite payer 0,6 BTC à Bob.

La transaction peut créer :

- une sortie de 0,6 BTC pour Bob ;
- une sortie de monnaie rendue pour Alice ;
- des frais pour le mineur, correspondant à la différence entre les valeurs consommées et créées.

```text
Input : 1 BTC
Outputs :
  - 0,6 BTC pour Bob
  - 0,39 BTC de monnaie rendue pour Alice
Frais : 0,01 BTC
```

Les valeurs de cet exemple sont illustratives.

### Règle de conservation

Hors transaction spéciale de création de monnaie (*coinbase*), une transaction ne peut pas créer de bitcoins :

```text
Somme(inputs) >= Somme(outputs)
Frais = Somme(inputs) - Somme(outputs)
```

### Double dépense

Une même sortie ne doit pas être dépensée plusieurs fois dans la même histoire validée de la blockchain.

Les nœuds vérifient que les inputs référencent des UTXO encore disponibles.

## 5. Diffusion et validation des transactions

Lorsqu'un utilisateur émet une transaction :

1. Son wallet la construit et la signe.
2. Il la diffuse à un ou plusieurs nœuds.
3. Les nœuds vérifient sa validité.
4. Les transactions valides peuvent être conservées dans la **mempool**.
5. Les nœuds relaient les transactions à leurs pairs.
6. Un mineur peut inclure la transaction dans un bloc.

Une transaction incluse dans un bloc est confirmée. Plus il y a de blocs construits au-dessus de celui qui la contient, plus son inclusion est considérée comme difficile à renverser.

## 6. Scripts Bitcoin

Bitcoin utilise un langage de script destiné à exprimer les conditions de dépense des sorties.

Une sortie contient un script de verrouillage (*locking script*). Pour la dépenser, l'input fournit les données nécessaires dans un script de déverrouillage (*unlocking script*).

Exemple conceptuel de vérification P2PKH :

```text
<signature> <clé_publique>
DUP HASH160 <adresse> EQUALVERIFY CHECKSIG
```

Le script vérifie notamment :

- que le hash de la clé publique correspond à l'adresse attendue ;
- que la signature est correcte pour la transaction.

Le cours souligne que l'interprétation d'une adresse par un explorateur peut dépendre du type de script standard utilisé.

## 7. Blocs et arbre de Merkle

Les nœuds complets reçoivent les transactions, les vérifient et les conservent dans leur mempool. Les mineurs sélectionnent ensuite des transactions pour construire un bloc.

Un bloc contient notamment :

- un en-tête ;
- des transactions ;
- une référence au bloc précédent ;
- un horodatage ;
- une racine de Merkle ;
- un nonce.

### Arbre de Merkle

L'arbre de Merkle permet de vérifier efficacement qu'une transaction appartient à un bloc.

- Les feuilles correspondent aux hashes des transactions.
- Les hashes sont combinés deux à deux.
- Le hash final est la **racine de Merkle**.

La racine est incluse dans l'en-tête du bloc. Le cours précise que l'arbre complet n'est pas stocké dans le bloc : les nœuds peuvent le conserver en mémoire pour produire des preuves d'inclusion.

Une preuve de Merkle permet de vérifier l'inclusion d'une transaction sans télécharger toutes les transactions du bloc.

## 8. Minage et Proof of Work

Le minage sert à sécuriser le réseau et à produire de nouveaux blocs. Les mineurs cherchent un bloc valide respectant les règles de consensus.

### Construction d'un bloc

Un mineur :

1. sélectionne des transactions valides dans sa mempool ;
2. ajoute une transaction spéciale appelée **coinbase** ;
3. calcule la racine de Merkle ;
4. construit l'en-tête du bloc ;
5. cherche une valeur de nonce qui satisfait la difficulté.

### Preuve de travail (Proof of Work, PoW)

La règle simplifiée du cours est que le hash de l'en-tête doit être inférieur à une cible de difficulté.

```text
Répéter :
  choisir un nonce
  calculer le hash de l'en-tête
Jusqu'à obtenir :
  hash(en-tête) < cible_de_difficulté
```

- Trouver une solution demande beaucoup de calculs.
- Vérifier une solution est rapide.
- La difficulté ajuste le coût de la recherche.
- Le nonce est une partie de l'en-tête utilisée dans cette recherche.

### Coinbase et récompense

La transaction coinbase est particulière :

- elle n'a pas d'input ordinaire ;
- elle crée la récompense autorisée par les règles du protocole ;
- elle peut également inclure les frais des transactions du bloc.

### Forks

Plusieurs mineurs peuvent produire des blocs concurrents au même niveau. La blockchain peut donc temporairement prendre la forme d'un arbre.

Les nœuds suivent les règles de consensus pour choisir la chaîne à retenir. La situation peut se résoudre lorsqu'une branche accumule davantage de travail.

## 9. Points importants à connaître sur Bitcoin

- Le réseau est P2P et ne dépend pas d'un serveur central.
- La clé privée autorise la signature des transactions.
- Une adresse n'est pas un mot de passe.
- Bitcoin utilise le modèle UTXO.
- Une transaction consomme des sorties et en crée de nouvelles.
- La mempool contient des transactions en attente d'inclusion.
- Le minage construit des blocs et repose, dans le modèle du cours, sur la Proof of Work.
- La racine de Merkle est placée dans l'en-tête du bloc.
- La coinbase est une transaction spéciale.
- Les confirmations renforcent la confiance dans l'inclusion d'une transaction.

---

# Partie II — Ethereum

## 1. Présentation générale

Ethereum est présenté dans le support comme un **ordinateur mondial décentralisé**.

Il permet d'exécuter des programmes appelés **smart contracts**. L'état du système est partagé et synchronisé par la blockchain.

Une application décentralisée, ou **DApp**, associe généralement :

```text
DApp = smart contracts + interface web (frontend)
```

Les contrats sont souvent écrits en Solidity, compilés en bytecode EVM, puis exécutés par l'Ethereum Virtual Machine.

## 2. Comptes Ethereum : EOA et contrats

Ethereum distingue deux grandes catégories de comptes.

| Type | Caractéristique |
|---|---|
| EOA (*Externally Owned Account*) | Compte contrôlé par une clé privée |
| Compte de contrat | Compte associé à du code exécuté par l'EVM |

- Un EOA peut initier une transaction.
- Un contrat possède une adresse et du code.
- Un contrat ne possède pas de clé privée comme un EOA.
- L'exécution d'un contrat est déclenchée par une transaction ou par un appel effectué pendant une exécution.

## 3. Transactions Ethereum

Une transaction peut notamment contenir :

- l'adresse du destinataire ;
- la valeur transférée ;
- les données d'appel ;
- un nonce ;
- des informations relatives au coût d'exécution ;
- une signature.

Le **nonce** d'un compte EOA permet notamment d'ordonner ses transactions et d'éviter leur réutilisation.

### Gas

L'exécution d'opérations sur Ethereum consomme du **gas**.

Le gas sert à mesurer les ressources utilisées par l'exécution :

- calcul ;
- lecture et écriture de l'état ;
- opérations de la machine virtuelle.

Le gas limite les ressources consommées et évite qu'un programme monopolise indéfiniment le réseau.

Dans une présentation simplifiée :

```text
Coût = gas utilisé × prix du gas
```

Sur Ethereum actuel, le calcul des frais tient notamment compte de la base fee et de la priority fee ; les champs de frais dépendent du type de transaction.

**À retenir :** le gas mesure le coût en ressources de l'exécution ; l'ETH sert à payer ces ressources.

## 4. EVM : Ethereum Virtual Machine

L'EVM est la machine virtuelle qui exécute le bytecode des contrats.

Cycle simplifié :

1. Le développeur écrit un contrat en Solidity.
2. Le compilateur traduit le contrat en bytecode EVM.
3. Le bytecode est déployé sur la blockchain.
4. Une transaction déclenche l'exécution du code.
5. L'exécution modifie éventuellement l'état.
6. Les nœuds exécutent les mêmes opérations déterministes pour parvenir au même résultat.

L'exécution doit être déterministe : pour un même état initial et les mêmes entrées, les nœuds doivent obtenir le même résultat.

## 5. Smart contracts

Un smart contract est un programme informatique déployé sur la blockchain.

Caractéristiques présentées dans le support :

- il est compilé en bytecode EVM ;
- il est associé à une adresse ;
- il est déterministe ;
- il opère sur des données restreintes ;
- il ne s'exécute pas spontanément en arrière-plan ;
- son exécution est déclenchée par une transaction ou une chaîne d'appels ;
- une transaction est atomique : elle est exécutée entièrement ou annulée en cas d'échec.

Un contrat n'est pas nécessairement « intelligent » au sens humain du terme, ni un contrat juridique.

## 6. État global et Merkle-Patricia Trie

Contrairement au modèle UTXO de Bitcoin, Ethereum maintient un état global organisé sous forme de données associées à des clés.

Le support présente notamment les opérations conceptuelles suivantes :

```text
get(address)
put(address, value)
get hash()
checkout(hash)
```

Le **Merkle-Patricia Trie** permet notamment :

- un accès efficace aux valeurs ;
- une représentation compacte de l'état ;
- un recalcul efficace du hash après modification ;
- un changement de vue vers un état antérieur.

Le hash de l'état est enregistré dans l'en-tête du bloc. Il permet de représenter cryptographiquement l'état obtenu après l'exécution des transactions.

## 7. Bitcoin et Ethereum : différence de modèle

| Bitcoin | Ethereum |
|---|---|
| Modèle UTXO | État global clé-valeur |
| Transactions qui consomment et créent des sorties | Transactions qui modifient l'état |
| Scripts de dépense | Smart contracts généralistes |
| Bitcoin (BTC) | Ether (ETH) |
| Proof of Work dans le modèle historique du support | Le support décrit le fonctionnement historique ; le réseau Ethereum actuel utilise la Proof of Stake |

---

# Partie III — Cours de Solidity

## 1. Qu'est-ce que Solidity ?

Solidity est un langage de programmation destiné principalement à écrire des smart contracts pour l'EVM.

Caractéristiques mises en avant dans le support :

- langage impératif ;
- syntaxe inspirée des langages orientés objet ;
- langage évolutif ;
- permet les séquences, conditions et répétitions ;
- permet d'écrire des programmes généraux exécutés par l'EVM.

Le compilateur transforme le code Solidity en bytecode exécutable par l'EVM.

## 2. Structure minimale d'un contrat

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MonContrat {
    uint256 public compteur;

    constructor() {
        compteur = 0;
    }

    function incrementer() public {
        compteur += 1;
    }
}
```

Explications :

- `SPDX-License-Identifier` indique la licence du code.
- `pragma solidity` indique la version du compilateur visée.
- `contract` déclare un contrat.
- `uint256 public compteur` déclare une variable d'état.
- `constructor` est exécuté lors du déploiement.
- `function` déclare une fonction.

## 3. Types de données

### Types simples

| Type | Rôle | Exemple |
|---|---|---|
| `bool` | Booléen | `true`, `false` |
| `uint` | Entier non signé | `uint256 x = 10;` |
| `int` | Entier signé | `int256 x = -10;` |
| `address` | Adresse Ethereum | `address utilisateur;` |
| `address payable` | Adresse pouvant recevoir de l'ETH | `address payable destinataire;` |
| `bytes` | Tableau d'octets | `bytes donnees;` |
| `string` | Chaîne de caractères | `string nom;` |

Les entiers non signés (`uint`) ne peuvent pas être négatifs.

### Tableaux

```solidity
uint256[] public nombres;

function ajouter(uint256 valeur) public {
    nombres.push(valeur);
}
```

Un tableau dynamique peut grandir avec `push`.

### Enum

Un `enum` définit un ensemble limité de valeurs possibles.

```solidity
enum Etat {
    EnAttente,
    Accepte,
    Refuse
}

Etat public etat;
```

### Struct

Une `struct` regroupe plusieurs champs.

```solidity
struct Etudiant {
    string nom;
    uint256 age;
    bool inscrit;
}

Etudiant public etudiant;
```

### Mapping

Un `mapping` associe une clé à une valeur.

```solidity
mapping(address => uint256) public soldes;
```

Ici, une adresse est associée à un entier.

Attention : un mapping n'est pas un tableau classique. Il ne fournit pas directement la liste de toutes ses clés.

## 4. Variables d'état, locales et globales

### Variables d'état

Elles sont déclarées dans le contrat, en dehors des fonctions. Leur valeur est conservée dans l'état du contrat.

```solidity
contract Exemple {
    uint256 public total;
}
```

### Variables locales

Elles sont déclarées à l'intérieur d'une fonction et ne constituent pas directement un champ persistant de l'état.

```solidity
function calculer() public pure returns (uint256) {
    uint256 resultat = 2 + 3;
    return resultat;
}
```

### Variables globales utiles

| Variable | Signification |
|---|---|
| `msg.sender` | Adresse de l'appelant immédiat |
| `msg.value` | Quantité d'ETH envoyée avec l'appel |
| `msg.data` | Données de l'appel |
| `block.timestamp` | Horodatage du bloc |
| `block.number` | Numéro du bloc |
| `block.chainid` | Identifiant de la chaîne |
| `address(this)` | Adresse du contrat courant |
| `gasleft()` | Quantité de gas restante pour l'exécution |

**Attention :** `tx.origin` désigne l'EOA à l'origine de la transaction. Il ne faut pas l'utiliser comme mécanisme d'autorisation à la place de `msg.sender`.

## 5. Fonctions et visibilité

Une fonction définit un comportement du contrat.

```solidity
function additionner(uint256 a, uint256 b)
    public
    pure
    returns (uint256)
{
    return a + b;
}
```

### Visibilité

| Mot-clé | Signification |
|---|---|
| `public` | Accessible depuis le contrat et depuis l'extérieur |
| `external` | Destinée à être appelée depuis l'extérieur |
| `internal` | Accessible dans le contrat et ses contrats dérivés |
| `private` | Accessible uniquement dans le contrat qui la déclare |

La visibilité `private` ne rend pas les données secrètes : les données stockées sur une blockchain publique peuvent être observées ou analysées.

### Paramètres et retour

```solidity
function carre(uint256 x) public pure returns (uint256) {
    return x * x;
}
```

- `uint256 x` : paramètre d'entrée.
- `returns (uint256)` : type de retour.
- `return` : valeur renvoyée.

## 6. Mutabilité : view, pure et payable

| Mot-clé | Rôle |
|---|---|
| `view` | Peut lire l'état, mais ne doit pas le modifier |
| `pure` | Ne lit ni ne modifie l'état |
| `payable` | Autorise la réception d'ETH par la fonction ou l'adresse concernée |

Exemples :

```solidity
uint256 public nombre = 10;

function lire() public view returns (uint256) {
    return nombre;
}

function calculer(uint256 x) public pure returns (uint256) {
    return x * 2;
}

function deposer() public payable {
    // La fonction peut recevoir de l'ETH.
}
```

## 7. Conditions et boucles

### Conditions

```solidity
function verifier(uint256 age) public pure returns (bool) {
    if (age >= 18) {
        return true;
    } else {
        return false;
    }
}
```

### Boucle for

```solidity
function somme(uint256 n) public pure returns (uint256) {
    uint256 resultat = 0;

    for (uint256 i = 1; i <= n; i++) {
        resultat += i;
    }

    return resultat;
}
```

Les boucles coûteuses ou non bornées sont dangereuses dans un contrat : elles peuvent dépasser le gas disponible et rendre une fonction inutilisable.

## 8. Gestion des erreurs : require, assert et revert

### require

Utilisé pour vérifier une condition attendue, par exemple un paramètre ou une autorisation.

```solidity
function retirer(uint256 montant) public {
    require(montant > 0, "Montant nul");
}
```

### assert

Utilisé pour vérifier une invariant ou une condition qui ne devrait jamais être fausse si le programme est correct.

```solidity
assert(total >= 0);
```

### revert

Permet de déclencher explicitement une erreur.

```solidity
function operation() public pure {
    revert("Operation impossible");
}
```

En cas de revert, les modifications d'état effectuées pendant l'appel sont annulées.

## 9. Ether et fonctions payable

L'ETH possède plusieurs unités :

| Unité | Équivalence |
|---|---|
| 1 ether | 10^18 wei |
| 1 gwei | 10^9 wei |
| 1 wei | unité de base |

Il est recommandé d'utiliser les suffixes d'unité plutôt que de manipuler directement de grands nombres :

```solidity
uint256 montant = 1 ether;
uint256 frais = 5 gwei;
```

Une fonction qui doit recevoir de l'ETH doit être déclarée `payable`.

```solidity
function deposer() public payable {
    require(msg.value > 0, "Depot vide");
}
```

`msg.value` représente la quantité d'ETH envoyée avec l'appel.

## 10. receive et fallback

Ces fonctions spéciales permettent de gérer certains appels entrants.

```solidity
receive() external payable {
    // Appel avec données vides et transfert d'ETH.
}

fallback() external payable {
    // Appel lorsque la fonction demandée ne correspond pas
    // à une fonction connue, ou selon les règles de dispatch.
}
```

- `receive()` est appelée lorsque les données d'appel sont vides, si elle est définie.
- `fallback()` intervient notamment lorsqu'aucune fonction ne correspond aux données reçues.
- Pour recevoir de l'ETH dans ces cas, la fonction concernée doit être `payable`.

## 11. Transférer de l'ETH

Un contrat peut envoyer de l'ETH à une adresse payable.

```solidity
payable(destinataire).transfer(montant);
```

Le support utilise `transfer` dans l'exemple du Faucet. Dans le développement Solidity moderne, `call` est également couramment utilisé, avec des précautions de sécurité.

Exemple avec `call` :

```solidity
(bool succes, ) = payable(destinataire).call{value: montant}("");
require(succes, "Transfert echoue");
```

Il faut protéger les fonctions qui envoient de l'ETH contre les attaques de réentrance.

## 12. Modifiers

Un `modifier` permet de factoriser une condition appliquée à plusieurs fonctions.

```solidity
contract Administration {
    address public proprietaire;

    constructor() {
        proprietaire = msg.sender;
    }

    modifier seulementProprietaire() {
        require(msg.sender == proprietaire, "Non autorise");
        _;
    }

    function actionAdmin() public seulementProprietaire {
        // Action réservée au propriétaire.
    }
}
```

Le symbole `_` indique l'endroit où le corps de la fonction est exécuté.

## 13. Events

Les événements permettent au contrat d'émettre des informations consultables par les applications et les outils externes.

```solidity
contract Journal {
    event DepotEffectue(address indexed utilisateur, uint256 montant);

    function deposer() public payable {
        emit DepotEffectue(msg.sender, msg.value);
    }
}
```

- `event` déclare un événement.
- `emit` déclenche l'événement.
- `indexed` permet de faciliter certains filtrages dans les logs.

## 14. Héritage, interfaces et bibliothèques

### Héritage

Un contrat peut hériter d'un autre contrat.

```solidity
contract Parent {
    function direBonjour() public pure returns (string memory) {
        return "Bonjour";
    }
}

contract Enfant is Parent {
}
```

### Interface

Une interface déclare des fonctions que le contrat doit exposer, sans fournir leur implémentation habituelle.

```solidity
interface IStockage {
    function lire() external view returns (uint256);
}
```

### Bibliothèque

Une bibliothèque regroupe des fonctions réutilisables. Elle permet d'organiser et de mutualiser du code.

## 15. ABI et appels de fonctions

L'**ABI** (*Application Binary Interface*) décrit la manière d'interagir avec un contrat depuis une application ou un autre contrat.

Elle indique notamment :

- les fonctions disponibles ;
- les types de leurs paramètres ;
- les types de retour ;
- les événements.

Les données d'appel contiennent notamment un sélecteur de fonction, dérivé de sa signature, puis les paramètres encodés.

## 16. Exemple 1 — Compteur

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Compteur {
    uint256 public valeur;

    function incrementer() public {
        valeur += 1;
    }

    function decrementer() public {
        require(valeur > 0, "Compteur deja a zero");
        valeur -= 1;
    }

    function reinitialiser() public {
        valeur = 0;
    }
}
```

Notions illustrées :

- variable d'état ;
- fonctions publiques ;
- lecture automatique via le getter de `valeur` ;
- vérification avec `require`.

## 17. Exemple 2 — Gestion de soldes internes

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SoldesInternes {
    mapping(address => uint256) public soldes;

    function crediter(uint256 montant) public {
        require(montant > 0, "Montant invalide");
        soldes[msg.sender] += montant;
    }

    function consulter(address utilisateur)
        public
        view
        returns (uint256)
    {
        return soldes[utilisateur];
    }
}
```

**Important :** cet exemple modifie un registre interne au contrat. Il ne transfère pas réellement d'ETH.

## 18. Exemple 3 — Banque simplifiée

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Banque {
    mapping(address => uint256) public depots;

    function deposer() public payable {
        require(msg.value > 0, "Depot nul");
        depots[msg.sender] += msg.value;
    }

    function retirer(uint256 montant) public {
        require(montant > 0, "Montant nul");
        require(depots[msg.sender] >= montant, "Solde insuffisant");

        // Effets : mettre a jour l'etat avant l'appel externe.
        depots[msg.sender] -= montant;

        // Interaction : envoyer l'ETH.
        (bool succes, ) = payable(msg.sender).call{value: montant}("");
        require(succes, "Transfert echoue");
    }
}
```

Cet exemple illustre le modèle **Checks-Effects-Interactions** :

1. **Checks :** vérifier les conditions.
2. **Effects :** mettre à jour l'état.
3. **Interactions :** effectuer l'appel externe.

Pour un contrat de production, il faut également envisager des protections complémentaires contre la réentrance et tester soigneusement les chemins d'exécution.

## 19. Exemple 4 — Faucet

Un Faucet est un contrat qui reçoit de l'ETH et permet à des utilisateurs de retirer une quantité limitée.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Faucet {
    receive() external payable {}

    function withdraw(uint256 montant) public {
        require(montant <= 1 ether, "Montant trop eleve");
        require(address(this).balance >= montant, "Fonds insuffisants");

        (bool succes, ) = payable(msg.sender).call{value: montant}("");
        require(succes, "Transfert echoue");
    }
}
```

Notions illustrées :

- `receive()` pour accepter l'ETH ;
- `payable` ;
- `msg.sender` ;
- `address(this).balance` ;
- contrôle avec `require` ;
- transfert d'ETH.

Le support présente un Faucet comme premier exemple de contrat Solidity.

## 20. Sécurité des smart contracts

Un contrat déployé peut gérer des fonds. Une erreur de programmation peut donc entraîner une perte financière.

### Réentrance

Une attaque de réentrance se produit lorsqu'un contrat externe rappelle une fonction vulnérable avant que le premier appel ait terminé son exécution.

Schéma dangereux :

1. Le contrat vérifie le solde.
2. Il envoie l'ETH à l'utilisateur.
3. Le contrat externe rappelle la fonction.
4. Le solde n'ayant pas encore été mis à jour, le retrait peut être répété.

**Protection :** appliquer Checks-Effects-Interactions et, lorsque nécessaire, utiliser un verrou anti-réentrance.

### Dépassement et sous-dépassement d'entiers

Depuis Solidity 0.8.x, les opérations arithmétiques sur les entiers vérifient les dépassements par défaut et revert en cas de dépassement, sauf utilisation explicite de `unchecked`.

### Front-running

Un utilisateur peut observer une transaction en attente et tenter de faire passer sa propre transaction avant elle.

Des mécanismes comme *commit-reveal* peuvent être utiles selon le cas d'usage.

### Déni de service par boucle

Une boucle dont le nombre d'itérations dépend d'une liste qui peut grandir sans limite peut finir par dépasser le gas disponible.

**Protection :** éviter les boucles non bornées dans les fonctions indispensables au fonctionnement du contrat.

### `tx.origin` et `msg.sender`

- `msg.sender` : appelant immédiat.
- `tx.origin` : compte externe à l'origine de la transaction.

Pour contrôler les permissions, utiliser généralement `msg.sender` et non `tx.origin`.

### Confidentialité

Les données d'un contrat sur une blockchain publique ne doivent pas être considérées comme secrètes. Le mot-clé `private` contrôle l'accès depuis le code Solidity, pas la visibilité des données sur la blockchain.

## 21. ERC-20 : standard de jetons

ERC-20 est un standard de jetons fongibles sur Ethereum.

Il définit une interface commune pour gérer des tokens.

Fonctions fréquemment rencontrées :

| Fonction | Rôle |
|---|---|
| `totalSupply()` | Quantité totale de tokens |
| `balanceOf(address)` | Solde d'une adresse |
| `transfer(address, amount)` | Transfert de tokens |
| `approve(address, amount)` | Autorisation de dépense |
| `allowance(owner, spender)` | Montant autorisé |
| `transferFrom(from, to, amount)` | Transfert utilisant une autorisation |

Événements principaux :

- `Transfer`
- `Approval`

**À retenir :** un token ERC-20 n'est pas nécessairement de l'ETH. Il s'agit d'un actif géré par un smart contract.

---

# Partie IV — Comparaisons et révision de l'examen

## 1. Comparaison Bitcoin / Ethereum

| Critère | Bitcoin | Ethereum |
|---|---|---|
| Objectif principal dans le cours | Monnaie électronique décentralisée | Ordinateur mondial décentralisé |
| Unité native | BTC | ETH |
| Modèle de données | UTXO | État global clé-valeur |
| Programmabilité | Scripts de dépense | Smart contracts |
| Exécution | Vérification des règles de transaction et scripts | EVM |
| Langage étudié | Bitcoin Script | Solidity |
| Structure | Blocs liés par hash | Blocs contenant des transactions et un état résultant |
| Coût | Frais de transaction | Gas et frais en ETH |

## 2. Questions de révision — Bitcoin

1. Quel problème la décentralisation de Bitcoin cherche-t-elle à résoudre ?
2. Quelle est la différence entre une clé privée, une clé publique et une adresse ?
3. Pourquoi faut-il utiliser un générateur aléatoire cryptographiquement sûr ?
4. Qu'est-ce qu'un UTXO ?
5. Pourquoi une transaction peut-elle avoir plusieurs outputs ?
6. À quoi sert la monnaie rendue (*change*) ?
7. Comment les nœuds évitent-ils la double dépense ?
8. Quel est le rôle de la mempool ?
9. Qu'est-ce qu'une transaction coinbase ?
10. Quel est le rôle de la racine de Merkle ?
11. Pourquoi la Proof of Work est-elle coûteuse à trouver mais facile à vérifier ?
12. Pourquoi plusieurs blocs concurrents peuvent-ils apparaître temporairement ?

## 3. Questions de révision — Ethereum

1. Qu'est-ce qu'une DApp ?
2. Quelle différence existe-t-il entre un EOA et un compte de contrat ?
3. Quel est le rôle de l'EVM ?
4. Pourquoi l'exécution des smart contracts doit-elle être déterministe ?
5. Qu'est-ce que le gas ?
6. Pourquoi une transaction possède-t-elle un nonce ?
7. Quel est le rôle du Merkle-Patricia Trie ?
8. Quelle différence existe-t-il entre le modèle UTXO de Bitcoin et l'état global d'Ethereum ?
9. Comment un contrat est-il déployé ?
10. Pourquoi un contrat ne s'exécute-t-il pas spontanément en arrière-plan ?

## 4. Questions de révision — Solidity

1. À quoi servent `pragma` et `contract` ?
2. Quelle différence existe-t-il entre une variable d'état et une variable locale ?
3. Quelle différence existe-t-il entre `public`, `private`, `internal` et `external` ?
4. Quelle différence existe-t-il entre `view` et `pure` ?
5. Pourquoi une fonction qui reçoit de l'ETH doit-elle être `payable` ?
6. À quoi sert `msg.sender` ?
7. Que représente `msg.value` ?
8. À quoi sert `require` ?
9. Quelle différence existe-t-il entre `receive()` et `fallback()` ?
10. À quoi sert un `modifier` ?
11. À quoi servent les événements ?
12. Qu'est-ce qu'un mapping ?
13. À quoi sert une interface ?
14. Qu'est-ce que la réentrance ?
15. Pourquoi faut-il mettre à jour l'état avant un appel externe ?
16. Pourquoi `private` ne garantit-il pas la confidentialité ?
17. À quoi sert l'ABI ?
18. Quelle différence existe-t-il entre ETH et un token ERC-20 ?

## 5. Exercices pratiques à savoir faire

### Exercice A — Compteur

Écrire un contrat qui :

- possède une variable `uint256` ;
- permet de l'incrémenter ;
- permet de le décrémenter sans passer sous zéro ;
- expose sa valeur.

### Exercice B — Gestion de membres

Écrire un contrat qui :

- utilise `mapping(address => bool)` ;
- permet d'inscrire une adresse ;
- permet de vérifier si une adresse est inscrite.

### Exercice C — Caisse

Écrire un contrat qui :

- accepte de l'ETH ;
- mémorise le montant déposé par chaque adresse ;
- permet à un utilisateur de retirer ses fonds ;
- vérifie le solde avant le retrait ;
- met à jour le solde avant le transfert.

### Exercice D — Compréhension de code

Pour chaque fonction Solidity, savoir identifier :

- les paramètres ;
- le type de retour ;
- la visibilité ;
- la mutabilité (`view`, `pure`, `payable`) ;
- les variables lues ou modifiées ;
- les conditions d'erreur ;
- les éventuels transferts d'ETH.

---

# Partie V — Fiche express à mémoriser

## Bitcoin

```text
Clé privée → clé publique → adresse
Transaction = inputs + outputs
UTXO = sortie non dépensée
Mempool = transactions en attente
Coinbase = transaction spéciale de récompense
Merkle root = hash racine des transactions
PoW = recherche d'un hash inférieur à une cible
```

## Ethereum

```text
DApp = smart contracts + frontend
EOA = compte contrôlé par une clé
Contrat = adresse + code
EVM = machine virtuelle d'exécution
Gas = mesure des ressources consommées
Nonce = séquencement des transactions d'un compte
État = données clé-valeur
Merkle-Patricia Trie = structure de représentation de l'état
```

## Solidity

```text
contract  → déclarer un contrat
uint256   → entier non signé
address   → adresse Ethereum
mapping   → association clé / valeur
struct    → regrouper plusieurs champs
public    → accessible depuis l'extérieur
private   → accessible uniquement dans le contrat déclarant
view      → lecture de l'état
pure      → ni lecture ni modification de l'état
payable   → autorise la réception d'ETH
msg.sender → appelant immédiat
msg.value  → ETH envoyé avec l'appel
require   → vérifier une condition
emit      → émettre un événement
modifier  → factoriser une condition
```

## Les pièges classiques

- Une adresse Bitcoin n'est pas une clé privée.
- Un wallet ne contient pas littéralement les bitcoins : il gère les clés permettant de dépenser les UTXO.
- Un UTXO est une sortie non dépensée, pas un compte avec un solde.
- Une transaction Bitcoin peut créer une sortie de monnaie rendue.
- La racine de Merkle est dans l'en-tête du bloc ; l'arbre complet n'est pas stocké dans l'en-tête.
- Un contrat Ethereum ne s'exécute pas tout seul : il faut déclencher son exécution.
- Le gas est une unité de mesure des ressources, pas une monnaie distincte.
- `view` autorise la lecture de l'état ; `pure` n'autorise ni lecture ni modification de l'état.
- `payable` est nécessaire pour recevoir de l'ETH via une fonction ou un point d'entrée concerné.
- `private` ne signifie pas que les données sont secrètes.
- Toujours vérifier les conditions avant une opération sensible.
- Lors d'un retrait, appliquer Checks-Effects-Interactions pour limiter le risque de réentrance.

---

# Méthode de révision conseillée

1. Apprendre les définitions et les différences entre Bitcoin et Ethereum.
2. Savoir expliquer le cycle de vie d'une transaction Bitcoin.
3. Savoir expliquer la construction d'un bloc et la Proof of Work.
4. Comprendre la différence entre UTXO et état global.
5. Apprendre la structure d'un contrat Solidity.
6. S'entraîner à lire et écrire des fonctions avec `mapping`, `require`, `payable`, `view` et `pure`.
7. Savoir expliquer la réentrance et le modèle Checks-Effects-Interactions.
8. Refaire les exercices pratiques sans regarder les solutions.

---

## Supports de référence

- **Bitcoin.pdf** — Fausto Spoto, *Blockchain Course — Bitcoin*, 90 diapositives.
- **Ethereum.pdf** — Fausto Spoto, *Blockchain Course — Ethereum*, 160 diapositives.

> Cette fiche synthétise les notions des deux supports. Les exemples Solidity sont présentés avec une syntaxe compatible avec Solidity 0.8.x ; ils servent à comprendre les concepts et doivent être testés avant tout déploiement réel.
