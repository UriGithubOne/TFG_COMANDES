# Comandes utilitzades

Aquest document recull les ordres que s'han confirmat com a executades correctament. Les ordres
de configuració poden variar segons la instal·lació i l'usuari del sistema.

## 1. Preparació de Debian

En aquesta fase actualitzem els paquets del sistema i instal·lem eines per administrar, monitorar i
protegir el servidor, així com les utilitats necessàries per configurar zRAM.

### Comandes utilitzades

```bash
apt update && apt upgrade -y
apt install -y curl wget git htop iotop net-tools ufw sudo zram-tools
```

### Explicació detallada

#### Actualitzar Debian

- `apt`: gestor de paquets de Debian, utilitzat per obtenir informació sobre els paquets i instal·lar-los o actualitzar-los.
- `apt update`: actualitza l'índex local de paquets disponibles als repositoris configurats. Aquesta ordre, per si sola, no actualitza els programes instal·lats.
- `&&`: executa la segona ordre només si la primera acaba correctament.
- `apt upgrade`: instal·la les versions noves disponibles dels paquets ja instal·lats.
- `-y`: respon afirmativament a les preguntes de confirmació d'APT.

Per tant, la primera línia actualitza primer l'índex de paquets i, si això funciona, actualitza els paquets instal·lats.

#### Instal·lar eines d'administració

- `apt install`: instal·la els paquets que s'indiquen a continuació.
- `-y`: accepta automàticament les confirmacions durant la instal·lació.
- `curl`: transfereix dades des d'adreces URL; pot servir, per exemple, per consultar o descarregar recursos d'Internet.
- `wget`: descarrega fitxers des d'adreces URL.
- `git`: permet gestionar repositoris i l'historial de canvis dels fitxers.
- `htop`: mostra els processos i l'ús de CPU i memòria en una interfície interactiva.
- `iotop`: mostra l'activitat d'entrada i sortida dels discos, incloent-hi l'activitat associada als processos.
- `net-tools`: proporciona eines tradicionals de xarxa, com ara `ifconfig` i `netstat`.
- `ufw`: facilita la configuració de regles del tallafoc del sistema.
- `sudo`: permet que els usuaris autoritzats executin ordres amb privilegis d'administració.
- `zram-tools`: proporciona eines per configurar zRAM, que utilitza memòria comprimida com a espai d'intercanvi (swap).

Les ordres que necessiten privilegis d'administració s'han d'executar com a `root` o, si l'usuari té els permisos necessaris, amb `sudo` al davant.

## 2. Configuració de zRAM

En aquesta fase configurem zRAM perquè utilitzi memòria RAM comprimida com a espai d'intercanvi (swap). Després reiniciem el servei perquè carregui la configuració.

### Comandes utilitzades

```bash
nano /etc/default/zramswap
systemctl restart zramswap
```

Al fitxer `/etc/default/zramswap` es van configurar aquests valors:

```ini
ALGO=zstd
PERCENT=50
SIZE=512
PRIORITY=100
```

### Explicació detallada

- `nano /etc/default/zramswap`: obre el fitxer de configuració de zram-tools amb l'editor de text Nano. S'hi defineixen els paràmetres que el servei farà servir.
- `ALGO=zstd`: selecciona `zstd` com a algorisme de compressió. Segons els comentaris del mateix fitxer, prioritza la compressió per sobre de la velocitat respecte de `lz4`.
- `PERCENT=50`: configura la mida de zRAM com un percentatge de la memòria disponible. En aquesta configuració, s'indica el 50%.
- `SIZE=512`: especifica una mida fixa de 512 MiB, però el comentari del fitxer indica que `PERCENT` té prioritat i sobreescriu `SIZE`. Per tant, mentre `PERCENT=50` estigui definit, `SIZE=512` no determina la mida utilitzada.
- `PRIORITY=100`: estableix la prioritat del dispositiu swap de zRAM. Una prioritat més alta fa que el sistema el prefereixi davant de dispositius swap amb prioritat inferior.
- `systemctl`: eina per consultar i controlar serveis gestionats per systemd.
- `restart`: atura i torna a iniciar el servei indicat.
- `zramswap`: nom del servei de zram-tools que es reinicia per carregar la configuració actualitzada.

## 3. Ajust del *swappiness* i de la memòria del kernel

