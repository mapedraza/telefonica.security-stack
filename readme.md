# Previous terms definitions:

## Components 

Those components are based on 3 Openstack security components and an aditional Telefonica's component:
- [Keystone](https://github.com/telefonicaid/fiware-keystone-spassword) as IDM
- [Keypass](https://github.com/telefonicaid/fiware-keypass) as PDP
- [Steelskin](https://github.com/telefonicaid/fiware-pep-steelskin) as PEP
- [Orchestrartor](https://github.com/telefonicaid/orchestrator) to administrate the rest of the components

## Definitions

The conceps we are goin to use are:

- **Service**: Is a tenant space, the organization, and defines the `fiware-service` header when making a request. A service can contains severals subservices. Is represented as Domain on Openstack.
- **Subservice**: It is a project space inside a Service, and define the `fiware-servicepath` header. Is represented as a project in Openstack

## User roles

- **Cloud admin**: It is the user with more privileges. It is able to create and delete services.
- **Service admin**: It is able to administrate a service, creating subservices inside a service.
- **Subservice admin**: It is able to create and modify entities and other operation within the subservice

## Aditional information

* [Orchestrator API](https://orchestrator2.docs.apiary.io/#)
* [Keystone API](https://docs.openstack.org/api-ref/identity/v3/)

# Operations

## Sanity Check Orion

```bash
curl -L -X GET 'http://localhost:10026/version'
```

## Create a Service

```bash
curl -X POST http://localhost:8084/v1.0/service/ -H 'Content-Type: application/json' -H 'Accept: application/json' -d \
'{
  "NEW_SERVICE_NAME": "smartcity",
  "NEW_SERVICE_DESCRIPTION": "smartcity",
  "NEW_SERVICE_ADMIN_USER": "usertest",
  "NEW_SERVICE_ADMIN_PASSWORD": "s3rv1c34dm1n",
  "NEW_SERVICE_ADMIN_EMAIL": "adm@tid.es",
  "DOMAIN_ADMIN_USER": "cloud_admin",
  "DOMAIN_ADMIN_PASSWORD": "proxypwd1234",
  "DOMAIN_NAME": "admin_domain"
}'
```

## List services

```bash
curl -L -X GET 'http://localhost:8084/v1.0/service/' \
-H 'Content-Type: application/json' \
-H 'Accept: application/json' \
--data-raw '{
  "SERVICE_ADMIN_USER": "cloud_admin",
  "SERVICE_ADMIN_PASSWORD": "proxypwd1234",
  "DOMAIN_NAME": "admin_domain"
}'
```

## Create subservice

We are going to create a Service into a given subservice. In this case, we have to provide the Service ID as a parameter into the URL.

```bash
curl -L -X POST 'http:/localhost:8084/v1.0/service/{{serviceId}}/subservice' \
-H 'Content-Type: application/json' \
-H 'Accept: application/json' \
--data-raw '{
    "SERVICE_NAME": "smartcity",
    "SERVICE_ADMIN_USER": "usertest",
    "SERVICE_ADMIN_PASSWORD": "s3rv1c34dm1n",
    "SERVICE_ADMIN_TOKEN":"{{admin_token}}",
    "NEW_SUBSERVICE_NAME": "test_subservice",
    "NEW_SUBSERVICE_DESCRIPTION": "test subservice desc",
    "NEW_SUBSERVICE_ADMIN_USER": "subservice_admin_user",
    "NEW_SUBSERVICE_ADMIN_PASSWORD": "pass1234qwer",
    "NEW_SUBSERVICE_ADMIN_EMAIL": "email@dot.com"
}'
```

`NEW_SUBSERVICE_DESCRIPTION`, `NEW_SUBSERVICE_ADMIN_USER`, `NEW_SUBSERVICE_ADMIN_PASSWORD` and `NEW_SUBSERVICE_ADMIN_EMAIL` are optional (in the case you want to create a subservice admin, you have to provide it)

## List subservices

```bash
curl -L -X GET 'http://localhost:8084/v1.0/service/{{serviceId}}/subservice' \
-H 'Content-Type: application/json' \
-H 'Accept: application/json' \
--data-raw '{
    "SERVICE_NAME": "smartcity",
    "SERVICE_ADMIN_USER": "usertest",
    "SERVICE_ADMIN_PASSWORD": "s3rv1c34dm1n",
    "SERVICE_ADMIN_TOKEN":"{{admin_token}}"
}'
```

## Get a Token

```bash
curl -i -s -X POST http://localhost:5001/v3/auth/tokens -H 'content-type: application/json' -d \
'{
  "auth": {
    "identity": {
      "methods": [
        "password"
      ],
      "password": {
        "user": {
          "domain": {
            "name": "smartcity"
          },
          "name": "usertest",
          "password": "s3rv1c34dm1n"
        }
      }
    },
    "scope": {
      "domain": {
        "name": "smartcity"
      }
    }
  }
}'
```

The token is provided through `X-Subject-Token` response header


## Query entities to Context Broker

Now we have a valid token, we are ready to make requests to the Context Broker through PEP. We have to provide the token received on the previous request 
(as `X-Subject-Token`) throug the `X-Auth-Token` Header.

```bash
curl -L -X GET 'http://localhost:1026/v2/entities' \
-H 'Fiware-Service: smartcity' \
-H 'Fiware-ServicePath: /' \
-H 'X-Auth-Token: {{token}}'
```





