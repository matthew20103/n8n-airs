# n8n-airs

## n8n Server with HTTPS Setup
This section provides the necessary Docker Compose and Caddy configuration files to run n8n with SSL encryption.
1. Create a Project Directory
First, create a directory for your n8n project:

        mkdir n8n-server
        cd n8n-server
2. Create the `docker-compose.yml` file
Create a file named `docker-compose.yml` inside your `n8n-server` directory and paste the following content. Remember to replace `your.domain.com` with your actual domain name and generate a strong `N8N_ENCRYPTION_KEY`.

        # docker-compose.yml
        version: '3.8'
        
        services:
          n8n:
            # CORRECTED: Changed image name to your preferred version
            image: n8nio/n8n:1.109.2
            restart: always
            environment:
              # Replace with your actual domain name
              N8N_HOSTNAME: your.domain.com
              N8N_PROTOCOL: https
              # This is crucial for webhooks to be generated with the correct HTTPS URL
              WEBHOOK_URL: https://your.domain.com/
              # Set your desired timezone, e.g., "Asia/Hong_Kong", "America/New_York"
              GENERIC_TIMEZONE: "Asia/Hong_Kong"
              # IMPORTANT: Generate a strong, random key for encryption.
              # You can use `openssl rand -base64 32` to generate one.
              N8N_ENCRYPTION_KEY: "YOUR_STRONG_RANDOM_ENCRYPTION_KEY_HERE"
              # Optional: Database configuration (Postgres recommended for production)
              # DB_TYPE: postgresdb
              # DB_POSTGRESDB_HOST: postgres
              # DB_POSTGRESDB_PORT: 5432
              # DB_POSTGRESDB_DATABASE: n8n
              # DB_POSTGRESDB_USER: n8nuser
              # DB_POSTGRESDB_PASSWORD: n8npassword
            volumes:
              # Persistent storage for n8n data (workflows, credentials, etc.)
              - n8n_data:/home/node/.n8n
            # n8n listens internally on port 5678
            expose:
              - "5678"
        
          caddy:
            image: caddy:latest
            restart: always
            ports:
              # Caddy listens on standard HTTP and HTTPS ports
              - "80:80"
              - "443:443"
            volumes:
              # Mount the Caddyfile from the current directory
              - ./Caddyfile:/etc/caddy/Caddyfile
              # Persistent storage for Caddy's certificates and state
              - caddy_data:/data
            # Ensure Caddy starts after n8n is ready
            depends_on:
              - n8n
        
        # Define named volumes for persistent data
        volumes:
          n8n_data:
          caddy_data:
3. Create the `Caddyfile`
Create a file named `Caddyfile` in the same `n8n-server` directory (alongside `docker-compose.yml`) and paste the following content. Again, replace `your.domain.com` with your actual domain name.

        # Caddyfile
        # Replace 'your.domain.com' with your actual domain name
        your.domain.com {
            # Enable automatic HTTPS (Caddy will obtain and renew Let's Encrypt certificates)
            tls your_email@example.com # Optional: Provide an email for urgent notices from Let's Encrypt
        
            # Reverse proxy all requests to the n8n service
            reverse_proxy n8n:5678
        
            # Optional: Enable Caddy's access logs
            log {
                output stdout
                format json
            }
        
            # Optional: Gzip compression
            encode gzip
        }
4. Generate an Encryption Key
It's crucial to generate a strong encryption key for n8n. You can use the following command on your Linux terminal:

        openssl rand -base64 32
    Copy the output and paste it as the value for `N8N_ENCRYPTION_KEY` in your `docker-compose.yml` file.

5. Run the N8N Server
Once you have both `docker-compose.yml` and `Caddyfile` in your `n8n-server` directory, navigate to that directory in your terminal and run:

        docker compose up -d
    This command will:
- Download the `n8nio/n8n` and `caddy:latest` Docker images (if not already present).
- Create the `n8n` and `caddy` containers.
- Start both services in detached mode (`-d`), meaning they will run in the background.
- Caddy will automatically try to obtain an SSL certificate for `your.domain.com` from Let's Encrypt.

6. Verify Installation
After a few moments, you should be able to access your n8n instance securely via HTTPS by navigating to `https://your.domain.com` in your web browser.
You can also check the logs to ensure everything is running smoothly:

        docker compose logs -f

    To stop the services, run:

        docker compose down