En aquesta fase ajustem alguns paràmetres del nucli de Linux relacionats amb l'ús de l'espai
d'intercanvi, la memòria cau de fitxers i l'escriptura de dades pendents al disc. Desem aquests
valors en un fitxer de configuració de `sysctl` i els apliquem al sistema.

### Comandes utilitzades

Obrir el fitxer de configuració:

```bash
nano /etc/sysctl.d/99-tfg-optimization.conf
```

Valors afegits al fitxer:

```ini
# Reduir la tendència de fer swap a disc per prioritzar la RAM
vm.swappiness=10

# Millorar la gestió de la memòria cau de fitxers
vm.vfs_cache_pressure=50

# Ajustar l'escriptura de dades pendents al disc
vm.dirty_ratio=10
vm.dirty_background_ratio=5
```

Aplicar els paràmetres:

```bash
sysctl --system
```

### Explicació detallada

- `nano /etc/sysctl.d/99-tfg-optimization.conf`: obre o crea el fitxer on es desen els paràmetres del kernel que gestiona `sysctl`.
- `vm.swappiness=10`: estableix un valor baix de *swappiness*. Aquest paràmetre influeix en la tendència del kernel a utilitzar espai d'intercanvi; no desactiva el swap.
- `vm.vfs_cache_pressure=50`: ajusta la tendència del kernel a recuperar memòria utilitzada per memòria cau d'inodes i dentries. Un valor inferior al valor habitual de `100` redueix aquesta pressió de recuperació.
- `vm.dirty_ratio=10`: fixa en el 10% el llindar de memòria ocupada per dades modificades pendents d'escriure, abans que els processos que generen aquestes dades puguin haver d'escriure-les.
- `vm.dirty_background_ratio=5`: fixa en el 5% el llindar a partir del qual el kernel comença a escriure en segon pla les dades modificades pendents.
- `sysctl`: consulta o modifica paràmetres del kernel en execució.
- `--system`: carrega els fitxers de configuració de `sysctl` del sistema i aplica els valors que contenen, inclòs el fitxer creat en aquesta fase.

## 4. Configuració d'una IP estàtica a la xarxa local

En aquesta fase identifiquem la interfície de xarxa del servidor i configurem una adreça IP fixa
a la xarxa local. Això permet que el servidor conservi la mateixa adreça després de reiniciar-se.
Els valors concrets de la xarxa s'han substituït per marcadors per no publicar-ne els detalls.

### Comandes utilitzades

Consultar les interfícies i adreces de xarxa:

```bash
ip a
```

Editar la configuració de xarxa:

```bash
nano /etc/network/interfaces
```

Exemple de configuració amb marcadors que cal substituir pels valors corresponents abans d'utilitzar:

```ini
# La interfície de loopback
auto lo
iface lo inet loopback

# Interfície de xarxa principal
auto <INTERFICIE>
iface <INTERFICIE> inet static
    address <IP_SERVIDOR>
    netmask <MASCARA_XARXA>
    gateway <IP_ROUTER>
    dns-nameservers <DNS_1> <DNS_2>
```

Reiniciar el servei de xarxa per aplicar la configuració:

```bash
systemctl restart networking
```

### Explicació detallada

- `ip a`: mostra les interfícies de xarxa i les adreces IP que tenen assignades. La sortida permet identificar el nom de la interfície que cal configurar.
- `nano /etc/network/interfaces`: obre el fitxer de configuració de les interfícies gestionades per `ifupdown`.
- `auto <INTERFICIE>`: indica que la interfície s'ha d'activar automàticament durant l'arrencada.
- `iface <INTERFICIE> inet static`: declara una configuració IPv4 estàtica per a la interfície.
- `address <IP_SERVIDOR>`: defineix l'adreça IPv4 fixa del servidor.
- `netmask <MASCARA_XARXA>`: defineix la màscara de la subxarxa.
- `gateway <IP_ROUTER>`: defineix l'adreça del router que s'utilitza com a passarel·la per arribar a altres xarxes.
- `dns-nameservers <DNS_1> <DNS_2>`: especifica els servidors DNS que s'utilitzaran per resoldre noms de domini.
- `systemctl restart networking`: demana a `systemd` que reiniciï el servei de xarxa perquè torni a carregar la configuració. Reiniciar xarxa remotament pot interrompre la connexió SSH si els valors no són correctes.

## 5. Seguretat bàsica amb SSH i UFW

