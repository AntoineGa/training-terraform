# Coding Dojo Terraform

<!-- toc -->

- [Prérequis](#prerequis)
- [Installation de Terraform](#installation-de-terraform)
  * [Installation manuelle](#installation-manuelle)
  * [Installation via Chocolatey](#installation-via-chocolatey)
- [Exercice 1 : Initialisation de Terraform et création d'un resource group](#exercice-1--initialisation-de-terraform-et-creation-dun-resource-group)
- [Exercice 2 : Création d'un Virtual Network](#exercice-2--creation-dun-virtual-network)
- [Exercice 3 : Utilisation de variables](#exercice-3--utilisation-de-variables)
- [Exercice 4 : Création de workspaces (environnements)](#exercice-4--creation-de-workspaces-environnements)
- [Exercice 5 : Travailler en équipe sur le même projet (remote tfstate)](#exercice-5--travailler-en-equipe-sur-le-meme-projet-remote-tfstate)

<!-- tocstop -->

## Prérequis

* Une souscription Azure active (dont vous êtes l'administrateur)
* Visual Studio Code
  * Terraform extension : https://marketplace.visualstudio.com/items?itemName=mauve.terraform
  * Terraform snipets extension : https://marketplace.visualstudio.com/items?itemName=mindginative.terraform-snippets
* Azure CLI : https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest
* Git (optional) : https://git-scm.com/download/win

## Installation de Terraform

Vous pouvez installer Terraform de deux façons

### Installation manuelle

* Télécharger le package sur le site de Terraform : https://www.terraform.io/downloads.html
* Mettre à jour le PATH pour rendre le binaire Terraform accessible : https://stackoverflow.com/questions/1618280/where-can-i-set-path-to-make-exe-on-windows

### Installation via Chocolatey

Chocolatey est un package manager Windows. Il permet également d'installer Terraform. La seule différence avec l'installation manuelle est qu'il ajoute automatiquement Terraform dans le PATH. Il permet également de le mettre à jour facilement.

Si vous n'avez pas Chocolatey, vous pouvez l'installer via PowerShell en une ligne de commande : https://chocolatey.org/install#install-with-powershellexe

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

Vous pouvez ensuite installer Terraform

```bash
choco install terraform
```

## Exercice 1 : Initialisation de Terraform et création d'un resource group

Un projet terraform est constitué d'un ensemble de fichiers `*.tf`. Les fichiers Terraform sont écrits en HCL (HashiCorp Configuration Language).

Dans un répertoire vide, créer un premier fichier `main.tf`

```bash
provider "azurerm" { }

# Snippet tf-azurerm_resource_group
resource "azurerm_resource_group" "rg_coding_dojo" {
   name = "rg_coding_dojo"
   location = "West Europe"
}
```

* La première ligne indique à Terraform quel provider utiliser
* Le deuxième bloc va créer un Resource Group `rg_coding_dojo`

Dans un terminal, aller dans le répertoire dans lequel vous avez créé ce fichier.

La première étape est d'initialiser le projet Terraform. Taper la commande suivante :

```bash
terraform init
```

Terraform va télécharger la dernière version disponible du provider Azure, et va créer un répertoire `.terraform` dans lequel seront stockés les artefacts nécessaires au fonctionnement de Terraform. Ce dossier doit être dans le `.gitignore` si vous mettez vos sources sur Git.

Authentifiez vous sur votre souscription Azure en utilisant la CLI Azure (cela ne fonctionnera pas si vous vous authentifiez avec une commande PowerShell)

```bash
az login

# Si vous avez plusieurs souscriptions associées à votre compte, vous pouvez en sélectionner une en particulier
az account set --subscription <subscription_id>
```

> Il y a plusieurs façons de s'authentifier sur Azure pour faire fonctionner Terraform. La façon ci-dessus est juste la plus rapide, mais en situation réelle, il faudra préférer une authentification à l'aide d'une App déclarée dans Azure AD

Après être identifié sur Azure, taper la commande :

```bash
terraform plan
```

La commande ne va rien faire sur Azure, mais va indiquer en output les opérations qui seront réalisées. C'est une sorte de dry run. Vous devriez voir qu'un resource group va être créé, et qu'aucune resource ne va être mise à jour ou détruite.

Pour lancer l'opération, tapez la commande :

```bash
terraform apply

# Tapez yes pour confirmer la commande
```

Avec la commande `az group list` ou bien directement depuis le portail Azure, vous devez maintenant voir le resource gorup `rg_coding_dojo`

Un fichier `terraform.tfstate` a été créé à la racine du répertoire. Il contient l'état de l'infrastructure déployée par votre projet Terraform. Encore une fois, ne pas mettre ce fichier dans votre Source Control, même si vous êtes plusieurs à modifier l'infrastructure.

## Exercice 2 : Création d'un Virtual Network

Nous allons maintenant ajouter un Virtual Network et un Subnet dans notre resource group. Pour cela, créer un deuxième fichier `vnet.tf` et ajouter les deux ressources :

```bash
# Snippet : tf-azurerm_virtual_network
resource "azurerm_virtual_network" "vnet_coding_dojo" {
   name = "vnet_coding_dojo"
   location = "West Europe"
   resource_group_name = "${azurerm_resource_group.rg_coding_dojo.name}"
   address_space = ["10.0.0.0/16"]
   dns_servers = ["10.0.0.4"]
}

# Snippet : tf-azurerm_subnet
resource "azurerm_subnet" "subnet_coding_dojo" {
   name = "subnet_coding_dojo"
   resource_group_name = "${azurerm_resource_group.rg_coding_dojo.name}"
   virtual_network_name = "${azurerm_virtual_network.vnet_coding_dojo.name}"
   address_prefix = "10.0.1.0/24"
}
```

Exécutez la commande `terraform plan`

Le resource group étant déjà créé, Terraform indique seulement deux ressources à créer. Faites un `terraform apply` pour créer les ressources.

Pour détruire les ressouces, faites un `terraform destroy`.

## Exercice 3 : Utilisation de variables

## Exercice 4 : Création de workspaces (environnements)

## Exercice 5 : Travailler en équipe sur le même projet (remote tfstate)