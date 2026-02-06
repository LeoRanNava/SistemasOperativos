#!/bin/bash

#Creamos el directorio el proyecyo que se respaldara
DIR_PROYECTO="$HOME/Documentos/scripts"
#Directorio donde se guarda los respaldos
DIR_RESPALDOS="$HOME/respaldos"
#Fecha que sera actual en formato como la practica anterio AÑO-MES-DIA
FECHA=$(date +%Y-%m-%d)
#Nombre del archivo de respaldo
ARCHIVO_RESPALDO="proyecto_$FECHA.tar.gz"
#Ruta completa del respaldo
RUTA_RESPALDO="$DIR_RESPALDOS/$ARCHIVO_RESPALDO"
#Nombre del archivo de bitacora
ARCHIVO_BITACORA="bitacora_respaldo_$FECHA.json"
#Ruta completa de la bitacora
RUTA_BITACORA="$DIR_RESPALDOS/$ARCHIVO_BITACORA"

#Procedemos a verificar que exista el directorio del proyecto
if [ ! -d "$DIR_PROYECTO" ]; then #Prueba si es un directorio, el ! indica NO por lo tanto la condicion completa significa SSI NO EXISTE EL DIRECTORIO DEL PROYECTO, entra en el if
	echo "Error: el directorio del proyecto no existe: $DIR_PROYECTO" #Esto informa que paso al usuario manda el texto y agrega la ruta para que sepa el usuario
	exit 1 #el 0 es todo bien y el 1 es error
fi

#Creamos el directorio de respaldos si no existen
if [ ! -d "$DIR_RESPALDOS" ]; then #La pregunta es no existe el directorio de respaldos 
	mkdir -p "$DIR_RESPALDOS" #aqui crea el directorio y la -p no marca error si ya existe, en conclusion si no existe lo crea y si existe no se queja eso
fi

#Creamos la aprte del tiempo
#Tiempo inicial

TIEMPO_INICIO=$(date +%s) #devuelve los segundos 
#pasamos a la creacion del respaldo comprimido
tar -czf "$RUTA_RESPALDO" -C "$HOME/Documentos" scripts  #tar:empaquetador   -c: crea el archivo, -z:comprime con gzip, -f:nombre del archivo resultante,  -C HOME cambia temp al direc HOME. proyecto es la carpeta a respaldar

#verificamos si el respaldo se creo correctamente
if [ $? -ne 0 ]; then #$? codigo de salida del ultimo comando, el cero es exito, si es distinto de 0 da error
	ESTADO_RESPALDO="fallido"
else
	ESTADO_RESPALDO="exitoso"
fi

#Medimos el tiempo final
TIEMPO_FIN=$(date +%s)
#Duracion del respaldo en segundos
DURACION=$((TIEMPO_FIN - TIEMPO_INICIO))

#Obtendremos el tamño del archivo (si fue exitoso)
#Obtenemos el tamaño del respaldo en bytes

if [ "$ESTADO_RESPALDO" = "exitoso" ]; then
	TAMANIO_BYTES=$(stat -c %s "$RUTA_RESPALDO")
else
	TAMANIO_BYTES=0
fi

#HAREMOS UN ARREGLO PARA GUARDAR LOS RESPALDOS YA ELIMINADOS
RESPALDO_ELIMINADOS=()

#BUSCAMOS RESPALDOS CON MAS DE 7 DIAS CON EL COMANDO FIND

#Buscamos respaldos con mas de 7 dias
ARCHIVOS_VIEJOS=$(find "$DIR_RESPALDOS" -name "proyecto_*.tar.gz" -mtime +7) ##find:busqueda de archivos, $DIR_RESPALDOS:donde buscar, -name proyecto_*.tar.gz":solo respaldos valido, -mtime +7 modificados hace mas de 7 dias

##Eliminamos respaldos y guardamos nombres

for ARCHIVO in $ARCHIVOS_VIEJOS; do #recorre cada archivo encontrado
	NOMBRE=$(basename "$ARCHIVO") #extrae solo el nombre del archivo
	rm "$ARCHIVO" #elimica el archivo
	RESPALDOS_ELIMINADOS+=("$NOMBRE") #agrega el nombre al arreglo
done



##EVALUAMOS SI EL RESPALDO FUE EXITOSO Y, SEGUN ESO, DEFINE EL ESTADO FINAL DEL PROCESO

if [ "$ESTADO_RESPALDO" = "exitoso" ]; then ##guarda el resultado del proceso de respaldo 
	ESTADO_FINAL="completado" #aqui el if entra si el resultado fue correcto en
else
	ESTADO_FINAL="error" #en otro caso entra si no fue correcto
fi

##FORMATO JSON: RESPALDOS QUE FUERON ELIMINADOS

JSON_ELIMINADOS=""
for ARCH in "${RESPALDOS_ELIMINADOS[@]}"; do
	JSON_ELIMINADOS+="	\"$ARCH\", \n"
done


##QUitamos la ultima coma si hubo elementos 
JSON_ELIMINADOS=$(echo -e "$JSON_ELIMINADOS" | sed '$ s/, $//')


##CREAMOS BITACORA JSON

cat << EOF > "$RUTA_BITACORA"
{
	"fecha": "$FECHA",
	"operacion": "respaldo_proyecto",
	"respaldo_generado": {
	  "archivo": "$ARCHIVO_RESPALDO",
	  "ruta": "$RUTA_RESPALDO",
	  "tamano_bytes": $TAMANIO_BYTES,
	  "estado": "$ESTADO_RESPALDO",
	  "duracion_seg": $DURACION
	},
	"respaldos_eliminados": [

  $(echo -e "$JSON_ELIMINADOS")
    ],
    "estado_final": "$ESTADO_FINAL"
}

EOF
