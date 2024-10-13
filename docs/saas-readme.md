# Artifact Management SAAS
## Frontend

## Backend
### Datastore
#### Postgres (RDS)
- Popular and well supported, lots of documentation
- Good / great performance
- Support for things that might require another datastore
  - Can store data as a JSON column, and index and query on data vs using another tool such as Mongodb to store this
- Replication supported, can have multiple replicas

#### S3
- S3 for storage of package objects
- No need to worry about scaling
- Things such as encryption provided 

### Document Store
#### Elastcisearch
- Allow advanced and quick search against packages and metadata
- Can handle complex queries

### Application
Application built and deployed using Docker
- Docker is popular and well supported
- Allows deployment to various environments such as EC2, ECS, Kubernetes
- Can use secure docker images with only minimum installation required to run, no root etc

##### API nodes
Nodes for API application.
- Might be single monolith application, or smaller microservices based on domain e.g. packages-api, user-api
- Stateless, all data stored in S3 or 

##### Worker nodes
Nodes for "worker" application. Worker application is responsible for async work such as
- User uploads a new package, API returns a response and worker node will validate, scan, and persist package
- Running scheduled tasks such as cleaning up old packages, generating reports and metrics
- Might be a long running application, or instances spun up on demand e.g. kubernetes jobs, lambdas

## Other
### Load Balancing and Redundancy: 
- API instances behind load balancer to distribute requests
- CDN to cache content
  - Reduce pressure on backend and allow returning cached content if backend fails

### Data Replication and Backup: 
- Replicas for datastores?
  - Might allow read-only access incase of issues
- Backups of all data e.g. backup databases on regular schedule and before any maintenance
 
### Scalability Strategy: 
- Use auto scaling where possible to react based on metrics e.g. increased queue size, requests
  - Need to set limits on how many allow instances e.g. too many API instances can overload databsae with connections
- Use replicas to distribute read-only requests
- Use larger instances with more CPU/memory 
  - e.g. for worker nodes that have 1 thread per core consuming from a queue, increasing core count can be cheaper and quicker than scaling out to more instances

### Failover and Disaster Recovery: 
- Deploy to multiple regions in same locale if possible e.g. for US deploy to multiple US regions
  - Region suffers outage, users and be redirected to nearest region in same area
- Deploy all services to multiple availablility zones
  - minimum of at least 2/3 to allow for cloud provider having issues
- Datastores should be in multiple availablility zones, with hot standby if possible
  - Allows switching to standby if main instance fails
- Backups of all data e.g. backup databases on regular schedule and before any maintenance
- Circuit-breaker pattern in backend
  - e.g. if database returns X timeouts over 30 seconds, open circuit breaker and prevent further build up of traffic
  - can return default response if circuit-breaker open
  
### Security Measures: 
- Permissions for all CRUD operations 
  - User must have explicit permission to do actions such as read/delete/update a package
  - Permissions per package/repository/customer
    - Allow giving user read access to everything in repository, or only select packages
- Audit events
  - Store audit for CRUD operations e.g. who uploaded a package and when, who modified a package and when
- Zero-trust by default
  - New user should have no access by default, only give required access
- Have processes to check users access and generate reports/notifications
  - e.g. users who have access to packages they haven't accessed in X days, recommend to remove access
- API gateway 
  - Rate limiting based on type of user, prevent malicious users from spamming requests 
  - Allow/blocklists to control who has access
    
### Monitoring and Alerts:
- SAAS such as Datadog for monitoring and metrics, tools such as Pagerduty for on-call to handle issues
- Standardise metrics for components
  - API record metrics on number of requests, duration, number of 2xx, 4xx, 5xx responses
  - Metrics on queue size, duration, age of message
  - Metrics for datastores, such as size, number of connections, duration of requests

### Integration Framework: 
- APIs should implement OpenAPI documentation
  - Allow generation of API documentation that is in sync with API
  - Tools such as Swagger allow generation of API clients to make interacting with REST endpoints easier
- Webhooks
  - Webhooks for backend events e.g. package_uploaded, package_deleted
  - Allow developers to react to events, build their own tools and integrations e.g. Slackbot that consumes package webhooks to notify channel when new release happens
  
### Global Reach  
- CDN
  - Let CDN handle serving content to user based on location e.g. user in US is redirected to US regions