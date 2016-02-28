FranceData permet de crawler le site de l'assemblée nationnale pour en extraire
tout les votes.

Fait pour la Societe Citoyenne, pour IlsTravaillentPourVous.fr

Initialiser le serveur
----------------------

FranceData inclut un serveur d'API simple basé sur django-representatives et
django-representatives-votes.

Synchronisez la base de données (fichier SQLite3 local par défaut) :

    ./manage.py migrate

Puis, lancez le serveur :

    ./manage.py


Mettre à jour les données
-------------------------

Lancez les crawlers pour extraire les données parlementaires :

    rm data/parlementaires.json
    scrapy crawl parlspider -o data/parlementaires.json
    rm data/dossiers.json
    scrapy crawl dossierspider -o data/dossiers.json
    rm data/scrutins.json
    scrapy crawl scrutinspider -o data/scrutins.json
    rm data/votes.json
    scrapy crawl votespider -o data/votes.json

Note: le crawler des votes fonctionne de manière incrémentale.  Il se base sur
le fichier `data/scrutins.json`, il faut donc exécuter le crawler des scrutins
avant.  Par ailleurs il utilise les fichiers dans `data/votes/` pour éviter de
re-crawler des scrutins déjà exportés, ne videz pas ce dossier !

Lancez les scripts d'import :

    cat data/parlementaires.json | francedata_import_representatives
    cat data/dossiers.json | francedata_import_dossiers
    cat data/scrutins.json | francedata_import_scrutins
    cat data/votes.json | francedata_import_votes

Déployer sur Openshift
----------------------

Installer et configurer RHC puis créer une app comme suit:

    rhc app-create \
        python-2.7 cron-1.4 postgresql-9.2 \
        -a francedata \
        -e OPENSHIFT_PYTHON_WSGI_APPLICATION=apiserver/wsgi.py \
        --from-code http://github.com/SocieteCitoyenne/FranceData.git \
        --no-git

Il est conseillé d'exécuter manuellement le script `bin/update_all` une première
fois pour initialiser les données.  Par la suite un job cron quotidien tiendra
les données à jour.