En aquesta fase configurem una política inicial per al tallafoc UFW i ajustem opcions del servidor
SSH per limitar l'accés directe de `root` i els intents d'autenticació. També cal disposar d'un
usuari administratiu no-root abans de desactivar l'accés SSH de `root`. En aquesta configuració
inicial només s'obre el port SSH; els ports dels serveis s'afegeixen més endavant quan cal accedir-hi.

### Comandes utilitzades

Establir les polítiques per defecte del tallafoc:

```bash
ufw default deny incoming
ufw default allow outgoing
```

Permetre SSH abans d'activar el tallafoc:

```bash
ufw allow 22/tcp comment 'Accés SSH'
ufw enable
```

Crear un usuari no-root i afegir-lo al grup `sudo`:

```bash
adduser <USUARI>
usermod -aG sudo <USUARI>
```

Editar la configuració del servidor SSH:

```bash
nano /etc/ssh/sshd_config
```

Directives configurades al fitxer:

```text
PermitRootLogin no
MaxAuthTries 3
```

Reiniciar el servei SSH per aplicar els canvis:

```bash
systemctl restart ssh
```

### Explicació detallada

- `ufw default deny incoming`: estableix com a política d'entrada per defecte rebutjar les connexions entrants que no estiguin permeses explícitament.
- `ufw default allow outgoing`: estableix com a política de sortida per defecte permetre les connexions iniciades des del servidor.
- `ufw allow 22/tcp comment 'Accés SSH'`: crea una regla per permetre connexions SSH entrants pel port TCP 22 i hi afegeix una descripció.
- `ufw enable`: activa el tallafoc amb les regles configurades. És important permetre abans el port SSH utilitzat per no perdre l'accés remot.
- `adduser <USUARI>`: crea un compte d'usuari interactiu. Cal substituir `<USUARI>` pel nom del compte al servidor.
- `usermod -aG sudo <USUARI>`: afegeix l'usuari existent al grup `sudo`. `-aG` afegeix el grup sense eliminar l'usuari dels altres grups.
- `nano /etc/ssh/sshd_config`: obre el fitxer de configuració del dimoni SSH, és a dir, del servidor SSH. Aquest fitxer és diferent de `/etc/ssh/ssh_config`, que configura el client SSH.
- `PermitRootLogin no`: desactiva l'inici de sessió SSH directe com a `root`.
- `MaxAuthTries 3`: limita a tres els intents d'autenticació per connexió SSH.
- `systemctl restart ssh`: reinicia el servei del servidor SSH perquè carregui la configuració. Si el fitxer conté errors, es pot interrompre l'accés remot; convé mantenir una sessió existent fins a verificar una nova connexió.

Aquestes ordres d'administració s'han d'executar com a `root` o amb `sudo`. Els valors i les regles del tallafoc s'han de revisar si el servidor utilitza un port SSH diferent del 22 o ofereix altres serveis de xarxa.

## 6. Verificació de la instal·lació

En aquesta fase fem comprovacions bàsiques de l'estat de la memòria, de l'espai dels sistemes de
fitxers i de les regles del tallafoc. La sortida descriu l'estat del servidor en el moment en què
s'executen les ordres.

### Comandes utilitzades

```bash
free -h
df -h
sudo ufw status
```

### Explicació detallada

- `free -h`: mostra la memòria física i l'espai d'intercanvi (swap) totals, utilitzats i disponibles. L'opció `-h` presenta les mides en unitats llegibles per a les persones. En el meu cas la RAM total és `3,7 GiB`, amb `1,4 GiB` en ús i `2,3 GiB` disponibles; la swap és `5,7 GiB` total i no n'hi ha cap ús en aquell moment. L'ús de RAM varia segons els serveis actius. La sortida mostra la swap disponible, però no identifica per si sola quin dispositiu o mecanisme de swap s'està utilitzant.
- `df -h`: mostra l'espai total, utilitzat i disponible dels sistemes de fitxers muntats. L'opció `-h` presenta les mides en unitats llegibles. En el meu cas, el sistema de fitxers arrel `/` apareix amb `435G` totals, `5,5G` utilitzats i `407G` disponibles. `df` no identifica per si sola si el dispositiu físic és un SSD.
- `sudo ufw status`: mostra si UFW està actiu i les regles configurades. Es va utilitzar `sudo` perquè la consulta es va fer des d'un usuari no-root.

## 7. Instal·lació i verificació de K3s

