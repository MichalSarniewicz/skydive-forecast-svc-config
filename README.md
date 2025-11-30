# Skydive Forecast - Service Configuration Repository

Centralized configuration repository for all Skydive Forecast microservices.

## Structure

```
skydive-forecast-svc-config/
├── gateway/
│   ├── gateway.yaml              # Default configuration
│   ├── gateway-dev.yaml          # Development profile
│   ├── gateway-prod.yaml         # Production profile
│   ├── gateway-test.yaml         # Test profile
│   ├── gateway-consul.yaml       # Consul-specific config
│   └── gateway-swagger.yaml      # Swagger aggregation config
├── user-service/
│   ├── user-service.yaml
│   ├── user-service-dev.yaml
│   ├── user-service-prod.yaml
│   └── user-service-test.yaml
├── analysis-service/
│   ├── analysis-service.yaml
│   ├── analysis-service-dev.yaml
│   ├── analysis-service-prod.yaml
│   └── analysis-service-test.yaml
├── location-service/
│   ├── location-service.yaml
│   ├── location-service-dev.yaml
│   ├── location-service-prod.yaml
│   └── location-service-test.yaml
├── shared/
│   └── application-consul.yaml   # Shared Consul configuration
└── README.md
```

## Naming Convention

### Service-Specific Configuration
- `{service-name}.yaml` - Default configuration loaded for all profiles
- `{service-name}-{profile}.yaml` - Profile-specific configuration (dev, test, prod)
- `{service-name}-{feature}.yaml` - Feature-specific configuration (consul, swagger)

### Shared Configuration
- `application.yaml` - Shared across all services (all profiles)
- `application-{profile}.yaml` - Shared for specific profile
- `application-{feature}.yaml` - Shared feature configuration

## Configuration Loading Order

Spring Cloud Config loads files in this order (later overrides earlier):

1. `application.yaml` (shared)
2. `application-{profile}.yaml` (shared)
3. `{service-name}.yaml` (service-specific)
4. `{service-name}-{profile}.yaml` (service + profile)

## Usage

### Config Server Configuration

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: file:///path/to/skydive-forecast-svc-config
          search-paths: '{application}'
          default-label: master
```

The `search-paths: '{application}'` tells Config Server to look in folders named after the service.

### Client Configuration (bootstrap.yaml)

```yaml
spring:
  application:
    name: user-service  # Matches folder name
  cloud:
    config:
      uri: http://config-server:8888
  profiles:
    active: dev,consul
```

This will load:
1. `shared/application-consul.yaml`
2. `user-service/user-service.yaml`
3. `user-service/user-service-dev.yaml`

## Adding New Service Configuration

1. Create folder with service name:
```bash
mkdir new-service
```

2. Create configuration files:
```bash
touch new-service/new-service.yaml
touch new-service/new-service-dev.yaml
```

3. Add configuration:
```yaml
# new-service/new-service.yaml
server:
  port: 8084
spring:
  application:
    name: new-service
```

4. Commit changes:
```bash
git add new-service/
git commit -m "Add new-service configuration"
```

## Environment-Specific Configuration

### Development (dev)
- Local database connections
- Debug logging enabled
- Relaxed security

### Test (test)
- Test database
- Integration test settings
- Mock external services

### Production (prod)
- Production database
- Error-level logging
- Strict security
- Performance optimizations

## Sensitive Data

**DO NOT** commit sensitive data like:
- Passwords
- API keys
- Certificates
- Private keys

Use environment variables or external secret management:

```yaml
spring:
  datasource:
    password: ${DB_PASSWORD}
```

Or use Spring Cloud Config encryption:

```bash
# Encrypt value
curl http://config-server:8888/encrypt -d mysecret

# Use in config
spring:
  datasource:
    password: '{cipher}AQA...'
```

## Profiles

### Active Profiles

Services can activate multiple profiles:

```yaml
spring:
  profiles:
    active: dev,consul,swagger
```

This loads:
- `{service}-dev.yaml`
- `{service}-consul.yaml`
- `{service}-swagger.yaml`

### Common Profiles

- `dev` - Development environment
- `test` - Testing environment
- `prod` - Production environment
- `consul` - Consul service discovery
- `swagger` - Swagger/OpenAPI configuration
- `kubernetes` - Kubernetes-specific settings

## Refresh Configuration

Update configuration without restarting:

1. Update config file
2. Commit to Git
3. Trigger refresh:
```bash
curl -X POST http://service:8081/actuator/refresh
```

## Best Practices

1. **Organize by service** - Each service has its own folder
2. **Use profiles** - Separate configs for different environments
3. **Shared configs** - Common settings in `shared/` folder
4. **No secrets** - Use environment variables or encryption
5. **Version control** - Commit all changes with meaningful messages
6. **Documentation** - Comment complex configurations

## Troubleshooting

### Config not loading

Check Config Server logs:
```bash
curl http://config-server:8888/{service-name}/dev
```

### Wrong configuration loaded

Verify:
1. Service name matches folder name
2. Profile is correctly set
3. File naming follows convention

### Changes not reflected

1. Ensure changes are committed to Git
2. Config Server may cache - restart if needed
3. Trigger `/actuator/refresh` on service

## Integration

This repository is used by:
- **Config Server** (Port 8888) - Serves configurations
- **Gateway** (Port 8080)
- **User Service** (Port 8081)
- **Analysis Service** (Port 8082)
- **Location Service** (Port 8083)

## License

This project is part of the Skydive Forecast system.
