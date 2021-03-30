# BorgBackup

Utilisation de "borg" pour l'édition d'un script de sauvegarde de serveur web qui sera automatisé par "cron".


# Prérequis

Installation de borg :

    sudo yum install borgbackup

## Borg initialisation

Pour sauvegarder via borg, il faut préparer un repertoire de backup que l'on assigne à borg :

    mkdir saves
    // set user borg root for backup
    useradd root --create-home --home-dir saves/
    borg init --encryption=none saves/
    

## Exemple d'une sauvegarde borg

    borg create saves::"Machine name" "/path of/dir/to save/"

## Script coté serveur web

Utilisation de prune pour une meilleur gestion des backup (cleaning).
   

     !/bin/sh

    # Script de sauvegarde.
    
    set -e

    BACKUP_DATE=`date +%Y-%m-%d`
    
    LOG_PATH=/var/log/borg-backup.log

    BORG_REPOSITORY=root@172.26.0.10:/root/saves/web1
    
    BORG_ARCHIVE=${BORG_REPOSITORY}::${BACKUP_DATE}

    borg create \
    
    -v --stats --compression lzma,9 \
    
    $BORG_ARCHIVE \
    
    "/Here the /path of the /dir to save" \
    
    >> ${LOG_PATH} 2>&1
    
    # Prune backup clean
    
    # - 1 save per day for the last 7 days,
    
    # - 1 save per week for the last 4 weeks,
    
    # - 1 save per month for the last 6 month,.

    borg prune -v $BORG_REPOSITORY \
    
    --keep-daily=7 \
    
    --keep-weekly=4 \
    
    --keep-monthly=6 \
    
    >> ${LOG_PATH} 2>&1