En aquesta fase instal·lem K3s amb els components integrats per defecte i configurem `kubectl`
perquè l'usuari no-root pugui administrar el clúster. La instal·lació per defecte de K3s inclou,
entre altres components, CoreDNS, Traefik, metrics-server i el provisionador d'emmagatzematge
local-path. Aquesta és una instal·lació d'un sol node; la comanda, per si sola, no constitueix una
validació completa de preparació per a producció.

### Comandes utilitzades

Instal·lar K3s:

```bash
curl -sfL https://get.k3s.io | sh -
```

Configurar `kubectl` per a l'usuari actual:

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
chmod 600 ~/.kube/config
sudo chmod 600 /etc/rancher/k3s/k3s.yaml
echo "export KUBECONFIG=~/.kube/config" >> ~/.bashrc
source ~/.bashrc
```

Comprovar els permisos dels dos fitxers:

```bash
sudo ls -la /etc/rancher/k3s/k3s.yaml
ls -la ~/.kube/config
```

Verificar l'estat del node, els pods del sistema i les mètriques:

```bash
kubectl get nodes -o wide
kubectl get pods -n kube-system
kubectl top nodes
```

### Explicació detallada

- `curl -sfL https://get.k3s.io | sh -`: descarrega l'script oficial d'instal·lació de K3s i el passa a la shell perquè s'executi. Les opcions `-s`, `-f` i `-L` fan que `curl` funcioni en mode silenciós, retorni error davant d'un error HTTP i segueixi redireccions. Com que executa codi descarregat, cal utilitzar només la font oficial i revisar les instruccions vigents abans d'instal·lar.
- `mkdir -p ~/.kube`: crea el directori `.kube` a la carpeta personal de l'usuari. `-p` evita errors si el directori ja existeix i crea els directoris intermedis necessaris.
- `sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config`: copia el kubeconfig generat per K3s a la ubicació estàndard que utilitza `kubectl`. El fitxer conté credencials d'administració del clúster i no s'ha de publicar ni afegir al repositori.
- `sudo chown $(id -u):$(id -g) ~/.kube/config`: canvia el propietari i el grup del fitxer perquè corresponguin a l'usuari actual. `$(id -u)` i `$(id -g)` substitueixen l'UID i el GID de l'usuari.
- `chmod 600 ~/.kube/config`: limita la lectura i l'escriptura del kubeconfig al propietari del fitxer.
- `sudo chmod 600 /etc/rancher/k3s/k3s.yaml`: restringeix també el kubeconfig original de K3s perquè només `root` el pugui llegir o modificar. Això és especialment important si durant la instal·lació s'havia configurat amb permisos més oberts, com ara `644`.
- `sudo ls -la /etc/rancher/k3s/k3s.yaml` i `ls -la ~/.kube/config`: mostren els permisos i la propietat dels fitxers per comprovar que el kubeconfig original és de `root` amb mode `600` i que la còpia personal pertany a l'usuari amb mode `600`.
- `echo "export KUBECONFIG=~/.kube/config" >> ~/.bashrc`: afegeix la variable `KUBECONFIG` al fitxer `.bashrc`, perquè les noves sessions Bash utilitzin aquest fitxer de configuració.
- `source ~/.bashrc`: carrega el fitxer `.bashrc` a la sessió actual sense haver d'obrir un terminal nou.
- `kubectl get nodes -o wide`: mostra els nodes del clúster i informació addicional. L'estat `Ready` indica que el node està preparat per rebre càrregues de treball.
- `kubectl get pods -n kube-system`: mostra els pods de l'espai de noms del sistema, inclosos els components de K3s. Cal revisar-ne la columna d'estat; la presència d'un pod a la llista no garanteix per si sola que estigui operatiu.
- `kubectl top nodes`: mostra l'ús actual de CPU i memòria dels nodes a partir de les mètriques publicades per metrics-server. És una comprovació que les mètriques estan disponibles; no verifica per si sola que una política HPA escali correctament una aplicació.

No publiquis mai cap dels dos fitxers kubeconfig ni el seu contingut: proporcionen credencials
d'accés administratiu al clúster. Si K3s està configurat per tornar a generar el fitxer original
amb mode `644`, comprova els permisos després de reiniciar el servei, ja que la configuració del
servei pot tornar a aplicar aquest mode.

## 8. Organització del clúster amb namespaces

