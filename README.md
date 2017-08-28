#!/bin/bash
#	@file firewall.sh
#	@Authors Alejandro (2) && Jennifer (22) 
#	@date 27/08/17

start (){ 
echo "FIREWALL USB."
CONTROL=0    #Creando Variable para iniciar un ciclo de busqueda de DISPOSITIVOS USB 
while [ $CONTROL=0 ] ;
	do	#SE INICIA LA BUSQUEDA DE DISPOSITIVOS Y MIENTRAS NO ENCUENTRE ESTE SEGUIRA ACTIVO 
	   	cat /etc/mtab | grep media >> /dev/null
		if [ $? -ne 0 ]; then    
		 	CONTROL=0      
		 	echo "BUSCANDO DISPOSITIVOS USB..."
  		else
			CONTROL=1
			for USBDEV in `df | grep media | awk {'print $6'}` ;
			do
			#USBNAME=`echo $USBDEV | cut -d "/" -f 4`
			#UNA VEZ ENCONTRADO CON ESTA LINEA OBTENEMOS UN IDENIFICADOR UNICO DE CADA USB QUE NOS SERVIRA 
			#PARA NUESTROS REGISTRO
       			USB=$( dmesg | tail -n 20 | grep ": Serial" | awk '{print $5}' )
			echo   "Se ha detectado un dispositivo nuevo: $USB "
			Titulo="Menu"
        		Pregunta="Se detecto una USB, ¿Que desa hacer con el ?:"
###   			LO PRIMERO QUE HACEMOS ES BUSCAR EN LA LISTA BLANCA PARA VER SI ESTE DISPOSITIVO YA HA SIDO APROBADO ANTES
        		if [ $( grep -c $USB lista1.text ) == 1 ]; then # SI LO ENCUENTRA REGRESARA UN 1 Y ENTRARA AL IF
	           		echo "USB EN LISTA BLANCA $USB "
	           		echo "SELECCIONA UNA OPCION ."
	           		echo "1)Es mi USB (montar) "
	           		echo "2)Desmontar"
				echo "3)salir"
				
	           		read n
	        		case $n in
	              			(1)	
					echo "Montando usb... No lo extraiga"
					sudo mount -t vfat /dev/sdb1 /media/usb #MONTA LA USB EN LA RUTA SEÑALADA 
					sleep 2s
					echo "Dispositivo listo para usarse"
					echo "Contenido de la unidad USB $USBNAME"
					ls  /media/usb #NOS MUESTRA EL CONTENIDO DEL DISPOSITIVO 
					exit
					
	              			;;
	            			(2)
					sudo umount /media/usb #DESMONTA LA USB
					echo "Listo"
					exit	
					;;
					(3)
	          			exit
	         		esac
	         		sudo dmesg --clear
	         		break;
####
			 elif [ $( grep -c $USB lista2.text ) == 1 ];then #BUSCA EN LA LISTA NEGRA, SI LO ENCUENTA REGRESA UN 1 Y DESPLIEGA EL MENU 
				#if [ $? -eq 0 ]; then
	       				echo "USB EN LISTA NEGRA PELIGROSA  "
	       				echo " USB PELIGROSA"
					echo " (1)¿Mandar a lista Blanca? (2) No confiar y Salir"
	       				read  op
	       				case $op in
	           				(1)
						echo "Mandado a lista Blanca"
						echo $USB >> lista1.text #SE AGREGA A LA LISTA BLANCA
						echo "Montando usb... No lo extraiga"
						sudo mount -t vfat /dev/sdb1 /media/usb #SE MONTA
						sleep 2s
						echo "Dispositivo listo para usarse"
						echo "Contenido de la unidad USB $USB"
						ls  /media/usb#DESPLIEGA EL CONTENIDO DEL DISPOSITIVO 
						exit
						;;

	           				(2)
						echo "bye."
	           				exit
	       				esac
	       				sudo dmesg --clear
	       				#//break;
				else
					Opciones=("1) Reconozco este dispositivo (Mandar a lista blanca)" "2) No lo Reconozco (Mandar a lista negra)" "3) salir")
					while opt="$(zenity --title="$Titulo" --text="$Pregunta" --list --column="Opciones" "${Opciones[@] $Versiones}")"; do
						case $opt in
          					"${Opciones[0]}" )
                					echo "Has elegido $opt"
                					zenity --info --text="Has elegido $opt"
                					echo $USB >> lista1.text #SE AGREGA LISTA BLANCA
                					echo "Se ha mandado a la lista blanca. ¿Deseas montar la USB en tu Computadora?"
                					echo "1) montar usb " # COMANDO MOUNT 
                					echo "2) desmontar usb" #COMANDO UMOUNT
                					read n
                					case $n in
							
#######
      							 (1) echo "Montando usb... No lo extraiga"
                					sudo mount -t vfat /dev/sdb1 /media/usb
                					sleep 2s
							echo "Dispositivo listo para usarse"
                					echo "Contenido de la unidad USB $USB"
                					ls /media/usb
							exit
                					;;
                					(2) echo "Desmontando usb..."
                					sudo umount /media/usb
                					echo "Desmontada ya puedes retirarla "
                					;;
                					(0) exit
                					;;
                					(*)
                					echo "Error! Esa opcion no vale, concentrate."
                					;;
              						esac
							exit
              						;;
          					"${Opciones[1]}")
                					echo "Has elegido $opt"
                					zenity --info --text="Has elegido $opt"
							echo $USB >> lista2.text
							echo "La unidad USB se ha mandado a la lista negra"
							echo "No podra ser usada en este equipo"
                					;;
          					"${Opciones[2]}")
                					echo "Has elegido $opt"
                					zenity --info --text="Has elegido $opt"
        						exit
        						;;#SE CREA UNA VALIDACION DE LA ENTRADA PARA EL USUARIO
          					"${Opciones[-1]}")
                					zenity --error --text="Opcion Incorrecta , Intenta con otra."
                					;;
            					esac
					done
				fi
			done
		fi
	sleep 10
done
exit 0


######## PARA DEMONIZAR AGRAGAMOS LAS SIGUIENTES FUNCIONES
}
#PARAR EL DEMONIO
stop(){
 echo -n $"Stopping service: "
 $shutdown
 RETVAL=$?
 echo
}
#REINICIAR
restart(){
 stop
 sleep 10
 start
}

# Dependiento del parametro que se le pase
#start - stop - restart ejecuta la función correspondiente.
case "$1" in
start)
 start
 ;;
stop)
 stop
 ;;
restart)
 restart
 ;;
*)
 echo $"Usar: $0 {start|stop|restart}"
 exit 1
esac

exit 0
