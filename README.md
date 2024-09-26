resource "google_container_cluster" "primary" {
  # Other necessary cluster config...

  maintenance_policy {
    recurring_window {
      start_time = contains(local.maintenance_window_overrides, local.cluster_key)
        ? local.maintenance_window_overrides[local.cluster_key]["maintenance_start_time"]
        : var.maintenance_start_time

      end_time = contains(local.maintenance_window_overrides, local.cluster_key)
        ? local.maintenance_window_overrides[local.cluster_key]["maintenance_end_time"]
        : var.maintenance_end_time

      recurrence = contains(local.maintenance_window_overrides, local.cluster_key)
        ? local.maintenance_window_overrides[local.cluster_key]["maintenance_recurrence"]
        : var.maintenance_recurrence
    }
  }
}
