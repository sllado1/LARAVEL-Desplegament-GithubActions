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

