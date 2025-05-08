# Docker Setup Guide for Database Workshops

This guide will help you set up Docker environments for our hands-on database sessions covering PostgreSQL (RDBMS), MongoDB (Document DB), and Neo4j (Graph DB).

## Prerequisites

- Computer with at least 8GB RAM, 20GB free disk space
- Administrative privileges to install software
- Internet connection to download Docker and images

## Step 1: Install Docker Desktop

### For Windows:
1. Download Docker Desktop from [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Run the installer and follow the prompts
3. Ensure WSL 2 is enabled if prompted
4. Start Docker Desktop after installation

### For macOS:
1. Download Docker Desktop from [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Drag Docker to your Applications folder
3. Start Docker from your Applications 
4. Allow any permission requests

### For Linux:
1. Follow the official Docker installation guide for your distribution:
   [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)
2. Configure Docker to start on boot and add your user to the docker group:
   ```
   sudo systemctl enable docker.service
   sudo systemctl enable containerd.service
   sudo usermod -aG docker $USER
   ```
3. Log out and log back in for changes to take effect

## Step 2: Verify Docker Installation

Open a terminal or command prompt and run:
```
docker --version
docker-compose --version
```

You should see version information for both commands.

## Step 3: Create Workshop Directory

Create a directory to store all workshop materials:

```
mkdir -p database-workshops/{postgres,mongodb,neo4j}
cd database-workshops
```

## Step 4: Create Docker Compose File

Create a file named `docker-compose.yml` in the root of your workshop directory:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:14
    container_name: workshop-postgres
    environment:
      POSTGRES_PASSWORD: workshop
      POSTGRES_USER: workshop
      POSTGRES_DB: workshop
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./postgres/init:/docker-entrypoint-initdb.d
    networks:
      - workshop-network

  mongodb:
    image: mongo:6
    container_name: workshop-mongodb
    environment:
      MONGO_INITDB_ROOT_USERNAME: workshop
      MONGO_INITDB_ROOT_PASSWORD: workshop
    ports:
      - "27017:27017"
    volumes:
      - mongodb-data:/data/db
      - ./mongodb/init:/docker-entrypoint-initdb.d
    networks:
      - workshop-network

  neo4j:
    image: neo4j:5
    container_name: workshop-neo4j
    environment:
      NEO4J_AUTH: neo4j/workshop
    ports:
      - "7474:7474"  # HTTP
      - "7687:7687"  # Bolt
    volumes:
      - neo4j-data:/data
      - ./neo4j/import:/import
    networks:
      - workshop-network

networks:
  workshop-network:
    driver: bridge

volumes:
  postgres-data:
  mongodb-data:
  neo4j-data:
```

## Step 5: Starting the Environments

From your workshop directory, run:

```
docker-compose up -d
```

This will download the necessary images and start all three database environments in the background.

## Step 6: Verifying Services

### PostgreSQL:
```
docker exec -it workshop-postgres psql -U workshop -d workshop -c "SELECT version();"
```

### MongoDB:
```
docker exec -it workshop-mongodb mongosh --username workshop --password workshop --eval "db.version()"
```

### Neo4j:
Open your browser and navigate to http://localhost:7474/
Login with:
- Username: neo4j
- Password: workshop

## Step 7: Stopping the Environments

When you're done with the workshop, you can stop the containers:

```
docker-compose down
```

If you want to remove all data volumes as well:

```
docker-compose down -v
```

## Troubleshooting

### Port Conflicts
If you get an error about ports already being in use, edit the `docker-compose.yml` file to change the external ports (the first number in the port mappings).

### Memory Issues
If containers crash or fail to start properly, your Docker might not have enough memory allocated. Go to Docker Desktop settings and increase the RAM allocation.

### Connection Issues
- For PostgreSQL: Ensure you're using the correct username, password, and database name
- For MongoDB: Check connection string includes authentication credentials
- For Neo4j: Verify browser can access localhost:7474

## Next Steps

- Proceed to the database-specific workshop materials
- Import the provided sample datasets
- Try running the example queries