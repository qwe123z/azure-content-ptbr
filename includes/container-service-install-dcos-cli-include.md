<properties
   pageTitle="Instalar a CLI do DC/OS | Microsoft Azure"
   description="Instale a CLI do DC/OS."
   services="container-service"
   documentationCenter=""
   authors="rgardler"
   manager="timlt"
   editor=""
   tags="acs, azure-container-service"
   keywords="Contêineres, Microsserviços, DC/OS, Azure"/>

<tags
   ms.service="container-service"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="05/10/2016"
   ms.author="rogardle"/>

>[AZURE.NOTE] Isso é para trabalhar com clusters ACS baseados no DC/OS. Não é necessário fazer isso para clusters ACS baseados no Swarm.

Primeiro, [conectar-se ao cluster ACS baseado no DC/OS](../articles/container-service/container-service-connect.md). Depois de fazer isso, você poderá instalar a CLI do DC/OS em seu computador cliente com os comandos abaixo:

```bash
sudo pip install virtualenv
mkdir dcos && cd dcos
wget https://raw.githubusercontent.com/mesosphere/dcos-cli/master/bin/install/install-optout-dcos-cli.sh
chmod +x install-optout-dcos-cli.sh
./install-optout-dcos-cli.sh . http://localhost --add-path yes
```

Se estiver usando uma versão antiga do Python, talvez você observe algumas “InsecurePlatformWarnings”. Você pode ignorá-los com segurança.

Para começar a usar sem reiniciar o shell, execute:

```bash
source ~/.bashrc
```

Esta etapa não será necessária quando você iniciar novos shells.

Agora você pode confirmar se a CLI está instalada:

```bash
dcos --help
```