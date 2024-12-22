# INTRODUCTION


- create mysql database aws aurora db
  - create the application user on the database
  - update the vault file to contain both the admin and the user credentials
  - update the fact gatherer playbook wih the aurora details
  - verify that fact gatherer can pull new aurora details(added rds endpoint to the  vault file instead)
- create keycloak admin user credentials in the vault
- create en entry in the global_vars for the keycloak global_var_services_details
- create a dns to point to the ALB
- create Billbox ROOT CA (UAT/PROD)
- create client certificate (UAT/PROD)
- configure nginx for setup with client cert
- create new NLB
- create new client certificate
- create new pfx for using in a browser
- point mtls for core to new NLB domain
- deploy keycloak
- deploy securecore