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
