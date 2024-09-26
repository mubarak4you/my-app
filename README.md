maintenance_policy {
  recurring_window {
    start_time = coalesce(
      lookup(local.maintenance_window_overrides, 
        "${var.project_id}-${local.cluster_name}",
        null)["maintenance_start_time"],
      var.maintenance_start_time
    )

    end_time = coalesce(
      lookup(local.maintenance_window_overrides, 
        "${var.project_id}-${local.cluster_name}",
        null)["maintenance_end_time"],
      var.maintenance_end_time
    )

    recurrence = coalesce(
      lookup(local.maintenance_window_overrides, 
        "${var.project_id}-${local.cluster_name}",
        null)["maintenance_recurrence"],
      var.maintenance_recurrence
    )
  }
}
