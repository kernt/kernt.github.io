log sammler

```sh
#!/bin/bash
 
if [ "$#" -lt 1 ]
then
        echo "USAGE:    dev_get-contrail-logs.sh [CASEID]"
        echo "or        dev_get-contrail-logs.sh [CASEID] [MACHINE NUMBER] <-- optional"
        echo " "
        echo "Example:          dev_get-contrail-logs.sh 12345          (collects logs from all contrail-containers from ALL controllers)"
        echo "or                dev_get-contrail-logs.sh 12345 67       (collects logs from contrail controller on machine 67 (--> juju status 67)"
        exit 1
fi

caseid="$1"
machine="$2"
tmpdir="/tmp/CASES/${caseid}"
 
echo "CASE ID: ${caseid}"
 
if [ ! -e "${tmpdir}" ]
then
        mkdir -p "${tmpdir}"
fi

if [ "${machine}" ]                     # dedicated machine is given as optional parameter
then
        CONTRAIL_UNITS=( "${machine}" )
else                                    # do for all machine units
        CONTRAIL_UNITS="$(juju status contrail4-controller |& sed -n '/^Machine/,//p' | tail -n +2 | awk '{ print $1 }' | tr -d " ")"
fi
 
for unit in ${CONTRAIL_UNITS[@]}
do
        CONTAINERS=$(juju ssh ${unit} sudo docker ps | tail -n +2 | awk '{ print $1 }' | tr -d " ")
        for container in ${CONTAINERS[@]}
        do
                echo "======================================="
                echo " Unit ${unit} - Container ${container} "
                echo "---------------------------------------"
 
                echo "Erzeuge Contrail Status Bericht..."
                juju ssh ${unit} "sudo docker exec ${container} bash -c 'contrail-status > /tmp/contrail-status.txt'"
 
                echo "Erzeuge Contrail Log File Archiv auf Container ${container}..."
                juju ssh ${unit} "sudo docker exec ${container} bash -c 'tar --overwrite -czf /tmp/contrail-${unit}-${container}.tgz /tmp/contrail-status.txt /var/log/'"
 
                echo "Kopiere Archiv vom Container auf Maschine ${unit}..."
                juju ssh ${unit} "sudo docker cp ${container}:/tmp/contrail-${unit}-${container}.tgz /tmp/contrail-${unit}-${container}.tgz" &&
 
                echo "Loesche Archiv und Contrail Status auf Container ${container}..."
                juju ssh ${unit} "sudo docker exec ${container} bash -c 'rm -f /tmp/contrail-status.txt /tmp/contrail-${unit}-${container}.tgz'"
 
                echo "Kopiere Archiv von Maschine ${unit} auf JumpHost..."
                juju scp ${unit}:/tmp/contrail-${unit}-${container}.tgz "${tmpdir}" &&
 
                echo "Loesche Archiv auf Maschine ${unit}..."
                juju ssh ${unit} "sudo rm -f /tmp/contrail-${unit}-${container}.tgz"
        done
done

tar -czf ${tmpdir}/${caseid}_juniper_$(date +"%Y%m%d_%H%M").tgz ${tmpdir}/contrail*.tgz
 
echo "Fertig !"
```

