FROM nginx:latest

# supprime le fichier de config par défaut
RUN rm /etc/nginx/conf.d/default.conf

# ajoute ta config
COPY nginx.conf /etc/nginx/conf.d/default.conf

