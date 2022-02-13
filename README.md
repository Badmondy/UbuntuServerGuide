# Ubuntu server guide


For this guide i have installed Ubuntu Server version 20.04.03 LTS(<sub><sup>most stable</sup></sub>)\
running on a Raspberry Pi 8gb. Vilken hårdvara du väljer att installera på är upp till dig.




En grund i ett säkert system är att läckor och skadliga programvaror patchas/uppdateras,
detta är vad dessa kommandon ovan gör. 

</br>
</br>

* För att hämta alla senaste uppdateringar:

   		   apt update

* För att uppdatera alla tillgängliga:

  		   apt upgrade

* Istället för att manuellt uppdatera alla nytillkomna uppdateringar kan man installera automatiska uppdateringar:

   		  apt install unattended-upgrades

    | :zap:        <sub> Ska du driva servern i produktion med många klienter är det en bättre praxis att uppdatera manuellt.</sub>|
    |------------------------------------------------------------------------------------------------------------------------------|
    | ℹ️        <sub> För att stänga av automatiska uppgraderingar: `sudo dpkg-reconfigure -plow unattended-upgrades`</sub>|
    |------------------------------------------------------------------------------------------------------------------------------|
</br>



## Modifiera säkerheten
`sudo dkkg-reconfigure --priority=low unattended-upgrades`
Kommandot verkställer automatiska uppdateringar,\
du kommer bli presenterad med denna ruta:
![unattended upgrades](https://user-images.githubusercontent.com/37629838/153732400-b8cf6516-2ea8-42b5-93a6-c0c555ce2f8c.png)\
Klicka ENTER på tangentbordet.


Vi är förtillfället inloggade som root, vi ska nu skapa en användare som får "root" privlegier genom att lägga till sudo framför kommandot t.ex:
när vi upppdaterade ovan skrev vi `apt update`, när vi skapat våran användare kommer vi behöver vi lägga till:\
`sudo` framför kommandot = `sudo apt update`.

`usermod -aG sudo networkmondy`

Detta gör vi för att säkra servern.

Lösenord är ofta lätta att komma ihåg och kan vara bekvämt för en själv, men för en säker server skippar vi lösenord helt.
Vi kommer istället använda authentication keys. [Läs mer om authentication keys](https://origo-map.github.io/origo-documentation/latest/#origo-api).

Vi ska nu skapa två nycklar en privat och en public.










