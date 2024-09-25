locals {
  maintenance_window_overrides = {
    "<project_id>-<cluster_name>" = {
      "maintenance_start_time" = "xxx",
      "maintenance_end_time"    = "xxx",
      "maintenance_recurrence"  = "xxx"
    }
  }

  # Define your defaults
  default_maintenance_start_time = "12:00"
  default_maintenance_end_time   = "14:00"
  default_maintenance_recurrence = "WEEKLY"

  # Lookup key for a specific cluster/project
  cluster_key = "<project_id>-<cluster_name>"
}

resource "google_container_cluster" "example" {
  maintenance_policy {
    recurring_window {
      start_time = lookup(
        local.maintenance_window_overrides[local.cluster_key],
        "maintenance_start_time",
        local.default_maintenance_start_time
      )

      end_time = lookup(
        local.maintenance_window_overrides[local.cluster_key],
        "maintenance_end_time",
        local.default_maintenance_end_time
      )

      recurrence = lookup(
        local.maintenance_window_overrides[local.cluster_key],
        "maintenance_recurrence",
        local.default_maintenance_recurrence
      )
    }
  }
}