En aquesta fase organitzem els recursos de Kubernetes en namespaces per separar-los lògicament
segons el tipus de servei. Els namespaces ajuden a ordenar i administrar els recursos, però no
imposen límits de CPU o memòria ni proporcionen aïllament de seguretat per si sols; per a això cal
configurar, per exemple, `ResourceQuota`, `LimitRange` i polítiques de xarxa.

### Comandes utilitzades

Crear namespaces per als serveis de xarxa, multimèdia, emmagatzematge i monitoratge:

```bash
kubectl create namespace network-services
kubectl create namespace media
kubectl create namespace storage
kubectl create namespace monitoring
```

Comprovar els namespaces:

```bash
kubectl get namespaces
```

### Explicació detallada

- `kubectl`: eina de línia d'ordres per comunicar-se amb el clúster de Kubernetes.
- `create namespace`: crea un namespace amb el nom indicat.
- `network-services`, `media`, `storage` i `monitoring`: noms dels namespaces utilitzats per agrupar lògicament els recursos segons el seu propòsit.
- `get namespaces`: mostra els namespaces del clúster perquè se'n pugui comprovar la creació.

## 9. Preparació de carpetes per a dades persistents

En aquesta fase creem directoris locals sota `/mnt/storage` per organitzar dades de configuració i
mitjans dels serveis. Les carpetes, per si soles, no connecten aquestes dades als pods: cal
referenciar-les correctament en volums i muntatges dels manifests de Kubernetes.

### Comandes utilitzades

Crear les carpetes:

```bash
sudo mkdir -p /mnt/storage/adguard/work
sudo mkdir -p /mnt/storage/adguard/conf
sudo mkdir -p /mnt/storage/jellyfin/config
sudo mkdir -p /mnt/storage/jellyfin/media
sudo mkdir -p /mnt/storage/ocis/data
```

Assignar la propietat a l'usuari actual i restringir l'accés:

```bash
sudo chown -R $USER:$USER /mnt/storage
sudo chmod -R 700 /mnt/storage
```

Comprovar l'estructura:

```bash
ls -la /mnt/storage/
```

### Explicació detallada

- `sudo`: executa l'ordre següent amb privilegis d'administració.
- `mkdir -p`: crea els directoris indicats i els seus pares si encara no existeixen; no falla si ja existeixen.
- `/mnt/storage/adguard/work` i `/mnt/storage/adguard/conf`: directoris destinats, respectivament, a dades de treball i configuració d'AdGuard Home.
- `/mnt/storage/jellyfin/config` i `/mnt/storage/jellyfin/media`: directoris destinats a la configuració de Jellyfin i als fitxers multimèdia.
- `/mnt/storage/ocis/data`: directori destinat a les dades d'oCIS.
- `chown -R $USER:$USER /mnt/storage`: canvia recursivament el propietari i el grup de `/mnt/storage` a l'usuari actual. `$USER` és una variable de l'entorn de la shell.
- `chmod -R 700 /mnt/storage`: aplica recursivament els permisos `700`: el propietari pot llegir, escriure i accedir als directoris; el grup i la resta d'usuaris no tenen permisos.
- `ls -la /mnt/storage/`: mostra els fitxers, directoris ocults, propietaris i permisos del directori.

Els permisos `700` protegeixen les dades d'altres usuaris locals, però aplicats recursivament
també marquen els fitxers com a executables, cosa que normalment no és necessària. Jellyfin
s'executa al manifest amb UID/GID `1001`, de manera que cal que els volums siguin accessibles per
a aquesta identitat. Comprovació realitzada a l'host: l'usuari té UID/GID `1001:1001`, que
coincideix amb l'usuari del
contenidor. `/mnt/storage`, `/mnt/storage/jellyfin`, `config` i `media` són propietat de
`1001:1001` i tenen mode `700`; `cache` és propietat de `1001:1001` i té mode `775`. La sortida
de `namei -l` també confirma que els directoris pares permeten a aquest UID travessar la ruta.
Per tant, els modes i propietaris dels directoris mostrats són compatibles amb l'accés de
Jellyfin: configuració i caché poden ser accessibles en escriptura, i els mitjans en lectura,
tal com correspon al muntatge de només lectura del manifest. No s'han canviat permisos arran
d'aquesta comprovació.

