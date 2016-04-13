#master ja kaksi slavea

Aloitin tehtävän asentamalla virtual boxilla yhden ubuntu serverin v 14.04 ja kaksi xubuntua v 14.04.  Laitoin serverille 4 gigaa rammia ja työasemille gigan. Vaihdoin verkkoadapterista natin bridgen adapteriksi, jotta koneet näkisivät toisensa.

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
logeja katsomalla selvisi nopeasti, että vika oli ssl sertifikaateissa. Puppetin dokumentaatiosta löytyi onneksi ratkaisu tämän korjaamiseksi

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

https://docs.puppet.com/puppet/4.3/reference/quick_start_helloworld.html
tein ohjeen mukaan ja lopetin uuten sertifikaatti ongelmaan ensi kerralla avaimet uusiksi.
https://docs.puppet.com/puppet/4.4/reference/ssl_regenerate_certificates.html

lisää hosts tiedostoihin koko domain nimet ws1.tielab.haaga-helia.fi ws2.tielab.haaga-helia.fi srv.tielab.haaga-helia.fi

less /etc/motd


