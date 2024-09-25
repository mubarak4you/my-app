maintenance_policy {
  recurring_window {
    start_time = contains(keys(local.maintenance_window_overrides), local.cluster_key) ?
      lookup(local.maintenance_window_overrides[local.cluster_key], "maintenance_start_time", var.maintenance_start_time) :
      var.maintenance_start_time

    end_time = contains(keys(local.maintenance_window_overrides), local.cluster_key) ?
      lookup(local.maintenance_window_overrides[local.cluster_key], "maintenance_end_time", var.maintenance_end_time) :
      var.maintenance_end_time

    recurrence = contains(keys(local.maintenance_window_overrides), local.cluster_key) ?
      lookup(local.maintenance_window_overrides[local.cluster_key], "maintenance_recurrence", var.maintenance_recurrence) :
      var.maintenance_recurrence
  }
}
