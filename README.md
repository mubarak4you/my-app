locals {
  cluster_name = lower("gke-${var.name}-${local.environment}-${var.region}-${local.vsad}")

  maintenance_start_time = lookup(var.maintenance_start_time, local.cluster_name, "2019-01-01T01:00:00Z")
  maintenance_end_time   = lookup(var.maintenance_end_time, local.cluster_name, "2019-01-01T09:00:00Z")
  maintenance_recurrence = lookup(var.maintenance_recurrence, local.cluster_name, "FREQ=WEEKLY;BYDAY=WE,FR")

  cluster_maintenance_window_is_recurring = local.maintenance_recurrence != "" && local.maintenance_end_time != "" ? [1] : []
}





resource "google_container_cluster" "primary" {
  name     = var.name
  project  = var.project_id
  location = var.location

  maintenance_policy {
    dynamic "recurring_window" {
      for_each = local.cluster_maintenance_window_is_recurring
      content {
        start_time = local.maintenance_start_time
        end_time   = local.maintenance_end_time
        recurrence = local.maintenance_recurrence
      }
    }
  }
}






variable "maintenance_start_time" {
  type        = map(string)
  description = "Time window specified for daily or recurring maintenance operations in RFC3339 format. Optional per project/cluster."
  default     = {}
}

variable "maintenance_end_time" {
  type        = map(string)
  description = "Time window specified for recurring maintenance operations in RFC3339 format. Optional per project/cluster."
  default     = {}
}

variable "maintenance_recurrence" {
  type        = map(string)
  description = "Frequency of the recurring maintenance window in RFC5545 format. Optional per project/cluster."
  default     = {}
}
