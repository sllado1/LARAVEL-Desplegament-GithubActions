# Índex

- [Introducció] (#introduccio-a-github-actions)


# Introducció a GitHub Actions

Actualment, tenim moltes opcions per automatitzar el desplegament d’una aplicació. En aquest document, explicaré com fer un desplegament bàsic utilitzant **GitHub Actions**.

## Per què hem d’automatitzar el desplegament?

Desplegar una aplicació és una tasca especialment crítica, ja que si es fa manualment sempre hi ha el perill d’introduir errors durant el procés. Aquest tipus d’errors poden afectar el rendiment de l’aplicació, la seva disponibilitat, o fins i tot provocar caigudes inesperades del sistema.

Amb **GitHub Actions**, podem automatitzar aquest procés de manera eficient i segura. Això ens permet evitar errors humans i tenir més confiança en el procés de desplegament. 

### Què és GitHub Actions?

GitHub Actions és una eina integrada dins de GitHub que permet definir **Workflows**. Aquests workflows s'inicien automàticament quan es produeixen determinats esdeveniments, com ara un **push** a una branca o un **pull-request**. Un cop es dispara l'esdeveniment, s'executa una sèrie de tasques definides en contenidors de **Docker** o en màquines virtuals proporcionades per GitHub.

### Avantatges de l’automatització

Les tasques que definim en un workflow són **idempotents**, és a dir, que per la mateixa entrada generen sempre la mateixa sortida. Això vol dir que podem definir el procés de desplegament de manera que s’executarà amb resultats previsibles i sense sorpreses.

**Alguns avantatges d’automatitzar el desplegament amb GitHub Actions**:
- **Seguretat**: Reducció d’errors humans.
- **Previsibilitat**: Resultats consistents en cada desplegament.
- **Eficiència**: Desplegaments més ràpids i fàcils de gestionar.
- **Escalabilitat**: Permet desplegar en diferents entorns (producció, staging, etc.).

Amb aquest enfocament, podem estar segurs que el nostre procés de desplegament serà repetible, fiable i sempre executat de la mateixa manera.

# Desplegament d'una aplicació Laravel amb GitHub Actions
Per definir un worflow que s'iniciï cada vegada que es produeixi un esdeveniment, s'ha de crear un fitxer yaml a la carpeta ``` .github/workflows ``` del repositori.

![Imatge path workflow](/img/img-path-workflow.png)

Defineix el nom del workflow, en aquest cas es diu Laravel.
 ```yaml
 on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:
```
Aquesta secció defineix quan s'executarà el workflow:

- *push*: S'executa quan es fa un push a la branca main.
- *pull_request*: S'executa quan es crea un pull request cap a la branca main.
- *workflow_dispatch*: Permet executar el workflow manualment des de la interfície de GitHub.

```yaml  
jobs:
  build:
    runs-on: ubuntu-latest
```
Especifica que el treball (*build*) es realitzarà en un runner de Ubuntu amb la imatge més recent disponible. *(imatge Docker)*

## Passos del workflow

A continuació es defineixen els passos que s'executaran dins d'aquest treball:



```yaml
    - uses: shivammathur/setup-php@v2
      with:
        php-version: '8.4'
        extensions: curl, mbstring, pdo_mysql, tokenizer, xml, zip, zlib, ctype, openssl, fileinfo, bcmath
```
Instal·la PHP 8.4 i les extensions necessàries per a l’aplicació Laravel, com curl, mbstring, pdo_mysql, etc... a la màquina virtual Ubuntu (Màquina runs-on).

```yaml 
    - uses: actions/checkout@v4
```
Fa un git clone del repositori de la branca que ha desencadenat l'acció a la màquina virtual Ubuntu (Màquina runs-on).

```yaml
    - name: Install Dependencies
      run: composer install -q --no-ansi --no-interaction --no-scripts --no-progress --prefer-dist
```
Instal·la les dependències de PHP definides en el fitxer composer.json del projecte Laravel utilitzant Composer

```yaml
    - name: Node JS 
      uses: actions/setup-node@v4
      with:
        node-version: 20
```

Instal·la Node.js 20 per gestionar les dependències de JavaScript del projecte (com Vue.js).

```yaml
    - name: Instal·la les dependències de Node JS
      run: npm install
```

Instal·la les dependències de Node.js definides al fitxer ```package.json```.
```yaml
    - name: Compila vue.js
      run: npm run build
```

Compila el codi de Vue.js per generar els fitxers de producció, normalment dins d’una carpeta de public.
```yaml
    - name: Fent neteja
      run: rm -rf node_modules
```
Elimina la carpeta node_modules per evitar problemes de dependències i eliminar espai.

```yaml
    - name: Comprimir el projecte  (incloent els ocults)
      run: tar -czf app.tgz $(ls -A)
```
Comprimeix tot el projecte en un fitxer .tgz per facilitar el seu enviament al servidor de producció. Inclou també fitxers ocults, per això es fa ```$(ls -A) ```.

```yaml
    - name: ssh scp ssh pipelines
      uses: cross-the-world/ssh-scp-ssh-pipelines@latest
      env:
        WELCOME: "ssh scp ssh pipelines"
        LASTSSH: "Doing something after copying"
      with:
        host: ${{ secrets.SSH_HOST }}
        user: ${{ secrets.SSH_USER }}
        pass: ${{ secrets.SSH_PASS }}
        port: ${{ secrets.SSH_PORT }}
        connect_timeout: 10s
        first_ssh: | 
          ls -la  
          echo $WELCOME
        scp: |
          './app.tgz' => ${{ secrets.DOCUMENT_ROOT }}
        last_ssh: |
          echo $LASTSSH
          cd ${{ secrets.DOCUMENT_ROOT }}
          tar -xzf app.tgz
          pwd
          ls -lisa
          echo "Fem un alias amb la versió de php 8.4"
          echo "Copiar el fitxer env si no el tenim"
          $(which php84) -r "file_exists('.env') || copy('.env.example', '.env');"
          echo "Canvia el fitxer env"
          sed -i "s/DB_CONNECTION=mysql/DB_CONNECTION=${{ secrets.DB_CONNECTION }}/g" .env
          $(which php84) artisan config:clear
          $(which php84) artisan key:generate
          echo "Executar migracions, cache i altres operacions de Laravel"
          $(which php84) artisan migrate --force         
```

Aquest pas utilitza l'acció ```ssh-scp-ssh-pipelines``` per connectar-se al servidor de producció via SSH i SCP per transferir el fitxer comprimit del projecte (app.tgz). Un cop transferit, es descomprimeix al servidor i es configuren les variables d'entorn del projecte Laravel, incloent la configuració de la base de dades. També s'executen diverses ordres de Laravel per migrar la base de dades i neteja la cau i fer els enllaços simbòlics de les imatges.

### Consideracions

- Tot el que es posi dins de l'etiqueta ```last_ssh```es farà al servidor.

- Per executar php s'utilitza la comanda ```$(which php84)```perquè quan s'executa php des d'un script utilitza una versió antiga, aquesta comanda busca la ruta absoluta del php 8.4 i l'executa, evitant errors de versions. Per conèixer les versions de php que hi ha instal·lades al servidor ```ls -l /usr/local/bin/php* ``` 

![Mostra versions php instal·lades](/img/php-versions.png)

- ```sed ``` és una eina molt potent a Linux/Unix que permet modificar text en línia sense necessitat d'editar manualment un fitxer. Es pot utilitzar per buscar, substituir, inserir, eliminar i manipular text d’una manera ràpida i eficient. 
```sed 's/text_original/text_nou/' fitxer ```
Exemple: Canviar "hola" per "bon dia" en un fitxer text.txt
```sed 's/hola/bon dia/' text.txt ```
:warning: Això només canvia la primera coincidència.



# Secrets de GitHub: Què són i com s'utilitzen?
Els Secrets de GitHub són una manera segura d'emmagatzemar informació sensible dins d'un repositori o una organització de GitHub. Aquests secrets es poden utilitzar dins dels GitHub Actions sense exposar-los en el codi font.

## Per què utilitzar secrets?
Quan creem workflows per automatitzar processos (com desplegaments o integracions), sovint necessitem utilitzar credencials sensibles com:

- Claus d’API
- Contrasenyes
- Tokens d’accés
- Credencials SSH
- Paràmetres de configuració de bases de dades

GitHub ofereix secrets per protegir aquesta informació i evitar que sigui exposada públicament al codi del repositori, però a la vegada es pugui utilitzar aquesta informació sensible a l'hora de fer un desplegament.

### Secrets a nivell de repositori
Es defineixen per a un únic repositori. Només són accessibles dins dels workflows d'aquest repositori.

Exemple de creació:

1- Aneu al repositori a GitHub.
2- Aneu a Settings > Secrets and variables > Actions.
3- Feu clic a New repository secret.
4- Poseu-li un nom (exemple: ```SSH_HOST ```).
5- Introduïu el valor del secret i deseu-lo.

## Com utilitzar els secrets a GitHub Actions?
Dins d'un workflow de GitHub Actions, podem accedir als secrets de la següent manera:

```yaml 
name: Deploy Laravel

on:
  push:
    branches: [ "main" ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Connectar-se via SSH i executar ordres
        uses: cross-the-world/ssh-scp-ssh-pipelines@latest
        with:
          host: ${{ secrets.SSH_HOST }}
          user: ${{ secrets.SSH_USER }}
          pass: ${{ secrets.SSH_PASS }}
          port: ${{ secrets.SSH_PORT }}
          first_ssh: |
            echo "Ens hem connectat correctament al servidor!"

```
- ```$\{{ secrets.NOM_DEL_SECRET }}```: Permet accedir als secrets dins del workflow.
- Els secrets no es mostren mai en els logs, ni tan sols si hi ha un error.
  
 # Exemple complet

 ```yaml
 name: Laravel

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    # Fa un git clone de la branca que ha generat l'acció a la màquina ubuntu
    - uses: actions/checkout@v4
    # Instal·la el php
    - uses: shivammathur/setup-php@v2
      with:
        php-version: '8.4'
        extensions: curl, mbstring, pdo_mysql, tokenizer, xml, zip, zlib, ctype, openssl, fileinfo, bcmath
    # Fa un git clone de la branca que ha generat l'acció a la màquina ubuntu
    - uses: actions/checkout@v4
    # Instal·la les dependències de php
    - name: Install Dependencies
      run: composer install -q --no-ansi --no-interaction --no-scripts --no-progress --prefer-dist
    # Instal·la el Node JS
    - name: Node JS 
      uses: actions/setup-node@v4
      with:
        node-version: 20
    # Instal·la les dependències Node.js
    - name: Instal·la les dependències de Node JS
      run: npm install
    # Compila el Vue
    - name: Compila vue.js
      run: npm run build
    - name: Fent neteja
      run: rm -rf node_modules
 #   - name: Esborrar la carpeta storage
 #     run: rm -rf storage
    - name: Comprimir el projecte  (incloent els ocults)
      run: tar -czf app.tgz $(ls -A)
    - name: ssh scp ssh pipelines
      uses: cross-the-world/ssh-scp-ssh-pipelines@latest
      env:
        WELCOME: "ssh scp ssh pipelines"
        LASTSSH: "Doing something after copying"
      with:
        host: ${{ secrets.SSH_HOST }}
        user: ${{ secrets.SSH_USER }}
        pass: ${{ secrets.SSH_PASS }}
        port: ${{ secrets.SSH_PORT }}
        connect_timeout: 10s
        first_ssh: | 
          ls -la  
          echo $WELCOME
        scp: |
          './app.tgz' => ${{ secrets.DOCUMENT_ROOT }}
        last_ssh: |
          echo $LASTSSH
          cd ${{ secrets.DOCUMENT_ROOT }}
          tar -xzf app.tgz
          pwd
          ls -lisa
          echo "Copiar el fitxer env si no el tenim"
          $(which php84) -r "file_exists('.env') || copy('.env.example', '.env');"
          echo "Canvia el fitxer env"
          sed -i "s/DB_CONNECTION=mysql/DB_CONNECTION=${{ secrets.DB_CONNECTION }}/g" .env
          $(which php84) artisan config:clear
 ``` 