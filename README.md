# Cloudformation und CICD-Automatisierung für AWS-Deployment
## Anleitung
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
