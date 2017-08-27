# firewall
#1
#!/bin/bash
start (){ 
echo "FIREWALL USB."
CONTROL=0
while [ $CONTROL=0 ] ;
	do
	   	cat /etc/mtab | grep media >> /dev/null
		if [ $? -ne 0 ]; then
		 	CONTROL=0
		 	echo "BUSCANDO DISPOSITIVOS USB..."
  		else
			CONTROL=1
			for USBDEV in `df | grep media | awk {'print $6'}` ;
			do
			USBNAME=`echo $USBDEV | cut -d "/" -f 4`
        		#USBSERIAL= $(dmesg | tail -n20 | grep ": Serial" | awk '{print $6}' )
			echo -e  "Se ha detectado un dispositivo nuevo:$USBNAME "
			Titulo="Menu"
        		Pregunta="Se detecto una USB, ¿Que desa hacer con el ?:"
			#echo "Se ha detectado la usb con No de Serie : $USBSERIAL "
	      		echo $(grep -c $USBNAME lista_blanca1.txt )
			if [ $? -ne 0 ]; then
	           		echo "USB EN LISTA BLANCA $USBNAME "
	           		echo "SELECCIONA UNA OPCION ."
	           		echo "1)Es mi USB (montar) "
	           		echo "2)Desmontar"
				echo "3)salir"
	           		read n
	        		case $n in
	              			(1)	
					echo "Montando usb... No lo extraiga"
					sudo mount -t vfat /dev/sdb1 /media/usb
					sleep 2s
					echo "Dispositivo listo para usarse"
					echo "Contenido de la unidad USB $USBNAME"
					ls  /media/usb
	              			;;
	            			(2)
					sudo umount /media/usb
					echo "Listo"
					exit	
					;;
					(3)
	          			exit
	         		esac
	         		sudo dmesg --clear
	         		break;
	    		 else
				 echo $(grep -c $USBNAME lista_negra.txt )
				 if [ $? -ne 0 ]; then
	       				echo "USB EN LISTA NEGRA PELIGROSA  "
	       				echo "$USBNAME USB PELIGROSA"
					echo " (1)¿Mandar a lista Blanca? (2) No confiar y Salir"
	       				read  op
	       				case $opcion in
	           				(1) 
						echo "Mandar a lista Blanca"
						echo $USBNAME >> lista_blanca1.text
	           				(2)
						echo "bye."
	           				exit
	       				esac
	       				sudo dmesg --clear
	       				break;
				else
Opciones=("1) Reconozco este dispositivo (Mandar a lista blanca)" "2) No lo Reconozco (Mandar a lista negra)" "3) salir")
while opt="$(zenity --title="$Titulo" --text="$Pregunta" --list --column="Opciones" "${Opciones[@] $Versiones}")";
	do
		case $opt in
          		"${Opciones[0]}" )
                	echo "Has elegido $opt"
                	zenity --info --text="Has elegido $opt"
                	echo $USBSERIAL >> lista_blanca1.text
                	echo "Se ha mandado a la lista blanca. ¿Deseas montar la USB en tu Computadora?"
                			echo "1) montar usb "
                			echo "2) desmontar usb"
                	read n
                	case $n in
                		(1) echo "Montando usb... No lo extraiga"
                		sudo mount -t vfat /dev/sdb1 /media/usb
                		sleep 2s
				echo "Dispositivo listo para usarse"
                		echo "Contenido de la unidad USB $USBNAME"
                		ls  /media/usb
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
					echo $USBNAME >> lista_negra.text
					echo "La unidad USB se ha mandado a la lista negra"
					echo "No podra ser usada en este equipo"
                			;;
          			"${Opciones[2]}")
                			echo "Has elegido $opt"
                			zenity --info --text="Has elegido $opt"
        				exit
        				;;	
          			"${Opciones[-1]}")
                			zenity --error --text="Opcion Incorrecta , Intenta con otra."
                			;;
            		esac
				done	
			fi

	 	fi
	done
  fi
	sleep 10
done
exit 0
}

stop(){
 echo -n $"Stopping service: "
 $shutdown
 RETVAL=$?
 echo
}

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
