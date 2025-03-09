# Exercici: Enviar un fitxer al servidor amb GitHub Actions
## Objectiu
- Crear un repositori a GitHub.
- Configurar un GitHub Actions per enviar un fitxer al servidor via SCP.
- Aprendre a utilitzar secrets de GitHub per emmagatzemar credencials segures.
  
## Passos a seguir
- Crear un fitxer.txt al repositori
- Crea el workflow a .github/workflows/deploy.yml
    - Mostrar el contingut de fitxer mitjançant cat
  ```yml
  - uses: actions/checkout@v4
  - name: Mostra el fitxer 
    run: cat fitxer.txt
  ```
- Crea els secrets per connectar-se al servidor mitjançant SCP
- Transmetre el fitxer al servidor a una carpeta pactada amb els companys de grup 