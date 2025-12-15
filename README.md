# Mock data-tooling Minio

> Ce repo est fourni à titre d'exemple afin de simuler un service S3. Ce repo n'est pas une offre de service CPiN, il permet de gagner du temps pour la mise en place d'un service S3 mais est à adapter par les équipes de dev aux besoins projet.

Minio servant de mock S3 pour le projet mock data-tooling

Afin d'avoir un certificat valide, les ingress doivent être un sous le domaine *.apps.app1.numerique-interieur.com

## Installation

Ce chart Helm peut être déployé sans changement via la console DSO. Mais il convient de modifier les values Helm depuis Argocd (`détails`, `parameters` puis `Edit` pour surcharger les values Helm), notamment pour éviter des conflits d'urls avec d'autres projets utilisant ce chart helm d'exemple.

### URL

Les urls pour accéder au minio sont données à titre d'exemple.

```yaml
minio:
  ingress:
    enabled: true
    hostname: minio-mock-data-tooling.apps.app1.numerique-interieur.com
  apiIngress:
    enabled: true
    hostname: api-minio-mock-data-tooling.apps.app1.numerique-interieur.com
```

L'URL pour l'ingress permet d'accéder à la web ui de minio, tandis que celle sous apiIngress est utile pour la connexion S3 (voir après pour créer une paire AK/SK).

### Secrets

Si vous souhaitez modifier les mots de passe par défaut, il faut créer un secret nommé secrets.yml.dec avec le contenu suivant (changer le mot de passe avant):

```yaml
apiVersion: isindir.github.com/v1alpha3
kind: SopsSecret
metadata:
  name: secret-mock-data-tooling-minio
spec:
  secretTemplates:
    - name: minio-root
      stringData:
        root-user: admin
        root-password: MonSuperMotDePasse1234!
```

Voir fichier **conf/secrets/ovh/dev/secrets.yml.dec.exemple**

### Buckets

Par défaut, ce chart helm créer 3 buckets:

- app
- logs
- database

Une persistence de 30Go est aussi paramétrée par défaut.

## Post-config

### Créer un compte S3

Pour créer un compte S3 et sa paire de clés AK/SK, il faut se connecter à l'interface web de minio avec l'utilisateur admin (celui désigné dans le secret minio-root).

![connexion](docs/img/connexion.png)

Après la connexion, la fênetre principale de minio s'affiche

![fenetreprincipale](docs/img/fenetreprincipale.png)

Dans le menu de gauche, dérouler le menu **Identity** et cliquer sur **Users**.

Dans la nouvelle fenêtre, cliquer sur le bouton **Create User +**
![menuusers](docs/img/menuusers.png)

La page de création d'un utilisateur s'affiche. Cet utilisateur est un utilisateur au sens minio qui pourra se connecter à l'interface web de minio, ce n'est pas un compte S3 (mais le compte S3 sera lié à cet utilisateur et à ses droits).

Cocher la policy **readwrite** pour autoriser l'utilisateur à lire et écrire dans les différents buckets.

Cliquer sur le bouton **Save** pour créer l'utilisateur
![createuser](docs/img/createuser.png)

La page des utilisateurs s'affichent avec le nouvel utilisateur (ici dev), cliquer sur son nom pour pouvoir créer un compte S3 lié à cet utilisateur
![useraftercreate](docs/img/useraftercreate.png)

Sur la nouvelle page, un menu s'affiche avec les choix Groups, Service Accounts et Policies.

Choisir **Service Accounts**, puis cliquer sur le bouton **Create Access Key +**
![userserviceaccount](docs/img/userserviceaccount.png)

Une fenêtre s'affiche avec une Access Key et une Secret Key déjà pré-remplies. Si cela convient, cliquer sur le bouton **Create**
![createserviceaccount](docs/img/createserviceaccount.png)

Une popup s'affiche pour être sûr d'avoir bien copier les informations concernant le compte S3, cliquer sur la croix en haut à droite pour fermer la popup
![aksk](docs/img/aksk.png)

**Cette paire de clés AK/SK sera à transmettre aux développeurs, ainsi que l'url de l'api**
