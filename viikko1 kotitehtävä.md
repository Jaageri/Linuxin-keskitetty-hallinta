Aloitin puppet-kotitehtävän asentamalla puppet agentin komennoilla:
sudo apt-get update<br>
sudo apt-get install puppet<br>

Tämän jälkeen loin tiedoston puppetin testaamista varten komennolla:<br>
puppet apply -e 'file {"/tmp/testifilu": content => "Ensimmäinen puppetmoduulini!\n" }'<br>

Terminaali antoi kolmirivisen tulosteen, joka tarkoitti onnistunutta tiedoston luomista<br>
Notice: Compiled catalog for xubuntu.tielab.haaga-helia.fi in environment production in 0.03 seconds<br>
Notice: /Stage[main]/Main/File[/tmp/testifilu]/ensure: defined content as '{md5}c156bdd99ad07511ef5aab99710f1e6a'<br>
Notice: Finished catalog run in 0.01 seconds<br>

Kävin vielä tämän järkeen tarkastamassa manuaalisesti oliko tiedosto oikeasti luotu avaamalla tiedoston komennolla:<br>
less /tmp/testifilu<br>

Tämän jälkeen loin Puppet moduuleille uuden kansion<br>
mkdir puppet<br>

Luodaan puppet kansioon modules kansio, testi kansio, ja manifests kansio jonne luodaan init.pp tiedosto.<br>
mkdir -p modules/testi/manifests/<br>
init.pp tiedostoon laitoin seuraavan koodin<br>

class testi { <br>
        file { '/tmp/testiModuuli':<br>
                content => "Toimiiks tää?\n"<br>
}}
Tämän jälkeen pääsin ajamaan moduulia. Tässä komento ja tuloste<br>
xubuntu@xubuntu:~/puppet$ puppet apply --modulepath modules/ -e 'class {"testi":}'<br>
Notice: Compiled catalog for xubuntu.tielab.haaga-helia.fi in environment production in 0.04 seconds<br>
Notice: /Stage[main]/Testi/File[/tmp/testiModuuli]/ensure: defined content as '{md5}dfa00ec11f8fe814ebd83b40f9cd93c3'<br>
Notice: Finished catalog run in 0.02 seconds<br>

Tämän jälkeen menin vielä temppiin katsomaan toimiko moduuli<br>
less /tmp/testiModuuli<br>
<br>
Moduuli näytti toimineen oikein, koska tiedosto aukesi oikealla sisällöllä<br>