La comprovació des del pod s'ha executat i ha retornat UID/GID `1001:1001`, amb lectura i
escriptura a `/config` i `/cache`, i només lectura a `/media`. El resultat concorda amb els
permisos i el muntatge de només lectura del manifest. Aquesta prova verifica l'accés als
directoris muntats, però no els permisos de cada fitxer que hi pugui haver a dins.

La comanda utilitzada per comprovar-ho:

```bash
kubectl -n media exec deployment/jellyfin -- sh -c '
id
for p in /config /cache /media; do
  printf "%s: " "$p"
  test -r "$p" && printf "lectura " || printf "sense lectura "
  test -w "$p" && printf "escriptura\n" || printf "sense escriptura\n"
done
'
```

## 10. Diagnòstic inicial del port DNS per a AdGuard Home

Abans de desplegar AdGuard Home com a servei DNS, comprovem si algun procés de Debian ja escolta al
port 53. En aquesta etapa es va editar la configuració de `systemd-resolved`, però no es va confirmar
que es desactivés el servei ni que es reiniciés. No s'ha de desactivar el resolvedor sense preparar
abans una configuració DNS alternativa per al mateix servidor.

### Comandes utilitzades

Comprovar si hi ha un procés escoltant al port 53:

```bash
sudo netstat -tulpn | grep :53
```

Editar el fitxer de configuració:

```bash
nano /etc/systemd/resolved.conf
```

### Explicació detallada

- `sudo netstat -tulpn`: mostra sockets TCP i UDP en escolta, amb ports numèrics i informació del procés. `sudo` permet veure dades de processos que un usuari normal pot no tenir permís per consultar.
- `|`: envia la sortida de `netstat` com a entrada a l'ordre següent.
- `grep :53`: filtra les línies que contenen `:53`, per localitzar possibles sockets del port DNS.
- `nano /etc/systemd/resolved.conf`: obre la configuració de `systemd-resolved`. Editar el fitxer no aplica per si sol els canvis; cal validar la configuració i reiniciar el servei només després d'assegurar la resolució DNS del sistema.

## 11. Desplegament inicial d'AdGuard Home

En aquesta fase despleguem AdGuard Home al namespace `network-services`. El manifest desat al
repositori especifica l'estratègia `Recreate`, `hostPort` als ports 53 (DNS UDP/TCP) i 30080 (web),
límits i sol·licituds de recursos, i volums `hostPath` sota `/mnt/storage/adguard`. El port 53 es
va comprovar com a lliure abans del desplegament. El pod va arribar a l'estat `Running`; aquest
estat, per si sol, no confirma que totes les funcions de l'aplicació o la interfície web siguin
accessibles.

### Comandes utilitzades

Des de l'arrel del repositori, editar i aplicar el manifest:

```bash
nano manifests/adguard-deployment.yaml
kubectl apply -f manifests/adguard-deployment.yaml
```

Permetre el port TCP `3000` al tallafoc:

```bash
sudo ufw allow 3000/tcp comment 'AdGuard Home Web Initial Setup'
```

Iniciar el reenviament del port cap al Deployment:

```bash
kubectl port-forward -n network-services deployment/adguard-home --address 0.0.0.0 3000:3000
```

Mentre el reenviament continua executant-se, obrir la interfície web al navegador. L'assistent
va carregar i s'hi va completar la configuració inicial, inclosa la creació de l'usuari
administrador d'AdGuard Home:

```text
http://<IP_SERVIDOR>:3000
```

Comprovar l'estat del pod:

```bash
kubectl get pods -n network-services -o wide
```

Després de completar la configuració inicial, es va canviar manualment el port web d'AdGuard Home
a `30080`. El manifest del repositori reflecteix aquesta configuració: el contenidor escolta al
port `30080`, `hostPort` publica aquest port al node i el Service intern també hi apunta. Per
aplicar el manifest actualitzat des de l'arrel del repositori:

```bash
kubectl apply -f manifests/adguard-deployment.yaml
```

Després d'aplicar-lo, la interfície web s'obre al port `30080` del servidor:

```text
http://<IP_SERVIDOR>:30080
```

El port `30080` de `hostPort` queda exposat a les interfícies del node, subjecte al tallafoc.
Si UFW està actiu i bloqueja connexions entrants, cal permetre TCP `30080` per accedir-hi des
d'altres dispositius; aquesta regla no consta com a executada.

### Explicació detallada

