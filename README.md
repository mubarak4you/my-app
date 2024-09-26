# Maintenance Window
maintenance_policy {
  recurring_window {
    start_time = lookup(
      [for key, value in local.maintenance_window_overrides : 
        value["maintenance_start_time"] if key == local.cluster_key][0], 
      var.maintenance_start_time
    )

    end_time = lookup(
      [for key, value in local.maintenance_window_overrides : 
        value["maintenance_end_time"] if key == local.cluster_key][0], 
      var.maintenance_end_time
    )

    recurrence = lookup(
      [for key, value in local.maintenance_window_overrides : 
        value["maintenance_recurrence"] if key == local.cluster_key][0], 
      var.maintenance_recurrence
    )
  }

  dynamic "maintenance_exclusion" {
    for_each = var.maintenance_exclusions
    content {
      exclusion_name = maintenance_exclusion.value.name
      start_time     = maintenance_exclusion.value.start_time
      end_time       = maintenance_exclusion.value.end_time
    }
  }
}
