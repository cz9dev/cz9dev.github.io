---
layout: post
title: Convertir MBR BIOS a GPT UEFI sin perder datos
subtitle: Convertir disco MBR BIOS a GPT UEFI sin perder nuestros datos
## cover-img: /assets/img/convert-mbr2gpt/mbr2gpt.png
thumbnail-img: /assets/img/convert-mbr2gpt/mbr2gpt.png
tags: [Windows, MBR, GPT, UEFI]
comments: true
---

Hola, hoy les traigo una entrada para los que se les ha presentado un problema a la hora de migrar de una Motherboard antigua a una nueva o algo similar, donde tengan que cambiar la forma de buteo de sus PC de MBR Bios a GPT UEFI lo puedan hacer sin perder sus datos. Bueno acá les dejo la forma sin más:

# Convertir MBR BIOS a GPT UEFI sin perder datos de nuestro HDD

1. Lo primero será iniciar nuestra PC con una Memoria USB que tengamos con el instalador de Windows
2. dfgh
3. En la terminal abierta escribimos:
```bash
diskpart
```

4. Una ves dentro de DISKPART escribimos:
```bash
DISKPART> list disk
```

***exit*** para salir de DISKPART

Ahí veremos los discos que tenemos instalado, debemos fijarnos bien en el numero del disco que queremos cambiar a GPT UEFI, como ejemplo tomaremos que es el **Disco 0**

5. Ahora lo que aremos es comprobar si podemos hacer la conversion, para eso lo aremos escribiendo la siguiente instrucción:
```bash
mbr2gpt.exe /validate /disk:0
```

Si al intentar validar nos mostró el error **mbr2gpt cannot find os partition(s) for disk 0** lo mas seguro es que necesitemos marcar a C como activa para eso deberíamos ejecutar la siguiente instrucción:
```bash
bcdboot C:\Windows /s C:
```
si con esto no corrigen el problema les recomiendo ver esta otra solución: [ver en YouTube](https://www.youtube.com/watch?v=E2a5Uqj02OM)

6. Si todo fue correcto y no hay problemas entonces convertimos nuestro disco con la siguiente instrucción:
```bash
mbr2gpt /convert /disk:0
```

Con esto ya podremos iniciar nuestra PC correctamente
