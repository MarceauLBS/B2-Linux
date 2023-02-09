# TP Léo rattrapage

## I. Le Gentil Setup

Salut mon petit chat !

Je vais te montrer comment installer un serveur Emby, y mettre un reverse proxy et installer un client nfs + un script de backup ^^

Installation trop dure (Emby server) :

Voici le démarrage des services :
```
[marceau@localhost ~]$ sudo dnf install https://github.com/MediaBrowser/Emby.Releases/releases/download/4.7.11.0/emby-server-rpm_4.7.11.0_x86_64.rpm
[marceau@localhost /]$ sudo systemctl start emby-server
[marceau@localhost /]$ sudo systemctl status emby-server
● emby-server.service - Emby Server is a personal media server with apps on just about every device
     Loaded: loaded (/usr/lib/systemd/system/emby-server.service; enabled; vendor preset: disabled)
     Active: active (running) since Mon 2023-02-06 22:03:12 CET; 14min ago
   Main PID: 44074 (EmbyServer)
      Tasks: 13 (limit: 5905)
     Memory: 109.1M
        CPU: 5.961s
     CGroup: /system.slice/emby-server.service
             └─44074 /opt/emby-server/system/EmbyServer -programdata /var/lib/emby -ffdetect /opt/emby-server/bin/ffdet>

Feb 06 22:03:12 web emby-server[44074]: Info App: Entry point completed: Emby.Notifications.Notifications. Duration: 0.>
Feb 06 22:03:12 web emby-server[44074]: Info App: Starting entry point Emby.Server.Sync.SyncManagerEntryPoint
Feb 06 22:03:12 web emby-server[44074]: Info App: SyncRepository Initialize taking write lock
Feb 06 22:03:12 web emby-server[44074]: Info App: SyncRepository Initialize write lock taken
Feb 06 22:03:12 web emby-server[44074]: Info App: Entry point completed: Emby.Server.Sync.SyncManagerEntryPoint. Durati>
Feb 06 22:03:12 web emby-server[44074]: Info App: Starting entry point Emby.Server.Sync.SyncNotificationEntryPoint
Feb 06 22:03:12 web emby-server[44074]: Info App: Entry point completed: Emby.Server.Sync.SyncNotificationEntryPoint. D>
Feb 06 22:03:12 web emby-server[44074]: Info App: Starting entry point EmbyServer.Windows.LoopUtilEntryPoint
Feb 06 22:03:12 web emby-server[44074]: Info App: Entry point completed: EmbyServer.Windows.LoopUtilEntryPoint. Duratio>
Feb 06 22:03:12 web emby-server[44074]: Info App: All entry points have started
lines 1-20/20 (END)

[marceau@localhost /]$ sudo systemctl enable emby-server
```
Voila mon petit chat maintenant nous allons consulter nos logs ^^
```
[marceau@localhost /]$ sudo journalctl -u emby-server
```
Je vais te montrer maintenant quels processus sont lancés par ce service :3
```
[marceau@localhost /]$ ps | grep emby
[marceau@localhost /]$ ps aux | grep emby-server
emby       44074  0.7 17.2 3173268 169516 ?      Ssl  22:19   0:06 /opt/emby-server/system/EmbyServer -programdata /var/lib/emby -ffdetect /opt/emby-server/bin/ffdetect -ffmpeg /opt/emby-server/bin/ffmpeg -ffprobe /opt/emby-server/bin/ffprobe -restartexitcode 3 -updatepackage emby-server-rpm_{version}_x86_64.rpm
marceau     44276  0.0  0.2   6408  2152 pts/0    S+   22:25   0:00 grep --color=auto emby-server
```
Maintenant imagine des petits bateaux ... nous allons voir à quel port ces processus sont amarrés :
```
[marceau@localhost /]$ sudo ss -altnp | grep EmbyServer
LISTEN 0      512                *:8096            *:*    users:(("EmbyServer",pid=44294,fd=193))
```
Nous allons voir qui peut lancer le serveur ^^ (il faut suivre c'est encore long)
```
[marceau@localhost /]$ ps aux | grep emby-server
emby       44294  1.6 16.2 3172408 160164 ?      Ssl  22:28  0:06 /opt/emby-server/system/EmbyServer -programdata /var/lib/emby -ffdetect /opt/emby-server/bin/ffdetect -ffmpeg /opt/emby-server/bin/ffmpeg -ffprobe /opt/emby-server/bin/ffprobe -restartexitcode 3 -updatepackage emby-server-rpm_{version}_x86_64.rpm
marceau     44377  0.0  0.2   6408  2172 pts/0    S+   22:32   0:00 grep --color=auto emby-server
```
Pour accéder à l'interface web, nous allons faire quelques manip' ! Suit bien !

ATTENTION ! Il ne faut pas oublier d'ouvrir le firewall petit chat !!

Nous allons donc ouvrir un port firewall (ou pare-feu pour les pures Frenchy)

Pour ce faire nous allons nous souvenir des petits bateaux (pour trouver le port) ^^
```
[marceau@localhost /]$ sudo firewall-cmd --permanent --add-port=8096/tcp
success
[marceau@localhost /]$ sudo firewall-cmd --reload
success
```
Pour la suite, nous allons voir comment accueillir nos médias !

Alors mon petit chat qu'est-ce qu'on fait maintenant ?

(Toi): *hmmmmm*

Bon je vais t'aider : nous allons créer un répertoire pour les mettre biens au chaud ! 
```
[marceau@localhost /]$ sudo mkdir /srv/media
```
Parfait! Maintenant, pour s'en servir il faut donner les droits au bon user :
```
[marceau@localhost media]$ sudo chown emby:emby /srv/media/
```
Mais bon faut pas abuser non plus, on va le restreindre un peu les autres marioles
```
[marceau@localhost media]$ sudo chmod 750 /srv/media/
```
Envoyer le fichier audio :
```
PS C:\WINDOWS\system32> scp C:\Users\marce\Music\ChantDeChats.mp3 emby@10.25.1.11:/srv/media
emby@10.25.1.11's password:
ChantDeChats.mp3                                                                  100% 4057KB  99.1MB/s   00:00
```
Bon mon petit chat, la Partie 1 est terminée !! Bravo à toi !! Pour fêter ça ... Tu veux t'ambiancer ? 

Tu peux le faire grâce à quelques manip' ! Il te suffit de mettre le chemin de tes médias directement depuis ton serveur Emby !

Enjoy !! <3

## 2. Le Méchant Reverse Proxy

BON ! Tu vas m'écouter maintenant (stp)

On est sur la Partie 2 et c'est pas de la rigolade !!!

Tu vas mettre en place NGINX et plus vite que ça 😡
```
[marceau@localhost ~]$ sudo dnf install nginx
[...]
Y
[...]
Complete !
```
Bon ça va t'as réussis... mais vas-tu réussir à démarer NGINX ???
```
[marceau@localhost ~]$ sudo systemctl start nginx
[marceau@localhost ~]$ sudo systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
    [...]
     Active: active (running) since Tue 2023-02-07 10:52:23 CET; 15s ago
    [...]

Feb 07 10:52:23 proxy systemd[1]: Starting The nginx HTTP and reverse proxy server...
Feb 07 10:52:23 proxy nginx[40372]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Feb 07 10:52:23 proxy nginx[40372]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Feb 07 10:52:23 proxy systemd[1]: Started The nginx HTTP and reverse proxy server.
```
Ok ok je vois ... tu es coriace ...

Tu sais me trouver tous les ports ?
```
[marceau@localhost ~]$ sudo ss -alpnt
State  Recv-Q Send-Q  Local Address:Port   Peer Address:Port Process
LISTEN 0      511           0.0.0.0:80          0.0.0.0:*     users:(("nginx",pid=40375,fd=6),("nginx",pid=40374,fd=6))
LISTEN 0      128           0.0.0.0:22          0.0.0.0:*     users:(("sshd",pid=21221,fd=3))
LISTEN 0      511              [::]:80             [::]:*     users:(("nginx",pid=40375,fd=7),("nginx",pid=40374,fd=7))
LISTEN 0      128              [::]:22             [::]:*     users:(("sshd",pid=21221,fd=4))
```
Bien ... OUVRE LE FIREWALL TOUT DE SUITE !
```
[marceau@localhost ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
[marceau@localhost ~]$ sudo firewall-cmd --reload
success
```
Tu es plutôt fort je dois le reconnaître ... 

Détermine les user de NGINX "Monsieur je sais tout" !
```
[marceau@localhost ~]$ ps -ef | grep nginx
root       40374       1  0 10:52 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx      40375   40374  0 10:52 ?        00:00:00 nginx: worker process
marceau     40402     960  0 10:52 pts/0    00:00:00 grep --color=auto nginx
```
Ouais ouais c'est ça c'est ça ... *a le seum* 

Et tout ça tu peux y accéder ?
```
[marceau@localhost ~]$ curl 10.25.1.12:80
<!doctype html>
<html>
  <head>
    [...]
```
Bon c'est bien tu travailles bien à l'école...

Configure bien NGINX et on verra si je suis plus doux !

Et je veux pas un mot !
```
[marceau@localhost ~]$ cat /etc/nginx/conf.d/nginx.conf
server {
    # On indique le nom que client va saisir pour accéder au service
    # Pas d'erreur ici, c'est bien le nom de web, et pas de proxy qu'on veut ici !
    server_name web.peche.linux;

    # Port d'écoute de NGINX
    listen 80;

    location / {
        # On définit des headers HTTP pour que le proxying se passe bien
        proxy_set_header  Host $host;
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-Proto https;
        proxy_set_header  X-Forwarded-Host $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;

        # On définit la cible du proxying
        proxy_pass http://10.25.1.11:8096;
    }

    # Deux sections location recommandés par la doc NextCloud
    location /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }

    location /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }
}
```
Tu m'fais c***r à la fin !!!!

Dépêche toi de mettre en place le HTTPS
```
[marceau@localhost ~]$ openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout server.key -out server.crt
Common Name (eg, your name or your server's hostname) []:web.peche.linux
[...]
[marceau@localhost ~]$ sudo mv server.crt /etc/pki/tls/certs/
[marceau@localhost ~]$ sudo mv server.key /etc/pki/tls/private/
```
Aller ! Modifie la conf' pour prendre en compte les certificats ssl ... tu le savais ça aussi ?!
```
[marceau@localhost ~]$ sudo cat /etc/nginx/conf.d/nginx2.conf
server {
    # On indique le nom que client va saisir pour accéder au service
    # Pas d'erreur ici, c'est bien le nom de web, et pas de proxy qu'on veut ici !
    server_name web.peche.linux;


    # Port d'écoute de NGINX
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    ssl_certificate     /etc/pki/tls/certs/server.crt;
    ssl_certificate_key   /etc/pki/tls/private/server.key;

    location / {
        # On définit des headers HTTP pour que le proxying se passe bien
        proxy_set_header  Host $host;
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-Proto https;
        proxy_set_header  X-Forwarded-Host $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;

        # On définit la cible du proxying
        proxy_pass http://10.25.1.11:8096;
    }

    # Deux sections location recommandés par la doc NextCloud
    location /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }

    location /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }
}
server {
    listen 80 default_server;
    server_name _;
    return 301 https://$host$request_uri;
}
```
Modifie le fichier host c'est horrible là ...
````
*À faire sur le pc*
[...]
10.25.1.12    emby.peche.linux
````
Tu te débrouilles bien ... 

Redémarre NGINX pour la suite !

````
[marceau@localhost ~]$ sudo systemctl restart nginx
````
T'as presque finis ... MAIS CROIS PAS QUE T'ES FORT HEIN !

On va faire un mini peu de sécu ... tiens toi prêt !

Fais en sorte qu'il soit joignable uniquement par ````proxy.peche.linux````.
```
[marceau@localhost ~]$ cat /etc/sysctl.conf | tail -n 1
net.ipv4.icmp_echo_ignore_all = 1
[marceau@localhost ~]$ sudo sysctl -p
net.ipv4.icmp_echo_ignore_all = 1
```
Bon t'as fais du bon boulot... 

C'est bien mais t'as pas finis à temps malheureusement ... 

Tu n'as réussis à faire que les 2 premières parties 😥

C'est dommage mais peut-être une prochaine fois !

![Triste](./pics/unnamed.jpg)