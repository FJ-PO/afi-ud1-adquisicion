# Notas de trabajo - UD1 - FORENSE-06

Mi cuaderno de la intervención, en orden y con lo que hice, vi y decidí.

- Marca: AFI-FranciscoJosePerezOrtiz-20261031
- Equipo investigado: máquina FORENSIC_10 (Windows 7 Pro, usuario Administrador)

La Parte 1 (en vivo) la hice el 31/10 en el equipo con Windows donde estaba la máquina. La Parte 2 (en frío) y la Parte 3 (buscar la marca) las hice en un
equipo con Linux.


## Parte 1 - Adquisición en vivo (31/10/2026)

Antes de tocar nada dejé mi marca:

- Escribí AFI-FranciscoJosePerezOrtiz-20261031 en el Bloc de notas y lo dejé abierto sin guardar.
- Creé en el escritorio el archivo marca-franciscojoseperezortiz.txt con el
  mismo texto.
- Copié ese archivo al pendrive que hace de pendrive encontrado.

Estado inicial: Hice una captura de la pantalla. La hora del equipo era
14:07:04 y la hora real 14:07:50, así que el equipo iba 46 segundos atrasado.
Lo anoté para tenerlo en cuenta al interpretar las horas.

Memoria RAM. Primero probé DumpIt, pero en Windows 7 no cargaba su
controlador, así que usé WinPmem y volqué la RAM a FORENSE-06-ram.raw.

Datos volátiles: El script de respuesta en vivo de los apuntes no funcionaba en PowerShell 2.0. 

Lo adapté y lo copié al USB. 
Generó la carpeta AFI-UD1_FORENSE-06_20261031-133614 con procesos, conexiones, usuarios, etc.

Tráfico de red: Capturé el tráfico desde fuera de la máquina, sin instalar nada dentro, usando VBoxManage y al terminar paré la captura y guardé red.pcap.

Triaje. KAPE no funcionó y Velociraptor 0.77 tampoco. Usé Velociraptor 0.6.9 y obtuve FORENSE-06-triaje.zip.

Cifrado: Lo comprobé con EDD. 
Resultado: no hay volúmenes cifrados. Guardé la captura en FORENSE-06-EDD.txt.

Apagado: Apagué por desconexión directa. Ya tenía todo lo volátil y no había
cifrado, un apagado normal podría ejecutar scripts del atacante y escribir en
el disco.

Al final comprobé los hashes de todas las evidencias en mi ordenador: todos
coincidían. Todo quedó en el USB de respuesta.


## Parte 2 - Adquisición en frío

### El disco de la máquina

Traje el disco de la VM (FORENSIC_10_disk0.vmdk) del otro ordenador por red
(scp) y lo dejé en solo lectura. Comprobé que el hash era el mismo en origen y destino:

    sha256sum FORENSIC_10_disk0.vmdk
    # 556a7dacca69bf029d6a8de9ca0ec38379935cad25f9e0211b4a2479566832d8
    chmod 444 FORENSIC_10_disk0.vmdk 

Con la máquina apagada, documenté que tenía 1 disco y ninguna instantánea, y calculé el hash del disco y de sus archivos de configuración:

sha256sum FORENSIC_10_caso2.vbox Logs/VBox.log Logs/VBoxHardening.log

Hacer la imagen: El .vmdk guarda un disco de 32 GB comprimido, así que dd no sirve. Lo abrí como disco crudo en solo lectura con qemu-nbd y lo adquirí en formato E01 con ewfacquire:

    sudo modprobe nbd
    sudo qemu-nbd --connect=/dev/nbd0 --read-only -f vmdk FORENSIC_10_disk0.vmdk
    sudo ewfacquire /dev/nbd0        # salida: FORENSE-06-disco.E01
    sudo qemu-nbd --disconnect /dev/nbd0

Después volví a calcular el hash del .vmdk: seguía siendo 556a7d…, así que el
disco original no se tocó. Y verifiqué la imagen:

    ewfverify FORENSE-06-disco.E01
    # ewfverify: SUCCESS

### El pendrive encontrado

El pendrive lo adquirí el 03/11 con el bloqueo de escritura por software. En la captura se ve el estado PROTEGIDO antes de conectarlo y FTK
Imager ya creando la imagen. La imagen la verifiqué:

    ewfverify pendrive_encontrado.E01
    # ewfverify: SUCCESS


## Parte 3 - Buscar mi marca

Mi marca tiene que aparecer en todas las evidencias. La busqué desde Linux.

En la RAM:

    strings -el FORENSE-06-ram.raw | grep "AFI-"
    # AFI-FranciscoJosePerezOrtiz-20261031

En el triaje. El triaje no copia los archivos del escritorio, pero sí la $MFT, donde está el nombre de mi archivo de marca:

    strings -el '$MFT' | grep "marca-"
    # marca-franciscojoseperezortiz.txt

En las imágenes del disco y del pendrive: Las monté en solo lectura con
ewfmount y encontré mi archivo de marca en el escritorio del Administrador y en el pendrive, con el texto AFI-FranciscoJosePerezOrtiz-20261031.
