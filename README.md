  # Maintenance Window
  maintenance_policy {
    recurring_window {
      start_time = lookup(
        local.maintenance_window_overrides[local.cluster_name],
        "maintenance_start_time",
        var.maintenance_start_time
      )

      end_time = lookup(
        local.maintenance_window_overrides[local.cluster_name],
        "maintenance_end_time",
        var.maintenance_end_time
      )

      recurrence = lookup(
        local.maintenance_window_overrides[local.cluster_name],
        "maintenance_recurrence",
        var.maintenance_recurrence
      )
    }
  }
  
  
    # Cluster Maintenance Window
  maintenance_window_overrides = {
    "<project_id>-<cluster_name>" = {
      "maintenance_start_time" = "xxx",
      "maintenance_end_time"    = "xxx",
      "maintenance_recurrence"  = "xxx"
    }
  }


  cluster_name                            = lower("gke-${var.name}-${local.environment}-${var.region}-${local.vsad}")


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