---
title: Erstellen eines Cassandra-Keyspace und einer Cassandra-Tabelle mit Autoskalierung für Azure Cosmos DB
description: Erstellen eines Cassandra-Keyspace und einer Cassandra-Tabelle mit Autoskalierung für Azure Cosmos DB
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: sample
ms.date: 7/29/2020
ms.openlocfilehash: 7e128b9a1e76ddf9374aa7129a51fc3639b49236
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/20/2021
ms.locfileid: "107786824"
---
# <a name="create-an-azure-cosmos-cassandra-api-account-keyspace-and-table-with-autoscale-using-azure-cli"></a>Erstellen eines Kontos, eines Keyspace und einer Tabelle mit Autoskalierung für die Azure Cosmos-Cassandra-API mit der Azure-Befehlszeilenschnittstelle (Azure CLI)
[!INCLUDE[appliesto-cassandra-api](../../../includes/appliesto-cassandra-api.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../../includes/azure-cli-prepare-your-environment.md)]

- Für diesen Artikel ist mindestens Version 2.9.1 der Azure CLI erforderlich. Bei Verwendung von Azure Cloud Shell ist die aktuelle Version bereits installiert.

## <a name="sample-script"></a>Beispielskript

[!code-azurecli-interactive[main](../../../../../cli_scripts/cosmosdb/cassandra/autoscale.sh "Create an Azure Cosmos DB Cassandra API account, keyspace, and table with autoscale.")]

## <a name="clean-up-deployment"></a>Bereinigen der Bereitstellung

Nach Ausführung des Skriptbeispiels können mit dem folgenden Befehl die Ressourcengruppe und alle damit verbundenen Ressourcen entfernt werden.

```azurecli-interactive
az group delete --name $resourceGroupName
```

## <a name="script-explanation"></a>Erläuterung des Skripts

Das Skript verwendet die folgenden Befehle. Jeder Befehl in der Tabelle ist mit der zugehörigen Dokumentation verknüpft.

| Get-Help | Notizen |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Erstellt eine Ressourcengruppe, in der alle Ressourcen gespeichert sind. |
| [az cosmosdb create](/cli/azure/cosmosdb#az_cosmosdb_create) | Erstellt ein Konto für Azure Cosmos DB. |
| [az cosmosdb cassandra keyspace create](/cli/azure/cosmosdb/cassandra/keyspace#az_cosmosdb_cassandra_keyspace_create) | Erstellt einen Azure Cosmos-Cassandra-Keyspace. |
| [az cosmosdb cassandra table create](/cli/azure/cosmosdb/cassandra/table#az_cosmosdb_cassandra_table_create) | Erstellt eine Azure Cosmos-Cassandra-Tabelle. |
| [az group delete](/cli/azure/resource#az_resource_delete) | Löscht eine Ressourcengruppe einschließlich aller geschachtelten Ressourcen. |

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zur Azure Cosmos DB-CLI finden Sie in der [Dokumentation zur Azure Cosmos DB-CLI](/cli/azure/cosmosdb).

Alle CLI-Skriptbeispiele für Azure Cosmos DB finden Sie im [GitHub-Repository zur Azure Cosmos DB-CLI](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).
