#!/bin/bash

# Couleur verte pour l'affichage du texte
GREEN='\033[0;32m'
RED='\033[0;31m'
NC='\033[0m' # Réinitialisation de la couleur

# Demander à l'utilisateur de choisir entre "prod" ou "qual"
read -p "Veuillez choisir entre 'prod' ou 'qual': " choice

if [ "$choice" = "prod" ]; then
    echo -e "${RED}Vous avez choisi 'prod'.${NC}"
    container_prefix="monitoring_prod"
    volume_prefix="monitoring_data"
    container_port=8080
elif [ "$choice" = "qual" ]; then
    echo -e "${GREEN}Vous avez choisi 'qual'.${NC}"
    container_prefix="monitoring_qual"
    volume_prefix="monitoring_data"
    container_port=8081
else
    echo -e "${RED}Choix invalide.${NC}"
    exit 1
fi

# Demander à l'utilisateur de choisir l'image Docker à utiliser
read -p "Veuillez saisir le nom de l'image Docker : " image_name

# Extraire la version de Checkmk à partir du nom de l'image
version_checkmk=$(echo "$image_name" | sed -E 's/.*:([0-9.]+).*/\1/')

# Vérifier si la version de Checkmk a été trouvée
if [ -z "$version_checkmk" ]; then
    echo -e "${RED}Impossible de récupérer la version de Checkmk à partir du nom de l'image.${NC}"
    exit 1
fi

echo -e "${GREEN}Version de Checkmk : $version_checkmk${NC}"

# Variables pour les noms des volumes et des conteneurs
container_name="${container_prefix}_${version_checkmk}"
volume_name="${volume_prefix}_${version_checkmk}"

# Étape 3 : Copier l'ancien volume vers le nouveau volume
echo -e "${GREEN}Étape 3 : Copie de l'ancien volume vers le nouveau volume${NC}"
docker run --rm -v monitoring_data_source:/source -v "$volume_name":/destination alpine cp -a /source/. /destination
if [ $? -eq 0 ]; then
    echo -e "${GREEN}La copie de l'ancien volume vers le nouveau volume a été effectuée avec succès.${NC}"
else
    echo -e "${RED}Une erreur s'est produite lors de la copie de l'ancien volume vers le nouveau volume.${NC}"
    exit 1
fi

# Étape 4 : Exécuter le nouveau conteneur avec la nouvelle version de Checkmk
echo -e "${GREEN}Étape 4 : Exécution du nouveau conteneur avec la nouvelle version de Checkmk${NC}"
docker run --name "$container_name" -p "$container_port:80" -v "$volume_name":/checkmk_data "$image_name"
if [ $? -eq 0 ]; then
    echo -e "${GREEN}Le nouveau conteneur a été exécuté avec succès.${NC}"
else
    echo -e "${RED}Une erreur s'est produite lors de l'exécution du nouveau conteneur.${NC}"
    exit 1
fi

# Étape 5 : Vérification de la mise à jour
echo -e "${GREEN}Étape 5 : Vérification de la mise à jour"
# Ajoutez ici vos étapes de vérification supplémentaires
echo -e "Veuillez effectuer les vérifications nécessaires.${NC}"

# Étape 6 : Nettoyage des ressources inutilisées
echo -e "${GREEN}Étape 6 : Nettoyage des ressources inutilisées"
# Ajoutez ici vos étapes de nettoyage supplémentaires
echo -e "Veuillez effectuer le nettoyage approprié.${NC}"

echo -e "${GREEN}La mise à jour de la version Docker de Checkmk a été effectuée avec succès.${NC}"
