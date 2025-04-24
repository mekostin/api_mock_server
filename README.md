# API Mock Server

A server for emulating various APIs with configurable response delays. Allows creating multiple independent services, each with its own delay.

## Project Structure

```
.
├── docker-compose.yml    # Docker configuration
├── Dockerfile           # Image build
├── nginx.conf          # Nginx configuration
└── services/           # Services directory
    ├── users/          # Users service
    │   └── responses/  # JSON responses
    ├── products/       # Products service
    │   └── responses/  # JSON responses
    └── orders/         # Orders service
        └── responses/  # JSON responses
```

## Running

```bash
docker-compose up --build
```

## Configuration

### Ports

- Users API: 8081
- Products API: 8082
- Orders API: 8083

### Delays

Delays are configured through environment variables in `docker-compose.yml`:

```yaml
environment:
  USERS_DELAY_MS: 500    # 500 ms
  PRODUCTS_DELAY_MS: 5000 # 5 seconds
  ORDERS_DELAY_MS: 10000  # 10 seconds
```

## Usage

### Get Users List
```bash
curl 'http://localhost:8081/responses/list.json'
```

### Get Products List
```bash
curl 'http://localhost:8082/responses/list.json'
```

### Get Orders List
```bash
curl 'http://localhost:8083/responses/list.json'
```

## Adding New Services

1. Create a new directory in `services/`
2. Add JSON files to `services/new_service/responses/`
3. Add a new `server` block to `nginx.conf`:
```nginx
server {
    listen 8084;  # New port
    
    set_by_lua_block $new_service_delay_ms {
        local delay = os.getenv("NEW_SERVICE_DELAY_MS")
        return delay
    }
    
    location / {
        root /usr/local/openresty/nginx/html/services/new_service;
        
        access_by_lua_block {
            if ngx.var.new_service_delay_ms then
                ngx.sleep(tonumber(ngx.var.new_service_delay_ms) / 1000)
            end
        }
        
        try_files $uri $uri/ =404;
        add_header Content-Type application/json;
    }
}
```
4. Add an environment variable to `docker-compose.yml`:
```yaml
environment:
  NEW_SERVICE_DELAY_MS: 1000  # 1 second
```

## Changing Delays

To change delays, edit the values in `docker-compose.yml` and restart the server:

```bash
docker-compose down
docker-compose up
``` 