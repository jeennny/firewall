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
			if [ $? -eq 0 ]; then
	           		echo "USB EN LISTA BLANCA $USBNAME "
	           		echo "SELECCIONA UNA OPCION ."
	           		echo "1)Es mi USB (montar) "
	           		echo "2)No la Reconozco (salir)"
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
	          			exit
	         		esac
	         		sudo dmesg --clear
	         		break;
	    		 else
				 echo $(grep -c $USBNAME lista_negra.txt )
				 #################if
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
