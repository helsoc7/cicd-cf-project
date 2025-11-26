# Cloudformation und CICD-Automatisierung für AWS-Deployment
## Anleitung - Frontend
### Projekt anlegen
Lege dir ein neues Projekt an mit den Ordnern
- infra: Hier kommt das Cloudformation-Template rein
- public: Hier kommt das Frontend, also die index.html rein
- .github/workflows: Hier kommt später unsere Datei rein mit der Pipeline für Github Actions
### IAM User
Als nächstes legen wir uns in der AWS Konsole einen IAM User für die Pipeline an, damit die Pipeline auch im AWS Konto Sachen machen kann, wie z.B. einen Bucket erstellen etc.
**Achtung**: Der bekommt für Testzwecke erstmal volle Admin-Berechtigungen, im Produktiv-Betrieb später braucht er restriktivere Berechtigungen
#### Keys vom IAM User
- Navigiere auf den erstellten IAM User
- Geh auf den Tab Sicherheitsanmeldeinformationen
- Dann auf Zugriffsschlüssel erstellen
- Dann auf Sonstiges
- Kopiere dir den Zugriffsschlüssel und den geheimen Zugriffsschlüssel raus (Achtung, der ist nur einmal sichtbar)
- Diese Daten müssen nun als Github Secret in ein Repo
##### Github Repo anlegen
- Leg dir ein neues Repo an
- Push den bisherigen Stand
- Lege dir Secrets an unter Settings > Secrets and Variables > Actions > New Repository Secret
- Pack da bitte AWS_ACCESS_KEY_ID und AWS_SECRET_ACCESS_KEY rein
### CloudFormation
1. Wir wollen erst einmal das CloudFormation Template anlegen
2. Den Stack wollen wir einmalig einmal deployen. CloudFront dauert tatsächlich auch ein wenig länger..
```bash
aws cloudformation deploy \
  --template-file infra/template.yaml \
  --stack-name frontend-stack \
  --capabilities CAPABILITY_NAMED_IAM
```
3. Wenn das erstellt worden ist, wollen wir mit der Pipeline weiter machen. Stand jetzt haben wir ein Bucket (+ stat. Website Hosting) erstellt mit vorgeschalteten CloudFront auf AWS. Jetzt benötigen wir natürlich noch die Dateien in dem Bucket, die wir über eine CICD-Pipeline ständig uploaden bzw. updaten wollen.
### CICD-Pipeline
1. Erstell eine Datei in .github/workflows 
2. Danach committen wir den Stand und schauen mal ob die Pipeline korrekt startet
3. Scheint zu laufen
### Kontrolle über AWS Management Konsole
1. Navigiere in den CloudFront-Service und geh auf den gerade erstellten
2. Kopiere die Distribution domain name und gib ihn in einem neuen Browser-Fenster ein
3. Du solltest nun dein Frontend sehen... 

## Anleitung - Backend
Für das Backend wollen wir eine EC2-Instanz per CloudFormation Stack erstellen und ihr einen selbst erstellten SSH-Key zuordnen, damit wir uns aus der Pipeline mit der Instanz verbinden können und die aktuelle docker-compose-Datei hochladen können. Ziel der Geschichte ist es, dass Backend und eine Datenbank mit Volume in einem Docker Compose Konstrukt auf der EC2-Instanz laufen sollen.
**Achtung** Wir kopieren uns noch eben den Code des Backends rüber inkl. Dockerfile. Das ist ein einfaches Flask-Backend
### SSH-Key lokal erstellen
1. Erzeuge dir über das Terminal einen SSH-Key
```bash
ssh-keygen -t rsa -b 4096 -C "backend-deploy"
```
Beim Dateinamen antworte bitte mit dem kompletten Pfad, häng aber hinten ein id_rsa_backend an
2. Navigier in das .ssh Verzeichnis auf deinem User-Verzeichnis rein
3. Da sehen wir haben wir einen öffentlichen und einen privaten Key vom erzeugten SSH-Schlüssel. 
Der Private Key kommt als Github Secret ins Repo
```bash
cat ~/.ssh/id_rsa_backend
```
**Achtung** Der öffentliche Schlüssel kommt nachher auf die EC2-Instanz per CloudFormation Stack
### Cloudformation Template fürs Backend
1. Lege ein template fürs Backend-Deployment an
2. Danach deploye das mit dem folgenden Befehl
```bash
aws cloudformation deploy \
  --template-file infra/backend.yaml \
  --stack-name backend-stack \
  --parameter-overrides SshPublicKey="$(cat ~/.ssh/id_rsa_backend.pub)" \
  --capabilities CAPABILITY_NAMED_IAM
```
3. Wir müssen noch die Public Ip der Instanz als Github Secret anlegen. Außerdem noch den EC2-User...
- EC2_USER = ec2-user
- EC2_HOST = bekommen wir raus mit `aws cloudformation describe-stacks --stack-name backend-stack` z.B. 18.192.119.206
### Docker Compose File anlegen
1. Lege ein docker-compose.yml im Repo an
2. Committe den aktuellen Stand
### CICD-Pipeline fürs Backend
1. Lege eine neue Workflow-Datei an in .github/workflows
2. In der Pipeline soll immer das aktuelle Backend und die docker compose Datei auf die EC2-Instanz gepushed werden.
3. Die Pipeline braucht dazu dann natürlich den privaten SSH Key, den EC2-User und den Hostnamen (IP-Adresse) fürs Kopieren per SCP
4. Testen wir das mal aus... läuft

## Verbindung Frontend - Backend
Damit wir das testen können müssen wir im Frontend die richtige API_URL fetchen. Hol dir also wieder die öffentliche IP der EC2-Instanz und füg die ein. z.B. http://3.122.247.91:3000/users"