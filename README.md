# web-app-1

full docker compose 



version: '3.8'

services:
  app011:
    build:
      context: ./application-011
      dockerfile: Dockerfile
    container_name: app011
    ports:
      - "8081:80"
    environment:
      - NODE_ENV=production
      - API_KEY=${APP011_API_KEY}
    volumes:
      - app011-data:/usr/src/app/data
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
    depends_on:
      - db
    networks:
      - appnet

  app022:
    build:
      context: ./application-022
      dockerfile: Dockerfile
    container_name: app022
    ports:
      - "8082:80"
    environment:
      - NODE_ENV=production
      - API_KEY=${APP022_API_KEY}
    volumes:
      - app022-data:/usr/src/app/data
    depends_on:
      - db
    networks:
      - appnet

  db:
    image: postgres:15
    container_name: appdb
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: securepassword
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - appnet

volumes:
  app011-data:
  app022-data:
  db-data:

networks:
  appnet:







To Use:
Make sure your two apps are in:

./application-011/Dockerfile

./application-022/Dockerfile

Create a .env file to store:

dotenv
Copy
Edit
APP011_API_KEY=your_key_011
APP022_API_KEY=your_key_022
Then run:

bash
Copy
Edit
docker-compose up --build
Would you like me to help you write the Dockerfile for either app too?









Ask ChatGPT

