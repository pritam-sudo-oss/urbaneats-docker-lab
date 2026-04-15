services:
  frontend:
    build:
      context: ./web
    restart: unless-stopped
    depends_on:
      - backend
    networks:
      - app-network

  backend:
    build:
      context: ./api
    restart: always
    depends_on:
      - postgresdb
      - rediscache
    environment:
      DB_HOST: postgresdb
      CACHE_HOST: rediscache
    networks:
      - app-network

  postgresdb:
    image: postgres:15-alpine
    container_name: urbaneats-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin123
      POSTGRES_DB: urbaneats
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - app-network

  rediscache:
    image: redis:7-alpine
    container_name: urbaneats-cache
    restart: unless-stopped
    networks:
      - app-network

  rabbitmq_service:
    image: rabbitmq:3-management-alpine
    container_name: urbaneats-queue
    restart: unless-stopped
    ports:
      - "15672:15672"
      - "5672:5672"
    networks:
      - app-network

  reverseproxy:
    image: nginx:alpine
    container_name: urbaneats-proxy
    restart: unless-stopped
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "8080:80"
    depends_on:
      - frontend
      - backend
    networks:
      - app-network

volumes:
  pgdata:

networks:
  app-network:
    driver: bridge
