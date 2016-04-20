#master ja kaksi slavea  

Aloitin tehtävän asentamalla virtual boxilla yhden ubuntu serverin v 14.04 ja kaksi xubuntua v 14.04.  Laitoin serverille 4 gigaa rammia ja työasemille gigan. Vaihdoin verkkoadapterista natin bridgen adapteriksi, jotta koneet näkisivät toisensa.  

teen kaikki toimenpiteet roottina, joten
sudo -i  

Lisätään puppetin lähteet apt repoihin ja asennetaan puppet kaikkiin kolmeen koneeseen  
wget https://apt.puppetlabs.com/puppetlabs-release-pc1-trusty.deb  
dpkg -i pupeptlabs-release-pc1-trusty.deb  
apt-get update  

Serverille apt-get install puppetserver  
Työasemille apt-get install puppet-agent  
  
Tämän jälkeen vaihdoin serverin hostnamen lyhemmäksi, joten vaihdoin /etc/hostname tiedostoon nanolla hostnameksi srv  
Tein saman työasemille ja laitoin nimiksi ws1 ja ws2  
Lisäsin kaikkien koneiden /etc/hosts tiedostoon kaikkien koneiden nimet ja ip:t ja pingasin niitä keskenään varmistaakseni, että kaikki toimii  

Tämän jälkeen määrittelin työasemien puppetclienttien serverin lisäämällä nanolla niiden /etc/puppetlabs/puppet/puppet.conf -tiedostoon rivit    
[main]    
server = srv    

Seuraavaksi boottasin puppetserverin  
/etc/init.d/puppetserver restart  
Ensimmäisellä yrityksellä tuli ...fail. -ilmoitus  
logeja katsomalla selvisi nopeasti, että vika oli ssl-sertifikaateissa. Puppetin dokumentaatiosta löytyi onneksi ratkaisu tämän korjaamiseksi  

puppet resource service puppet ensure=stopped pysäytetään puppet agent  
puppet resource service puppetserver ensure=stopped pysäytetään puppet master  
poistetaan puppetin ssl-kansio rm -r /etc/puppetlabs/puppet/ssl  
sudo puppet cert list -a luodaan sertifikaattilista uudestaan  
puppet master --no-daemonize --verbose luodaan puppet masterille uudet sertifikaatit  
puppet resource service puppetserver ensure=running käynnistetään puppet master uudestaan  
puppet resource service puppet ensure=running käynnistetään puppet agent uudestaan  

tämän jälkeen yritin hakea sertifikaatteja työasemilla  
käynnistin puppet servicen komennolla  
puppet resource service puppet ensure = running enable =true  

Siirryin takaisin serverille jossa hain sertifikaattilistaa komennolla  
puppet cert-list --all  

sertifikaattilista näytti ainoastaan serverin sertifikaatin. Työasemien sertifikaattipyynnöt puuttuivat kokonaan, joten palasin lukemaan puppetin dokumentaatiota.  

Suljin puppet agentit työasemilta  
puppet resource service puppet ensure=stopped  

poistin ssl kansion   
rm -r /etc/puppetlabs/puppet/ssl  

käynnistin puppet agentit uudestaan  
sudo puppet resource service puppet ensure=running  

Tämän jälkeen hain sertifikaattilistoja uudestaan serverillä ja näin työasemien sertifikaattipyynnöt  

allekirjoitin pyynnöt  
puppet cert sign ws1.tielab.haaga-helia.fi  
puppet cert sign ws2.tielab.haaga-helia.fi  

tein tämän jälkeen hello worldin ohjeen https://docs.puppet.com/puppet/4.3/reference/quick_start_helloworld.html mukaan, kunnes törmäsin uuteen sertifikaattiongelmaan käyttäessäni työasemilla puppet agent-t -komentoa.  

Ilmeisesti sertifikaattiongelma johtui siitä, että olin laittanut työasemien /etc/hosts tiedostoon koneiden nimet ilman   domainia. Vaihdoin ne siis muotoon ws1.tielab.haaga-helia.fi Tämän jälkeen suoritin aiemmin tehdyt sertifikaattitoimenpiteet uusiksi  

tämän jälkeen ajoin puppet agent -t komennon uusiksi työasemilla ja tekemäni motd tekstitiedosto ilmestyi /etc -kansion alle  





käynnistys automaattisesti cd /etc/rc2.d  
  
ln -s /etc/init.d/puppetserver 99puppetserver

manuaalisesti sudo puppet resource service puppetserver ensure=running

https://docs.puppet.com/puppet/4.3/reference/quick_start_helloworld.html  
https://docs.puppet.com/puppet/4.4/reference/ssl_regenerate_certificates.html  

#paketin asennus toiselle työasemalle puppetin avulla  

käyttäjän määrittely serverille: lisätään tiedostoon /etc/puppetlabs/code/environments/production/manifests/site.pp 
määritellään siis mitä yksittäinen node hakee puppetilla, työaseman nimen perusteella  


node ws1 {   
        package { 'openssh-server':  
          ensure => 'installed',  
          }  
    }  
node ws2 {  
        user { 'jaakko':  
        name => 'jaakko',  
        ensure => 'present',  
        home => '/home/jaakko',  
        managehome => 'yes'.  
        password => '"pitkä rimpsu merkkejä"'  


loin myös valmiiksi serverille käyttäjän 'jaakko' saadakseni salasanan kryptatyssa muodossa site.pp tiedostoon.Tiedot siirsin site.pp tiedostoon tällä tavalla.  

grep jaakko /etc/shadow >> /etc/puppetlabs/code/environments/produciton/manifests/site.pp  
Itse salasanaosa tuosta on $6 merkeillä alkava kaksoispisteiden välissä oleva osa, jonka laitoin passwd kohtaan  

tämän jälkeen ajoin puppet agent -t ws2:sella ja käyttäjä luotiin    
Info: Using configured environment 'production'  
Info: Retrieving pluginfacts  
Info: Retrieving plugin  
Info: Caching catalog for ws2.tielab.haaga-helia.fi  
Info: Applying configuration version '1461138443'  
Notice: /Stage[main]/Main/Node[ws2]/User[jaakko]/ensure: created  
Notice: Applied catalog in 0.27 seconds  

Tein saman ws1:sella ja openssh-server asentui
Info: Using configured environment 'production'  
Info: Retrieving pluginfacts  
Info: Retrieving plugin  
Info: Caching catalog for ws1.tielab.haaga-helia.fi  
Info: Applying configuration version '1461138466'  
Notice: /Stage[main]/Main/Node[ws1]/Package[openssh-server]/ensure: created  
Notice: Applied catalog in 10.62 seconds  