- `nano manifests/adguard-deployment.yaml`: obre el manifest des de l'arrel del repositori amb l'editor Nano.
- `kubectl apply -f manifests/adguard-deployment.yaml`: crea o actualitza al clúster els recursos declarats al manifest.
- `sudo ufw allow 3000/tcp comment 'AdGuard Home Web Initial Setup'`: afegeix una regla d'entrada persistent a UFW per al port TCP `3000`. La regla observada permetia l'accés des de qualsevol origen; no és temporal automàticament i cal retirar-la manualment en acabar si ja no és necessària.
- `kubectl get pods -n network-services -o wide`: mostra els pods del namespace `network-services`; `-o wide` afegeix informació com ara el node i les adreces assignades.
- `kubectl port-forward`: crea el reenviament indicat entre un port del servidor i el port del pod. `-n network-services` selecciona el namespace, `deployment/adguard-home` selecciona el Deployment, `--address 0.0.0.0` fa que el procés escolti a totes les interfícies i `3000:3000` associa el port local al del pod. Atura el procés amb `Ctrl+C` en acabar.
- La URL inicial obre la interfície HTTP d'AdGuard al port `3000`; s'hi va completar l'assistent i es va crear l'usuari administrador. La IP s'ha substituït per un marcador i les credencials no s'han d'incloure al repositori.
- Després de la configuració inicial, el port web d'AdGuard es va canviar manualment a `30080`. El port intern configurat a l'aplicació ha de coincidir amb `containerPort` i amb `targetPort` del Service al manifest.
- `strategy: Recreate`: indica que, durant una actualització del Deployment, se suprimeix el pod existent abans de crear-ne un de nou. Pot evitar que dues instàncies intentin reservar simultàniament els mateixos `hostPort`, però durant el canvi hi ha una interrupció.
- `hostPort`: publica un port del pod directament al mateix port del node. Això requereix que el port corresponent no estigui ocupat al node i exposa el servei a les interfícies de xarxa del node subjectes al tallafoc.
- `hostPath`: munta dins del pod un directori del sistema de fitxers del node. El directori ha d'existir i tenir permisos compatibles amb l'usuari amb què s'executa el contenidor.
- `resources.requests` i `resources.limits`: indiquen, respectivament, els recursos que Kubernetes reserva per al contenidor i els límits màxims que pot utilitzar.

Per reduir l'exposició de l'assistent durant la configuració inicial, una alternativa recomanada
(no consta com a executada) és fer que el reenviament escolti només a `127.0.0.1` i accedir-hi
amb un túnel SSH:

```bash
kubectl port-forward -n network-services deployment/adguard-home --address 127.0.0.1 3000:3000
```

Des del client que inicia la connexió SSH, substitueix els marcadors i crea el túnel:

```bash
ssh -L 3000:127.0.0.1:3000 <USUARI>@<SERVIDOR>
```

Obre `http://127.0.0.1:3000` al navegador del client. En acabar, atura el reenviament de
Kubernetes amb `Ctrl+C`; comprova les regles numerades amb `sudo ufw status numbered` i elimina la
regla d'accés ampli si ja no es necessita. Si només hi ha una regla per al port TCP `3000`, es pot
eliminar amb:

```bash
sudo ufw delete allow 3000/tcp
```

Si no es pot utilitzar un túnel SSH, restringeix la regla a la subxarxa local en lloc de permetre
el port des de qualsevol origen.

## 12. Instal·lació i inici de K9s

Després de posar en marxa K3s, es va instal·lar K9s, una interfície interactiva de terminal per
consultar i administrar recursos de Kubernetes. Es va descarregar el paquet `.deb` de la versió
`v0.51.0`, se'n va verificar el SHA-256 i després es va instal·lar.

### Comandes utilitzades

Anar al directori temporal:

```bash
cd /tmp
```

Descarregar el paquet de K9s:

```bash
curl -fLO https://github.com/derailed/k9s/releases/download/v0.51.0/k9s_linux_amd64.deb
```

Verificar la suma SHA-256 del fitxer descarregat:

```bash
echo '56b539a509eb2d6357cf4f575ed38c089f0e4880c95f79a70196b54f14954908  k9s_linux_amd64.deb' | sha256sum -c -
```

Instal·lar el paquet local i iniciar K9s:

```bash
sudo apt install ./k9s_linux_amd64.deb
k9s
```

### Explicació detallada

