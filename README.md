## ---------------------------------------------------------------------------------------------------------------------
## Required variables
## ---------------------------------------------------------------------------------------------------------------------

variable "name" {
  description = "The name of the cluster"
  type        = string
  validation {
    condition     = length(var.name) < 19
    error_message = "The name must be < 19 characters"
  }
  validation {
    condition     = can(regex("^[[:lower:]]+[0-9a-z-]*", var.name))
    error_message = "Must match ^[[:lower:]]+[0-9a-z-]*"
  }
}

variable "region" {
  description = "The region to host the cluster in."
  type        = string
  validation {
    condition = anytrue([
      var.region == "us-east4",
      var.region == "us-west1",
    ])
    error_message = "Must be us-east4 or us-west1"
  }
}

variable "project_id" {
  description = "The project ID to host the cluster in."
  type        = string
  validation {
    condition     = can(regex("^vz-(it|nonit)-(np|pr)-[a-z0-9-]*$", var.project_id))
    error_message = "Must match ^vz-(it|nonit)-(np|pr)-[a-z0-9-]*$"
  }
}

variable "user_id" {
  description = "The user ID of the cluster owner."
  type        = string
  validation {
    condition     = can(regex("^[[:alpha:]]+[[:alnum:]]*", var.user_id))
    error_message = "Must match ^[[:alpha:]]+[[:alnum:]]*"
  }
}

variable "user_email" {
  description = "The user email of the cluster owner."
  type        = string
}


## ---------------------------------------------------------------------------------------------------------------------
## Optional variables
## ---------------------------------------------------------------------------------------------------------------------

variable "kubernetes_version" {
  type        = string
  description = "The Kubernetes version of the masters."
  default     = "1.29"
}

variable "hub_project_id" {
  description = "The project in which the GKE Hub belongs. Defaults to GKE cluster project_id."
  type        = string
  default     = ""
}

variable "environment" {
  description = "The environment to host the cluster in."
  type        = string
  default     = ""
}

variable "vsad" {
  description = "The VSAD of the cluster."
  type        = string
  default     = ""
}

variable "timeouts" {
  type        = map(string)
  description = "Timeout for cluster operations."
  default     = {}
  validation {
    condition     = !contains([for t in keys(var.timeouts) : contains(["create", "update", "delete"], t)], false)
    error_message = "Only create, update, delete timeouts can be specified."
  }
}
variable "maintenance_exclusions" {
  type        = list(object({ name = string, start_time = string, end_time = string, exclusion_scope = string }))
  description = "List of maintenance exclusions. A cluster can have up to three"
  # The Google Cloud API requires that the end_time be within
  # 180 days from the day the maintenance exclusion is created
  default = []
}

# Maintenance window policy is
# 6PM - 2AM ET every Wednesday and Friday
# ETOKE-1131 change time to 01AM to 9AM 
variable "maintenance_start_time" {
  type        = string
  description = "Time window specified for daily or recurring maintenance operations in RFC3339 format"
  default     = "2019-01-01T01:00:00Z"
}

variable "maintenance_end_time" {
  type        = string
  description = "Time window specified for recurring maintenance operations in RFC3339 format"
  default     = "2019-01-01T09:00:00Z"
}

variable "maintenance_recurrence" {
  type        = string
  description = "Frequency of the recurring maintenance window in RFC5545 format."
  default     = "FREQ=WEEKLY;BYDAY=WE,FR"
}

variable "config_sync_infra_branch" {
  description = "The branch of the infra repository to sync from. Defaults to the release branch matching the cluster version. This variable is meant for development purposes only."
  type        = string
  default     = "main"
}

variable "config_sync_security_branch" {
  description = "The branch of the security repository to sync from. Defaults to the release branch matching the cluster version. This variable is meant for development purposes only."
  type        = string
  default     = "main"
}

variable "pipeline_id" {
  description = "The ID of the last GitLab pipeline to update this cluster."
  type        = string
  default     = ""
}

variable "cluster_module_version" {
  description = "Semantic version of the module applied."
  type        = string
  default     = ""
}

variable "deletion_protection" {
  description = "Whether or not to allow Terraform to destroy the cluster. Terraform cannot delete this cluster unless this field is set to false in Terraform state."
  type        = bool
  default     = true
}
