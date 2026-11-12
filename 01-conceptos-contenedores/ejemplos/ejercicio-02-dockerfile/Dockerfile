FROM nginx:1.27-alpine

# Copiar la página personalizada al directorio público de Nginx
COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
