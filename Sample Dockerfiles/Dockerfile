FROM node:6-alpine
EXPOSE 3000
RUN apk add --update tini
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY package.json package.json
RUN npm install && npm cache clean  
#run 2. command if the first command is run. if you use ';' then the second command will execute regardless of first command success.
COPY . .
#copy everything from current working directory of the host to current working directory of the image.
CMD ["tini", "--", "node", "./bin/www"]