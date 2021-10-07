# k4o-platform-technical-challenge-terraform

Below is the Code:

```
#Added the Provider
terraform {
  required_providers {
    mongodbatlas = {
      source  = "mongodb/mongodbatlas"
      version = "1.0.1"
    }
  }
}

data mongodbatlas_clusters this {
  project_id = var.mongdbatlas_project_id
}

data mongodbatlas_cluster this {
  for_each = toset(data.mongodbatlas_clusters.this.results[*].name)

  project_id = var.mongdbatlas_project_id
  name       = each.value
}

#Connection_string added
output connection_strings = { 
  for_each = var.service_configuration
    { Cluster = each.value.mongoCluster 
      Database = each.value.mongoDatabase 
      Collection = each.value.mongoCollection 
      conn_string = "mongodb+srv://${mongodbatlas_database_user.store-service-user.username}:${random_password.store-service-password.result}@${Cluster}/${Database}/@{Collection}"
     }
 }    


resource random_password store-service-password {
  # Generate a unique new password for the DB user
    length           = 16
    special          = true
    override_special = "_%@"
}

resource mongodbatlas_database_user store-service-user {
  # create a username for the service (e.g. the service name)
  username           = "${var.environment}-${each.key}" 
  # create a password for the service 
  password           = random_password.store-service-password
  # Create the right role (read only permissions) for this user and service
  dynamic roles {
    for_each = each.value.mongoCollection[*]
    content {
      role_name       = "read"
      database_name   = each.value.mongoDatabase
      collection_name = roles.value
    }
  }
}

 variable service_configuration = 
 { type = set(string)
 default = [
  {
    serviceName     = "possums-data-store"
    mongoCluster    = "animals-mongo"
    mongoDatabase   = "marsupials-dev"
    mongoCollection = ["possums"]
  },
  {
    serviceName     = "numbats-data-store"
    mongoCluster    = "animals-mongo"
    mongoDatabase   = "marsupials-dev"
    mongoCollection = ["numbats"]
  }
]
}

```
1) Output commond support -json - If specified, the outputs are formatted as a JSON object, with a key per output. If NAME is specified, only the output specified will be returned. This can be piped into tools such as jq for further processing. "https://www.terraform.io/docs/cli/commands/output.html" 
2) Example - Return a Connection String "https://registry.terraform.io/providers/mongodb/mongodbatlas/latest/docs/resources/cluster"
3) For random password https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password

Few command I used for checking:
1) terraform validate ( command validates the configuration files in a directory )
2) terraform fmt ( command is used to rewrite Terraform configuration files to a canonical format and style )
3) terraform plan ( command creates an execution plan )
4) terraform init ( command is used to initialize a working directory containing Terraform configuration files. ) 


Reference Docs
https://www.terraform.io/docs/cli/commands/output.html
https://registry.terraform.io/providers/mongodb/mongodbatlas/latest/docs/resources/cluster
https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password
