
# Étape 1 : Build Angular
FROM node:22-alpine AS build

WORKDIR /app

# Installer les dépendances
COPY package*.json ./
RUN npm ci

# Copier le code source
COPY . .

# Compiler Angular pour la production
RUN npm run build

# Étape 2 : Serveur Nginx
FROM nginx:alpine

# Supprimer la configuration par défaut
RUN rm -rf /usr/share/nginx/html/*

# Copier les fichiers Angular compilés
COPY --from=build /app/dist/frontend/browser/ /usr/share/nginx/html/

# Exposer le port HTTP
EXPOSE 80

# Démarrer Nginx
CMD ["nginx", "-g", "daemon off;"]
