Aloitin puppet-kotitehtävän asentamalla puppet agentin komennoilla:
sudo apt-get update
sudo apt-get install puppet

Tämän jälkeen loin tiedoston puppetin testaamista varten komennolla:
puppet apply -e 'file {"/tmp/testifilu": content => "Ensimmäinen puppetmoduulini!\n" }'

Terminaali antoi kolmirivisen tulosteen, joka tarkoitti onnistunutta tiedoston luomista
Notice: Compiled catalog for xubuntu.tielab.haaga-helia.fi in environment production in 0.03 seconds
Notice: /Stage[main]/Main/File[/tmp/testifilu]/ensure: defined content as '{md5}c156bdd99ad07511ef5aab99710f1e6a'
Notice: Finished catalog run in 0.01 seconds

Kävin vielä tämän järkeen tarkastamassa manuaalisesti oliko tiedosto oikeasti luotu avaamalla tiedoston komennolla:
less /tmp/testifilu

Tämän jälkeen loin Puppet moduuleille uuden kansion
mkdir puppet

Luodaan puppet kansioon modules kansio, testi kansio, ja manifests kansio jonne luodaan init.pp tiedosto.
mkdir -p modules/testi/manifests/
init.pp tiedostoon laitoin seuraavan koodin

class testi { 
        file { '/tmp/testiModuuli':
                content => "Toimiiks tää?\n"
}}
Tämän jälkeen pääsin ajamaan moduulia. Tässä komento ja tuloste
xubuntu@xubuntu:~/puppet$ puppet apply --modulepath modules/ -e 'class {"testi":}'
Notice: Compiled catalog for xubuntu.tielab.haaga-helia.fi in environment production in 0.04 seconds
Notice: /Stage[main]/Testi/File[/tmp/testiModuuli]/ensure: defined content as '{md5}dfa00ec11f8fe814ebd83b40f9cd93c3'
Notice: Finished catalog run in 0.02 seconds

Tämän jälkeen menin vielä temppiin katsomaan toimiko moduuli
less /tmp/testiModuuli

Moduuli näytti toimineen oikein, koska tiedosto aukesi oikealla sisällöllä
