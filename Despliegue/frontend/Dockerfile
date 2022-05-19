# Uso la imagen de node (imagen oficial para node.js) version 18 con la distribucion alpine
FROM node:18-alpine

LABEL maintainer="ignacio bergantinos <ignaciobergantinos@gmail.com>"

# Creo mi directorio de trabajo
RUN mkdir -p /usr/src/app/frontend
WORKDIR /usr/src/app/frontend

# Copio Package.json y package-lock.json en mi working directory
# Para luego instalar dependencias
COPY package*.json /usr/src/app/frontend/
RUN npm install


# Copias la app en el working directory
COPY . /usr/src/app/frontend

# Puerto usado para Express apps
EXPOSE 3000
CMD ["npm", "start"]
