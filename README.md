# Maintenance Window Overrides
locals {
  maintenance_window_overrides = {
    # Example Override for a specific project-cluster
    "project1-cluster1" = {
      "maintenance_start_time" = "2024-09-01T02:00:00Z"
      "maintenance_end_time"    = "2024-09-01T06:00:00Z"
      "maintenance_recurrence"  = "FREQ=WEEKLY;BYDAY=MO,TH"
    }
    # Another Example Override
    "project2-cluster2" = {
      "maintenance_start_time" = "2024-10-01T01:00:00Z"
      "maintenance_end_time"    = "2024-10-01T05:00:00Z"
      "maintenance_recurrence"  = "FREQ=WEEKLY;BYDAY=TU,FR"
    }
  }
}

# Default Maintenance Start Time
variable "maintenance_start_time" {
  type        = string
  description = "Time window specified for daily or recurring maintenance operations in RFC3339 format"
  default     = "2019-01-01T01:00:00Z"
}

# Default Maintenance End Time
variable "maintenance_end_time" {
  type        = string
  description = "Time window specified for recurring maintenance operations in RFC3339 format"
  default     = "2019-01-01T09:00:00Z"
}

# Default Maintenance Recurrence (RFC5545 format)
variable "maintenance_recurrence" {
  type        = string
  description = "Frequency of the recurring maintenance window in RFC5545 format."
  default     = "FREQ=WEEKLY;BYDAY=WE,FR"
}


# Example of how you might define your cluster key from project ID and cluster name
locals {
  project_id  = "project1"
  cluster_name = "cluster1"
  cluster_key  = "${local.project_id}-${local.cluster_name}"
}


# Maintenance Policy with Overrides or Default Values
resource "google_container_cluster" "my_cluster" {
  # Other necessary cluster config...

  maintenance_policy {
    recurring_window {
      start_time = lookup(
        flatten([
          for key, value in local.maintenance_window_overrides :
          value if key == local.cluster_key
        ])[0]["maintenance_start_time"],
        var.maintenance_start_time
      )

      end_time = lookup(
        flatten([
          for key, value in local.maintenance_window_overrides :
          value if key == local.cluster_key
        ])[0]["maintenance_end_time"],
        var.maintenance_end_time
      )

      recurrence = lookup(
        flatten([
          for key, value in local.maintenance_window_overrides :
          value if key == local.cluster_key
        ])[0]["maintenance_recurrence"],
        var.maintenance_recurrence
      )
    }
  }
}
