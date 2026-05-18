FROM node:20
WORKDIR /app
COPY packpage*.json ./
RUN npm install
COPY . .
EXPOSE 5173:5173
CMD ["npm", "run", "dev", "--", "--host"]      