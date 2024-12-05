a)

### test.jenkinsfile

### Cleanup Stage
Dit beeld toont de stage voor het opruimen van de workspace.

![CleanUpScript](images/CleanUpScript.png)

### Dependencies Installation
Dit beeld toont de installatie van de benodigde Node.js dependencies.

![DependenciesScript](images/DependenciesScript.png)

### Post Stage
Dit beeld toont de post-verwerking die wordt uitgevoerd na de pipeline.

![PostScript](images/PostScript.png)

### Tools Stage
Dit beeld toont de installatie van de gebruikte tools in de pipeline.

![ToolsScript](images/ToolsScript.png)

### Console Output
Dit beeld toont de console-uitvoer van de jenkinsfile

![ConsoleOutput](images/ConsoleOutput.png)


### Docker build, push en deploy

Om docker te kunnen pushen ga ik credentials username met password toevoegen in jenkins:

![Credentials](images/docker-credential.png)

Docker image is succesvol naar DockerHub gepusht 

![DockerHub](images/dockerhub.png)

Console output:

![ConsoelOutput](images/console_deploy.png)


b)


# Production Server
> **Author - Raphael *(everything below this point)***

## Installeren Docker
Installatiegids: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04

Ondernomen stappen:
1. Updaten huidige packages
   1. sudo apt update
   2. PowerShell: ![updaten packages adhv sudo apt update](images/prod-dock-install-aptupdate.png)
2. Installeren packages om de package manager via HTTPS te laten communiceren (Docker repos gebruiken HTTPS)
   1. sudo apt install apt-transport-https ca-certificates curl software-properties-common
   2. PowerShell: ![enable https communication](images/prod-dock-install-https.png)
3. Toevoegen GPG key (authenticatie met Docker repositories)
   1. curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   2. PowerShell: ![toevoegen gpg key](images/prod-dock-install-gpgkey.png)
4. Toevoegen Docker repo aan package lijst (resource list)
   1. echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   2. PowerShell: ![adding repo to apt](images/prod-dock-install-addtoapt.png)
5. Refreshen van de resources lijst
   1. sudo apt update
   2. PowerShell: ![refresh lijst](images/prod-dock-install-refresh.png)
6. Kijken of de docker-ce package beschikbaar is om te downloaden
   1. apt-cache policy docker-ce
   2. PowerShell: ![check package en download](images/prod-dock-install-check.png)
7. Installeren Docker Community Edition
   1. sudo apt install docker-ce
   2. PowerShell: ![installeer docker](images/prod-dock-install-actualinstall.png)
8. Checken of Docker aan het runnen is
   1. sudo systemctl status docker
   2. PowerShell: ![checking docker status](images/prod-dock-install-checkstatus.png)

## Vermijden van sudo
1. Add user to docker group
   1. sudo usermod -aG docker ${USER}
   2. PowerShell: ![add user to group](images/prod-sudo-addtogroup.png)
2. AWS Ubuntu instance heeft geen passwoord, deze eerst instellen
   1. sudo passwd ubuntu      
   2. PowerShell: ![set password](images/prod-sudo-setpasswd.png)
3. Log terug in
   1. su - ${USER}
   2. PowerShell: ![log back in](images/prod-sudo-relog.png)
4. Kijken of de gebruiker in de "docker" groep is
   1. groups
   2. PowerShell: ![check groups](images/prod-sudo-checkgroups.png)

## Setting up the pipeline & other preparations
1. Pipeline integratioe met production.jenkins. Ik heb besloten SSH te gebruiken omwille van de volgende redenen
   1. het vermijdt het gebruik van gebruikersnamen en passwoorden, het is dus veiliger
   2. het is in het algemeen makkelijker om hiermee te werken, eens de keypairs matchen moet je niet nog eens een passwoord ingeven, het is dus ook deels uit gemakzucht (zoals een echte programmeur)
   3. we hebben de vorige keer pijnlijk ondervonden dat HTTPS niet werkt op een privé repository, het gebruik van SSH is dus flexiebeler en staat ons toe in de toekomst ooit onze repo van publiek naar privé om te zetten
   4. ![pipeline configuration](images/prod-jenk-pipelineconfiguration.png)
2. Installeer/Update SSH Agent in Jenkins
   1. ![installing ssh agent](images/prod-jenk-installSSHAgent.png)
3. Configureren van de SSH Agent: een nieuwe SSH key genereren voor toegang van de testserver naar de production server
   1. ![generate new ssh key](images/prod-jenk-newsshkey.png)
4. Toevoegen nieuwe inbound rule: IP adress testserver moet geaccepteerd worden door production server (vergeet de "/32" niet)
   1. ![configure inbound rules](images/prod-jenk-inboundrules.png)
5. Kijken of we kunnen verbinden vanuit de test server naar de productie server
   1. ![check connection test to prod](images/prod-jenk-checkconnection.png)

## The actual pipeline
1. Connectie werkt! De Jenkins (testserver) kan nu dus communiceren met de productieserver. Tijd om de pipeline te runnen...
   1. Pipeline Script: ![the pipeline script](images/prod-jenk-pipelinescript.png)
   2. Pipeline Output Pt. 1:![Jenkins Pipeline Output 1](images/prod-jenk-piplineoutput1.png)
   3. Pipeline Output Pt. 2:![Jenkins Pipeline Output 2](images/prod-jenk-piplineoutput2.png)
   4. Pipeline Output Pt. 3:![Jenkins Pipeline Output 3](images/prod-jenk-piplineoutput3.png)