- `cd /tmp`: canvia el directori actual al directori temporal del sistema, on es desa el paquet descarregat.
- `curl -fLO <URL>`: descarrega el fitxer indicat. `-f` retorna un error si el servidor respon amb un error HTTP, `-L` segueix redireccions i `-O` desa el fitxer amb el nom que té a l'URL.
- `sha256sum -c -`: comprova que el fitxer coincideix amb la suma SHA-256 proporcionada a l'entrada estàndard. La suma s'ha de contrastar amb una font oficial o de confiança.
- `sudo apt install ./k9s_linux_amd64.deb`: instal·la el paquet `.deb` local i resol les dependències disponibles als repositoris configurats.
- `k9s`: inicia la interfície interactiva. K9s utilitza la configuració de Kubernetes disponible per a l'usuari, habitualment `~/.kube/config`; el kubeconfig no s'ha de compartir ni publicar.

## 13. Desplegament de Jellyfin

En aquesta fase despleguem Jellyfin al namespace `media` i li proporcionem directoris persistents
per a la configuració, la caché i els fitxers multimèdia. El manifest complet és a
[`manifests/jellyfin-deployment.yaml`](./manifests/jellyfin-deployment.yaml). El Deployment va
quedar operatiu i s'hi va accedir pel port NodePort `30096`.

### Comandes utilitzades

Crear els directoris de configuració i caché:

```bash
mkdir -p /mnt/storage/jellyfin/config /mnt/storage/jellyfin/cache
```

Crear la carpeta de pel·lícules:

```bash
mkdir -p /mnt/storage/jellyfin/media/Peliculas
```

Des de l'arrel del repositori, aplicar el manifest:

```bash
kubectl apply -f manifests/jellyfin-deployment.yaml
```

Esperar que el Deployment acabi el desplegament i comprovar els pods:

```bash
kubectl -n media rollout status deployment/jellyfin --timeout=180s
kubectl -n media get pods -o wide
```

Adreça d'accés al navegador (s'omet la IP concreta del servidor):

```text
http://<IP_SERVIDOR>:30096
```

### Explicació detallada

- `mkdir -p`: crea les carpetes indicades i els directoris pares necessaris; no falla si ja existeixen.
- `/mnt/storage/jellyfin/config`: directori de l'host muntat al contenidor com `/config` per conservar la configuració de Jellyfin.
- `/mnt/storage/jellyfin/cache`: directori de l'host muntat com `/cache` per a dades temporals.
- `/mnt/storage/jellyfin/media/Peliculas`: carpeta de l'host per organitzar les pel·lícules. El directori multimèdia s'exposa al contenidor com `/media` en mode només lectura.
- `kubectl apply -f manifests/jellyfin-deployment.yaml`: crea o actualitza els recursos descrits al manifest, incloent-hi el namespace, el Deployment i el Service.
- `kubectl -n media rollout status deployment/jellyfin --timeout=180s`: espera fins a 180 segons que el desplegament del Deployment finalitzi i informa del resultat.
- `kubectl -n media get pods -o wide`: mostra els pods del namespace `media` i informació addicional sobre els nodes on s'executen.
- `http://<IP_SERVIDOR>:30096`: obre la interfície web de Jellyfin. El Service del manifest és de tipus `NodePort`: rep les peticions al port `30096` del node i les envia al port de servei `8096`, que apunta al port HTTP `8096` del contenidor.

El contenidor s'executa com a usuari no-root amb UID/GID `1001`, i el manifest desactiva
l'escalada de privilegis i elimina les capabilities addicionals. Cal que els directoris de
configuració i caché siguin accessibles per aquest UID/GID; el muntatge de mitjans és de només
lectura dins del contenidor. Es va pujar un fitxer multimèdia al servidor i Jellyfin el va
detectar a la biblioteca.

El manifest utilitza `jellyfin/jellyfin:latest` amb `imagePullPolicy: Always`; per tant, una
reaplicació pot descarregar una imatge diferent i els resultats no són reproduïbles només amb
aquest YAML. Per registrar la imatge exacta observada en un pod existent, es pot executar:

```bash
kubectl -n media get pods -l app=jellyfin -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{range .status.containerStatuses[*]}{.image}{"\t"}{.imageID}{"\n"}{end}{end}'
```

Anota el digest `imageID` i, abans de repetir proves o desplegaments, fixa una versió explícita o
el digest de la imatge al manifest. El tag/digest utilitzat durant la primera prova no consta en
l'historial disponible.