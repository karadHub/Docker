# used offical node image of node.js runtime
FROM node:22-alpine

# set the working directory in the container
WORKDIR /app

# copy package.json and package-lock.json to the working directory
COPY package*.json ./

# install dependencies
RUN npm install

# copy the rest of the application code to the working directory
COPY . .

ENV PORT=3000

CMD [ "node", "index.js" ]