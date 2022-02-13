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
    
    
</br>



## Modifiera säkerheten
`sudo dkkg-reconfigure --priority=low unattended-upgrades`
Kommandot verkställer automatiska uppdateringar,\
du kommer bli presenterad med denna ruta:
![unattended upgrades](https://user-images.githubusercontent.com/37629838/153732400-b8cf6516-2ea8-42b5-93a6-c0c555ce2f8c.png)\
Klicka ENTER på tangentbordet. 
|  ℹ️            <sub> För att stänga av automatiska uppgraderingar: `sudo dpkg-reconfigure -plow unattended-upgrades`</sub>    |
|------------------------------------------------------------------------------------------------------------------------------|
</br>

Vi är förtillfället inloggade som root, vi ska nu skapa en användare som får "root" privlegier genom att lägga till sudo framför kommandot t.ex:
när vi upppdaterade ovan skrev vi `apt update`, när vi skapat våran användare kommer vi behöver vi lägga till:
`sudo` framför kommandot: `sudo apt update`

`usermod -aG sudo dittanvändarnamn`

Detta gör vi för att säkra innehållet i server(*Det finns många anledningar att man tilldelar en användare sudo rättigheter istället för att köra allt genom root, det går jag dock inte in på här.*)

Lösenord är ofta lätta att komma ihåg och kan vara bekvämt för en själv, men ytterligare ett steg mot bättre säkerhet utesluter vi lösenord.
Vi kommer istället använda authentication keys. [Läs mer om authentication keys](https://www.ssh.com/academy/ssh/public-key-authentication).

Vi ska nu skapa två nycklar en privat och en public.


`mkdir ~/.ssh && chmod 700 ~/.ssh`  - Vi skapar en mapp i vår "home directory" `~` = detta indekerar på just på användaren. Samt ändrar rättigheter för mappen med kommandot `chmod`. Just 700 sätter rättigheter för användaren att användaren i detta fall oss själva att läsa, skriva och köra program i mappen. `mkdir` Står för make directory.  [Här kan du leka runt med inställningar och generera rädd chmod nummer](https://chmodcommand.com/chmod-400/)


För att kontrollera att att mapppen verkligen har skapats kan vi använda cd(<sub>*change directory*</sub>.
`cd ~/.ssh` Detta tar oss in i mappen, använder vi kommandot `ls` vilket listar obejekt i mappen kommer inget upp då vi ännu inte lagt in något där.


Gå nu ut ur SSH tunneln och skriv in kommandot `ssh-keygen -b 4096` i windows powershell.  (*För att komma ut ur ssh sessionen använd skriv in `logout`

Du kan även välja att lägga ett lösenord på nyckeln, det blir som ett hänglås till nyckeln som öppnar huvuddörren.

Skriv sedan in:
`scp $env:USERPROFIEL/.ssh/id_rsa.pub ubuntuanvändarnamn@ip.address.till.servern:~/.ssh/authorized_keys`

Nu blir du ombedd att skriva in lösenordet en sista gång för din server.


Logga nu in på servern med ssh igen `ssh din användarnamn@din.ip.adress.ubuntu`. Om allt blivit korrekt kommer du denna gång inte behöva skriva in lösenordet något mer. Du är istället inloggad genom att nyckeln vi skickade till servern kollar om den överenstämmer med nyckeln på din dator.


##Vi fortsättter täppa till hålen till servern.

Standard porten för att ansluta sig genom ssh är port 22, och en god praxis är att försöka undvika standardportar. Det blir yttreligare ett steg för någon som vill ta sig in att lista ut våran port. Vi vill använda någon slumpmässig port, dock viktigt att inte använda någon som redan används. 


`sudo nano /etc/ssh/sshd_config` nano efter sudo används för att kortfattat öppna programmet/filen på destinationen. I denna fil ska vi ändra standardporten till någon slumpmässig.

![portnummer](https://user-images.githubusercontent.com/37629838/153737834-c30ba26f-785b-414e-84b4-b79deb8305b1.png)

Ta bort # som står före dessa och byt ut port 22 till något valfritt, AdressFamily ändrar vi till inet, för att endast använda ipv4.

Leta sedan upp raden PermitRootLogin och byt ut till `PermitRootLogin no`, fortsätt neråt med pilarna på tangentbordet och hitta raden PasswordAuthentication byt ut till `PasswordAuthentication no`. Vi har nu stängt av rootlogin och inloggning med lösenord. Nu kan man inte logga in på servern utan våra nycklar vi skapade ovan.

På tangentbordet klicka nu CTRL + X sedan Y sen enter. Detta sparar våra ändringar.

`sudo systemctl restart sshd` Detta gör att våra nya inställningar blir aktiva när tjänsten startats om.

Innan vi loggar ut ur tjänsten, för att kontrollera att allt står rätt till. Försök ansluta till ssh i en ny windows powershell. Detta ska nu inte funka. Lägg till `-p dittvaldaportnummer`. 

Kommer du nu in har du gjort allt korrekt 🥇


##Brandvägg & portar
`sudo ss -tupln` kollar vilka portar som är öppna just nu. Kan komma upp saker här, var dock försiktigt att stänga portar då vissa kan ha funktioner du behöver.

?????????????`sudo nano /etc/default/avahi-deamon` Avahi deamon är ett sätt att koppla upp sig till skrivare osv, eftersom vi ej vill ha detta på våran servern avvaktiverar vi den. Vi kommer nu till filen och ändra där 1an till 0. Sedan CTRL + X sen Y samt ENTER.

`sudo kill -9 dittPID ` Du hittar pid med i listan vi tog upp med sudo ss -tupln.

`sudo reboot` - starta om servern.?????????????????????


`sudo apt install ufw` - Installera brandvägg.

`sudo enable ufw` - Aktivera brandväggen.

Det kommer upp en ruta som varnar: Bara till att trycka y sen ENTER.

`sudo ufw status` - Indikerar att den är aktiv.

Vi öppnade ju en port för SSH, den vi ställde in själva. 

`sudo ufw allow dinport` - detta tillåter din port för ssh.

`sudo ufw status` - kommer nu visa din öppna port i brandväggen.

Logga nu in från ett annat fönster windows powershell för att se att allt funkar.

Ett sätt att stänga ner servern ytterligare är att stänga av möjligheten att pinga oss.


`sudo nano /etc/ufw/before.rules`

leta upp # ok icmp codes for INPUT, lägg till:

`-A ufw-before-input -p icmp --icmp-type echo-request -j DROP` CTRL + X sedan y sen ENTER

`sudo reboot` Starta nu om servern, när servern startat testa nu att pinga din servern från windows.


Funkar inte? Bra 🥇

#fail2ban
scans logs files and bans ip addresses som försöker komma åt din ssh server.


`sudo apt update` - innan vi forstätter säkerställer vi att allt är uppdaterat. Om inte använd `sudo apt upgrade`

Ibland har man fått med sig paket som inte längre behövs kommandot `sudo apt autoremove` löser detta.


`sudo apt install fail2ban` - installera fail2ban

De mesta du installerar hamnar under /etc/, i detta falla /etc/fail2ban.

`cd /etc/fail2ban` - förflytta dig till mappen. `ls` för att se innehållet.

När vi nu ska ändra i fail2ban.conf gör vi först en kopia.

`sudo cp fail2ban.conf fail2ban.local`

`sudo nano fail2ban.local` öppna våran nya fil.
Gå längst ner i filen och lägg till detta:
![fail2ban](https://user-images.githubusercontent.com/37629838/153739318-a9ff201c-f676-478c-9d32-88bfe7488d07.png)

Går även att modifiera hur länge man vill att folk blir utelåsta. Standard är 600 sekunder. Vill man sätta högre kan man sätta 

bantime = sekunder.

Om du vill banna dem för alltid kan man sätta bantime = -1 ( Tänk dock på att du kan banna dig sig på detta sätt).


`sudo service fail2ban restart` starta om fail2ban

`sudo fail2ban-client status` - ser våra "fängelser" för folk som försöker göra intrång.

`sudo fail2ban-client status sshd` - visar loggen.

`sudo fail2ban-client set sshd unbanip ipadress` om du vill ta bort någon från din lista som blivit bannad.


## Authenticator

sudo apt install libpam-google-authenticator
google-authenticator
y
y
y
n
y

 sudo nano /etc/pam.d/sshd
comment out Include common-auth

längst ner lägg in :

auth required pam_google_authenticator.so

sudo nano /etc/ssh/sshd_config


CTRL + w sök efter ChallengeResponseAuthentication och change to yes.
CTRL + W sök efter Use pam och lägg till under det.
AuthenticationMethods publickey,password publickey,keyboard-interactive


`sudo service sshd restart` restart the deamon
